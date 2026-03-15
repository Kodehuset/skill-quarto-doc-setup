---
name: skill-quarto-doc-setup
description: Set up professional Quarto documentation projects with PDF output via LaTeX templates. Use this skill when the user wants to create a Quarto project, set up a documentation project with PDF output, configure LaTeX templates for Quarto, render markdown to professional PDF, or asks about Quarto document setup. Triggers on mentions of "quarto", "pdf template", "latex template for documents", "documentation project setup", or "professional PDF from markdown".
version: 1.0.0
---

# Quarto Document Project Setup

This skill sets up professional documentation projects that render Markdown to PDF using Quarto and custom LaTeX templates.

## Workflow

Follow these steps in order when setting up a new Quarto document project.

### Step 1: Check Prerequisites

Run these checks and report results to the user:

```bash
# Check Quarto
quarto --version

# Check TeX Live
pdflatex --version

# Check required packages (attempt kpsewhich for each)
for pkg in libertinus.sty inconsolata.sty multirow.sty mdframed.sty zref.sty needspace.sty titlesec.sty enumitem.sty; do
  kpsewhich "$pkg" > /dev/null 2>&1 && echo "OK: $pkg" || echo "MISSING: $pkg"
done
```

If Quarto is missing, tell the user to install it from https://quarto.org/docs/get-started/

If TeX packages are missing, provide this command:
```bash
sudo tlmgr install libertinus libertinus-fonts libertinus-otf libertine inconsolata multirow mdframed zref needspace titlesec enumitem
```

### Step 2: Choose a Template

Present the user with these template options using AskUserQuestion:

1. **Corporate Report** — Professional business documents with logo support, navy blue accents, clean tables. Good for: annual reports, proposals, policy documents.

2. **Technical Manual** — Optimized for technical content with styled code blocks, admonition boxes (note/warning/tip), dark teal accents. Good for: API docs, technical specs, runbooks, SOPs.

3. **Assessment Report** — Color-coded status indicators, metadata boxes, summary tables, blue accents. Good for: compliance assessments, audits, gap analyses, security reviews.

4. **Minimal Clean** — No title page, black/gray only, wide margins, elegant typography. Good for: memos, short reports, internal documents, drafts.

### Step 3: Gather Document Metadata

Ask the user for:
- **Title** (required)
- **Subtitle** (optional)
- **Author** (optional)
- **Organization** (optional)
- **Document status** (e.g., "Draft", "Final", "Under Review")
- **Logo file path** (optional, for corporate-report template)

### Step 4: Create Project Files

The templates are located in this skill's `templates/` directory. Copy the selected template and create the QMD file.

**4a. Copy the LaTeX template** to the user's project directory:
```bash
cp <skill-path>/templates/<selected-template>.tex <project-dir>/<template-name>.tex
```

**4b. Create the QMD document** with this frontmatter structure:

```yaml
---
title: "<title>"
subtitle: "<subtitle>"
author: "<author>"
organization: "<organization>"
document_status: "<status>"
short_title: "<abbreviated title for headers>"
titlepage: true
toc: true
format:
  pdf:
    template: <template-name>.tex
    number-sections: true
    keep-tex: false
---

# First Section

Your content here.
```

For the **minimal-clean** template, omit `titlepage: true` (it uses an inline title instead).

**4c. Add to .gitignore** (if a git repo):
```
# Generated PDF output
*.pdf
```

### Step 5: Render and Verify

```bash
quarto render "<document-name>.qmd" --to pdf
```

Open the generated PDF and confirm it renders correctly. If there are LaTeX errors, check for missing packages first.

## Template Details

### Shared Features (All Templates)

- KOMA-Script `scrartcl` document class (A4 paper, 11pt)
- Libertinus serif + Inconsolata monospace fonts
- 2.5cm margins on all sides
- Zero paragraph indent with 8pt paragraph spacing
- Styled hyperlinks matching accent color
- Pandoc-compatible (`\tightlist`, `Highlighting` environment)
- Table of contents support
- Professional `booktabs`-style tables
- Customizable headers/footers with document status

### Frontmatter Variables

All templates support these QMD YAML variables:

| Variable | Used In | Description |
|----------|---------|-------------|
| `title` | All | Document title |
| `subtitle` | All except minimal | Subtitle below title |
| `author` | All | Author name |
| `organization` | All except minimal | Organization name on title page |
| `document_status` | All | Shown in footer (e.g., "Draft") |
| `short_title` | All | Abbreviated title for page headers |
| `logo` | corporate-report | Path to logo image for title page |
| `titlepage` | All except minimal | Set `true` to generate a separate title page |
| `toc` | All | Set `true` for table of contents |
| `framework` | assessment-report | Assessment framework name |
| `total_requirements` | assessment-report | Number of requirements assessed |
| `assessment_date` | assessment-report | Date of assessment |

### Template-Specific Commands

**Assessment Report** provides these LaTeX commands for use in QMD with raw LaTeX blocks:

- `\statusfully` — Green "Fully met" badge
- `\statuspartially` — Orange "Partially met" badge
- `\statusnotmet` — Red "Not met" badge
- `\assessmentsummary{fully}{partial}{not}` — Summary count table
- `\requirementmeta{title}{asset}{function}{description}` — Requirement metadata box

**Technical Manual** provides:

- `\notebox{text}` — Blue info box
- `\warningbox{text}` — Orange warning box
- `\tipbox{text}` — Green tip box

## Example QMD

An example document is available at `examples/example-document.qmd` in this skill's directory. Use it as a reference for frontmatter and content structure.
