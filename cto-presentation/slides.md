<!-- .slide: data-background="#071448" -->
<!-- .slide: data-state="terminal" -->

# Hello; Brandon

By <a href="http://bkase.com">Brandon Kase</a>  <a href="http://o1labs.org">CTO @ O(1) Labs</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

!!!

### About me

<img src="img/kid-brandon.png" width="55%" height="55%"></img>

Note: Obsessed with computers since I was a kid. I knew I wanted to build software early. And then continued to be obsessed with it until I was like 28.

!!!

### About me2

<img src="img/evan-brandon.png" width="55%" height="55%"></img>

Note: Studied CS @ CMU (studied with Evan Shapiro)

!!!

### Prefounding

<img src="img/evan-izzy-brandon.png" width="55%" height="55%"></img>

Note: Started nights and weekends with izzy and Evan

!!!

### First prototype

<img src="img/javascript-mina.png" width="55%" height="55%"></img>

Note: The first prototype of Mina was in Flow-typed JavaScript!

!!!

### Company Founded, now OCaml

<img src="img/ocaml-mina.png" width="55%" height="55%"></img>

Note: But then we moved to OCaml

!!!

### Folks joined

<img src="img/bigger-team.png" width="55%" height="55%"></img>

Note: Lots of protocol engineering for me, and Nacera and the first wave of protocol engineers joined

!!!

### My path

<img src="img/brandon-hats.png" width="55%" height="55%"></img>

Note: I went through protocol engineering, ramping up other folks, product engineering, manager of a few folks, early devrel and partner engineering, build tooling, getting obsessesed with defi and learning a lot about competitors and then wearing random hats. And now CTO

!!!

### In this talk

* ~~Intro myself~~
* <!-- .element: class="fragment" data-fragment-index="1" --> Background on the CTO role <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### CTO Responsibilities

* Defining and communicating technical vision and stories; to vendors, to the media, to ourselves; thought-leadering
* <!-- .element: class="fragment" data-fragment-index="1" --> Keeping up with competitors/market trends/eng and technology trends <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Contributing to the high-level strategy/roadmap <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> Ensuring our team is building maintainable software <!-- .element: class="fragment" data-fragment-index="3" -->

Note: And I try hard to do so without being in a vaccuum. I want you to give me your ideas and your feedback and tell me if you disagree with something. Let's argue about it (constructively). I work a lot with Aneesha. and Emre and Steve and many engineers. I come up with ideas and then try to get folks' feedback and convince myself and then others that it makes sense. Then we do it.

!!!

### Brandon CTO what do

<img src="img/multitool.png" width="55%" height="55%"></img>

Note: In practice, since I've become CTO -- at first I had a lot of random x-organizational work + little things, but then I had a streak of offsites: eng leads offsite, architect offsite, MF+juraj join offsite, zksummit, sdk team offsite, platform eng offsite, now this LT offsite+company offsite. The point really getting into strategy conversations with different teams, digging into issues that different folks are seeing. Plus at the same time, I'm working closely enough with zeko team, viable systems, and lambdaclass that I can observe how different folks work and the pros and cons of those styles. Now I want to act on some of these learnings over the next few months.

!!!

### In this talk

* ~~Intro myself~~
* ~~Background on the CTO role~~
* <!-- .element: class="fragment" data-fragment-index="1" --> Reflections on what we've built <!-- .element: class="fragment" data-fragment-index="1" -->

Note: But before we go into that, let's reflect!

!!!

### We've built a lot of cool stuff

<img src="img/o1-collage.png" width="55%" height="55%"></img>

!!!

### Snarky v1

<img src="img/snarkyv1.png" width="85%" height="85%"></img>

Note: Custom embedded DSL in OCaml for building zero knowledge proofs. Used exentensively within Mina Protocol codebase. Function abstraction and types.

!!!

### Kimchi: Recursive-friendly modular proof system

<img src="img/kimchi.png" width="55%" height="55%"></img>

