# Principle 6: Composable Recursion

Workflows are not pipelines. Skills and sub-agents should be callable by any agent, in any order, at any point.

The default model for workflow automation - and by inheritance, for most early agentic implementations - is the pipeline. Step 1 feeds Step 2 feeds Step 3 feeds Step 4. The sequence is predetermined. The flow is linear. If Step 3 needs information from Step 1, it was passed forward. If Step 4 reveals that Step 2 was wrong, the pipeline either fails or restarts from the beginning.

Pipelines are comprehensible, debuggable, and fundamentally inadequate for intelligent orchestration.

An agent pursuing an outcome (Principle 4) does not think in sequences. It thinks in goals. It retrieves data, evaluates what it has, realizes it needs more, retrieves again from a different source, calls a skill, evaluates the result, decides the result is insufficient, calls a different skill, revisits an earlier retrieval with new parameters informed by what it has learned, and iterates until the outcome is achieved. The execution path is not 1 → 2 → 3 → 4. It is 1, which calls 2, 3, 4, then 2 again, then 4 with different inputs, then 1 again to verify.

This is composable recursion. The "composable" means that each skill and sub-agent is self-contained - it has defined inputs, defined outputs, and no dependency on being called at a specific point in a specific workflow. The "recursion" means that the orchestrating agent can call any skill any number of times, in any order, including calling skills it has already called, until the outcome is satisfied.

The SOA Lineage

This principle has the most direct lineage to Service Oriented Architecture. SOA's foundational insight was that services should be loosely coupled, reusable, and composable across multiple consuming applications. A well-designed service did not assume it was "Step 3 of Workflow X." It exposed a capability - retrieve customer data, validate an address, calculate a price - and any consumer could call it for any purpose.

Theotropic Architecture applies the same insight to agentic skills, with one critical extension: the orchestration is non-linear and adaptive. In SOA, the orchestration layer - BPEL, ESBs, workflow engines - was itself predetermined. A human designed the flow, and the engine executed it. In Theotropic Architecture, the orchestrating agent determines the flow at runtime based on the goal, the available data, and the results of previous steps. The services are SOA. The orchestration is something new.

This extension is what makes composable recursion powerful. A predetermined orchestration can only handle scenarios that the designer anticipated. An intelligent orchestration can adapt to scenarios that were never explicitly designed for - as long as the skills it has access to are composable enough to be called in unanticipated combinations.

Design Requirements for Composable Skills

For composable recursion to work, skills must be designed with specific properties:

Statelessness. A skill should not depend on being called in a specific sequence or after a specific prior skill. It receives its inputs, performs its function, and returns its outputs. It does not maintain state between invocations. If state is needed across a multi-step orchestration, it belongs in the memory layer of the AIL (Principle 3), not in the skill itself.

Self-describing contracts. A skill should clearly declare what it needs (inputs), what it produces

(outputs), and what it does (function). This contract is what the orchestrating agent uses to decide whether and when to call the skill. A skill with an ambiguous or undocumented contract cannot be reliably composed.

Granular scope. A skill should do one thing - consistent with the Unix philosophy and Principle 5's emphasis on deterministic, testable actions. A skill that "retrieves customer data, validates it, transforms it, and writes it to the target system" is four skills compressed into one. The compression makes it harder to reuse the individual capabilities and harder for the orchestrating agent to compose them in novel ways.

Structured failure responses. When a skill fails, it must return enough information for the orchestrating agent to adapt. This connects directly to Principle 8 (Self-Healing) and is described in full there.

The Skill Registry

At scale, composable recursion requires a mechanism for the orchestrating agent to know what skills are available. This is the skill registry - a catalog of available capabilities, their contracts, their requirements, and their domains. It is the action-layer equivalent of the data definition store in the AIL.

The skill registry enables several important capabilities:

Discovery. An agent pursuing an outcome can query the registry to identify which skills might be relevant, rather than having every possible skill hardcoded into its configuration. This is particularly important as the skill library grows across maturity stages.

Intent-based routing. Combined with Principle 7 (Hash, Cache, and Match on Intent), the registry enables the orchestrating agent to match a semantic intent to an available skill without exact keyword matching. "I need to notify the finance team" can route to the appropriate notification skill regardless of how the skill is named.

Governance. The registry is where skill-level permissions are enforced. Not every agent should be able to call every skill. The registry, in conjunction with the permission store (Principle 3), determines which agents can invoke which capabilities.

Safeguards

Composable recursion without governance is dangerous. An agent that can call anything in any order any number of times can enter infinite loops, trigger cascading failures, and accumulate costs without bound. OpenClaw's heartbeat scheduler - which wakes the agent at configurable intervals to act without any human prompting - is an example of ungoverned recursive capability: the agent can initiate actions indefinitely, with no depth limits, no cost ceilings, and no cycle detection. The following safeguards are structural requirements, not optional additions:

Depth limits. A maximum recursion depth prevents infinite loops. If the agent has not achieved its outcome within N recursive calls, it should escalate or fail gracefully rather than continuing indefinitely.

Cost ceilings. Each recursive call consumes tokens, API calls, and potentially external service costs. A per-execution cost ceiling ensures that a runaway orchestration is terminated before it becomes expensive.

Cycle detection. If the agent is calling the same skills with the same inputs repeatedly without making progress, the system should detect the cycle and intervene - either by escalating, by forcing a different approach, or by terminating with a structured failure report.

Timeout policies. A maximum wall-clock time per orchestration ensures that long-running recursive loops do not consume resources indefinitely.

Circuit breakers. If a specific skill is failing repeatedly across multiple orchestrations, the system should temporarily disable it and route around it, rather than allowing every orchestration to fail against the same broken skill.

These safeguards do not diminish the power of composable recursion. They make it production-safe. The God Machine is governed. Its recursion must be governed as well.

Tradeoffs and Limitations

Composable recursion is harder to debug than linear workflows. When something goes wrong in a pipeline, the failure point is usually obvious - Step N failed, examine Step N. When something goes wrong in a recursive orchestration, the failure may be a consequence of a decision made three recursive calls ago, informed by data retrieved five calls ago, compounded by a partial failure that was self-healed two calls ago. The execution path is non-linear by design, which makes post-hoc analysis more complex.

Transparent Feedback Loops (Principle 9) mitigate this - comprehensive logging of every decision and every skill invocation creates an audit trail that can be replayed. But the audit trail for a recursive orchestration is inherently more complex than for a linear workflow. Tooling for visualizing and debugging recursive agentic flows is still maturing.

There is also a design tension between composability and efficiency. The most composable design - many small, atomic skills - maximizes reuse but increases the number of orchestration steps, which increases token consumption and latency. The most efficient design - fewer, larger skills - reduces orchestration overhead but limits composability. The right balance depends on the specific use case and the maturity of the architecture. Early stages may favor fewer, larger skills for pragmatism. Later stages, as the skill registry grows and reuse becomes more valuable, may favor decomposition.
