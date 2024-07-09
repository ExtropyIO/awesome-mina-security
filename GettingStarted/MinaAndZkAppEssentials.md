## Mina and zkApp essentials
>This page can be seen as a brief summary of the [official documentation](https://docs.minaprotocol.com/). It aims to highlight key concepts relating to the Mina blockchain and zkApps. For a more in-depth study, please refer to the [official documentation](https://docs.minaprotocol.com/).

### Mina blockchain:
- The entire blockchain is an easily verifiable cryptographic proof of 22kb. Even if the network grows, the proof size will be constant. More [here](https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference).
- The chain functioning relies on recursive [zk-SNARKS](https://minaprotocol.com/blog/what-are-zk-snarks) and particularly on the `Pickles` proof system, mainly because it allows infinite recursive proofs, has no trusted setups, short proving time and acceptably small proof size (8kb). More [here](https://minaprotocol.com/blog/meet-pickles-snark-enabling-smart-contracts-on-coda-protocol).
- There is no distinction between full nodes and light nodes like in Ethereum or Bitcoin. In Mina every node is a full node. Instead nodes are divided into:
  - `non-consensus nodes` -> achieve full nodes security by only storing the protocol state, the account, a merkle path to this account and a verification key. They do not participate in consensus but can still fully verify the zero knowledge proof to trustlessly validate the state of the chain. More [here](https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference#:~:text=to%20full%20nodes.-,This,-client%20software%20for)
  - `archive nodes` -> store the historical chain data to a persistent data source so it can later be retrieved. A zkApp can retrieve events and actions from one or more Mina archive nodes. More [here](https://docs.minaprotocol.com/node-operators/archive-node/getting-started).
  - `block producers` -> More [here](https://docs.minaprotocol.com/mina-protocol/block-producers).
  - ... TO-DO
- The maximum number of zkApp transactions per block is currently capped at 24. This restriction will be gradually lifted after the Mainnet upgrade.
- There is no strong distinction between normal "user accounts" and "zkApp accounts". A zkApp account is an account on the Mina blockchain where a zkApp smart contract is deployed and has a verification key associated with it. Read below section for understanding the verification key.



### zkApps:
- smart contracts on Mina are called zkApps.
- Each zkApp consists in two parts: a smart contract and a user interface UI
- Contract code is not stored on chain since zkApps use an off-chain execution and off-chain state model allowing for private computation
- each zkApp has a limited on-chain storage: 8 elements of type Field (32 bytes each) denoted with the decorator `@state`
```js
class HelloWorld extends SmartContract {
  @state(Field) x = State<Field>(); // States are initialized with the State() function.

   @method async setX(x: Field) {
    this.x.set(x); // allow anyone to overwrite the state by using this.<state>.set():
  }

   @method async radState() {
    const x = this.x.get(); // fetches the on-chain state
    this.x.requireEquals(x);
    // Above line is CRUCIAL:
    // 1. You must link "x at verification time" to be the same as "x at proving time", where this.x is the on-chain state at verification time and the x within requireEquals() is the value fetched from the chain on the client side. 
    // 2. When you use an on-chain value, you have to prove that this value is the on-chain value
  }
}
```
- the Field data type is the base datatype and represents the order of the Finite field the math operations are conducted on. 
- since a zkApp has limited storage a solution can be to use structures such as Merkle Trees: you keep all the data off-chain and only commit the Merkle root on chain. More [here](https://docs.minaprotocol.com/zkapps/o1js/merkle-tree).
- When you deploy a zkApp to a new Mina address, the Mina Protocol charges a 1 MINA fee for account creation.
- Two entites are involved when running zkApp code and sending transactions to the Mina blockchain:
  - the **Prover**, who executes the code locally, creates a ZK proof and sends it on chain signing the transaction and paying for the fees 
  - the **Verifier**, which is the zkApp deployed on chain that uses its verification key to verify the received proof and eventually update its internal state.
- The code for the smart contract of a zkApp is written with o1js and when the project is built using `npm run build` two functions are derived:
  - Prover function receives Private and Public inputs and generates a proof of the executed code. It also generates a list of state updates (called `Account updates`) which is a JSON file in plain text. The integrity of these account updates is ensured by passing a hash of the account updates as a public input to the smart contract. The account updates must be present and unmodified for the verification function to pass successfully when it runs on Mina. 
  - Verifier function receives in input the proof and the Public inputs (probably the `Account updates` ?? Need to check) and validates the proof checking that all the constraints defined in the prover function are valid. In terms of speed the verifier function runs a lot faster than the prover function. The Mina network rejects any transactions that do not pass the verifier function.
- The verification Key it's a key that lives on chain, it's associated to each zkApp and it's generated considering the current deployment code. It's able to detect if we try to submit a proof of execution of a different code. Indeed if we want to change the contract code, we need to redeploy a new zkApp: the verification key will change.
- appState -> TO-DO
- A smart contract instance is created passing the public key of the zkApp 
```js
// random private key
let zkAppKey = PrivateKey.random(); 
// get the public key associated to the random private key
let zkAppAddress = PublicKey.fromPrivateKey(zkAppKey);
//create a new instance of the HelloWorld contract
let zkApp = new HelloWorld(zkAppAddress);
```
- A smart contract extends the base class `SmartContract` and inherits its constructor that cannot be overriden. Interaction with a smart contract happens by calling one or more of its @method functions. 
```js
class HelloWorld extends SmartContract {
  @method async myMethod(x: Field) { //x is a private input
    x.mul(2).assertEquals(5); // constraint example
  }
}
```
- Since internally every @method defines a zk-SNARK circuit, a smart contract is a collection of circuits, all of which are compiled into a single prover and a verification key.
- When a SmartContract is compiled into prover and verification keys, method inputs don't have any concrete values attached to them. In contrast, all the variables have actual values attached to them (cryptographers call them "witnesses") during proof generation. To log these values for debugging, use a special function for logging from inside your method:
```js
Provable.log(x);
```
