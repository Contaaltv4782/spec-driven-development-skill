# Contributing

Thanks for your interest in contributing to the Spec-Driven Development Skill!

## How to Contribute

### Reporting Issues

- Check if the issue already exists
- Use a clear and descriptive title
- Provide as much context as possible

### Suggesting Enhancements

- Open an issue describing the enhancement
- Explain why this would be useful
- Include examples if applicable

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit with clear messages (`git commit -m 'Add amazing feature'`)
5. Push to your branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

## Content Guidelines

When updating skill documentation:

| Content Type | File | Guidelines |
|--------------|------|------------|
| Workflow phases | `references/workflow-phases.md` | Step-by-step, include examples |
| Artifact templates | `references/artifact-templates.md` | Copy-paste ready, no placeholders without instructions |
| Prompt patterns | `references/prompt-patterns.md` | Include phase context, inputs, expected outputs |
| Quality gates | `references/quality-gates.md` | Use checkbox format, keep checks actionable |
| Anti-patterns | `references/anti-patterns.md` | Include symptom, wrong example, correct example, fix |
| AI agent patterns | `references/ai-agent-patterns.md` | Include context requirements, token budgets |

## Anti-Pattern Format

When adding a new anti-pattern to `references/anti-patterns.md`, follow this structure:

```markdown
### AP-N: Title

**Symptom**: What you observe when this happens
**Cause**: Why it happens
**Fix**: What to do instead

**Wrong:**
[example of the anti-pattern]

**Correct:**
[example of the correct approach]
```

## Questions?

Feel free to open an issue for any questions about contributing.
