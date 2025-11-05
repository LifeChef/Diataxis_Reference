# Contributing to Documentation

Thank you for contributing! This guide helps you create quality documentation using Diataxis.

## Quick Start

1. Identify document type (Tutorial/How-To/Reference/Explanation)
2. Use template from `/templates/`
3. Place in correct folder
4. Follow style guide
5. Submit for review

---

## Choosing Document Type

| User Goal | Type | Example |
|-----------|------|---------|
| Learn by doing | Tutorial | "Build your first API" |
| Solve specific problem | How-To | "How to deploy" |
| Look up details | Reference | "API endpoints" |
| Understand concepts | Explanation | "Why microservices" |

---

## Guidelines by Type

### Tutorials
**Must have**: Learning objectives, prerequisites, step-by-step instructions, checkpoints, troubleshooting
**Don't**: Explain concepts in depth, skip steps, show multiple approaches

### How-To Guides
**Must have**: Problem statement, prerequisites, direct solution, verification
**Don't**: Teach fundamentals; move deep conceptual background to explanations
**Do**: Present a single recommended approach; optionally note alternatives at the end

### Reference
**Must have**: Complete accurate info, consistent structure, parameter tables, examples
**Don't**: Explain concepts, include opinions, mix versions

### Explanations
**Must have**: Context, concept explanations, comparisons, real-world examples
**Don't**: Include step-by-step instructions, assume prior knowledge

---

## File Naming

- Use kebab-case: `setup-environment.md`
- Folders: category folders may be plural (e.g., `concepts/`, `decisions/`)
- Update README.md when adding docs

---

## Quality Checklist

- [ ] Title clear and descriptive
- [ ] Correct Diataxis category
- [ ] Follows template structure
- [ ] Prerequisites listed
- [ ] Code examples tested
- [ ] Links to related docs
- [ ] No broken links
- [ ] Follows style guide
- [ ] No spelling errors

---

## Code Examples

- Must work (test them!)
- Include imports and setup
- Specify language in code blocks
- Add clarifying comments

````markdown
```javascript
const client = new APIClient();
```
````

---

## Linking Documents

Link related documentation:
- Tutorials → How-To and Reference
- How-To → Reference and Tutorials
- Reference → How-To examples
- Explanations → All types

---

## Maintenance

Review documentation:
- After major releases
- Every 6 months
- When errors reported

Mark deprecated content:
```markdown
> **⚠️ Deprecated**: Use [new approach](link) instead.
```

---

For detailed guidelines, see `STYLE_GUIDE.md`
