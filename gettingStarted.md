## Getting started with Mina zkApps:
Used [this](https://www.youtube.com/playlist?list=PLItixFkgfjYEvV0SwNpguAu-sNQ4cjyZ0) playlist and official [docs](https://docs.minaprotocol.com/zkapps/writing-a-zkapp).

### Setup:

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
  npm install zkapp-cli  -g
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
1. Move to a desired directory and create a new project  
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


### Deploying the Verifier contract:
1. Create a local test network and give it a name when asked (e.g. `testnet`)
  ```c
  zk config 
  ```
2. Set an API URL to interact with it (e.g `https://api.minascan.io/node/devnet/v1/graphql` but this may change in the future)
3. Set tx fee (e.g `0.1`)
4. Select the account that should pay for fees. You can also create one directly when asked, giving it an alias (e.g. `testpayer`)
5. Done! A link will be prompted in the console with a link to a faucet to get some tMINA tokens for testing. Request them and wait for the tx to be processed.
6. Deploy the contract and select the newly created network when asked
  ```c
  zk deploy
  ```
7. The console will log all the informations and addresses related to the deployment together with a link to the testnet explorer of the deploy tx
8. Open that link and wait till the tx will be processed ([example tx](https://minascan.io/devnet/tx/5JtVRNaeF7qT9qbw2gexhirwNBnxWvFQn6U6SdHELAtwBrHdRJKd?type=zk-tx)). In it you can find:
   - the appState of the `Add.ts` contract just deployed with 1 in appState[0] 
   - the `verificationKeyHash` (if you can't find it enable `Raw JSON` toggle on the top-right corner of the page and search it in the page). This hash is used to identify if an account on Mina is a zkApp. Deploying this contract we actually created a Verifier contract which live on-chain and must have a verification key. 