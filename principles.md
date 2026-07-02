# Coding principles
- Prefer simple code over easy code. Easy code is fast to write; simple code is untangled. Often the simpler design takes more upfront thought, friction, or lines. Minimize complexity, not line count.
- Optimize for local reasoning. A future engineer should be able to understand the behavior by reading the domain boundary, type signature, and tests, not by chasing mutable state across the repo.
- Model domains by the concept they represent, not around generic buckets like `types` or `utils`.
- Prefer functional cores with imperative shells. Put business logic in pure, deterministic transformations when practical. Keep time, storage, network, mutation, and other side effects at the edges.
- Use records, smart constructors, and explicit transitions. Parse raw inputs into valid domain values, then transform those values with functions that return new state. Avoid hidden mutable state as a default.
- Let types carry intent. Use strong type boundaries to make invalid states hard or impossible to represent, so the compiler remembers rules engineers would otherwise have to keep in their heads.
- Test behavior, not implementation. Unit tests should encode intent: arrange inputs, run the behavior, assert the result. Focus on boundaries, invariants, and meaningful state, not internal steps.
- Mock only at true edges. Prefer real modules, fakes, fixtures, and runnable systems over mocks of your own code. If the test doubles your implementation, it can preserve the same misunderstanding.
- Build backpressure into the system. Use compilers, linters, formatters, and tests to guide code quality so engineers can focus on domain logic and tradeoffs.

# Review principles
Review by first inferring the author intent. Does the PR achieve what the author intends, or may it behave in a way they're not expecting or desiring, or perhaps haven't considered? Only list blocking issues.
