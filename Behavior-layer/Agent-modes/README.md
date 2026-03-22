# Agent Modes

A **mode** is a behavioral contract imposed on an AI agent that governs two things simultaneously:
1. **How much autonomy** the agent exercises (does it ask permission, or does it act?)
2. **What kind of behavior** the agent performs (does it research, plan, execute, critique, or monitor?)

Modes are distinct from prompts and skills. A prompt is a single instruction; a skill is a reusable capability. A mode is a persistent behavioral framework that shapes every decision the agent makes while active - how it selects tools, how it responds to uncertainty, when it pauses, and when it proceeds.

Think of modes as the "operating context" of an agent. The same underlying model, with the same tools, can behave like a cautious copilot or an autonomous executor depending solely on which mode is active.

## The two axes of modes

### Axis 1 — Autonomy level (How much the agent decides)

| Level | Name | Description |
|-------|------|-------------|
| 0 | Chat | Pure reactive: one input, one output, no tools |
| 1 | Copilot | Suggests actions, human executes |
| 2 | Supervised | Agent acts, human approves at checkpoints |
| 3 | Semi-autonomous | Agent acts on low-risk, escalates on high-risk |
| 4 | Fully autonomous | Agent plans, acts, and self-corrects end-to-end |

### Axis 2 — Behavioral type (What the agent does)

| Type | Name | Description |
|------|------|-------------|
| A | Task execution | Complete a defined deliverable |
| B | Research | Gather, synthesize, and cite information |
| C | Planning | Decompose goals into steps before acting |
| D | Critic/reflection | Self-evaluate and revise |
| E | Conversational | Clarify, explore, advise via dialogue |
| F | Monitoring | Passive observation, alert-on-condition |

These two axes combine. A "Research + Semi-Autonomous" mode gathers information independently but escalates when it encounters ambiguity or sensitive sources.

## Implementation Guide

### Step 1: Choose the autonomy level
Start from the least autonomous mode that satisfies our use case. Escalate only when human oversight creates clear bottlenecks.

```python
# Decision criteria for autonomy level
def choose_autonomy_level(task: dict) -> str:
    if task["requires_irreversible_actions"]:
        return "supervised"          # Always checkpoint before irreversible ops
    if task["risk_level"] == "high":
        return "semi-autonomous"     # Escalate only on high-risk steps
    if task["fully_automated_pipeline"]:
        return "fully-autonomous"    # Only when end-to-end trust is established
    if task["human_executes_suggestions"]:
        return "copilot"             # Agent advises, human acts
    return "chat"                    # Default: pure dialogue
```

**Practical rule:** The more irreversible an action, the lower the autonomy. Deleting records requires supervised or lower; reading a file is safe for fully autonomous.

### Step 2: Choose the behavioral Ttype
Behavioral type is orthogonal to autonomy - select based on the cognitive pattern required.

```python
BEHAVIORAL_MODE_SELECTOR = {
    # Input characteristic → recommended mode
    "open-ended question":         "conversational",   # dialogue and exploration
    "retrieve and summarize":      "research",         # gather, synthesize, cite
    "achieve a multi-step goal":   "planning",         # decompose before acting
    "validate or improve output":  "critic_reflection",# evaluate and revise
    "accomplish a defined task":   "task_execution",   # plan-execute-verify
    "observe and alert":           "monitoring",       # passive, threshold-based
}
```

### Step 3: Define the permission scope
Every mode must explicitly declare which tools are permitted and which are blocked. This prevents autonomous agents from accidentally using capabilities that are unsafe for their current mode.

### Step 4: Configure escalation rules
Escalation rules define exactly when the agent must pause and hand off to a human. They must be explicit - ambiguous rules lead to under-escalation.

### Step 5: Implement runtime mode switching
Agents need to change modes as context evolves. Mode switching must preserve conversational state.

### Step 6: Compose multiple modes
For complex agents, combine a compliance overlay with a behavioral type and an autonomy level.

### Step 7: Deploy and monitor
Track mode-level metrics to detect behavioral drift and optimize autonomy thresholds over time.


## Quick reference: Choosing a mode

| Use case | Autonomy level | Behavioral type |
|----------|---------------|-----------------|
| Customer-facing chatbot | Chat (0) | Conversational |
| Code review assistant | Copilot (1) | Critic/reflection |
| Research summarization | Supervised (2) | Research |
| Automated data pipeline | Fully autonomous (4) | Task execution |
| Complex project planning | Semi-autonomous (3) | Planning |
| Compliance monitoring | Supervised (2) | Monitoring |
| Medical or legal advice | Copilot (1) | Conversational + compliance |

## Best practices

1. **Default to simpler modes.** Start at chat or copilot. Escalate only when human oversight creates bottlenecks we can measure.
2. **Modes are behavioral contracts, not prompts.** They persist across turns, shape tool access, and define escalation rules - not just the tone of one response.
3. **Match autonomy to reversibility.** Irreversible actions (delete, deploy, send) need supervised or lower. Reversible actions (read, analyze, draft) can tolerate higher autonomy.
4. **Compose deliberately.** Combining modes (e.g., autonomous + compliance) creates powerful behavior, but conflicting modes create dangerous edge cases. Test all combination pairs explicitly.
5. **Build the trust ladder.** Begin a new use case in supervised mode. Collect approval-rate data. If approval rate exceeds 95% for 30 days for example, graduate to semi-autonomous. Track closely for 2 weeks before considering fully autonomous.
6. **Separate what from how much.** Behavioral type (what the agent does) and autonomy level (how much it decides) are independent. A monitoring agent can be copilot-level (flags only) or fully autonomous (self-remediates). Choose each axis independently.