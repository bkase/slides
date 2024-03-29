- Title: ZK DSL Usability Challenges to Bring Web2 Devs to ZK/Web3
- Intro slide; we build o1js (formerly SnarkyJS)
  - A lot of my opinions about tradeoff spaces I'll discuss is reflected in the design of o1js so if you want to consume this in another form, go download o1js and play with it!
- Highest level Problem — we want to build tools for working with private applications, and zkSNARKs is one great conduit for such technology
  - Assumption: This audience is fairly familiar with the concept of zk proofs and that which goes into it, but I'll really quickly review
- zkSNARK stands for ...
- My mental model is: There exists data, such that predicates
- At runtime, (typically), this boils down to constraints in some kind of constraint system (whether it be R1CS or a PLONKish constraint system)
  - So something at some point needs to describe these constraints to represent some computation
  - This can be direct, or close to direct (like building your application directly on arkworks), slightly higher level like Circom, introducing more tools for abstraction like o1js or Noir, or extremely indirect (like describing a zkVM, for example EVM, Cairo, RiscZero)
- Tension: To what extent do you abstract away the lowest level system?
  - You can view this from two vantage points; there is a fundamental choice that a DSL induces AND individual choices of abstractions within the language
    - But this leads us to an open challenge: What's a clean way to integrate zkVMs and circuit DSLs
- To embed or not to embed
  - Custom language vs embedded DSL:
    - It's fun to build new programming languages. I PERSONALLY enjoy learning new programming languages. BECAUSE I am a have a programming language theory background
    - Application developers IN GENERAL don't want to learn new languages, especially if they're trying to build and ship a product, and they sure as hell don't want to be stuck with shit tooling
    - Why go through the hell of not just building a new language, but building a
    - Okay you can go ALL THE way, and then you end up with something that looks RiscZero-like (great job ya'll!)
  - Embed it in TypeScript to get web devs
- Challenge: How can we make the system more approachable within the DSL?
  - Case study: Action/reducer vs React
  - Include the full stack in the same tool
    - You can use o1js directly for arbitrary public/private inputs, or the fixed public input that Mina's network expects.
    - Support web execution from day1 + make it first-class
  - More magical things are easier for folks to get something working quicker, but then harder when you get more advanced to understand what's happening. This is an active tension that we explore
    - Right now, our Field abstraction merges the concept of constants and constraint variables, and it's supposed to be simple, but it's confusing for people who want to understand if it's happening. We're not sure yet, but we're thinking about separating them again?

- This encompasses zk circuits, but also a layer on top for managing top-level circuits with fixed public inputs
- Some framework in the middle
  - Arkworks figures out when it has to add constraints, + is lazy, if you do more multiplications
  - Circom attempts to create a DSL; limited form of witness computation
    - Circom you have to hardcode character values
  - Maybe use cairo,
  - o1js
    - General purpose programming language for witness computation — embedded DSL a better idea. All in the same language
- xx
- Strong opinion alert (backed by anecdotal and objective data — uncited)
  - It's fun to build new programming languages. I PERSONALLY enjoy learning new programming languages. BECAUSE I am a have a programming language theory background
  - Application developers IN GENERAL don't want to learn new languages, especialy if they're trying to build and ship a product, and they sure as hell don't want to be stuck with shit toolin
  - Why go through the hell of not just building a new language, but building a
  - Okay you can go ALL THE way, and then you end up with something that looks RiscZero-like (great job ya'll!)
  -
- x
-
- Case studies?
  - Actions/reducer taken from React
- Open problems
  - There is a tension between low level access and native-ness of the underlying language
  - How to incorporate a VM-like model; it's powerful — but then it steps out of the language
  - Clean way to do if-statements and branching
    - You could write custom babel transforms, then you leave the langauge — that's thension. Is there a way to stay in the language, only have library functions, and still offer a normal experience
  - Tooling improvements
    - Show constraints
  - Making things self-explanatory
  - More magical things are easier for folks to get something working quicker, but then harder when you get more advanced to understand what's happening. This is an active tension that we explore
    - Right now, our Field abstraction merges the concept of constants and constraint variables, and it's supposed to be simplke, but it's confusing for people who want to understand if it's happening. We're not sure yet, but we're thinking about separating them again? ...
- Performance, account creation,
