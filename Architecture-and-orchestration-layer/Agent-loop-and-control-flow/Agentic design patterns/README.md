# AI agentic design patterns

This folder contains practice examples and implementations of agentic design patterns, which define the conceptual behaviors and reasoning capabilities of intelligent AI agents. Design patterns focus on how agents think, plan, act, and collaborate rather than just the task execution flow.

Agentic design patterns describe the architectures and methodologies that enable AI agents to make decisions, reason, use external tools, and improve themselves iteratively. They address how agents interact with their environment, utilize resources, and organize behaviors to solve complex problems effectively. These patterns empower the development of intelligent agents that are autonomous, adaptive, and capable of sophisticated cognitive functions such as planning, self-evaluation, and collaboration. They provide reusable blueprints for building advanced agent architectures that leverage capabilities like tool integration and multi-agent teamwork.

### Single-agent patterns
- **Tool use pattern:** The agent extends its reasoning by invoking external tools, APIs, or knowledge sources to enhance problem-solving capabilities beyond internal knowledge.
- **ReAct pattern:** Combines reasoning and acting in a loop, where the agent reasons to decide next actions and executes them, then reflects on outcomes to guide future steps.
- **Planning pattern:** The agent formulates an explicit strategy by breaking down the main task into smaller objectives and determining the sequence and methods to achieve them.
- **Reflection pattern:** The agent reviews and critiques its generated outputs or past decisions, enabling continuous self-improvement and error correction.

### Multi-agent patterns
Multiple specialized agents collaborate, each with their own expertise or role, to collectively address multifaceted tasks that are difficult for any single agent. It involves mechanisms for cooperation, negotiation, or competition between agents to achieve a collective goal or solve a problem that requires multiple perspectives.

#### Collaboration patterns
Agents working together through different interaction models:
- **Sequential pattern:** Agents organized in a pipeline where each agent processes and enhances the output before passing it to the next. Information flows linearly through stages with each agent building upon previous work.
- **Parallel pattern:** Multiple agents work simultaneously on different aspects of the same problem, with their independent outputs aggregated into a comprehensive solution. Enables concurrent processing of complementary perspectives.
- **Debate pattern:** Agents with different perspectives propose solutions, critique each other's work, and refine their proposals through adversarial collaboration. The iterative debate process produces more robust solutions than any single perspective.
- **Ensemble pattern:** Multiple agents independently solve the same problem using different approaches or configurations, with their diverse outputs combined through aggregation or voting. Reduces variance and improves reliability through wisdom of crowds.

#### Hierarchical patterns
Organizational structures where managers delegate to and coordinate workers:
- **Hierarchical (single-tier):** One supervisor agent directly manages multiple specialized worker agents. The supervisor delegates tasks, workers execute independently, and the supervisor aggregates results.
- **Hierarchical multi-tier:** Nested hierarchies with multiple management levels. Top managers coordinate middle managers, who each manage their own teams of workers.
- **Hierarchical dynamic teams:** A smart supervisor analyzes incoming tasks and dynamically selects the optimal team composition from a worker pool. Different tasks result in different team formations based on required expertise.