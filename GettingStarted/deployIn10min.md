## Deploy and interact with your first zkAPP under 10 minutes:
Used [this](https://www.youtube.com/playlist?list=PLItixFkgfjYEvV0SwNpguAu-sNQ4cjyZ0) playlist and official [docs](https://docs.minaprotocol.com/zkapps/writing-a-zkapp).

- [Deploy and interact with your first zkAPP under 10 minutes:](#deploy-and-interact-with-your-first-zkapp-under-10-minutes)
  - [Initial setup:](#initial-setup)
  - [Create your first project:](#create-your-first-project)
  - [Deploy the Verifier contract:](#deploy-the-verifier-contract)
  - [Interactions](#interactions)
  - [Testing](#testing)


### Initial setup:

1. Install Node js and verify your version with 
  ```c
  node --version // Example output: v18.18.0
  ```
2. Verify npm is installed
  ```c
  npm --version // Example output: 10.4.0
  ```
3. Install [zkApp Cli](https://github.com/o1-labs/zkapp-cli)
  ```c
  npm install -g zkapp-cli
  ```
4. Verify zkApp Cli is installed
  ```c
  zk --version // Example output: 0.17.2
  ```
5. Check the whole environment configurations and versions
  ```c
  zk system 
  ```

### Create your first project:
1. Move to a desired directory and create a new project. It automatically will install also 01js  
  ```c
  zk project myFirstProject 
  ```
2. Select `none` as the project template when asked and open it in your IDE
3. Install dependencies in project folder
  ```c
  npm install
  ```
4. A dummy contract named `Add.ts` will be generated in the `/src` folder. This contract allows to set `1` as the value for the state variable `num` and update it adding `2` when calling the `update()`.
```js
import { Field, SmartContract, state, State, method } from 'o1js';

/**
 * The Add contract initializes the state variable 'num' to be a Field(1) value by default when deployed.
 * When the 'update' method is called, the Add contract adds Field(2) to its 'num' contract state.
 */
export class Add extends SmartContract {
  @state(Field) num = State<Field>();

  init() {
    super.init();
    this.num.set(Field(1));
  }

  @method async update() {
    const currentState = this.num.getAndRequireEquals();
    const newState = currentState.add(2);
    this.num.set(newState);
  }
}
```
5. Another file named `interact.ts` will be generated in the `/src` folder. This file takes care of interacting with `Add.ts` and is where ZKP is actually generated using `tx.prove()` method.
6. Build the smart contract
  ```c
  npm run build
  ```

### Deploy the Verifier contract:
1. Create a local test network and give it a name when asked (e.g. `testnet`)
  ```c
  zk config 
  ```
2. Set the network type between testnet/mainnet when asked
3. Set an API URL to interact with it (e.g `https://api.minascan.io/node/devnet/v1/graphql` but this may change in the future)
4. Set tx fee (e.g `0.1`)
5. Select the account that should pay for fees. You can also create one directly when asked, giving it an alias (e.g. `testpayer`). A private key and public key will be generated and stored in the `testpayer.json` file stored in the xkapp-cli folder somewhere in your computer. 
6. Done! A link will be prompted in the console with a link to a faucet to get some tMINA tokens for testing to the newly created account. Request them and wait for the tx to be processed.
7. Deploy the contract and select the newly created network when asked
  ```c
  zk deploy myFirstProject
  ```
1. The console will log all the informations and addresses related to the deployment together with a link to the testnet explorer of the deploy tx
2. Open that link and wait till the tx will be processed ([example tx](https://minascan.io/devnet/tx/5JtVRNaeF7qT9qbw2gexhirwNBnxWvFQn6U6SdHELAtwBrHdRJKd?type=zk-tx)). In it you can find:
   - the appState of the `Add.ts` contract just deployed with 1 in appState[0] 
   - the `verificationKeyHash` (if you can't find it enable `Raw JSON` toggle on the top-right corner of the page and search it in the page). This hash is used to identify if an account on Mina is a zkApp. While deploying this contract we actually created a Verifier contract which live on-chain and must have a verification key. Important: the contract code is not stored on chain, the zkApp can only verify the proof you send it through the verification Key.

### Interactions
Within the file `interact.ts` there is the code for interacting with the on-chain state. Interaction take place between the **Prover** (the `feepayer`) and the **Verifier** (the zkApp deployed).

Let's explore the file `interact.ts` (some lines are omitted here):
```js
// reads privateKey and publickKey fields from testnet.json file
let zkAppKeysBase58: { privateKey: string; publicKey: string } = JSON.parse(
  await fs.readFile(config.keyPath, 'utf8') 
);
// creates a PrivateKey object converting from Base58 representation
let zkAppKey = PrivateKey.fromBase58(zkAppKeysBase58.privateKey)
// retrieves the corresponding public key of the zkApp
let zkAppAddress = zkAppKey.toPublicKey();

// creates a new instance of the Add contract and compile it
let zkApp = new Add(zkAppAddress);
await Add.compile();

// creates a tx object executing the Add.update() method locally and waits for its result
// then the tx object containing the result of the computation is used to generate a ZKP
// lastly the feepayer signs the tx and sends the proof on chain
  console.log('build transaction and create proof...');
  let tx = await Mina.transaction(
    // feepayerAddress should be specified and its data are read from the file which path is stored in config.json . In this example are read from testpayer.json we created during deployment
    { sender: feepayerAddress, fee },
    async () => {
      await zkApp.update();
    }
  );
  await tx.prove(); // generates the ZKP
  const sentTx = await tx.sign([feepayerKey]).send();

```

The command to execute `interact.ts` is:
```c
    npm run build && node build/src/interact.js DEPLOY_ALIAS // for this example DEPLOY_ALIAS=testnet
```
After it if we look to the [appState](https://minascan.io/devnet/account/B62qkebMxnT4U7vnDJ5WuRMMvAMfRZmLCe7DBS4Q4DLvGHnzuxAv498) of the zkApp we will see 3 in the field 0 since the update() method increased the initial state, which was 1, by 2.


### Testing
TO-DO:
- https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/getting-started-zkapps#3-add-testing-code 
- https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/getting-started-zkapps#5-create-integration-test
- https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/getting-started-zkapps#7-test-with-lightnet
- https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/testing-zkapps-lightnet
- https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/getting-started-zkapps#8-test-with-a-live-network
- https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/testing-zkapps-locally

  ```c
  npm run test
  ```