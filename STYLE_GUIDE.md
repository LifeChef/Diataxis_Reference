# Documentation Style Guide

Consistency guidelines for all documentation.

## Voice and Tone

### By Document Type

- **Tutorials**: Friendly, encouraging ("You'll build", "Let's create")
- **How-To**: Direct, professional ("Execute", "Configure")
- **Reference**: Neutral, factual ("Returns", "Accepts")
- **Explanations**: Conversational, thoughtful ("Consider", "This means")

## Writing Style

- **Active voice**: "The service processes" not "is processed"
- **Present tense**: "Returns" not "will return"
- **Second person**: "You configure"
- **Short sentences**: 15-20 words
- **Short paragraphs**: 2-4 sentences

## Formatting

### Headings
- Use ATX-style: `## Heading`
- Title case for H1
- Sentence case for H2+

### Lists
- Unordered: use `-`
- Ordered: use natural numbering or `1.` consistently

### Code
- Inline: Use backticks for `commands`, `files`, `variables`
- Blocks: Always specify language

```javascript
const example = true;
```

## Terminology

| Use | Not |
|-----|-----|
| repository | repo |
| directory or folder | - |
| command line | terminal, CLI |
| database | DB |
| configuration | config |

## Links

Use descriptive text:
```markdown
[See deployment guide](path/to/guide.md)
```

Not: "click here"

## Admonitions

```markdown
> **💡 Tip**: Helpful information
> **⚠️ Warning**: Important warning
> **ℹ️ Note**: Additional context
```

## Tables

Include headers:

```markdown
| Column 1 | Column 2 |
|----------|----------|
| Data | Data |
```

## Accessibility

- Alt text for images
- Descriptive link text
- Table headers
- Code explanations

---

**Questions?** Refer to CONTRIBUTING.md
