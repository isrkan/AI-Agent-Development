# Context Engineering

Context engineering is the practice of designing, structuring, and managing the information that flows into and out of AI agents during their operation. As agents become more sophisticated and handle increasingly complex tasks, the quality and organization of their context directly determines their effectiveness, reliability and ability to maintain coherent behavior across extended interactions. Context engineering encompasses everything from initial system prompts and memory management to tool integration and failure recovery, forming the foundation upon which production-level AI agents are built.

## Implementation guide
Follow these steps to develop effective context for our AI agent system:

### Part 1: Context components
Context components are the fundamental elements that make up an AI agent's operational context. Each component serves a specific purpose and contributes to the agent's ability to understand, reason, and respond effectively.

#### Component 1: System prompts and agent personas
The system prompt serves as the foundational context that shapes every interaction our agent has. This is where we establish the agent's identity, capabilities, behavioral guidelines, and operational boundaries. A well-crafted system prompt provides the agent with a clear sense of purpose and helps it maintain consistent behavior across diverse scenarios.

**What to define**:
* **Agent identity and role**: Clearly articulate what the agent is, what it does, and what domain expertise it possesses. For a customer support agent, this might include the company's values, product knowledge scope, and service philosophy. For a research assistant, it would encompass research methodologies, citation standards, and analytical approaches.
* **Behavioral guidelines and tone**: Specify how the agent should communicate—formal or casual, concise or detailed, proactive or reactive. Include guidelines for handling uncertainty, admitting knowledge gaps, and escalating issues beyond its capabilities.
* **Operational constraints and safety rules**: Define what the agent should never do, what information it cannot share, and what actions require human approval. This includes data privacy rules, ethical boundaries, and domain-specific regulations.
* **Response structure preferences**: Indicate whether responses should follow specific formats, include citations, provide step-by-step reasoning, or structure information in particular ways.

#### Component 2: Memory systems
Memory systems enable agents to maintain context across interactions, learn from past experiences, and provide personalized responses. Different types of memory serve distinct purposes in production AI systems.

* **Memory types**:
* **Transactional memory (state management)**: Tracks the current conversation flow, user intent, intermediate results, and execution metadata. In LangGraph, this is implemented through the agent's state dictionary that flows through each node.
* **Short-term memory (session context)**: Maintains context within a single conversation session, typically lasting from minutes to hours. This enables the agent to reference earlier parts of the conversation and maintain topic continuity.
* **Long-term memory (persistent knowledge)**: Stores information that persists across sessions, including user preferences, learned patterns and accumulated domain expertise. Specialized memory types:
    - Episodic memory: Timestamped events and experiences
    - Semantic memory: Facts and knowledge with confidence scores
    - Procedural memory: Workflow templates and successful action sequences
    - Associative memory: Relationships between concepts and inference patterns

#### Component 3: Knowledge base integration
Knowledge bases provide agents with access to domain-specific information, documentation, and factual data that extends beyond their base training. Effective integration involves determining when to retrieve information, how to formulate queries, and how to incorporate results into the agent's context.

**What to implement**:
* **Retrieval triggers**: Define when the agent should query the knowledge base - when it detects knowledge gaps, when users ask specific questions, or proactively to enrich responses.
* **Hybrid search strategies**: Combine semantic search (vector similarity) with keyword search (BM25) to capture both conceptual relevance and exact term matches. Blend results from multiple retrieval methods.
* **Query optimization**: Implement query rewriting to generate multiple variations of the user's question, expanding synonyms and reformulating for better retrieval. For complex questions, use multi-hop reasoning to retrieve and synthesize information from multiple documents.
* **Context integration**: Incorporate retrieved information into the agent's working context with proper source attribution. Format results to be easily parsable by the LLM, including relevance scores and metadata.
* **Knowledge base versioning**: Implement strategies for handling updates, including incremental indexing and cache invalidation to ensure the agent works with current information.

#### Component 4: User input processing
User input processing sits between the user and our agent's core logic, ensuring that incoming requests are clean, well-structured, and safe to process. This component prevents malformed inputs from corrupting context and enables better intent understanding.

