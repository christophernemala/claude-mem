<claude-mem-context>
# Recent Activity

### Jan 10, 2026

| ID | Time | T | Title | Read |
|----|------|---|-------|------|
| #39050 | 3:44 PM | 🔵 | Plugin commands directory is empty | ~255 |
</claude-mem-context>

# Universal Memory System

Claude-mem is configured as a universal, non-domain-locked memory system. The following describes the schema, routing rules, and retrieval discipline.

## Modes

| Mode | File | Use When |
|------|------|----------|
| `universal` | `modes/universal.json` | Multi-domain sessions or when domain is mixed/unclear |
| `finance` | `modes/finance.json` | Financial research, budgeting, investing |
| `ai-research` | `modes/ai-research.json` | AI/ML research, prompt engineering, model evaluation |
| `coding` | `modes/code.json` | Software development (default) |
| `job-search` | `modes/job-search.json` | Job applications, interviews, career strategy |
| `automation` | `modes/automation.json` | Workflows, integrations, scripts, triggers |

Switch mode: set `CLAUDE_MEM_MODE` in `~/.claude-mem/settings.json`, or use `/mode <name>` in session.

## Memory Types (Universal Mode)

The `universal` mode uses memory types as observation types:

| Type | id | When to use |
|------|----|-------------|
| Insight | `semantic` | Fact, pattern, or concept learned |
| Event | `episodic` | Experience, outcome, or milestone |
| Template | `procedural` | Reusable workflow, process, or prompt |
| Decision | `strategic` | Preference, policy, goal, or architectural choice |

## Concept Tagging (Universal Mode)

Every observation in `universal` mode MUST have:
- **Exactly 1 domain tag**: `finance` | `ai` | `coding` | `job-search` | `automation` | `general`
- **Exactly 1 function tag**: `research` | `execute` | `plan` | `review` | `automate`
- **0–3 signal tags**: `high-value` | `template` | `versioned` | `compressed`

Example valid concept arrays:
```
["finance", "research", "high-value"]
["coding", "execute", "template"]
["ai", "plan", "versioned"]
["automation", "automate", "compressed"]
["job-search", "review"]
```

## Ingestion Rules

Store an observation only if it is:
1. **Novel** — not captured in an earlier observation this session
2. **Reusable** — applicable beyond the current moment (template, pattern, decision)
3. **Decision-relevant** — shapes future behaviour across sessions
4. **Pattern-worthy** — a recurring dynamic worth naming

**Compression rule**: If 3+ similar operations occur in a session, skip individual observations and create ONE `procedural` observation synthesising the pattern as a reusable template. Tag it with `compressed`.

## Versioning

When an observation updates a prior known pattern:
- Set subtitle to `v2: [what changed]`
- Add concept `versioned` to the concepts array

## Retrieval

Search by domain + memory type for precise results:

```
# All strategic decisions in finance
search(query="...", obs_type="strategic") → filter results with concept "finance"

# All templates across all domains
search(query="...", obs_type="procedural")

# AI research insights
search(query="prompt pattern", obs_type="semantic") → filter concept "ai"

# High-value observations
search(query="...", obs_type="semantic") → filter concept "high-value"
```

The mem-search skill supports `obs_type` filtering natively. Domain filtering is applied via keyword matching on the `concepts` field.

## Output Discipline

- **Structured**: Every observation uses the 4-field format (title, subtitle, facts[], narrative)
- **Minimal**: Skip low-signal operations — no output is better than noisy output
- **Domain-specific**: Concept tags make domain-scoped retrieval reliable
- **No drift**: Concepts MUST come from the allowed list for the active mode — no ad-hoc tags