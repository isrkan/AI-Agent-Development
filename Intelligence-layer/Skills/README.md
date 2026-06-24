# Skills

A skill is a self-contained, on-demand package of knowledge and instructions that an AI agent loads to extend its capabilities for a specific category of work. At its core, a skill is a `SKILL.md` file - YAML frontmatter describing what the skill does, plus a Markdown body telling the agent how to do it - optionally accompanied by bundled resources such as reference documents, helper scripts and output templates. Where a base language model brings general competence, a skill brings the procedure, domain knowledge, constraints and output conventions that turn generic competence into reliable, repeatable performance on a particular kind of task.

It helps to contrast skills with tools. A tool is an executable function the agent *calls* to perform an action (`search_web(query)`, `read_file(path)`); its output is data. A skill is not called - it is *loaded into context* before and during planning, and its output is shaped agent behavior. Put simply, tools *do things*, while skills *know how to do categories of work*. A skill typically scopes which tools the agent may use while it is active.

Skills sit at the intersection of context engineering and knowledge retrieval. They are *selectively loaded context* that encodes expertise, workflow steps and tool scoping for specific task categories - working alongside context engineering (which manages what flows into the context window), planning and reasoning (which structures how the agent thinks), and RAG (which retrieves external knowledge). The design challenge throughout is the same one that governs all context engineering: deliver maximum capability for minimum token cost, loading detail only when it is actually needed.

## Implementation guide

### Step 1: Decide when a skill is the right tool (and define its type)
Not every task benefits from a skill. Skills add the most value when a task recurs, requires domain knowledge the base model lacks, follows a specific multi-step procedure, needs a consistent output format, or must be shared across several agents. For one-off tasks, or tasks fully handled by a single tool call, a skill is overhead.

A practical heuristic is to build a skill only when several of these indicators hold at once (a skill is justified when ≥ 3 of these conditions are true):
- task recurs frequently (e.g., > 5x per week).
- task requires domain knowledge not in the base model.
- task has a specific multi-step procedure.
- task requires a consistent output format.
- task needs tool access scoped narrowly.
- multiple agents need the same expertise.

Once we have decided to build a skill, naming its type up front shapes everything that follows - what the instruction body looks like, which resources it needs, and how it should be scoped:

- **Domain-expertise skills** encode specialized knowledge (legal review, medical triage, tax rules) the base model handles unreliably.
- **Capability skills** add a concrete competence (generate a commit message, write a SQL query, format a citation).
- **Workflow skills** orchestrate a multi-step procedure with a defined sequence (incident response, onboarding, release checklist).
- **Organizational skills** capture company-specific conventions, standards and institutional knowledge.

Start with the concepts before building anything: what skills are, how their lifecycle works, and the *progressive disclosure* principle that keeps them cheap (load lightweight metadata first, the full skill only on activation, and bundled resources only when cited).

### Step 2: Author the SKILL.md
Every skill is defined by a `SKILL.md` file combining YAML frontmatter (metadata) with a Markdown instruction body. The frontmatter is what a routing system reads to decide whether the skill applies; the body is what the agent follows once the skill activates.

```yaml
---
name: code-review          # kebab-case, unique identifier
description: |             # 1-2 sentences - this IS the routing signal
  Reviews code changes for correctness, security issues, and maintainability.
  Produces a structured report with CRITICAL/MAJOR/MINOR severity tiers.
version: 1.0.0             # semver
allowed-tools:             # only what this skill actually needs
  - read_file
  - search_web
tags: [engineering, review] # optional: aids discoverability
---

# Code Review Skill

## When to use this skill
When asked to review a pull request, diff, or code change for quality issues.

## Review protocol
1. Read the entire change before forming any judgment
2. Evaluate: correctness, security, clarity, maintainability
3. Classify issues: CRITICAL (must fix) | MAJOR (should fix) | MINOR (nice to fix)
4. Always include a Strengths section
5. Provide specific fix suggestions - never vague feedback
```

The most important field is `description`: it is the routing signal, so it must contain the domain terms a user would naturally use when requesting this kind of work. Treat the instruction body as a procedure, not prose - concrete steps and decision criteria, not background reading.

### Step 3: Bundle supporting resources
When a skill needs more than instructions, it can carry three kinds of supporting files. A skill can carry three kinds of supporting files, each in its own directory. Keeping each type in its own directory lets the agent load only what a given task requires:

