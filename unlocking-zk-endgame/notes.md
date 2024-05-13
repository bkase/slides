* Title: Unlocking the zk endgame
* zk everything
    ...
* It'd be like a zk rollup, but fully decentralized
    * decentralized proving + decentralized sequencing
* We'd have a proof of everything
    * A proof that recursively composes with any arbitrary stuff
* people would be able to prove stuff locally -- extreme edge compute
    * full privacy, predictable gas consumption (not proportional to compute)
* a sophisticated custom zk application model that handles these edge-compute races, supports off-chain contract composition, custom token, and fine-grained permissioning system

* I'm shilling Mina, payments on mainnet for a while.
    * Initial smart contracts release hardforked on Devent successfully yesterday

    * But a zk rollup that can do anything
* Proof of Everything
    * There exists a proof of the block before, an aggregated proof of transactions from some older blocks, and a bunch of other stuff
    * Such that: the proof verifies, the other stuff obeys consensus rules, and the aggregated proof is real
* Decentralized everything
* Decentralized sequencing
    * 
* Devnet is hardforking yesterday (Apr 9)




* Recursive modular proof system -- with folding, chunking, ramlookup -> zkvm support, etc
* First implementation of ouroboros (we beat cardano and ours was in a snark)
* o1js -- the most usable zk sdk (now with custom gates!)
* A standard library with cool treats like a powerful nullifier
* Staged ledger



1. Structure
2. Choose a linear path outline
3. Refine
4. Make headings of slides
5. Leave them blank if you don’t know
6. Revise


Imagine you had the ultimate zk platform

What might you want?
1. Some kind of mechanism for interacting with global shared state (typically a "blockchain")
2. Building composably on top of the shoulders of others through said shared state 
3. Developers can actually use the zk powers of the proof system — they don’t want to learn a new programming language. Something jacascripty. Maybe for some chunks they want to use standard Go or Rust through a riscv, mips, or wasm zkVM. But also they want the power to go deeper.
1. Scale through L2/3/4/5/6/7/infinity

And holding what properties?
1. Decentralization?
2. Privacy?
3. Computational efficiency?
4. Easy Composability?
5. Scaling


(and why for each)


How for each:

Decentralization1: decentralized sequencing — some kind of consensus algorithm
Decentralization3: we’d want as many network participants as we can to work with consumer hardware — so break up the work into little pieces (split blocks with txns and even chunks of txns)
Decentrlization2: decentralized proving — some kind of system for farming proofs to a marketplace — here’s how it could work… (pull from other deck)
Decentralization4: we’d want users to interact with applications without some centralized middleware and without sacrificing trustlessness … and so we need a proof of everything …
…decentralization1.5: that means we need this in a snark, for tricky bits like a VRF calculation that calls for real numbers we can use a Taylor series approximation 

Composability1: okay so composability is nice because you can mix and match and reuse logic,
What is composability? It’s building complex things out of LEGOs that can plug into
You need the ability to loop
Recursion needs to be first class for devs, and unbounded for ultimate power -- aggregate inside yourself
Composability2: in addition to recursion as a way of combining, we should be able to call smart contracts from other contracts. This is part of the power of web3 platforms like Ethereum 
Here’s how that might work….

Privacy: we need to run compute locally on the far edge, and so we should manage race conditions. Here’s how that might work…
Computational efficiency: run compute locally, on the far edge. Services can evaporate if users run their own compute. Gas is predictable.

Anyway, all that is Mina — if you knew me ahead of time you probably knew I was shilling , but hopefully I surprised some of you

On Mina mainnet for last 3+ yrs:
• proof of everything (for payments) using pickled kimchi proof system (plonkish halo2ish w/ IPA over pasta IPA)
• Decentralized sequencing (ouroboros in a snark)
• Decentralized proving (snarketplace)

On Mina mainnet soon (Devnet hf-ed successfully):
• client-side computation with zkApps , and composable through call forests
• no middleware, all users connect directly to zkApps -> browser node=open-mina
* programmable with o1js , iterating with devs for over 2yrs on a typescript/javascript framework for working with circuits and has first class recursion and ability to use custom gates ;; ecosystem of many devs, cool libs like ecdsa and nullifiers etc.
• Mina blockchain proof of everything zk state bridging to evm (nil or Lambdaclass or via alignedlayer)
• Some zk Rollups and zk sdks (protokit / zeko)

And on mainnet later:
• integrate zkVM (o1Labs has grafted a high performance one inside kimchi) into zkApps 
• More performance
• more proof system customization per app, but still rolling up into Mina
• More bridges
• More L2s and app chains 


Check it out, we hardforked Devnet successfully yesterday—mainnet hardfork up next, stay tuned 🎶

I expect Mina will be the first general purpose programmable zk L1 or L2 where you can actually tap into the underlying proof system, use recursion, custom gates, etc…

We’re hiring for loads of roles , and if you’re special we can bend a role to fit you. 

Conclusion CTA: …tell folks to check out o1js? And join the Mina Discord?  Tell them:  If you’re already part of the Mina ecosystem, you should be already getting ready for the big upgrade.  If you’re another chain that wishes it had verifiable compute, check out Mina as a modular layer for your chain. 


