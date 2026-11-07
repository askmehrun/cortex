# Cortex Template

Every cortex page follows this shape. It is written to be human-readable and RAG/agent-friendly: an agent reads the title, then the definition line, then the headings; a human skips the same way.

## Shape

```markdown
# <Concept Name>

<one-line definition: what it is + when/why it matters. This is the most important line.>

## When to Use
<triggers or prerequisites — when this applies. Add only if the content earns it.>

## How
<the mechanics: commands, config, code. Comment inline on WHY each flag/field matters.>

## Gotchas
<pitfalls, error cases, edge conditions. Add only if the content earns it.>

## Rule
<one-line invariant to remember. Every file ends with this.>
```

## Rules

- One concept per file.
- Open with the definition line — it is what humans and RAG both read first.
- Same heading names everywhere: `## When to Use`, `## How`, `## Gotchas`, `## Rule`.
- No tables — use prose or bullets (they parse cleanly in RAG).
- Tag every code block with a language.
- A file must make sense without following any cross-file link.