**What to implement**:
* **Input validation and sanitization**: Check input format, length, and content. Remove or escape potentially harmful characters, validate encoding, and ensure inputs meet expected schemas using Pydantic models.
* **Intent classification**: Determine what the user is trying to accomplish (question, command, clarification, feedback, greeting). This helps route requests appropriately and set context correctly.
* **Entity extraction**: Identify and extract key entities from user input including dates, names, locations, product IDs, monetary amounts, emails, and phone numbers. This structured information enriches context and enables more precise tool usage.
* **Ambiguity detection**: Identify when user input is ambiguous or incomplete. Detect pronouns without clear referents, incomplete date references, or vague quantities, and generate clarification questions.
* **Context-aware processing**: Use conversation history and user profile to interpret input correctly, resolving pronouns, implicit references, and context-dependent meanings.

**Implementation approach**:
Create a class that handles validation, intent classification, entity extraction, and ambiguity detection. Integrate it as a node in our LangGraph workflow that processes input before the main agent logic.

### Component 5: Tools and tool responses
Tools extend an agent's capabilities beyond text generation, enabling it to query databases, call APIs, perform calculations, and interact with external systems. Effective tool integration requires clear definitions and thoughtful response handling. The system prompt should also guide tool selection.

**What to define**:
* **Tool descriptions**: Provide detailed descriptions of what each tool does, when it should be used, what parameters it accepts, and what it returns. Use Pydantic schemas for type safety.
* **Usage guidelines**: Specify prerequisites, rate limits, and error handling strategies. Document common tool chaining patterns (how to chain tools together for complex workflows).
* **Response formatting**: Transform raw tool outputs into formats optimized for LLM consumption. For large datasets, provide summaries with drill-down options. For errors, provide context and recovery suggestions.
* **Response integration**: Define how tool outputs should be incorporated into the agent's reasoning. Include metadata like confidence scores, timestamps, and data quality indicators.
* **Conditional tool exposure**: Don't expose all tools for every task. Dynamically select which tools the agent can access based on current context, user permissions, and task requirements. This reduces decision complexity, improves security, and decreases context size.

#### Component 6: Constraints and guardrails
Constraints and safety rules ensure our agent operates within acceptable boundaries, protecting users, data, and systems from harm. This component encompasses input validation, output filtering, rate limiting, and feedback loops.

* **What to implement**:
* **Input constraints**: Validate input length, format, and content. Implement rate limiting per user or session. Detect and block malicious patterns including injection attempts.
* **Output constraints**: Filter sensitive information from responses (PII, credentials, internal data). Implement content moderation for inappropriate content. Validate output format and completeness.
* **Operational constraints**: Define resource limits (API calls, computation time, memory usage). Implement circuit breakers for failing external services. Set timeouts for long-running operations.
* **Safety rules**: Establish ethical boundaries and refusal conditions. Implement bias detection and mitigation. Define escalation triggers for human review.
* **Feedback loops**: Collect user feedback on responses. Monitor agent behavior for drift or degradation. Implement A/B testing for constraint effectiveness.

**Guardrail implementation**: Implement layered guardrails combining deterministic (rule-based) and model-based approaches:
* **Deterministic guardrails** use regex patterns, keyword matching, and explicit checks. They're fast, predictable, and cost-effective for enforcing clear boundaries like blocking specific words, validating data formats, or checking against predefined lists.
* **Model-based guardrails** leverage LLMs or specialized classifiers for semantic understanding. They catch subtle issues like implicit harmful intent, context-dependent violations, or sophisticated prompt injection attempts.

Apply guardrails at multiple points: before processing requests (input validation), after generating responses (output validation), around tool calls (action validation), and before final delivery (safety checks).

#### Component 7: Structured output
Structured output ensures that agent responses follow consistent formats, making them easier to parse, validate, and integrate into downstream systems. This is particularly important for agents that interface with other services.

* **What to implement**:
* **Output schemas**: Define Pydantic models that specify the structure of agent responses for different task types. Include validation rules to ensure outputs meet quality standards.
* **Format adaptation**: Enable the agent to adapt output format based on context - detailed for human consumption, structured for API integration.
* **Validation and error handling**: Implement validation logic to ensure outputs contain required information. Define how the agent should communicate failures, partial results, or uncertainty within the structured format.