Note: A plonkish proof system with an inner-product-argument. Hyper-fine-tuned for efficient recursion. No trusted setup. Super modular. Plonk-to-the-limit: eg supporting logarithmic lookup arguments. Chunking so we can run huge circuits, ramlookup so we can make zkVMs, folding so we can zkVM large execution traces, backends can be hot-swapped to an EVM-friendly kzg-bn254 so we can build the zkVM for OP and it can be used for a Mina bridge.

!!!

### Ourorboros in a Snark

<img src="img/ouroboros.png" width="55%" height="55%"></img>

Note: First implementaiton of ouroroboros in production -- we beat cardano whose paper we uses and ours was in a snark!

!!!

### Staged Ledger

<img src="img/staged-ledger.png" width="55%" height="55%"></img>

Note: A data structure that tracks transaction applications. Instantiates a parallel scan with user txns, fees, and coinbases. Handles all sorts of complex edge cases.

!!!

### o1js

<img src="img/o1js.png" width="85%" height="85%"></img>

Note: o1js the most usable zk sdk (don't ask me, the community says so). Also the most powerful! Now with custom gates! Not just for zkApp smart contracts but for circuits too. Comes with a standard library with merklized structures, nullifiers. As Steve shared, it's faster now too.

!!!

### o1js Accoutrement

<img src="img/o1js-extras.png" width="55%" height="55%"></img>

Note: zkApp CLI to makes it easy to get started and deploy and work with zkApps. A Mina signer tool to make it easy to sign transactions. Integrations with wallets. An archive node integration to access actions/events and other chain data. Beautiful documentation and examples and tutorials.

!!!

### Testing tooling

<img src="img/precision.png" width="55%" height="55%"></img>

Note: Integration testing framework for the protocol, nightly CI, lightnet (for o1js local testing)

!!!

### Cost-efficient testing substrate

<img src="img/cost-saving.png" width="55%" height="55%"></img>

Note: Cloud infra that can run testnets cheap

!!!

### Our Team

<img src="img/8bit-team.png" width="55%" height="55%"></img>

Note: Our team is top class and we've built such great stuff! Juraj told me the other day: "Literally every idea we have of something that can go wrong or a bug, we always find an open GitHub issue for it." Clap for yourselves

!!!

### In this talk

* ~~Intro myself~~
* ~~Background on the CTO role~~
* ~~Reflections on what we've built~~
* <!-- .element: class="fragment" data-fragment-index="1" --> Engineering values: Review and Instantiate <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Engineering Values

<img src="img/engineering-values.png" width="75%" height="75%"></img>

Note: Engineering values

!!!

### Engineering Values (that we idealize)

* We keep strong clarity of purpose and high visibility
* <!-- .element: class="fragment" data-fragment-index="1" --> We endeavor for engineering excellence <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> We strive for excellence over perfection <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> We are humans first <!-- .element: class="fragment" data-fragment-index="3" -->

Note: 1. Keep work well-defined, visible, using evidence, and sharing work quickly, keeping in mind the end-user as well as developers reading the code. 2. scope goes first, then speed, quality never suffers. High architecture and coding standards. Make sure we have a design and approach through RFCs first! Aggressively favor simplicity. Not easy. Simple. There's a difference. We finish things. Really finish. Ship them. It's not done until it's in a usable product 3. It's okay to ask for help, to fail, and to take calculated risks given constraints on time. Growth mindset. Always be open to learning. Help your teammates. 4. Make spaces for voices, culture of execution and delivery AND HAVING FUN. ... So this is our ideal, how have we gotten closer to this...

!!!

### Reflections of improvements in recent times

1. Better about PRDs before work starts
2. <!-- .element: class="fragment" data-fragment-index="1" --> More rigorous about RFCs before implementation starts <!-- .element: class="fragment" data-fragment-index="1" -->
3. <!-- .element: class="fragment" data-fragment-index="2" --> More rigorous data driven discourse for making technical decisions <!-- .element: class="fragment" data-fragment-index="2" -->
4. <!-- .element: class="fragment" data-fragment-index="3" --> Share more through demos <!-- .element: class="fragment" data-fragment-index="3" -->
5. <!-- .element: class="fragment" data-fragment-index="4" --> Testing game way up <!-- .element: class="fragment" data-fragment-index="4" -->
6. <!-- .element: class="fragment" data-fragment-index="5" --> Mistakes okay + retro to learn <!-- .element: class="fragment" data-fragment-index="5" -->
7. <!-- .element: class="fragment" data-fragment-index="6" --> Architects more mentoring <!-- .element: class="fragment" data-fragment-index="6" -->
8. <!-- .element: class="fragment" data-fragment-index="7" --> Empowered to put things in place to do our job well <!-- .element: class="fragment" data-fragment-index="7" -->

Note: (1) shoutout to crypto team, (3) shoutout to Deepthi with capturing data on berkeley discussions, (4) shoutout to the sdk team, (5) itn testing / sdk team / established a qa team ; reformated velocity to platform eng team, (6) retro/reflect often; ecdsa is a great example, (7) shoutout Matthew in crypto space, (8) new lead roles and spaces for easier x-team collaboration at the eng level shoutout steven and gregor + devrel guardian rotation + tests to nightlies 

!!!

### Wow so much!

Led by Aneesha's work on improving engineering culture and execution and the EMs!

Note: Clap for Aneesha and the EMs

!!!

### And more

<img src="img/engineering-values.png" width="55%" height="55%"></img>

Note: We've made this great progress, but there's more to do! And Aneesha and our team will continue! I want to dig deeper into the hands-on-the-keyboard part of the process. And contribute myself especially through the software we write and the systems in place to push us to write it better.

!!!

### Acting on the learnings

<img src="img/multitool.png" width="55%" height="55%"></img>

Note: And lots of ideas came out of synthesis from observations of how we work + some of the conversations I've had remotely and in person with folks over the last few months. ...

!!!

### Writing Software Things

<img src="img/write-software-culture.png" width="55%" height="55%"></img>

Note: I'm going to share a few of the most clear and relevant for the full company syntheses

!!!

### Wearing Hats (1)

<img src="img/hats1.png" width="55%" height="55%"></img>

Note: Problem: We have a lean organization. Sometimes we'll have a little extra high priority work on teams with a little bit fewer staff that need help. Sometimes we'll have projects where work crosses teams. 

!!!

### Wearing Hats (2)

<img src="img/hats2.png" width="55%" height="55%"></img>

Note: We have a culture where folks from other spaces contribute! and that's great! But what can happen is we end up doing work without getting the right folks involved, the right visibility, because even though we share "with our team", we're not sharing "with the right team". This leads to miscommunication, feeling loss of control, the wrong work getting done and needing to be redone, code not shipping, etc. I can find examples of this within the last few months with almost every pair of teams!

!!!

### Wearing Hats (3)

<img src="img/hats3.png" width="55%" height="55%"></img>

Note: Solution: Wear your hat! Continue being proactive, jumping in where you see you can help (tell your manager of course!), but acknowledge your hat. If you are doing work for the platform engineering team. You need to do that work in that context, add those folks for review on your RFC, your pull requests, discuss in the right slack channel. If you're contributing to the protocol team working on berkeley. Share with Deepthi! Work in that space. If you're writing cryptography code, either within Kimchi or o1js or some other tool -- let them know! Follow the crypto team's development process. Ask the managers or tech leads of the other teams if you have questions on how to do this, everyone wants to help you help O(1)!

!!!

### Writing Software Things

* ~~Wearing hats~~
* <!-- .element: class="fragment" data-fragment-index="1" --> [REDACTED] <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Brandon actions <!-- .element: class="fragment" data-fragment-index="2" -->

!!!

### Problem

<img src="img/house-fall.png" width="55%" height="55%"></img>

Note: You know when you've rushed to put something together and it struggles to stay up. I'm using this metaphor so the non-engineers can follow along, but all of the software engineers know what I'm takling about.

!!!

### Reality

<img src="img/slide.png" width="55%" height="55%"></img>

Note: It's no secret that in many places in our codebases across the different project domains, we've made this mistake. In fact, I can point to more than one example of where we've endured years worth of technical debt from tiny shortcuts and underinvested upfront thinking. Pushing this investment to the front improves our producitivity, readability of the code, interest in our work, happiness, ability to onboard our future selves and colleagues, etc.

!!!

### Is this at odds with being product-first?

<img src="img/no.png" width="55%" height="55%"></img>
<!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### This is product first

<img src="img/product-first.png" width="55%" height="55%"></img>

Note: You need this to be iterative and adapt to new information quickly. We need to build something that we can change. We can only change if we have something we understand, something broken into pieces, something that is well-tested.

!!!

### RFCs help!

<img src="img/rfcs-help.png" width="55%" height="55%"></img>

Note: RFCs help push us to invest in upfront thinking, but as is, it doesn't really push us to write our software in any particular way. We're missing a mechanism for highlighting part of the developer experience, and a mechanism for realizing this in a unified way across our engineering organization...

!!!

### Language-driven design

<img src="img/language-driven.png" width="55%" height="55%"></img>

Note: Myself and the architects believe that the mechanism that is most core to our culture to realize a unified devx system is Language-driven design thinking. Because when done properly it emits clear, readable, maintainable, and consistent software that is easy to test. I'll illustrate through examples: There are a few examples of powerful little languages, or DSLs, we've built for ourselves over the years, but rather than focus on those existing ones: I want to highlight three places across three codebases across three languages that could dramatically improve from this kind of thinking

!!!

### Blockchain Builder

<img src="img/blockchain-builder.png" width="55%" height="55%"></img>

Note: Problem: We struggle to represent blocks in a blockchain everytime we want to write a test. We never built a system that can cleanly represent such states. As a consequence, we naturally push tests that should be unit tests to the integration test layer because in order to build a blockchain, we need to actually run the whole Mina. This means we also struggle to reach edge cases due to the expensive nature of creating these blocks in integration tests leading to testing few cases.

!!!

### Blockchain Builder DSL

```ocaml
let chain_shape =
  ( exists_blocks 10 |> such_that any_txns
  , exists_block |> such_that (*...*)
  , ( exists_blocks 5
    , exists_blocks 5 ) )
in

let generator = solve chain_shape in
(* unit test with this blockchain generator *)
```

Note: Now we can represent shapes of chains with specific properties that we care about, and unit test with this generator.

!!!

### Kimchi proving logic 

<img src="img/kimchi-prover.png" width="55%" height="55%"></img>

Note: Problem: There are distinct operations in the proving logic which are the same for any kimchi, but our implementation is just inlines it all. This makes it hard to reconfigure the system for different bakcends (OP zkVM vs Mina for example). Plus our tests test both the bakcend and the proof logic simultaneously.

!!!

### Kimchi proving logic

```rust
trait Prover {
  fn compute_and_absorb_commitment()
  fn squeeze_challenge()
  // ...
  fn compute_evaluation_proof()

  // Works for any instantiation of the prover!
  fn prove() -> /*...*/ {
    for (col in vk) { /*..*/ }
    for (col in witness) { /*..*/ }
    // ...
  }
}
```

Note: Now we implement a trait where we can choose IPA vs kzg, poseidon vs keccak sponge, etc. This means our prover is implemented once and only once. We can test separately. We can trivially add new backends with assurance that our code holds.

!!!

### Pickles -> ZkProgram

<img src="img/pickle-to-program.png" width="55%" height="55%"></img>

Note: Problem: Pickles is one big ball of spaghetti, you can only call it all at once. It keeps track of its own keys and constraints and proving. Not only does this make understanding and testing the pieces of the system difficult, but it becomes hard and LIMITING in the ways that we can express our recursive proof API on the o1js side. All because we can't decouple laying out the constraints from proving them.

!!!

### Pickles -> ZkProgram

```typescript
// program for any backend!
const program = ZkProgram(/*...*/);

const [pk, vk] = program.compile();

return WasmKimchi.prove(
  program.sell(secretCode)
);
// or NativeKimchi.prove
// or O1cloud.prove
```

Note: By breaking pickles into specific clear steps on the backend, we empower the o1js sdk with new capabilities. We don't need to reach into the pickles code to add features. We can cache proving/verification keys by touching them here. We can call kimchi to prove for us either through wasm in the browser, native if we're running this o1js code from nodejs or o1cloud if in the cloud.

!!!

### Language-driven design

* Coming soon to an RFC template near you

!!!

### Writing Software Things

* ~~Wearing hats~~
* ~~Language-driven design~~
* <!-- .element: class="fragment" data-fragment-index="1" --> Brandon actions <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### Brandon Actions

<img src="img/buttons1.png" width="55%" height="55%"></img>

Note: We'll continue experimenting on different teams, RFCing things and asking the right questions in the RFC template, and automating the shit out of everything possible. Over the next few months, I intend on making time to personally push this. That means lots of collaboration with Aneesha + working with teams, iterating on the RFCs, and helping build some automations wearing my platform engineering team hat!

!!!

### Brandon Actions

<img src="img/buttons2.png" width="55%" height="55%"></img>

Note: I want to help guide us to making software better. I know a lot of you have ideas of what you want to change. Please send me ideas. And the best way is to talk to me. And the best way to talk to me, is to talk to me this week or next week in person (but we can also chat remote if we don't find time). What can make your time building more "fun". Tell me!

!!!

### In this talk

* Intro myself
* <!-- .element: class="fragment" data-fragment-index="1" --> Background on the CTO role <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> Reflections on what we've built <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="3" --> Engineering values: Review and Instantiate <!-- .element: class="fragment" data-fragment-index="3" -->

Note: via two examples: wearing hats, language driven design, and through my intention to help guide us

!!!

<!-- .slide: data-background="#071448" -->
<!-- .slide: data-state="terminal" -->
# Thanks!

By <a href="http://bkase.com">Brandon Kase</a>  <a href="http://o1labs.org">CTO @ O(1) Labs</a> / <a href="http://twitter.com/bkase_">@bkase_</a>

O(1) Labs: [https://o1labs.org](https://o1labs.org)

!!!

## Appendix

!!!

### Nomenclature

Domain-specific-language

Note: In the spirit of language, let's use language to define how we use languages. You'd think this is overkill, but it's not!

!!!

### DSL Kind

* External DSL (no) -- a language with its own syntax/semantics/runtime
* <!-- .element: class="fragment" data-fragment-index="1" --> Embedded DSL (yes) -- embedded in a host languages, leverage the syntax and features of the host language <!-- .element: class="fragment" data-fragment-index="1" -->

!!!

### External DSL

```html
<body>
    <div>
        <button>a button</button>
    </div>
</body>
```

Example: HTML is an external DSL for describing the content of web pages. It is a new language. Don't make this!

!!!

### Embedded DSL

```js
$('div.test')
  .on('click', handleTestClick)
  .addClass('foo');
```

Example: jQuery is an embedded DSL for manipulating web UIs (inside of the host JavaScript)

!!!

### DSL Encoding

* Initial encoding (rare) -- sentences in our language are represented by data and later interpreted
* Final encoding (common) -- sentences in our lanugage run directly

Note: Rare vs common for using these tools for everyday programming in our work.

!!!

### Initial Encoding Example

```ocaml
let%bind previous_state_hash =
  match constraint_constants.fork with
  | Some { previous_state_hash = fork_prev; _ } ->
      State_hash.if_ is_base_case
        ~then_:(State_hash.var_of_t fork_prev)
        ~else_:t.previous_state_hash
  | None ->
      Checked.return t.previous_state_hash
in (*...*)
```

Note: Snarky circuits are encoded initially

!!!

### Initial Encoding Example (pt2)

TODO

!!!

### Final Encoding Example

```typescript
...
```

Note: 

