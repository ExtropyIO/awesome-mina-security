## o1js basics
>This page can be seen as a brief summary of the [official documentation](https://docs.minaprotocol.com/zkapps/o1js). It aims to highlight key concepts relating to the 01js language. For a more in-depth study, please refer to the [official documentation](https://docs.minaprotocol.com/zkapps/o1js).

- o1js is a TypeScript library you need to import in your .ts file
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
- if statement is not working as usual
```js
// this will NOT work ❌
if (foo) {
  x.assertEquals(y);
}
// this WILL work ✅
const x = Circuit.if(new Bool(foo), a, b); // behaves like `foo ? a : b`
```
- It exists the concept of Foreign Fields which allows to create new finite fields of order different than 2^256, like 2^259. This allows to create zkApps that can communicate with the outside world of cryptography whch may use orders different than 2^256. Indeed it was mainly selected to enable efficient zk proofs calculations. More [hered](https://docs.minaprotocol.com/zkapps/o1js/foreign-fields).