## Mina and zkApp essentials
>This page can be seen as a brief summary of the [official documentation](https://docs.minaprotocol.com/). It aims to highlight key concepts relating to the Mina blockchain and zkApps. For a more in-depth study, please refer to the [official documentation](https://docs.minaprotocol.com/).



### Mina blockchain
- The entire blockchain is an easily verifiable cryptographic proof of 22kb. Even if the network grows, the proof size will be constant. More [here](https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference).
- The chain functioning relies on recursive [zk-SNARKS](https://minaprotocol.com/blog/what-are-zk-snarks) and particularly on the `Pickles` proof system, mainly because it allows infinite recursive proofs, has no trusted setups, short proving time and acceptably small proof size (8kb). More [here](https://minaprotocol.com/blog/meet-pickles-snark-enabling-smart-contracts-on-coda-protocol).
- There is no distinction between full nodes and light nodes like in Ethereum or Bitcoin. In Mina every node is a full node. Instead nodes are divided into:
  - `non-consensus nodes` -> achieve full nodes security by only storing the protocol state, the account, a merkle path to this account and a verification key. They do not participate in consensus but can still fully verify the zero knowledge proof to trustlessly validate the state of the chain. More [here](https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference#:~:text=to%20full%20nodes.-,This,-client%20software%20for)
  - `archive nodes` -> store the historical chain data to a persistent data source so it can later be retrieved. A zkApp can retrieve events and actions from one or more Mina archive nodes. More [here](https://docs.minaprotocol.com/node-operators/archive-node/getting-started).
  - `block producers` -> More [here](https://docs.minaprotocol.com/mina-protocol/block-producers).
  - ... TO-DO
- The maximum number of zkApp transactions per block is currently capped at 24. This restriction will be gradually lifted after the Mainnet upgrade.
- There is no strong distinction between normal "user accounts" and "zkApp accounts". A zkApp account is an account on the Mina blockchain where a zkApp smart contract is deployed and has a verification key associated with it. Read below section for understanding the verification key.
- Comparison between Mina and Ethereum [here](https://docs.minaprotocol.com/zkapps/advanced/zkapps-for-ethereum-developers)
- Retrieve on-chain data like current timestamp, network informations, account balance [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/on-chain-values)



### zkApps
- smart contracts on Mina are called zkApps.
- Each zkApp consists in two parts: a smart contract and a user interface UI
- Contract code is not stored on chain since zkApps use an off-chain execution and off-chain state model allowing for private computation
- each zkApp has a limited on-chain storage: 8 elements of type Field (32 bytes each) denoted with the decorator `@state`. Check below an example.
- the Field data type is the base datatype and represents the order of the Finite field the math operations are conducted on. 
- since a zkApp has limited storage a solution can be to use structures such as Merkle Trees: you keep all the data off-chain and only commit the Merkle root on chain. More [here](https://docs.minaprotocol.com/zkapps/o1js/merkle-tree).
- When you deploy a zkApp to a new Mina address, the Mina Protocol charges a 1 MINA fee for account creation.
- Two entites are involved when running zkApp code and sending transactions to the Mina blockchain:
  - the **Prover**, who executes the code locally, creates a ZK proof and sends it on chain signing the transaction and paying for the fees 
  - the **Verifier**, which is the zkApp deployed on chain that uses its verification key to verify the received proof and eventually update its internal state.
- The code for the smart contract of a zkApp is written with o1js and when the project is built using `npm run build` two functions are derived:
  - Prover function receives Private and Public inputs and generates a proof of the executed code. It also generates a list of state updates (called `Account updates` , one item for account) which is a JSON file in plain text. The integrity of these account updates is ensured by passing a hash of the account updates as a public input to the smart contract. The account updates must be present and unmodified for the verification function to pass successfully when it runs on Mina. 
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
- Smart contract:
  - extends the base class `SmartContract`
  - inherits its constructor that cannot be overriden
  - the contract storage is denoted with the decorator `@state`
  - to initialize contract state you can use `init()`
  - interactions happens by calling one or more of its @method functions
  - inputs to @method are private inputs unless they are explictly made public storing them in the contract storage
  - for methods that return some value of TYPE you should use @method.returns(TYPE) and return a Promise\<TYPE> object
  - if you want to send some MINA use `this.send()`
```js
class HelloWorld extends SmartContract {
  @state(Field) x = State<Field>(); // States are initialized with the State() function.

  init(){ // if you don't need to initialize anything for Helloworld you can avoid calling init()
    super.init(); //sets the entire state to 0
    this.x.set(Field(10)); // sets HelloWorld initial state x
  }

  @method async myMethod(x: Field) { //x is a private input
    x.mul(2).assertEquals(5); // constraint example
  }

   @method async setX(x: Field) {
    this.x.set(x); // allow anyone to overwrite the state by using this.<state>.set():
  }

   @method async readState() {
    const x = this.x.get(); // fetches the on-chain state
    this.x.requireEquals(x);
    // Above line is CRUCIAL:
    // 1. You must link "x at verification time" to be the same as "x at proving time", where this.x is the on-chain state at verification time and the x within requireEquals() is the value fetched from the chain on the client side. 
    // 2. When you use an on-chain value, you have to prove that this value is the on-chain value
  }

  @method.returns(Bool) async returnSomething(): Promise<Bool> { // annotated return type
  // ...
  return isSuccess;
    }

 @method async payout(amount: UInt64) {
    // REMEMBER:
    // MINA amounts, in all o1js APIs and elsewhere in the protocol, are always denominated in nanoMINA = 10^(-9) MINA, which is why you set const MINA = 1e9.
    this.send({ to: this.sender, amount });
  }
}
```
- Instad of `requireEquals(x)` also assertions can be used. If the assertion fails, o1js throws an error and does not submit the transaction. If the assertion succeeds, it becomes part of the proof that is verified on-chain. Using assertions in o1js means adding constraint logic to your code. E.g. if you call `a.assertLessThan(b)`, you prove that `a < b` under all circumstances.
```js
x.assertEquals(y); // x = y
x.assertBoolean(); // x = 0 or x = 1
x.assertLt(y);     // x < y
x.assertLte(y);    // x <= y
x.assertGt(y);     // x > y
x.assertGte(y);    // x >= y
```
- Since internally every @method defines a zk-SNARK circuit, a smart contract is a collection of circuits, all of which are compiled into a single prover and a verification key.
- When a SmartContract is compiled into prover and verification keys, method inputs don't have any concrete values attached to them. In contrast, all the variables have actual values attached to them (cryptographers call them "witnesses") during proof generation. To log these values for debugging, use a special function for logging from inside your method:
```js
Provable.log(x);
```
- As for Ethereum, also in Mina contracts can invoke functions from other contracts. 
```js
class HelloWorld extends SmartContract {
  @method async myMethod(otherAddress: PublicKey) { 
    //Two proofs are created: one for myMethod() and one for otherMethod()
    const calledContract = new OtherContract(otherAddress);
    calledContract.otherMethod();
  }
}

class OtherContract extends SmartContract {
  @method async otherMethod() {}
}
```
- You create transactions in o1js by calling `Mina.transaction(...)`, which takes the sender (a public key) and a callback that contains your transaction logic. More [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/interact-with-mina#transaction-structure). Then you can generate the proof with `tx.prove()`
```js
const sender = PublicKey.fromBase58('B62..'); // the user address
const zkapp = new MyContract(address); // MyContract is a SmartContract

await MyContract.compile(); 
// NOTE: MyContract.compile() creates prover and verification keys from your smart contract. It doesn't refer to literal "compilation" of JS into a circuit representation. The circuit representation of your code is created by executing it, not by compiling it.

const tx = await Mina.transaction(sender, async () => {
  await zkapp.myMethod(someArgument);
});

console.log(tx.toPretty()); // prints tx data with `undefined` in the authorization field
await tx.prove(); // creates proofs for all the account updates that came from method calls.
//In the case of zkApps, the public input is the account update that is passed in implicitly with tx.prove()

console.log(tx.toPretty()); // prints tx data with the proof in the authorization field
```
- Example of a deposit function for depositing MINA into MyContract:
```js
class MyContract extends SmartContract {
  @method async deposit(amount: UInt64) {
    // 1. instantiates the AccountUpdate class for the current tx sender
    let senderUpdate = AccountUpdate.create(this.sender);
    // 2. specifies that the update must be authorized with a signature
    senderUpdate.requireSignature();
    // 3. use .send() on the sender AccountUpdate to deposit into the zkApp itself
    senderUpdate.send({ to: this, amount });
  }
}
```
- In a user-facing zkApp, user signatures are typically added by a wallet, not within o1js. In that case, the missing signature is expected. However, in tests or when calling zkApps from a Node.js script, you must add the signatures with `tx.sign([...privateKeys])`, called after Mina.transaction on the finished transaction.
```js
const sender = senderPrivateKey.toPublicKey(); // public key from sender's private key
const tx = await Mina.transaction(sender, async () => {
  await zkapp.deposit(UInt64.from(5 * MINA));
});
await tx.prove();
tx.sign([senderPrivateKey]); 
// NOTE: .sign() takes an array, so you could provide multiple private keys for signing
// In this case we pass only senderPrivateKey since the fee payer and the depositor are the same entity
```
- To send a transaction, use `await tx.send()` after having done `tx.prove()` and `tx.sign(...)`, but before you need to specify what network to interact at the beginning of yout script:
```js
const Network = Mina.Network('https://example.com/graphql');
//The network URL must be a GraphQL endpoint that exposes a compatible GraphQL API
Mina.setActiveInstance(Network);

//use a simulated local blockchain for local testing:
const Local = Mina.LocalBlockchain();
Mina.setActiveInstance(Local);
```
- Permissions are realated to each zkApp and are stored on chain. They determine who has the authority to interact and make changes to a specific part of a smart contract. Full list [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#types-of-permissions).
- Authorization determines what resources can be accessed, while permissions just describe who has the ability to execute an action.
- The types of authorizations are:
  - `none`: Everyone has access to fields and can manipulate them as they please
  - `impossible`: nothing can ever change this field
  - `signature`: can only be manipulated by account updates that are accompanied and authorized by a valid signature. Although this is the simplest approach, it is the least flexible
  - `proof`: can be manipulated only by account updates that are accompanied and authorized by a valid proof. More flexible and asynchronous approach.
  - `proofOrSignature`: permissions with authorization set to proofOrSignature accept either a valid signature or a valid proof.
- Setting permissions arbitrarly may be dangerous (e.g `editState` set to `none` is very dangerous). This is way smart contracts, when first deployed, always start with [default permissions](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#default-permissions)
- The permission named `setVerificationKey` allows smart contract upgradeability. This is useful for letting zkApp developers to update the logic of their zkApp and keep them working if the Mina Protocol will undergo an update in the future.:
  - `setVerificationKey` has slightly different authorizations types: `none`, `signature`, `proofOrSignature` exist as they were, instead `impossible` now is `impossibleDuringCurrentVersion` and `proof` now is `proofDuringCurrentVersion`
  - - If set to `impossibleDuringCurrentVersion()` it will not be possible to update the verification key unless an hardfork happens, namely the new transaction version is greater than the old one. More [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#upgrading-after-an-update-to-the-mina-protocol) and [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#example-impossible-to-upgrade).
- Events -> TO-DO [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/events).
- Actions & Reducers -> TO-DO [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/actions-and-reducer)
- Concurrent state updates example using Actions&Reducers -> TO-DO [here](https://github.com/o1-labs/o1js/blob/main/src/examples/zkapps/reducer/reducer-composite.ts)
- Fetch events & actions -> TO-DO [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/fetch-events-and-actions)
- Time Locked accounts -> TO-DO [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/time-locked-accounts)
- Custom Tokens -> TO-DO [here](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/custom-tokens)