```
my-skill/
├── SKILL.md          # Required: instructions + frontmatter
├── references/       # Large domain docs, loaded on demand when cited
│   └── domain-guide.md
├── scripts/          # Executable helpers, run only when invoked
│   └── helper.py
└── assets/           # Templates and static files, loaded only when rendering
    └── report-template.md
```

The rule of thumb is to move anything large or rarely needed out of `SKILL.md` and into a bundled resource that is referenced by relative path:

| Resource | Use when | Token impact |
|----------|----------|-------------|
| `references/` | Domain knowledge exceeds ~1000 tokens | Lazy-loaded (0 until cited) |
| `scripts/` | The skill needs runnable code | Loaded only when invoked |
| `assets/` | The skill produces templated output | Loaded only when rendering |


### Step 4: Write for discovery and token efficiency
A skill's `description` is read on every routing decision, and its body loads into context on every activation - so both must earn their tokens. Descriptions should name the exact action verbs and domain nouns users say ("review", "analyze", "generate commit message"), not abstract labels like "code quality management". Bodies should stay lean, with everything large pushed to `references/`.

Progressive disclosure makes this concrete by loading content in phases:

```python
# Target budgets per phase of progressive disclosure
SKILL_TOKEN_BUDGETS = {
    "frontmatter_metadata": 50,   # index phase: name + description, per skill
    "full_skill_md":        2000, # activation phase: warn at 3000, hard limit 5000
    "reference_document":   3000, # execution phase: loaded from references/ when cited
    "script":               500,  # loaded only when the helper runs
}
# Rule: if the SKILL.md body exceeds ~500 lines or ~5000 tokens, move content to references/ and cite it by relative path.
```

### Step 5: Enable discovery and routing
A skill that never activates is worse than no skill at all, so routing must be tested before deployment. Routing typically works by comparing the embedding of a user's request against each skill's `description`, activating the closest match above a similarity threshold. Verify that a description reliably matches the natural phrasings of its task - and reliably *misses* unrelated ones:

```python
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer("all-MiniLM-L6-v2")

def routing_score(description: str, query: str) -> float:
    """Cosine similarity between a skill description and a user query."""
    d, q = model.encode(description), model.encode(query)
    return float(np.dot(d, q) / (np.linalg.norm(d) * np.linalg.norm(q)))

description = "Reviews code changes for correctness, security, and maintainability."
routing_score(description, "can you review this PR")   # should be high (> 0.5)
routing_score(description, "summarize this document")  # should be low
```

Before declaring a skill ready, run a short validation checklist: routing similarity above ~0.5 for several natural phrasings, full `SKILL.md` under the token budget, every referenced file present at its declared path, and `allowed-tools` listing only tools the skill actually uses. Routing can also be explicit (the user names the skill) or scoped by where skills are scanned from.

If routing is weak, the fix is almost always in the `description` - tighten it and re-test. Round out the check with real inputs: run the skill end-to-end on a few representative tasks and confirm the output matches expectations. 

### Step 6: Compose skills for multi-step workflows (optional)
Complex tasks often span several phases, each best served by a different skill. Rather than building one monolithic skill, chain focused skills together and pass results forward between phases - a research skill gathers, an analysis skill interprets, a report skill synthesizes:

```python
# Sequential skill chain: research → analyze → synthesize
SKILL_CHAIN = [
    {"skill": "research-assistant", "phase": "gather"},
    {"skill": "data-analyst",       "phase": "analyze"},
    {"skill": "report-writer",      "phase": "synthesize"},
]

def run_skill_chain(task: str, chain: list[dict], agent) -> dict:
    """Activate skills in sequence, carrying prior results forward."""
    results = {}
    for step in chain:
        skill_context = load_skill(step["skill"])
        results[step["phase"]] = agent.run(
            task, system_context=skill_context, prior=results
        )
    return results
```

The key constraint is that skills do not share state automatically - intermediate results must be explicitly threaded through the chain. Skills can also declare dependencies on one another and combine with tools.

### Step 7: Deploy at the right scope and keep skills portable
With the skill validated, place it where the agent will discover it. Skills resolve from multiple scope levels, with the innermost (most specific) winning - so the same name can mean different things in different contexts if we are not deliberate about it:

