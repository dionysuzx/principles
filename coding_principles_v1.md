# Coding principles v1

These are stylistic preferences/opinions that dionysuzx has adopted. It should be noted these are aspirational: programming in the wild is never perfect, or without violation. Don't let rules get in the way of shipping.

Additionally, it is recognized that a tool is picked for a job. Often times technology is great to get you up and running on day 1, month 1... but fall apart at year 1, year 2. These principles are biased towards maintainable and robust systems. Often times that means some upfront friction for long term gains. A quote from Richard Feldman: "Functional programming makes smaller problems hard, and big problems easy".

## Personas
This is a WIP section to distill coding principles into bounded personas. Here they are:
- "Code reviewer" (includes security)
- "Staff engineer" (includes backend + frontend)
- "Questioner" (for creating plans)

## Simple, complex, easy, hard
> ref: https://www.youtube.com/watch?v=SxdOUGdseq4

Code should be simple, sometimes that's hard. Easy can often make code complex. Sometimes more code can be more simple than less code which is more complex, so minimalism does not mean simplicity.

## Small pure functions
- If possible / straightforward, default to pure functions, that do pure data transformation, and unit test it ([functional core, imperative shell](https://testing.googleblog.com/2025/10/simplify-your-code-functional-core.html))
- To write a good unit test DON'T test implementation details, test behavior/state (decouple the unit test from the implementation so you can fearlessly refactor)
- Good unit tests treat your code like a black box, bad unit tests crack the box open and micromanage the internals
- Unit tests should 1) arrange input 2) call transform function 3) assert final state. If your unit test is doing more, your function/test is probably too complex.
- Test the boundaries in a unit test, not just the happy path (empty arrays, negative balances, etc)--hitting boundaries and the midpoint hits most of the entropy, you can also fuzz between the boundaries.
- Pure functions are easier to cache than side-effecty code. Functional style makes small problems hard but big problems easy.

## Composing cohesive domains
When all your code is written as pure functions with data transforms, it can become super decoupled, but low cohesion. A highly cohesive domain exists to **reduce cognitive entropy**. Cohesion is where we create the abstraction boundary of what the system is and our understanding of it. 

To avoid overloading terms, we establish a strict vocabulary to separate our boundaries from our data:
- **The Domain (The Concept):** The real-world system or business logic we are modeling (eg. the Wallet). This is the conceptual boundary that dictates how we group our code.
- **The Module (The File/Namespace):** The organizational boundary. Modules must never encapsulate hidden mutable state (eg. a global file-level variable or cache).
- **The Type (The Contract):** The compiler-enforced definition of our values (eg. primitives or unions like `'Active' | 'Locked'`). It exists to let the compiler do the heavy lifting of validation.
- **The Record (The Data Entity):** A specific composite Type representing the shape of our domain entity. We avoid "Object" because objects bundle state with time. Records never have private stateful fields (eg. OOP) because it binds behavior to time (and you need to trace the behavior which grows human and agent context window).

A cohesive functional domain should have:
- **1. Shape (The Record):** eg. `type WalletRecord = { address: string, balance: number, status: 'Active' | 'Locked' }`
- **2. Smart constructor:** this verifies the data inside the Record is correct when constructed; eg. `createWallet(address, initialBalance) -> WalletRecord | Error`, this means we can validate how our data should look.
- **3. Transitions:** you never call a method on the Record that changes internal state (because there is no internal mutable state), eg. `wallet.lock()`, instead do like `lockWallet(wallet) -> lockedWallet: WalletRecord` which returns a new Record representing the next state. Let the compiler do the work here.

Three rules for functional cohesion:
- **1. Vertical slicing.** Don't group files by what they are (data, types, etc.), group them by what they **mean**. Bad is `/types`. Good is a folder like `/domain/wallet` which contains the Record type, the wallet pure functions, and the wallet unit tests. Tests should also live right here next to their implementing code.
- **2. Opaque type barrier.** This prevents coupling to the internal details of the properties of the Record. By only relying on the smart constructor and transition functions, we can fearlessly refactor with validation pushed to the edges.
- **3. Decouple from time and storage.** Push time-bound side-effects and state mutation to the imperative shell. Again, this decreases context window for humans and agents.

