## 3EM Contract Spec

### Reasoning
Behind every successful project, there is always a successful pattern on the way things are done. These patterns are usually referred to as "spec" or "standards".

In Arweave, SmartWeave gives developer different standards to follow ([found here](https://github.com/ArweaveTeam/SmartWeave/blob/master/CONTRACT-GUIDE.md)) and while these are not completed, it has helped the community move forward with this technology which was initially what helped 3EM get conceived.

In the feedback received from the 3em community, developers have reached out to us with questions on how to accomplish things, whether we are fully backwards compatible with SmartWeave, and feature requests. We decided that the most constructive path forward would be to design a specification around feature additions and SmartWeave's existing SDK. From this, we hope to continue collaborating with the community on the future of 3em.

### The Problem

As 3EM gets more mature, our core team has come to realize waiting on SmartWeave specs to be updated with the community needs is a slow process, which is unfortunate for what our first goal was (being backwards compatible). The core team held a meeting with part of the Arweave team where the issue with Standards came up and it was mentioned _the community should be able to write their own specs, and the better the spec is the more people will follow it_ . From this reasoning, it was clear that SmartWeave can be considered a motivation but not a final spec as different execution engines might have different business purposes.

### The Proposal
As we continue to move forward with our roadmap, we want to solve the standard issue that has been mentioned, before the community grows and then it will be too late. This spec will be called **3EM Standard** .

#### Scope Globals
_The following globals will be kept_
```
globalThis.ContractError
globalThis.ContractAssert
```

#### New Scope Globals
```
globalThis.ContractEvolver
```

`globalThis.ContractEvolver` represents the address that owned the evolving.

1) `globalThis.ContractEvolver` is undefined before any evolve behavior has taken place.
2) It is a constant provided internally by 3EM and it cannot be mutated from inside the contract
3) Only 3EM internally can update `globalThis.ContractEvolver`

```
globalThis.ContractInitializer
```

`globalThis.ContractInitializer` represents the address that deployed the very first version of the contract.

1)  `globalThis.ContractInitializer` will always have its value set to the deployer of the contract & it cannot be mutated

#### SmartWeave Globals
_The following globals will be kept_

```
SmartWeave.contract.owner
SmartWeave.contract.id
SmartWeave.transaction.id  
SmartWeave.transaction.owner
SmartWeave.transaction.target
SmartWeave.transaction.quantity
SmartWeave.transaction.reward
SmartWeave.transaction.tags
SmartWeave.block.height
SmartWeave.block.indep_hash
SmartWeave.block.timestamp
SmartWeave.arweave.utils
SmartWeave.arweave.ar
SmartWeave.arweave.wallets (Partially)
```

#### Reading External States
3EM allows reading contract states including contracts written in different languages such as EVM-based contracts or WASM-based contracts. For this, `SmartWeave.contracts.readContractState` is kept.
```
SmartWeave.contracts.readContractState(contractId: string, height?: number, showValidity?: boolean): Promise<any>
```

1) Contracts are read up to the height of the current interaction
2) If `showValidity` is true, a struct of `{ state, validity }` is returned.

#### Unsafe Calls
```
SmartWeave.unsafeClient.transactions.getData
SmartWeave.unsafeClient.transactions.get
```

#### To Be Removed
3EM will focus on deterministic behavior thus removing most (if not all) of the calls that are considered non-deterministic and cannot be seeded.

`SmartWeave.arweave.wallets`
will have functions such as `generateJWT` removed as well as other crypto generation functions.

`SmartWeave.arweave.crypto`
will be fully removed

#### 3EM Contract Creation
- 3EM will be compatible with the SmartWeave `App-Name` tag standard that is used to fetch the contract, although, it will be recommended to use `App-Name=3EM` for fully-based 3EM contracts.

- Contracts can _still_ have a tag called `Init-State` with a string of what the initial state of the contract would be.
- Contracts can still reference the initial state by using the ID of a transaction

### Date Object
The `Date` object will be equivalent to `new Date(SmartWeave.block.timestamp)` and it will always represent the timestamp of the block of the current transaction. 

#### Evolve
- Evolve is kept as is. The functionality is added in the code where the user decides when to return `evolve` as an indication for the runtime that contract needs to be evolved.
- Evolve updates the variable `globalThis.ContractEvolver`
- Evolve can be done by interaction with the following tags (Option available only for 3EM contracts):
    - `Action` = `evolve`
    - `New-Source-Tx` = `{New Contract Id}`
    - **It's only allowed when:** `globalThis.ContractInitializer == interaction.caller`
    - **It's only allowed when contract was deployed with the following tag: `Allow-Evolution-By-Tags`=`true`**

## Authors
- Divy Srivastava - @littledivy 
- Andres Pirela - @andreespirela 