### Part 2: Context failures
Context failures occur when agents receive misleading information, exceed context limits, encounter conflicting data, or lose track of conversation state. Understanding and handling these failures is critical for production reliability.

#### Context poisoning
Context poisoning occurs when malicious or misleading information enters the agent's context, potentially leading to incorrect or harmful responses. This can happen through:
- Prompt injection: Users crafting inputs that override system instructions.
- Malicious tool responses: Compromised external systems returning poisoned data.
- Corrupted memory: Stored information that's been tampered with or degraded.

**Detection strategies**:
- Input validation checking for instruction override attempts.
- Tool response verification against expected formats and content. Verify the credibility and freshness of information sources.
- Memory integrity checks using checksums or signatures.
- Anomaly detection identifying unusual patterns in context.

**Mitigation approaches**:
- Instruction hierarchy where system prompts take precedence.
- Input sanitization removing or escaping suspicious content.
- Tool response sandboxing and validation.
- Memory versioning and rollback capabilities.
- Route suspected poisoning attempts to human reviewers for investigation.

#### Context overflow
Context overflow occurs when the total amount of information exceeds the model's context window, forcing the agent to drop important information or fail entirely.

**Detection strategies**:
- Continuously monitor context size in tokens. Set warning thresholds at 70-80% of maximum capacity.
- Maintain metadata about the importance and recency of each context element to inform compression decisions.
- Watch for degraded response quality or increased latency as context approaches limits.

**Mitigation approaches**:
- Summarize older or less critical information to reduce token usage while preserving essential meaning. Use LLM-based summarization for complex content.
- Keep high-priority information (system prompt, current task, recent messages) and compress or drop lower-priority content (old messages, background information).
- Organize context into priority layers and progressively compress lower-priority layers as space becomes constrained.
- Move detailed information to external storage (vector stores, databases) and retrieve only what's needed for the current task.

#### Context distraction
Context distraction occurs when irrelevant information overwhelms relevant context, causing the agent to lose focus on the actual task. This manifests as:
- Too much retrieved information drowning out key details.
- Conversation wandering away from original intent.
- Irrelevant details building up over long conversations.

**Detection strategies**:
- Relevance scoring for all context components.
- Topic coherence monitoring across conversation turns.
- Attention pattern analysis identifying when agent focuses on wrong information.

**Mitigation approaches**:
- Aggressive relevance filtering for retrieved information.
- Periodic context summarization removing irrelevant details.
- Topic anchoring keeping conversations focused on core objectives.
- Context pruning removing low-relevance information.

#### Context confusion
Context confusion occurs when contradictory, ambiguous or irrelevant information causes the agent to produce inconsistent or uncertain outputs. Sources include:
- Contradictory facts: Different sources providing conflicting information.
- Ambiguous instructions: Unclear or conflicting directives in system prompts.
- Temporal inconsistencies: Outdated information conflicting with current data.
- Irrelevant or extraneous context: Injecting unnecessary details, unrelated memories, unused tools, or non-task-specific content that the model mistakenly incorporates into its reasoning, leading to degraded or incorrect outputs.

**Detection strategies**:
- Contradiction detection comparing information from different sources.
- Confidence scoring identifying uncertain or ambiguous situations.
- Temporal consistency checking for outdated information.
- Relevance auditing detecting context components that score low on semantic similarity or task relevance but still appear in the context window.

**Mitigation approaches**:
- Source prioritization using authority, recency, and reliability.
- Explicit uncertainty communication when contradictions exist.
- Temporal tagging and freshness scoring for all information.
- Conflict resolution strategies (user clarification, majority voting, expert sources).
- Use RAG to pull only the most relevant context elements - including tool descriptions, memory items, knowledge snippets, and procedural rules.

#### Context clash
Context clash occurs when information from different sources conflicts, creating ambiguity about what the agent should believe or how it should respond.

**Detection strategies**:
- Compare information from different sources (memory, knowledge base, tool results, user input) to identify conflicts.
- Maintain confidence scores for information from different sources to help resolve conflicts.

