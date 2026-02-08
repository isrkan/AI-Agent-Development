# AI Agent Development Guide
This repository is a comprehensive educational and reference guide for developing AI agents applications. It follows a layered architecture that progressively covers foundational components through deployment and safety considerations.

## Repository structure

```
AI-Agent-Development-Guide/
├── Foundation-layer/                      # Core infrastructure
│   ├── LLMs/                              # Language model integration
│   ├── Memory/                            # Short-term and long-term memory
│   └── Tool_use/                          # Function calling and MCP
├── Behavior-layer/                        # Agent behavior shaping
│   ├── Prompt-engineering/                # Prompt engineering techniques
│   └── Structured-output/                 # Schema-based responses
├── Intelligence-layer/                    # Knowledge and reasoning
│   ├── RAG-system/                        # Retrieval-augmented generation
│   ├── Context-engineering/               # Context management strategies
│   └── Planning_and_reasoning/            # Extended thinking and reasoning
├── Architecture-and-orchestration-layer/  # Workflows
│   └── Agent-loop-and-control-flow/       # Design and workflow patterns
└── Safety-layer/                          # Guardrails and compliance
    └── Guardrails-system/                 # Safety monitoring
```

## Layer descriptions

### Foundation-layer
Core building blocks for AI agents:
- **LLMs/**: API-based (OpenAI, Gemini) and open-source (LLaMA) integrations.
- **Memory/**: In-memory and Redis-based persistence for session and long-term memory.
- **Tool_use/**: function calling examples and MCP server implementations.

### Behavior-layer
Shapes agent responses and output formatting:
- **Prompt-engineering/**: Prompt techniques and AI agent personas.
- **Structured-output/**: Pydantic models for schema-based generation.

### Intelligence-layer
Advanced reasoning and knowledge retrieval:
- **RAG-system/**: RAG techniques.
- **Context-engineering/**: Context management strategies across write, select, transform, isolate and refresh patterns.
- **Planning_and_reasoning/**: Thinking and reasoning techniques.

### Architecture-and-orchestration-layer
Complex workflow orchestration and agent coordination:
- **Agent-loop-and-control-flow/**: workflow patterns and agentic design patterns.

### Safety-layer
Guardrails integration with Qualifire for prompt injection detection, PII protection, and content safety.

## Purpose
Provide hands-on implementations, tutorials, and best practices for AI agent development across multiple layers of complexity.

### Note
This repository is constantly updated with new notebooks and techniques, so feel free to check back frequently for new content.

All the code in this repository are for educational purpose and self learning, it is not guaranteed that they will work in every situation.