*(Note on observable purity: While we favor immutability at these boundaries, don't be dogmatic. It is fine to use local mutation inside a transition function for performance, so long as it doesn't leak. The goal is predictable inputs and outputs without side-effects.)*

## The compiler as a boundary (Strong type checking)
Strong type checking structurally eliminates entire categories of state-mismatch and boundary bugs. It is the mechanism that enforces our abstraction boundaries at compile-time, drastically reducing the cognitive context window required to reason about the code.

- **The ultimate opaque type barrier:** Without strong types, a module's boundary is an honor system. Strong types hardcode that barrier into the environment. If the imperative shell tries to pass an ill-formed Record into the pure core, the compiler physically rejects it before the code can even run.
- **Shrink the context window:** In dynamic languages, seeing `processTransaction(wallet, tx)` forces you to read the implementation to understand what those inputs should be. With strong types, the signature `processTransaction(wallet: ActiveWallet, tx: SignedTransaction) -> TransactionResult` acts as a mathematical contract. The signature tells the whole story without expanding your context window.
- **Make illegal states unrepresentable:** By encoding business logic into the types themselves, you prevent bugs from ever existing. If you separate your Records into `LockedWallet` and `ActiveWallet`, the compiler makes it literally impossible to compile code that tries to pass a locked wallet into a function that requires an active one.
- **The Trade-off:** Strong typing introduces upfront friction; it is not strictly faster for writing throwaway scripts on day one. But it is strictly better for reading, maintaining, and **fearlessly refactoring** on day 100. Let the compiler memorize the shape of your domains so you don't have to hold it all in your head.

## Fun examples
Goku in OOP vs Functional.

in this example goku is mutable, so whenever you get this `goku` in your program you don't know what state it is in.
```
var goku = new Goku('brown hair', 'hair down', 'not super saiyan')
goku.makeHairYellow()
goku.hairUp()
goku.superSaiyan(2)
```
functional:
```
goku = GokuFactory.initialize() -> Goku
yellowHairedGoku = makeHairYellow(goku)
yellowHairedAndHairUpGoku = hairUp(yellowHairedGoku)
superSaiyanGoku = superSaiyan(yellowHairedAndHairUpGoku)
```
in the functional approach because your goku is immutable and shows all its traits up front, you know exactly the behavior without tracing outside the context of the code you're looking at.

