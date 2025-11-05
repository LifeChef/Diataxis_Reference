# Diataxis Quick Reference

Fast reference for choosing the right documentation type.

## Decision Matrix

| Question | Answer | Document Type |
|----------|--------|---------------|
| Are they learning? | Yes | **Tutorial** |
| Do they have a specific task? | Yes | **How-To Guide** |
| Are they looking up information? | Yes | **Reference** |
| Do they want to understand? | Yes | **Explanation** |

## Key Characteristics

### 📖 Tutorial
- **Goal**: Learning by doing
- **Tone**: Friendly, encouraging
- **Structure**: Step-by-step with checkpoints
- **Example**: "Build your first API in 30 minutes"

### 🔧 How-To Guide
- **Goal**: Solve specific problem
- **Tone**: Direct, professional
- **Structure**: Problem → Solution → Verification
- **Example**: "How to deploy to production"

### 📋 Reference
- **Goal**: Look up technical details
- **Tone**: Neutral, precise
- **Structure**: Organized by topic/component
- **Example**: "REST API endpoints"

### 💡 Explanation
- **Goal**: Understanding concepts
- **Tone**: Conversational, thoughtful
- **Structure**: Context → Concept → Implications
- **Example**: "Why we chose microservices"

## Template Locations

```
/templates/
  ├── tutorial-template.md
  ├── how-to-template.md
  ├── reference-template.md
  └── explanation-template.md
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Mixing tutorials with reference | Split into separate docs |
| How-to explains concepts | Link to explanation instead |
| Reference includes opinions | Keep factual, move opinions to explanation |
| Tutorial assumes knowledge | List all prerequisites clearly |

## Quick Commands

Create new documentation:
```bash
# 1. Copy template
cp templates/[type]-template.md [category]/[subcategory]/[name].md

# 2. Edit file
code [category]/[subcategory]/[name].md

# 3. Update category README
code [category]/README.md
```

## Validation Checklist

- [ ] Used correct template
- [ ] Placed in correct folder
- [ ] Updated category README
- [ ] All code examples tested
- [ ] Links to related docs added
- [ ] Follows style guide

---

**Need more details?** See [CONTRIBUTING.md](CONTRIBUTING.md) and [STYLE_GUIDE.md](STYLE_GUIDE.md)