**Mitigation approaches**:
- Establish a clear priority order for information sources (e.g., recent tool results > knowledge base > memory > training data).
- Consider the recency of information when detecting conflicts - newer information may supersede older data.
- When conflicts are detected, acknowledge them to the user and explain which source is being prioritized and why.
- Present conflicting information from multiple sources and let the user decide which to trust.
- Combine conflicting information weighted by confidence scores to produce a balanced response.

#### Implementation: Context management
Implement a context manager class that monitors context quality and handles failures. Integrate context management into your agent workflow as a preprocessing step that validates context quality before the agent processes each input.


### Part 3: Context engineering techniques
Context engineering techniques provide systematic approaches to managing context at scale. These strategies ensure agents remain effective as complexity grows, context windows fill, and requirements evolve.

#### *Write* strategies: Crafting effective context
Write strategies focus on creating high-quality context from the start, reducing the need for aggressive compression or selection later.

**Context production**:
- **Prompt engineering**: Apply systematic prompt engineering techniques to craft clear, effective instructions. Structure system prompts as composable modules that can be combined based on task requirements. Use few-shot examples strategically, ensure prompts are altitude-appropriate (neither too prescriptive nor too vague), and leverage structured delimiters to organize information clearly.
- **Tool system design**: Design tool interfaces and documentation that maximize clarity and usability from the outset. This includes establishing clear naming conventions that reflect tool purpose, structuring tool parameters with self-documenting schemas (e.g., Pydantic models with descriptive field names and validation), providing concise yet comprehensive descriptions that explain both what the tool does and when to use it, including usage examples, and organizing tools into logical categories to reduce decision ambiguity. Write tool specifications that are maximally informative with minimal tokens.
- **RAG system design**: Design RAG systems that produce well-structured, contextually rich outputs from the start. This includes implementing effective chunking strategies, semantic indexing with metadata, and retrieval pipelines that return information with clear provenance markers, relevance indicators, source attribution, and confidence scores. Structure the entire RAG workflow—from document processing to context injection to maximize signal-to-noise ratio.
- **Memory system architecture**: Design memory systems with intentional schemas and encoding strategies from the ground up. This includes defining memory types (episodic vs. semantic, short-term vs. long-term), establishing structured storage formats optimized for both retrieval and comprehension, implementing semantic tagging and metadata systems, assigning importance and relevance scores, and creating consolidation rules for merging or summarizing related memories. Design the memory lifecycle to determine what gets stored, how it's encoded, when it's retrieved, and when it's compressed or discarded.
* **Canonicalization**: Normalize data as it enters the system by standardizing entity formats (order numbers, phone numbers, emails), resolving temporal references (converting "yesterday" to specific dates), and ensuring consistent formatting before storage.

**Context maintenance**:
- **Append-only**: New information is always appended to context, never modifying existing content. This maintains a complete audit trail but can lead to context overflow.
- **Update-in-place**: Existing context elements are updated when new information supersedes them. This keeps context compact but loses historical information.
- **Versioned updates**: Maintain multiple versions of context elements, allowing rollback and comparison. This provides both history and current state but increases storage requirements.
- **Structured merging**: New information is merged with existing context using structured rules that handle conflicts and maintain consistency.

#### *Select* strategies: Choosing relevant information
Select strategies determine which information enters the context window, preventing context pollution and maintaining focus.
- **Semantic filtering**: Use vector similarity to select memories and retrieved documents most relevant to current queries. Apply similarity thresholds to exclude low-relevance information.
- **Conditional tool exposure**: Dynamically choose which tools to expose based on task type, user permissions, and current context. This reduces decision complexity and context size.
- **Query-based retrieval**: Apply multiple filtering stages to knowledge base queries: semantic similarity, metadata filters, recency requirements, and source authority.
- **Importance-based memory selection**: Prioritize high-importance memories and episodes when context space is limited. Use importance scores based on user feedback, task success, and business impact.
- **Context layering**: Organize information into priority-based layers, ensuring that critical information is always available while less important content can be compressed or dropped when space is constrained. When context approaches limits, compress lower-priority layers first while preserving system instructions and current task context. This ensures the agent maintains its core identity and understanding of the immediate task even when working memory is constrained.

