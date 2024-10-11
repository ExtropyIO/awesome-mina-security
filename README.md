# awesome-mina-security

A curated list of Mina resources with a focus on security from [Extropy.io](https://www.extropy.io/) .

## Getting started
👉🏻 [Mina and zkApp essentials](./GettingStarted/MinaAndZkAppEssentials.md)\
👉🏻 [Basics of o1js](./GettingStarted/o1jsBasics.md)\
👉🏻 [Deploy and interact with your first zkAPP under 10 minutes](./GettingStarted/deployIn10min.md) 

## Security aspects of zkApps
A list of security considerations to keep in mind when writing zkApps:
>NOTE: links should open the related page of the docs at the correct point where the issue is being mentioned. However for unknown reason after a page is opened it quickly goes at the top. At the moment please search for the mentioned words in the page, we will try to find a solution asap to this problem

Generic issues:\
🔒 [Underconstrained Proofs: unproved logic can be manipulated by a malicious prover](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#:~:text=exploit%20your%20application.-,Underconstrained,-proofs%3A%20Successfully%20%22calling)\
🔒 [Overriding `init()` is unnecessary if only `super.init()` is called inside without no other state initialization](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/smart-contracts#initializing-state)\
🔒 [Conversion between an o1js `Bool` and javascript `boolean` is always truthy](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#rolling-your-own-provable-methods:~:text=boolean%2C%20is%20always-,truthy,-%2C%20so%20this%20always)\
🔒 [Using `Provable.asProver()` on inputs moves them out from the zk proof: anyone can remove the Provable block, execute the off-chain code and send a valid proof passing constraints](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#fix-adding-the-missing-constraints:~:text=as%20well.%20Progress!%20%F0%9F%9A%80-,However,-%2C%20the%20statement%20about)\
🔒 [Writing custom low-level provable methods may add underconstrained logic](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#rolling-your-own-provable-methods)\
🔒 [Never trust methods callers](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#second-problem-we-trusted-the-caller)\
🔒 [Foreign Field Arithmetic should be used with caution](https://docs.minaprotocol.com/zkapps/o1js/foreign-fields#three-kinds-of-foreign-fields)\
🔒 [Usage of `requireNothing()` when retrieving On-chain Values may be dangerous](https://docs.minaprotocol.com/zkapps/o1js/foreign-fields#three-kinds-of-foreign-fields)\
🔒 [if condition is used instead of `const x = Circuit.if(new Bool(foo), a, b);`](https://docs.minaprotocol.com/zkapps/o1js/basic-concepts#conditionals)\
🔒 [Onchain merkle tree root not synced with offchain merkle root](https://docs.minaprotocol.com/zkapps/o1js/merkle-tree#:~:text=is%20always%20in-,sync,-with%20the%20actual)\
🔒 [Use on-chain values without checking them both at verification and proving time](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/smart-contracts#:~:text=off%2Dchain%20execution.-,When,-you%20use%20an)\
🔒 [Circumventing 01js security features should be avoided](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#fix-adding-the-missing-constraints:~:text=flag%20in%20general.-,Security,-advice%20%232%3A%20Don%27t)\
🔒 [Possible race conditions when many users read/write the state concurrently](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/smart-contracts#:~:text=and%20update%20state-,concurrently,-.%20It%20is%20applicable)\
🔒 [Always extend the official `TokenContract` standard instead of building a custom token from scratch](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#when-developing-a-token-extend-a-standard-token-contract) + [mina-fungible-token repo (built on top of `TokenContract`)](https://github.com/MinaFoundation/mina-le-token)

Actions & reducers issues:\
🔒 [The `reduce()` method breaks if more than the hard-coded number (default: 32) of actions are pending](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/actions-and-reducer)\
🔒 [Be careful when creating Account Updates from a reducer](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#be-careful-with-creating-account-updates-from-a-reducer)\

Permissions related issues:\
🔒 [Explicitly setting Permissions to default is unnecessary if no other permissions are modified in the `init()` function](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#default-permissions:~:text=send%3A%20Permissions.proof()%2C-,Alternatively,-%2C%20you%20can%20just)\
🔒 [Permissions not locked down enough + advices for setting permissions](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#lock-down-permissions-as-much-as-possible)\
🔒 [Calling external contracts with permissions not locked down enough](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#only-call-external-contracts-with-locked-down-permissions)\
🔒 [`editState` permission set to none](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#types-of-permissions:~:text=the%20smart%20contract.-,However,-%2C%20imagine%20if%20a)\
🔒 [`send` permission set to none](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#default-permissions)\
🔒 [if `access` and `receive` permissions are not set to `none` a deadlock may occur while doing concurrent state updates + safe way of doing an airdrop](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#dont-deadlock-your-zkapp-by-interacting-with-unknown-accounts)\
🔒 [smart contract interactions are limited to `signature` instead of `proof`](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#example-unsecurecontract:~:text=the%20transaction%20succeeds.-,However,-%2C%20this%20way%20of)\
🔒 [restrictive permissions can be circumvented if `setPermissions` is not set to impossible](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#example-impossible-to-upgrade:~:text=For%20the-,sake,-of%20security%2C%20it)\
🔒 [lack of access controls](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/feature-overview/permissions#example-unsecurecontract:~:text=not%20very%20secure%3A-,Anyone,-can%20call%20the)\
🔒 [Minting unlimited tokens to himself is possile for an attacker if a custom token contract does not change `access` permission from `none` to at least `proof`](https://docs.minaprotocol.com/zkapps/writing-a-zkapp/introduction-to-zkapps/secure-zkapps#dont-deadlock-your-zkapp-by-interacting-with-unknown-accounts:~:text=can%20mint%20an-,arbitrary,-number%20of%20tokens)\
🔒 Setting `access` to `impossible` makes your account inaccessible: no one will ever be able to call their contract againt. It's like deleting.

## Development best practices
- Do not assume that the fact that someone has to pay transaction fees doesn't have incentives to attack your zkApp
- Work with the compiler, don't try to get round the way that 01js works
- Be very aware of what is part of the proof and what is not
- Make sure that your code is not under constrained - all the necessary checks are
part of the proof.
- Don't make assumptions about the caller
- Be aware of pre conditions for the update
- Restrict the permissions as much as you can.
- Be careful when calling external contracts and accounts, you need to be sure your
call will succeed.
- Make sure the access control is accurate and enforced
- Have a test suite, don't just test the 'happy path'
- Put yourself in an adversarial mindset and try to break your code
- Get your code audited
- Have a plan to react to attacks
- Keep up to date, we are at a very early stage.

## Resources:
Documentation:
- [Official Docs](https://docs.minaprotocol.com/) 
- [Mina tutorials](https://docs.minaprotocol.com/zkapps/tutorials)

Tools & Frameworks:
- [Awesome mina tools](https://github.com/nerdvibe/awesome-mina) 
- [zkApp Cli](https://github.com/o1-labs/zkapp-cli)
- [OpenMina: setting up a node from the smartphone](https://openmina.com/)
- [zkApps official examples](https://github.com/o1-labs/o1js/tree/main/src/examples/zkapps) 
- [Mina Playground](https://www.minaplayground.com/)
- [Protokit](https://protokit.dev/)
- [GraphQL queries](https://graphql.minaexplorer.com/)

News:
- [zkok MinaBlog](https://minablog.zkok.io/) 
- [Mina June 4 2024 hard fork announcement](https://minaprotocol.com/blog/mina-protocols-upcoming-major-upgrade-everything-you-need-to-know) 

Projects on Mina:
- [Awesome Mina ZKApps](https://github.com/iam-dev/awesome-zkApps) 
- [zkApp list from zkok](https://zkok.io/)
- [list from Mina Foundation](https://github.com/MinaFoundation/list-of-projects?tab=readme-ov-file) 
- [Anomix](https://github.com/anomix-zk/anomix-network/tree/main)
- [Zeko](https://github.com/zeko-labs)
- [PunkPoll](https://www.punkpoll.io/)
- [ID-Mask](https://idmask.xyz/)
- [Hakata](https://hakata.io/) 
- [ZkNoid](https://www.zknoid.io/)
- [ZKPassport](https://github.com/MinaFoundation/Core-Grants/issues/18)
- [Snarky.bio](https://snarky.bio/) 
- [PaimaStudios](https://paimastudios.com/) 
- [Mina Email](https://github.com/0xStruct/moolah/tree/main)
- [Clor.io](https://clor.io/)
- [zk-invoices](https://github.com/kriss1897/zk-invoices/tree/main)
- [zkPass](https://zkpass.org/)
- [ZKON](https://www.zkon.xyz/)
- [zeko](https://zeko.io/)
- [Hazook](https://github.com/ycryptx/Hazook-Fast-Zk-Rollup)
- [Lumina DEX](https://luminadex.com/)
- [SocialCap](https://socialcap.app/)

  

---
## Old Links
> This section is a WIP

- https://www.di.ens.fr/~nitulesc/files/Survey-SNARKs.pdf
- https://arxiv.org/pdf/1906.07221.pdf 
- https://medium.com/magicofc/interactive-proofs-and-zero-knowledge-b32f6c8d66c3 
- https://blog.zkga.me/intro-to-zksnarks 
- https://media.consensys.net/introduction-to-zksnarks-with-examples-3283b554fc3b 
- https://z.cash/technology/zksnarks/
- https://masked.medium.com/the-coda-protocol-bbcb4b212b13 
- https://docs.minaprotocol.com/en/zkapps/how-to-write-a-zkapp 
- https://a16z.com/2022/04/15/zero-knowledge-proofs-hardware-decentralization-innovation/
- https://www.youtube.com/watch?v=bjSAf41PWWI
- https://www.youtube.com/watch?v=Zls_QlI8fn8
- https://www.youtube.com/watch?v=ByrymOorggc
- https://minaprotocol.com/blog/22kb-sized-blockchain-a-technical-reference
- https://minaprotocol.com/blog/kimchi-the-latest-update-to-minas-proof-system
- https://medium.com/minaprotocol/meet-pickles-snark-enabling-smart-contract-on-coda-protocol-7ede3b54c250
- https://minaprotocol.com/blog/what-are-zk-snarks
- https://medium.com/minaprotocol/https-and-snapps-bridging-cryptocurrency-and-the-real-world-962beb21cf2b 
