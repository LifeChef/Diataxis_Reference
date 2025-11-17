# Copy Guide for New Projects

**Purpose:** Explain which files and folders from this repository should be copied into **new project repositories**, and which are **template-only** or **example-only**.

This avoids duplicating meta files that only make sense in the **master template**, while keeping a clear, consistent structure in each project.

---

## 1. Files to copy into every new project

When you create a new project based on this template, copy (or recreate) the following items:

### 1.1 Root files (per-project core)

- `README.md` – Project-specific introduction and navigation
- `CONTRIBUTING.md` – How to contribute documentation/code
- `STYLE_GUIDE.md` – Writing style and formatting rules
- `FOLDER_STRUCTURE.md` – Folder layout and naming conventions
- `QUICK_REFERENCE.md` – Choosing the right Diataxis doc type
- `VALIDATION_CHECKLIST.md` – Quality checklist for the project docs
- `NEW_PROJECT_START_HERE.md` – Step-by-step bootstrap guide for new projects

> Tip: Adapt these files to your project (name, scope, links), but keep their **roles** and **structure**.

### 1.2 Folders to copy

- `templates/` – All documentation templates (tutorial, how-to, reference, explanation)
- `tutorials/` – Project-specific tutorials (initially you may have only 1–2)
- `how-to/` – Project-specific how-to guides
- `reference/` – Project-specific reference docs (API, configuration, architecture)
- `explanation/` – Project-specific explanation docs (concepts, decisions)

You should **also create** (if they do not exist yet in your project):

- `features/` – Feature-level docs (requirements, design, testing)
- `docs/` – System-wide docs (architecture, security, decisions), if needed
- `runbooks/` – Operational documentation

The exact content of these folders will be specific to each project.

---

## 2. Template-only files (do not copy into every project)

The following files describe this **template repository itself** and should normally stay only in the master template, not in each project:

- `PROJECT_SUMMARY.md` – Summary of this Diataxis template repo
- `IMPLEMENTATION_COMPLETE.md` – Status and validation report for this template

You can use them as **reference**, but per-project repos generally do not need their own copies of these exact files.

If a specific project needs its own summary or status document, create a project-specific variant (for example, `PROJECT_OVERVIEW.md` or `PROJECT_STATUS.md`) that describes that project.

---

## 3. Example-only content (reference, not to copy verbatim)

### 3.1 Root sample documents

The sample docs in the root-level Diataxis folders:

- `tutorials/`
- `how-to/`
- `reference/`
- `explanation/`

are **examples** of structure and writing style. For real projects, you should:

- Use them as patterns
- Replace them with your own project-specific content

### 3.2 `Blog_Engine_Example/`

The `Blog_Engine_Example/` folder is a **complete reference implementation** showing:

- How Diataxis works in a realistic system
- How `features/`, `docs/`, `runbooks/`, and Diataxis folders interact
- Example workflows (traditional and AI-augmented)

For new projects:

- **Do not copy this folder wholesale** into your repo.
- Instead, study it and copy **patterns** (file naming, linking, structure) into your own project-specific docs.

---

## 4. How to apply this when starting a new project

1. Create a new repository for your project.
2. Copy or recreate the **per-project core** items listed in section 1.
3. Use `NEW_PROJECT_START_HERE.md` as your detailed bootstrap guide.
4. Treat:
   - `PROJECT_SUMMARY.md` and `IMPLEMENTATION_COMPLETE.md` as template documentation only.
   - `Blog_Engine_Example/` as a **reference atlas**, not a folder to duplicate.

This keeps each project lean and focused, while preserving all rich examples and meta documentation in the central template.
