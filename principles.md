# Coding principles
- Prefer simple code over easy code. Easy code is fast to write; simple code is untangled, local to understand, and durable. Sometimes the simpler design takes more thought, more upfront friction, or more lines. Minimize complexity, not line count.
- Optimize for small context windows. A future engineer or agent should be able to understand a behavior by reading the domain boundary, type signature, and tests, not by chasing mutable state across the repo.
- Model domains by meaning, not by file category. Group records, constructors, transitions, and tests around the concept they represent, not around generic buckets like `types` or `utils`.
- Prefer functional cores with imperative shells. Put business logic in pure, deterministic transformations when practical. Keep time, storage, network, mutation, and other side effects at the edges.
- Use records, smart constructors, and explicit transitions. Parse raw inputs into valid domain values, then transform those values with functions that return new state. Avoid hidden mutable state as a default.
- Let types carry intent. Use strong type boundaries to make invalid states hard or impossible to represent, so the compiler remembers rules humans and agents would otherwise have to keep in their heads.
- Test behavior, not implementation. Unit tests should encode intent: arrange inputs, run the behavior, assert the result. Focus on boundaries, invariants, and meaningful state, not internal steps.
- Mock only at true edges. Prefer real modules, fakes, fixtures, and runnable systems over mocks of your own code. If the test doubles your implementation, it can preserve the same misunderstanding.
- Build backpressure into the system. Use compilers, linters, formatters, type checks, tests, CI, and pre-commit hooks to automate correctness and style so humans can focus on domain logic and tradeoffs.
- Ship valuable improvements iteratively. Choose tools for the lifecycle and problem at hand, prioritize user and maintainer value, review boundaries over nits, communicate clearly, and leave the codebase better without waiting for perfection.

# Review principles
Review by first inferring the author intent. Does the PR achieve what the author intends, or may it behave in a way they're not expecting or desiring, or perhaps haven't considered? Only list blocking issues.
