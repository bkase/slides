<!-- .slide: data-background="#071448" -->
<!-- .slide: data-state="terminal" -->

## One Proof To Rule them All

### Mina's Proof of Everything

By <a href="http://bkase.com">Brandon Kase</a>  <a href="http://o1labs.org">CEO @ o1Labs</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

Note: I'm brandon founding engineer & ceo at o1labs

!!!

### Prelude

<img src="img/prelude.png" width="60%" height="60%" />

Note: Our story is told in the style of a childrens story book illustration of stranger things -- not lord of rings, sorry

!!!

### Decentralized? Blockchains?

<img src="img/decentralized-blockchains.png" width="60%" height="60%" />

Note: Blockchains are supposed to be decentralized...

!!!

### Decentralized? Blockchains?

<img src="img/struggle.png" width="60%" height="60%" />

Note: They've struggled to balance scalability and decentralization. If I have a lot of activity, my chain grows, if my chain grows, it's harder to sync to and limits who can participate. If I try to take shortcuts I add trust assumptions

!!!

### ZK can help (of course!)

<img src="img/zk-duh.png" width="60%" height="60%" />

Note: We all know that zk can help, that's why we're here today

!!!

### Zk Rollups

<img src="img/zkrollup.png" width="80%" height="80%" />

Note: So a zk-rollup takes a bunch of transactions and proves them all, and then that proof of txns (sometimes combined with other txn proofs) eventually goes to a block on some base layer.

!!!

### ZK Rollups

<img src="img/zk-duh.png" width="60%" height="60%" />

Note: This helps right -- we can prove transactions are valid without needing to recompute them. We don't need to store them to replay them later. We're getting more scalable. But....

!!!

### Incremental decentralization

<img src="img/optimize-wrong.png" width="60%" height="60%" />

Note: Most people are optimizing for performance at the cost of scalability or decentralization or both, and then working to improve the scalability/decentralization vectors. Okay sure that's fair and useful: Mina is different...

!!!

### Mina is different

<img src="img/optimize-good.png" width="60%" height="60%" />

Note: Mina has always kept scalability and decentralization in their purest forms, and is now working to improve performance.

!!!

### Mina is different

<img src="img/upside-down.png" width="60%" height="60%" />

Note: it's the upside-down

!!!

### What is it

<img src="img/mina.png" width="60%" height="60%" />

!!!

### When is it?

* Designed late 2017/early 2018 
* Mainnet launch 2021

Note: Long long ago

!!!

### Decentralized! Really!

<img src="img/no-centralized.png" width="60%" height="60%" />

Note: But we didn't even consider centralizing a sequencer or provers. It's a base layer after all!

!!!

### Decentralized sequencing

* You do consensus [in a snark]

!!!

### Decentralized proving

* You do a proof marketplace

!!!

### L2s Decentralized?

<img src="img/tweet1.png" width="90%" height="90%" />

!!!

### Mina tho

<img src="img/tweet2.png" width="90%" height="90%" />

!!!

### What if we rolled up more

* One single proof
* Of everything
* That has ever happened

!!!

### Proof of Everything

<img src="img/proof-of-everything.png" width="60%" height="60%" />

Note: We had a successful L1. No downtime. No emergencies. Totally working. Strong foundation with fixed-function primitives like payments/stake delegations. But we obviously wanted to do more...

!!!

## Mainnet Berkeley upgrade!

<img src="img/graduate.png" width="60%" height="60%" />

Note: Now: The proof of everything can now do arbitrary compute

!!!

### On Mina Mainnet now: Client-side compute

* zkApps -- off-chain compute
  * predictable gas
* off-chain composability
* race-reconciliation (+more)

Note: client-side compute with zkApps , off-chain composable through call forests , action/reducer-style race-reconciliation + more --> fold it into the one proof

!!!

### On Mina Mainnet now: Programmable

* Programmable with TypeScript framework o1js
    * Recursive functions are recursive proofs
    * Custom gates
    * Cool libraries like ECDSA + nullifiers etc.

Note: Control our application -> for our one proof

!!!

### First

<img src="img/first.png" width="60%" height="60%" />

Note: I'm not good at tooting my horn, but this is something that we're really proud of: Mina is the first general purpose programmable zk L1 or L2 where you can actually tap into the underlying proof system, use recursion, custom gates, etc… . It's amazing! We're pioneering this.

!!!

### More Tweets

<img src="img/kobi-tweet.png" width="90%" height="90%" />

!!!

### ZkProgram

```typescript
const AddOne = ZkProgram({
  name: "add-one",
  publicInput: Field,
  //...
```

!!!

### ZkProgram

```typescript
  methods: {
    baseCase: {
      privateInputs: [],

      async method(publicInput: Field) {
        publicInput.assertEquals(Field(0));
      },
    },
```

!!!

### ZkProgram

```typescript
    step: {
      privateInputs: [SelfProof],

      async method(
        publicInput: Field,
        pi: SelfProof<Field, void>
      ) {
        pi.verify();
        pi.publicInput.add(1)
          .assertEquals(publicInput);
      },
    },
  },
});
```

!!!

### Try it

<img src="img/try-recursion.png" width="60%" height="60%" />

Note: If you want to play with recursion -- you should really try our stuff!

!!!

### Building

<img src="img/building-verifiable.png" width="60%" height="60%" />

Note: Folks are building verifiable applications today: game engines, L2s, DeFi experiments, voting, decentralized science -- it's early! Completely new computing paradigm, so it's hard to say what people will come up with.

!!!

### Modular Summit

<img src="img/modular-summit.png" width="60%" height="60%" />

Note: Modular summit: So how does this fit into a modular world?

!!!

### Today, tomorrow

<img src="img/today-tomorrow.png" width="60%" height="60%" />

Note: Today it's a proof of everything on Mina -- but we want to make it everything everything. That means more bridges. It means more proof system adapters. It means embedding a zkVM. It means email/tls/oauth circuits. Let's think about tomorrow...

!!!

### Proofs go to live

<img src="img/proof-go-to-live.png" width="60%" height="60%" />

Note: Mina is where proofs go to live. What does living mean? Combine them, mix and match them. And we're going to plug into Ethereum via a new AlignedLayer integration. Geometry is working on a Celestia integration. L2s settle to Mina. Other chains can settle back into Mina.

!!!

### Proofs go to live

* Reusable building blocks like Ethereum, but with real large datasets from the web2 world
* The power to scale verifiable compute through recursion (super simply)
* Prove lazily. Verify eagerly: Incrementally share (p2p?) and resume computation

Note: Curve / convex / etc (what's the thing people use these days?) but with web2 data... 

!!!

### Let's make an internet of true things

<img src="img/internet-of-true.png" width="60%" height="60%" />

Note: The proof of everything when it's really everytyhing can feed into this realy internet of true things. Specifically: Build in our developer grant programs, deploy your apps on our long-standing, quickly-updating Devnet and join the growing Mina community! Come make an internet of true things with us!

!!!

<!-- .slide: data-background="#071448" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a>  <a href="http://o1labs.org">CEO @ o1Labs</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

O(1) Labs: [https://o1labs.org](https://o1labs.org)
Mina: [https://minaprotocol.com](https://minaprotocol.com)


Note: Join the community. Build zkApps with o1js. Contribute to Mina.


