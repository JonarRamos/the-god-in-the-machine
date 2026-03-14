# Principle 4: Design for Outcomes, Not Outputs

Define the destination, not the route. Let the agent orchestrate the path.

There is a natural tendency, when designing agentic systems, to think in terms of tasks. The agent should generate a report. The agent should move data from System A to System B. The agent should classify incoming tickets and route them. These are outputs - specific, predefined artifacts or actions that the agent is instructed to produce.

Outputs are comfortable. They are testable, predictable, and easy to scope. But they constrain the agent to executing a predetermined solution path - which limits the value of having an intelligent orchestrator in the first place.

The alternative is to design for outcomes. An outcome is not a specific artifact - it is a desired state. The user should have an accurate, evidence-based understanding of an employee's performance. Two systems should be in sync. Incoming requests should reach the person best equipped to handle them. The difference is not semantic. It is architectural.

When you design for an output, you are encoding the solution path into the system. The agent becomes an executor of a predetermined workflow - handling variability within a fixed process, but not determining the process itself.

When you design for an outcome, the agent determines the path. It assesses what data is available, what skills it can call, what the current state of the world is, and orchestrates a solution dynamically. The same outcome might be achieved through different paths depending on what data is available, what systems are accessible, and what has changed since the last execution. The agent is not following instructions. It is pursuing a goal.

Why This Matters Architecturally

Outcome-oriented design has several structural advantages that compound across a Theotropic Architecture implementation:

Resilience to change. An output-oriented agent that is instructed to "pull data from Table X, join it with Table Y, and generate a CSV" breaks when Table X is renamed, when Table Y's schema changes, or when the CSV format is no longer what the consumer needs. An outcome-oriented agent that is told "ensure the finance team has an accurate view of Q3 expenditures" can adapt - querying different sources, using different joins, producing a different format - because its success is measured against the outcome, not against a specific execution path.

Reusability across contexts. An output-oriented agent is tightly coupled to a specific workflow. An outcome-oriented agent, combined with composable skills (Principle 6) and the Agentic Interface Layer (Principle 3), can pursue similar outcomes across different domains without being redesigned. "Keep these two systems in sync" is an outcome that applies to CRM-to-ERP sync, HR-to-payroll sync, inventory-to-fulfillment sync - each with different data, different systems, different edge cases, but the same structural goal. The agent needs to know the goal and where to look. It does not need a new workflow for each instance. This also aligns naturally with composable recursion (Principle 6): when an agent pursues an outcome rather than follows a script, it engages in the kind of non-linear, iterative orchestration that composable skills enable.

Better utilization of model intelligence. An LLM's core capability is reasoning - synthesizing information, identifying patterns, making judgments under ambiguity. Output-oriented design uses this capability to execute predetermined steps, which is a limited application of what the model can do. Outcome-oriented design puts the model's reasoning where it has the most leverage: deciding what to do, not just doing what it's told.

The Guardrail Requirement

Outcome-oriented design introduces a legitimate concern: if the agent determines its own path, how do you ensure it doesn't take a dangerous or unauthorized path?

This concern is valid, and it is addressed not by abandoning outcome-oriented design, but by combining it with other Theotropic Architecture principles:

Deterministic actions (Principle 5) ensure that whatever path the agent chooses, the individual steps it triggers are predictable and testable. The agent's reasoning about what to do is non-deterministic. The execution is deterministic. This separation is what makes outcome-oriented design safe - the agent has freedom in routing but not in execution.

Permission governance (Principle 3, AIL) ensures that the agent can only access data and invoke skills that it is authorized to use. The agent may determine its own path, but the path is bounded by its permission scope. It cannot accidentally - or deliberately - reach data or trigger actions outside its governed domain.

Transparent feedback loops (Principle 9) ensure that every decision the agent makes is logged and observable. If the agent takes an unexpected path to an outcome, that path is visible, auditable, and available for review. The organization can see why the agent did what it did, and can provide feedback that improves future routing.

Human interaction gates (Principle 9) provide approval checkpoints for critical actions. The agent proposes a path. A human approves or modifies it. This is particularly important in early maturity stages, where trust between the organization and the agentic system is still being established.

The principle is not "let the agent do whatever it wants." The principle is "define the goal and the guardrails, and let the agent handle the routing within those boundaries."

Tradeoffs and Limitations

Outcome-oriented design is harder to test than output-oriented design. When the success criterion is a specific output, testing is straightforward - did the agent produce the expected artifact? When the success criterion is an outcome, testing requires evaluating whether the outcome was achieved, which may involve subjective judgment, temporal assessment, or multi-system verification. Testing frameworks for outcome-oriented agents are less mature than those for output-oriented ones.

There is also a communication challenge. Stakeholders and product owners are accustomed to specifying requirements as outputs. "The system should generate a weekly reconciliation report" is a clear, scoped requirement. "The system should ensure that finance and operations data remain consistent" is a broader, fuzzier requirement that is harder to estimate, harder to sign off on, and harder to declare "done." Adopting outcome-oriented design requires a corresponding shift in how requirements are expressed and how success is measured.

Not every task benefits from outcome-oriented design. Some tasks genuinely are simple, well-defined output generation - formatting a document, sending a notification, executing a data transformation. For these, outcome-oriented design adds unnecessary abstraction. The principle applies most powerfully to tasks that involve reasoning across multiple data sources, tasks where the optimal path may vary, and tasks where the goal is a state rather than an artifact. The judgment is: if the task requires intelligence to determine the approach, design for the outcome.

If the task is mechanical, design for the output.