## Mocking / tests
- Never mock your own code. Only mock at the edges (eg. an external API); but in self-sovereigns apps, prefer to not have edges and always spin up real tests (no mocking, just fakes so you're running on real systems (like [bun](https://x.com/jarredsumner/status/2035450754291675582?s=20))). This is especially true for AI systems because if they hallucinate the implementation, they'll also hallucinate the mock.
- Generally use the test pyramid. Lots of unit, lots of testing composed modules; when testing integration boundaries keep a medium level, and only 1-2 sanity checks e2e. Why: same reason as perhaps an air filter (fine mesh, medium mesh, and large particulate filter). The big filter catches the big bugs (e2e).

## Language selection
Bias for languages with strong static type checking, preference to immutability, lack implementation inheritance, and most importantly: solve problems for our users. The reason is because humans and agents can examine a smaller section of context to understanding parts of the program behavior, this allows for more reliable code for our end users.

Good practical examples include Rust and Typescript (spiritually, Roc). These have massive ecosystems. For languages, pick things that are battle tested, be careful to select the "bleeding-bleeding edge". The goal is maintainable code that allows us to deliver consistent predictable valuable for end users, not fancy benchmarks or language features (sorry Roc, ily).

## Engineering values

1. Perfect is the enemy of the shipped. A PR doesn't need to be perfect.
2. 80/20: Prioritize work that has outsized impact. 80% of the outcomes come from 20% of the inputs.
3. Customer/value obsession: Never code or change code for the sake of code. Changes should be focused on driving value to end users or maintainers so they can help drive more value to end users ultimately.
4. Bias for action.
5. "Boy scout principle": leave the code in a better place than you found it. Resist large scale refactors and adopt incremental improvements.
6. No false idols. The way things have been done historically should never be the primary reason for doing something. Select the best solution for users. Stay curious. Stay laser focused on customer value.
7. Over communicate. Work in public. Express your messy thoughts. Share learnings. Get feedback. Engage in pair programming.
8. Iterate. Iteration means we hit the deadline but are flexible about the scope. Its okay and encouraged to have multiple iterations for a feature.
9. Don't overreact. If you find yourself reacting to change too much it usually means a process can be fixed at its root. Fix root causes and stay focused on the high level mission of driving value to your users.
10. Focus on the data. Data tells a story. Collect data examine it and make informed decisions that are evidence based.
11. Sometimes more is less. You may think because of DRY you should create an abstraction, sometime repeating yourself is simpler, and reduces coupling and maintenance of an abstraction. Minimize complexity, not LOC.

## Correctness: Unit tests, invariants, theorem provers, and dependent types
Why write tests at all? Just write good code, right? In an AI-driven future, shouldn't the LLM just read the code and understand the intent?

No, because **code is not intent**. Code is mechanical execution. 

If you only write the code, the actual *intent* of the system only exists in one place: your head. Because human context windows are incredibly limited, keeping that intent in your head guarantees you will eventually forget it, mess it up, or fail to communicate it to a teammate (or an AI agent). Furthermore, you cannot measure if code is "correct" on its own; you can only measure if it *matches an intent*. If the intent is stuck in your head, you have no yardstick.

Testing is fundamentally **double-entry bookkeeping for logic**. It forces you to get the intent out of your head and encode it into the system. You state your reality in two independent ledgers: the mechanism (the *how*) and the test (the *what*). If both ledgers balance, the probability that your system adheres to your actual intent skyrockets. 

Read Yudkowsky's "Complexity of Value" for more. It's difficult to measure program correctness on one vertical axis because mapping human intent perfectly to code is fundamentally hard. There are a few ways we externalize this intent:

- **Unit tests (Empiricism):** Fundamentally, a pure function is a logical statement. A unit test is simply an empirical sample of that function's truth table. It is inductive reasoning: we test specific inputs and observe the outputs to verify our ledger. It's cheap, fast, and covers the 80/20.
- **Invariants (The Anchor):** These coding preferences point to heavy unit testing, but especially **invariant** tests, because they are extremely dense encodings for system behavior. In a system with high entropy, an invariant is a constant that imposes what *must* or *must not* happen, even if everything else can happen. 
  - *Example:* If the goal is feeding a cat, an invariant test asserts `cat.isFed == true`. It doesn't micromanage the exact, step-by-step way the can is opened or the bowl is placed. Because you aren't testing the *how*, you open the door for **fuzzing**—throwing thousands of random actions at the system and ensuring the core invariant never breaks. You encode what you actually care about.
- **Theorem provers (Formal Verification):** Another option that can be bolted on for mission-critical code (eg. an EVM state transition func, fork choice rule) is to use a theorem prover like Lean4. It encodes the mathematical behavior of the program to generate deductive proofs. The reason this isn't preferred for most code is that formalizing intent flawlessly into math is prohibitively challenging. It is always better as a bolt-on to well-reasoned code.

In most cases, the most context-efficient way to ship robust software is to create small functional modules, favor immutability, and use heavy invariants / unit testing and strong type checking. Reach for formal verification tools when you need them as bolt-ons to maximize P(adheres to intent). Continue programming in English common languages, but remember that raw streams of English (or code) are too unstructured to perfectly hold your mental model: guide the structure by setting up boundaries, and externalize your intent into tests.

> ref: vitalik puts it nicely here: https://www.reddit.com/r/ethereum/comments/7bdm1g/so_can_we_again_have_a_talk_about_formal/

## Backpressure
Utilize lots of backpressure. Unit tests, linting, formatting, compiler settings, even the nitty gritty stuff; theres a lot of stuff you can do with these languages to even enforce program organization (eg. domains). Utilize your backpressure tools to your highest ability, these are structured ways to encode your intent and really helpful for engineers/agents. Enforce backpressure autonomously with precommit hooks.

## LLM reasoning review chain
When reviewing code in a loop, stage tour gates in low, medium, high reasoning (again think of a 3 stage air filter with big filter, medium filter, and fine mesh filter). Low reasoning catches big obvious bugs on the cheap. Refine that with medium reasoning, with a final pass of high reasoning. This is for token efficiency, you want to early return out of reasoning depth before spending more compute.

## Codex vs claude
This is the new vim vs emacs, tabs vs spaces, rust vs golang. Most languages support you wrestling it into your style of coding, pick tools that suit you as a driver. Its the driver that wins the race not the car. Learn your car well and it will take you far. Pick languages and LLM frameworks that suit your style of programming. The LLMs are all super smart, the performance is in the driver/harness. Theyre also not mutually exclusive. Experiment, have fun.

## Contextual decision making
Its important to make contextually aware decisions. You dont need sharded databases etc for something that can fit in memory on your laptop. Dont over-engineer. At the same time have the foresight to predict actual problems that may occur while solving the problem at hand.

Recognize product lifecycle, and use the appropriate tool/principles for the job at hand (eg. sometimes a bash script > a rust program; depends on the feature, what you're trying to do, etc).

## Code review principles
Building a cohesive codebase isn't just about how you write code; it's about how you review it. Human context windows are expensive, and PR reviews are where they are most heavily taxed. 

- **The standard of review (Accept if it's an improvement):** The primary purpose of code review is to ensure the overall health of the repository is improving. Approve a PR if it leaves the codebase in a better state than it was, **even if it isn't perfect**. Do not let perfect be the enemy of the shipped. Minor, non-blocking issues can be approved with comments for future cleanup.
- **Review the boundaries, not just the logic:** Bugs rarely hide in the middle of a pure function; they hide where state changes and data crosses boundaries. When reviewing, look directly at the Imperative Shell and the Type Signatures first. Is the state mutation contained? Is the type barrier opaque? If the boundaries are secure, the internal functional logic is much less risky.
- **Automate the nits:** As mentioned in the **Backpressure** section, human cycles should never be spent arguing about syntax, formatting, or imports. If it can be caught by a linter or compiler, enforce it autonomously in CI. Reserve human review strictly for domain logic and state management.
- **Prefix comments to signal intent:** Over-communicate and reduce ambiguity. Prefix your PR comments so the author knows exactly how to react without guessing your tone or the severity. We prefer semantic prefixes over priority tags (like P0, P1, P2) because "P0" dictates subjective urgency, while semantic prefixes clarify *actionability*.
  - `Issue:` A bug or architectural flaw that must be fixed before merging.
  - `Suggestion:` A better way to do it, but the author can decline and still merge.
  - `Nit:` A minor triviality (like a variable name).
  - `Question:` I don't understand this (which usually means the code needs a clearer name or a comment).

## Enjoy
Computer programming is the expression of vibes wishes and dreams into code. Have fun. Take a walk to clear your head when a problem is tough and youll usually find a solution. Focus on whats important to you.

## Sources & Further Reading
These principles are heavily inspired by legendary engineering guides that prioritize reading over writing, and system boundaries over strict paradigms.

- **[Google Engineering Practices (Code Review Guide)](https://google.github.io/eng-practices/):** The industry gold standard for how to review code quickly, effectively, and with empathy. Their core ethos is to optimize code for the reader, not the writer.
- **"A Philosophy of Software Design" by John Ousterhout:** The pragmatic counter-weight to traditional "Clean Code." It introduces the concept of **Deep Modules**, which perfectly maps to our "Opaque Type Barrier." A deep module is one that provides a very simple, narrow interface (a clean type signature) but hides a deep, complex implementation inside. The caller only needs a tiny context window to wield massive functionality. 
- **[Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) by Alexis King:** The philosophical backing for our "Smart Constructor" rule. Don't just check if data is valid and leave it as a raw string; *parse* it into a strict Type so the compiler carries that proof forward, eliminating the need for redundant checks deeper in the core.
- **[Functional Core, Imperative Shell](https://testing.googleblog.com/2025/10/simplify-your-code-functional-core.html) (Google Testing Blog):** The architectural pattern for isolating pure domain logic from messy side-effects and state mutations.
- https://lobste.rs/s/pef25i/language_for_agents
- Domain modeling made functional: https://www.youtube.com/watch?v=2JB1_e5wZmU

