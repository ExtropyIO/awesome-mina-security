## o1js basics
>This page can be seen as a brief summary of the [official documentation](https://docs.minaprotocol.com/zkapps/o1js). It aims to highlight key concepts relating to the 01js language. For a more in-depth study, please refer to the [official documentation](https://docs.minaprotocol.com/zkapps/o1js).

- o1js is a TypeScript library you need to import in your .ts file
- If your are new to TypeScript check [this](https://www.youtube.com/watch?v=ahCwqrYpIuM&ab_channel=Fireship) introductive video
- o1js provides data types and methods that are provable: You can prove their execution.
- the basic unit of o1js is the `Field` element which can store a number to almost 256 bits in size, like `uint256` in Solidity
- Other types are:
```js
new Bool(x);   // accepts true or false. NOTE: It's different than JS Boolean
new UInt64(x); // accepts a Field - useful for constraining numbers to 64 bits
new UInt32(x); // accepts a Field - useful for constraining numbers to 32 bits

PrivateKey, PublicKey, Signature; // useful for accounts and signing
new Group(x, y); // a point on our elliptic curve, accepts two Fields/numbers/strings
Scalar; // the corresponding scalar field (different than Field)

CircuitString.from('some string'); // string of max length 128
```
- It's possible to have custom data types creating a class that extends `Struct({ })` 
```js
class Point extends Struct({
  x: Field,
  y: Field,
}) {}
```
- if condition is not working as usual Typescript code
```js
// this will NOT work ❌
if (foo) {
  x.assertEquals(y);
}
// this WILL work ✅
const x = Circuit.if(new Bool(foo), a, b); // behaves like `foo ? a : b`
```
- Gadgets are small, reusable low-level building blocks that simplify the process of creating new cryptographic primitives. In o1js we have [Bitwise Operations](https://docs.minaprotocol.com/zkapps/o1js/bitwise-operations) and Foreign Field Arithmetic
- It exists the concept of Foreign Fields which allows to create new finite fields of order different than 2^256, like 2^259. This allows to create zkApps that can communicate with the outside world of cryptography whch may use orders different than 2^256. Indeed it was mainly selected to enable efficient zk proofs calculations. More [hered](https://docs.minaprotocol.com/zkapps/o1js/foreign-fields).
- Hashing functions:
  - The [Poseidon](https://o1-labs.github.io/proof-systems/specs/poseidon.html) zero knowledge native hash function operates over the native Pallas base field and uses parameters generated specifically for Mina which makes Poseidon the most efficient hash function available in o1js.
  - Keccak (later standardized as SHA-3) is an hash function that requires binary arithmetic and in o1js particularly uses the `Bytes` type. It operates over binary data and is not native to most zero knowledge proofs. For this reason, Keccak is not as efficient as Poseidon, but it is still very useful for verifying Ethereum transactions and blocks. Variants: Keccak256, Keccak384, Keccak512 and the SHA3_256, SHA3_384, SHA3_512. More [here](https://docs.minaprotocol.com/zkapps/o1js/keccak)
  - SHA-2 is also supported by 01js but only SHA-256 is available, SHA-512 is not. The same consideration done for Keccak are valid for SHA-2 compared to Poseidon hashes.