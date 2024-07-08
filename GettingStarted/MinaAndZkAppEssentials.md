## Mina and zkApp essentials
>This page can be seen as a brief summary of the [official documentation](https://docs.minaprotocol.com/). It aims to highlight key concepts relating to the Mina blockchain and zkApps. For a more in-depth study, please refer to the [official documentation](https://docs.minaprotocol.com/).

### Mina blockchain:
- The entire blockchain is an easily verifiable cryptographic proof of 22kb. More [here](https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference).
- The chain functioning relies onrecursive [zk-SNARKS](https://minaprotocol.com/blog/what-are-zk-snarks) and particularly on `Pickles` proof system, mainly because it allows recursive proofs, has no trusted setups, short proving time and acceptably small proof size (8kb). More [here](https://minaprotocol.com/blog/meet-pickles-snark-enabling-smart-contracts-on-coda-protocol).
- There is no distinction between full nodes and light nodes like in Ethereum or Bitcoin. In Mina every node is a full node. Instead nodes are divided into:
  - `non-consensus nodes` -> achieve full nodes security by only storing the protocol state, the account, a merkle path to this account and a verification key. They do not participate in consensus but can still fully verify the zero knowledge proof to trustlessly validate the state of the chain. More [here](https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference#:~:text=to%20full%20nodes.-,This,-client%20software%20for)
  - `archive nodes` -> store the historical chain data to a persistent data source so it can later be retrieved. A zkApp can retrieve events and actions from one or more Mina archive nodes. More [here](https://docs.minaprotocol.com/node-operators/archive-node/getting-started).
  - `block producers` -> More [here](https://docs.minaprotocol.com/mina-protocol/block-producers).
  - ... TO-DO


### zkApps:
- smart contracts on Mina are called zkApps. Contract code is not stored on chain.
- each zkApp has a limited storage: an array of 8 elements of type Field 
- the Field data type is the base datatype and represents the order of the Finite field the math operations are conducted on. 
- appState -> TO-DO
- verification Key -> It's a key associated to every zkApp and it's generated considering the current deployment code. Thus it's able to detect if we try to submit a proof of execution of a different code. Indeed if we want to change the contract code, we need to redeploy a new zkApp: the verification key will change.
- Two entites are involved when sending transactions to the Mina blockchain:
  - the **Prover**, who executes the code locally, creates a ZK proof and sends it on chain signing the transaction and paying for the fees 
  - the **Verifier**, which is the zkApp deplpyed on chain. It doesn't contain code but uses its verification key to verify the received proof and eventually update its internal appState.


