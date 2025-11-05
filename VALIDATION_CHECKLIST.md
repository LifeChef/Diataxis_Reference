# Repository Validation Checklist

Use this checklist to validate the documentation repository setup.

## Structure Validation

- [ ] Main README.md exists at root
- [ ] CONTRIBUTING.md exists
- [ ] STYLE_GUIDE.md exists
- [ ] QUICK_REFERENCE.md exists
- [ ] FOLDER_STRUCTURE.md exists
- [ ] .gitignore configured

### Folder Structure

- [ ] `/templates/` folder exists
- [ ] `/tutorials/` folder exists
- [ ] `/how-to/` folder exists
- [ ] `/reference/` folder exists
- [ ] `/explanation/` folder exists

### Templates

- [ ] tutorial-template.md exists
- [ ] how-to-template.md exists
- [ ] reference-template.md exists
- [ ] explanation-template.md exists

### Sample Documents

**Tutorials:**
- [ ] At least 2 tutorial examples exist
- [ ] Tutorials have proper structure
- [ ] Tutorials include prerequisites
- [ ] Tutorials have step-by-step instructions

**How-To Guides:**
- [ ] At least 3 how-to examples exist
- [ ] How-tos have problem statements
- [ ] How-tos have clear solutions
- [ ] How-tos include verification steps

**Reference:**
- [ ] At least 3 reference documents exist
- [ ] References are factual and complete
- [ ] References include technical specs
- [ ] References use consistent formatting

**Explanations:**
- [ ] At least 2 explanation documents exist
- [ ] Explanations provide context
- [ ] Explanations explain "why"
- [ ] Explanations are conversational

### Index Files

- [ ] Each major folder has README.md
- [ ] README files list available documents
- [ ] README files include brief descriptions
- [ ] Links in README files are correct

## Content Validation

### Documentation Quality

- [ ] All code examples use proper syntax highlighting
- [ ] Consistent terminology used throughout
- [ ] Active voice used in instructions
- [ ] Headings follow logical hierarchy
- [ ] No spelling errors

### Cross-References

- [ ] Documents link to related content
- [ ] Links use descriptive text
- [ ] Relative paths used for internal links
- [ ] No broken links (or intentionally documented exceptions)

### Formatting

- [ ] Markdown formatting is consistent
- [ ] Code blocks specify language
- [ ] Tables have headers
- [ ] Lists formatted correctly

## Template Quality

- [ ] Templates have clear placeholders
- [ ] Templates include all necessary sections
- [ ] Templates have examples/guidance
- [ ] Templates follow style guide

## Supporting Documentation

- [ ] CONTRIBUTING.md explains authoring process
- [ ] STYLE_GUIDE.md covers formatting rules
- [ ] QUICK_REFERENCE.md provides decision matrix
- [ ] FOLDER_STRUCTURE.md shows organization

## Team Readiness

- [ ] Team members understand Diataxis framework
- [ ] Clear process for adding new documentation
- [ ] Review process defined
- [ ] Maintenance schedule established

---

## Quick Validation Commands

Check all markdown files exist:
```bash
find . -name "*.md" -type f | wc -l
```

List directory structure:
```bash
tree -L 3
```

Check for broken internal links (basic):
```bash
grep -r "\[.*\](.*\.md)" --include="*.md" . | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  link=$(echo "$line" | grep -o '([^)]*.md)' | tr -d '()')
  if [ ! -f "$(dirname "$file")/$link" ]; then
    echo "Potential broken link in $file: $link"
  fi
done
```

---

## Sign-Off

**Validated By**: _____________  
**Date**: _____________  
**Notes**: 

---

**Version**: 1.0  
**Last Updated**: 2024
