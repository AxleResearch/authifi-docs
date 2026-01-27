# Contributing to Authifi Documentation

Thank you for contributing to the Authifi documentation! This guide explains how to make changes to the documentation.

## Quick Start (No Setup Required)

You can edit documentation directly on GitHub without installing anything on your computer.

### Option 1: Edit on GitHub.com

1. Navigate to the file you want to edit in the `docs/` folder
2. Click the **pencil icon** (Edit this file) in the top-right corner
3. Make your changes in the editor
4. Scroll down and add a commit message describing your changes
5. Select **"Create a new branch for this commit and start a pull request"**
6. Click **Propose changes**
7. Review your changes and click **Create pull request**

### Option 2: Use GitHub.dev (VS Code in Browser)

1. Go to the repository on GitHub
2. Press the `.` key (period) on your keyboard
3. This opens VS Code in your browser with the full repository
4. Edit files, then commit using the Source Control panel on the left
5. Create a pull request when ready

## Reviewing Your Changes

When you create a pull request, Cloudflare Pages automatically builds a **preview deployment**. You'll see a comment on your PR with a link like:

```
https://abc123.authifi-docs.pages.dev
```

Click this link to see exactly how your changes will look on the live site before merging.

## Documentation Structure

```
docs/
├── index.md                 # Home page
├── authorization/           # Authorization concepts and admin roles
├── guides/                  # Step-by-step administrator guides
└── security/               # Security configuration and best practices
```

## Writing Guidelines

### File Format

- All documentation is written in **Markdown** (`.md` files)
- Use standard Markdown syntax for headings, lists, links, and code blocks

### Headings

- Use `#` for the page title (only one per page)
- Use `##` for major sections
- Use `###` for subsections
- Don't skip heading levels (e.g., don't go from `##` to `####`)

### Code Blocks

Use triple backticks with a language identifier:

````markdown
```json
{
  "example": "code"
}
```
````

### Tables

Use Markdown tables for structured information:

```markdown
| Column 1 | Column 2 | Column 3 |
| -------- | -------- | -------- |
| Value 1  | Value 2  | Value 3  |
```

### Admonitions (Notes and Warnings)

Use admonition syntax for callouts:

```markdown
!!! note
    This is a note with additional information.

!!! warning
    This is a warning about potential issues.

!!! tip
    This is a helpful tip.
```

### Links

- **Internal links**: Use relative paths: `[Link Text](../guides/tenant-admin-guide.md)`
- **External links**: Use full URLs: `[Link Text](https://example.com)`

## Adding New Pages

1. Create a new `.md` file in the appropriate folder (`authorization/`, `guides/`, or `security/`)
2. Add a title as the first line: `# Page Title`
3. Update `mkdocs.yml` to add your page to the navigation

Example addition to `mkdocs.yml`:

```yaml
nav:
  - Guides:
    - Your New Guide: guides/your-new-guide.md
```

## Local Development (Optional)

If you want to preview changes locally before pushing:

### Prerequisites

- Python 3.8 or later
- pip (Python package manager)

### Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Start local development server
mkdocs serve
```

Open http://127.0.0.1:8000 in your browser to see the documentation.

Changes to `.md` files will automatically reload in the browser.

### Building the Site

```bash
mkdocs build
```

This creates the static site in the `site/` folder.

## Need Help?

- For documentation questions, open an issue on GitHub
- For Authifi product questions, contact your administrator