```
.agents/skills/      ← project scope: committed to the repo, checked first
~/.agents/skills/    ← user scope: applies across the user's projects
/etc/agents/skills/  ← global scope: system/admin level, org-wide policies
```

These are the cross-platform [agentskills.io](https://agentskills.io) standard paths; individual runtimes also scan their own conventions (Claude Code uses `.claude/skills/`, Cursor uses `.cursor/skills/`, and so on. Sticking to the standard paths keeps skills portable. Treat skills as versioned artifacts using semantic versioning - MAJOR for breaking changes to the instruction body or removed fields, MINOR for new backwards-compatible capabilities, PATCH for fixes and wording. Adhering to the open AgentSkills specification keeps skills portable across platforms rather than locked to one runtime.

### Step 8: Govern skills at enterprise scale
Once skills become shared organizational assets, they need governance. This means controlling who can author, edit and access which skills (scoping and access), reviewing skills before they go live, tracking them in version control, and capturing institutional knowledge in a maintainable, auditable form. Without this, skill libraries drift into stale, conflicting or insecure instructions.

## Best practices
1. **Write descriptions as routing triggers.** The description is what the routing system matches against user queries. Include the exact terms users will say - "review", "analyze", "generate commit message" - not abstract phrases like "code quality management".
2. **Keep SKILL.md small.** The instruction body loads on every activation, so every line costs tokens. Keep it under ~500 lines and move large reference content to `references/`, cited by relative path.
3. **One skill, one purpose.** A skill that does code review *and* deployment *and* monitoring routes poorly and is hard to maintain. Split broad skills into focused ones that compose via chaining.
4. **Test routing before deploying.** Use embedding similarity to confirm the description reliably matches the natural phrasings of the task. A skill with 60% routing accuracy is worse than no skill at all.
5. **Scope narrowly with allowed-tools.** List only the tools the skill actually uses. Over-permissive tool lists defeat the security benefit of skill-level scoping.
6. **Version in git, track in a catalog.** Treat skills as code: tag releases, maintain a changelog, and update the skill catalog when publishing. Unversioned skills in production are an operational risk.

## Common pitfalls
1. **Vague descriptions that break routing.** "A helpful assistant for engineering tasks" never matches reliably. Descriptions must contain specific action verbs and domain nouns.
2. **Encoding everything in the instruction body.** Putting a 5,000-word style guide in `SKILL.md` burns tokens on every invocation. Move it to `references/` and load it only when cited.
3. **Forgetting to validate referenced files exist.** If `SKILL.md` cites `./references/domain-guide.md` but the file is missing, the skill silently fails. Run `skill_validator.py` before deployment.
4. **Using too many allowed-tools.** A research skill doesn't need `write_file` or `execute_code`. Over-broad tool lists create security surface area and make debugging harder.
5. **Composing skills without state management.** When chaining skills, intermediate results must be explicitly passed forward - skills don't share state automatically.
6. **Ignoring scope resolution order.** A system-level and a project-level skill with the same name conflict; the project-level one wins, but only in that project, producing inconsistent behavior across projects if unintended.

## Relationship to other Intelligence-layer components
- **Context engineering** provides the *mechanism* for loading skill content into the context window. Skills are a specialized application of the context injection strategies in context engineering.
- **RAG systems** can serve as *backends* for skill resource libraries - retrieving the right reference docs on demand instead of bundling everything in `references/`.
- **Planning and reasoning** techniques determine *when* and *how* to invoke multi-step skills; workflow skills often encode a planning pattern.
- **Agent modes** interact with skills at the tool-access level: a mode's permitted tools intersect with a skill's `allowed-tools` to determine what the agent may actually use.

## Next steps
Once basic skills are working:
1. Build a small library of focused, single-purpose skills rather than a few broad ones, and compose them via chaining.
2. Instrument routing in production - track which skills activate, for which queries, and where routing collides or misses.
3. Establish a validation and review pipeline so every new or updated skill passes `skill_validator.py` and a routing test before shipping.
4. Adopt the AgentSkills standard and semantic versioning so skills stay portable and auditable as the library grows.

Remember: a good skill is the smallest possible package that reliably turns a recurring task into consistent, high-quality output. Start with one focused skill, prove its routing and value, then grow the library deliberately.