**Selection heuristics**:
- **Relevance-based**: Select context elements most semantically similar to the current task or query. Use embedding-based similarity search.
- **Recency-based**: Prioritize recent information over older content. Use time-decay functions to weight context elements.
- **Diversity-based**: Ensure context includes diverse perspectives and information types. Avoid redundancy by selecting representative examples.
- **Task-specific**: Select context based on the current task type. Different tasks (question answering, planning, execution) may need different context.

#### *Transform* strategies: Rewriting and restructuring context
Transform strategies focus on modifying the content and structure of context to improve clarity, coherence, and usability. Transform strategies ensure that context is clean, consistent, and optimized for LLM comprehension before being presented to the agent. These techniques are especially valuable in long-running conversations, multi-agent workflows, and retrieval-heavy systems.

##### *Compress* strategies: Reducing context size
Compress strategies reduce the size of necessary information without losing critical details.
- **Conversation summarization**: Periodically summarize older conversation turns, preserving key information while reducing token count. Use LLM-based summarization with explicit instructions about what to preserve.
- **Tool result compression**: Summarize verbose tool outputs, extracting key information and discarding unnecessary details.
- **Memory consolidation**: Merge related memories and episodes over time, preventing fragmentation while preserving essential information.
- **Hierarchical compression**: Compress at different granularities - keep recent messages verbatim, summarize older messages, and provide only high-level summaries of ancient history.
* **Instruction distillation**: Synthesize long or repetitive instructions into compact rules through redundancy elimination, rule consolidation and prioritization.

##### *Order* strategies: Structuring and sequencing context
These strategies control how information is arranged within the prompt and how ordering affects model attention. Because LLMs do not weigh all tokens equally, as LLMs prioritize some parts of the context window more than others (early + final tokens), the layout of context has a direct impact on reliability, alignment, and task performance.
* **Salience ordering:** placing the most important instructions early and late in the prompt where models weigh attention more strongly.
* **Task-first prompt layout**: Begin with high-level goals, followed by supporting details, examples, or constraints.
* **Temporal ordering**: Present events or facts chronologically to reduce confusion and maintain coherence.
* **Chunk grouping**: Organize related information into structured blocks (e.g., tools, memories, task details), helping the LLM parse and prioritize content efficiently.

#### *Isolate* strategies: Separating concerns
Isolate strategies separate different concerns into distinct contexts rather than cramming everything into a single agent's context.
- **Sub-agent architectures**: Decompose complex tasks into subtasks handled by specialized sub-agents, each with focused context. A coordinator agent orchestrates sub-agents and integrates results. Sub-agents operate with their isolated context and return results to the parent agent, which integrates them into the main context. This prevents sub-agents from accessing or modifying information outside their scope. Architecture patterns:
    - **Hierarchical coordination**: A coordinator agent breaks down tasks and delegates to specialized sub-agents. The coordinator maintains high-level context while sub-agents work with task-specific context.
    - **Peer collaboration**: Multiple agents work together as peers, sharing context through a common memory or message bus. Each agent maintains its own perspective and expertise.
    - **Pipeline processing**: Agents are arranged in a pipeline where each agent processes the output of the previous one, adding its own context and transformations.
- **Context chaining**: Break long workflows into stages where each stage has its own context. Pass only essential information between stages. When the agent needs information from previous contexts, it can retrieve them on-demand rather than keeping everything in active memory.
- **Layered context**: Organize context into layers (system, task, conversation, tool) with clear boundaries and access patterns. This prevents interference between layers.
- **Specialized memory stores**: Maintain separate memory systems for different purposes (user preferences, domain knowledge, procedural patterns) with independent retrieval and management.

#### *Refresh* Strategies: Periodically cleaning, updating, or rewriting context
These ensure long-running agents avoid degradation. This is important for agents with long-running loops or multi-day sessions. Examples:
* Scheduled context cleanup.
* Memory refactoring.
* Rebuilding context from authoritative sources.
* Refreshing task objectives to avoid drift.
