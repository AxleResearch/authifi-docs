# Authifi Documentation

Public documentation for the Authifi identity and access management platform.

**Live Site**: [https://authifi-docs.pages.dev](https://authifi-docs.pages.dev) *(after Cloudflare Pages setup)*

## For Documentation Writers

See [CONTRIBUTING.md](CONTRIBUTING.md) for instructions on editing documentation. **No local setup required** — you can edit directly on GitHub.com or using GitHub.dev (press `.` on the repo page).

## Cloudflare Pages Setup

To deploy this documentation site:

### 1. Connect Repository to Cloudflare Pages

> **Important**: Use **Pages**, not Workers. The setup screens look similar but Pages is for static sites.

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Go to **Workers & Pages** in the left sidebar
3. Click **Create** button
4. Select the **Pages** tab (not Workers)
5. Click **Connect to Git**
6. Authorize GitHub and select the `AxleResearch/authifi-docs` repository
7. Configure build settings:
   - **Project name**: `authifi-docs`
   - **Production branch**: `main`
   - **Framework preset**: None
   - **Build command**: `pip install -r requirements.txt && mkdocs build`
   - **Build output directory**: `site`
8. Click **Save and Deploy**

### 2. Verify Deploy Previews

Once connected, every pull request will automatically get a preview URL. Contributors will see a comment on their PR with a link like:

```
https://abc123.authifi-docs.pages.dev
```

### 3. Custom Domain (Optional)

To use a custom domain instead of `*.pages.dev`:

1. Go to your Pages project → **Custom domains**
2. Add your domain (e.g., `docs.authifi.com`)
3. Follow the DNS configuration instructions

## Local Development

### Prerequisites

- Python 3.8+
- pip

### Setup

```bash
# Set up a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
python3 -m pip install -r requirements.txt

# Start development server
mkdocs serve
```

Open http://127.0.0.1:8000 to view the documentation locally.

### Build

```bash
mkdocs build
```

The static site is generated in the `site/` directory.

## Project Structure

```
authifi-docs/
├── docs/                    # Documentation source files
│   ├── index.md            # Home page
│   ├── authorization/      # Authorization concepts
│   ├── guides/             # Administrator guides
│   └── security/           # Security documentation
├── mkdocs.yml              # MkDocs configuration
├── requirements.txt        # Python dependencies
├── CONTRIBUTING.md         # Contributor guide
└── README.md               # This file
```

## Technology Stack

- **Static Site Generator**: [MkDocs](https://www.mkdocs.org/)
- **Theme**: [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- **Hosting**: [Cloudflare Pages](https://pages.cloudflare.com/)
- **Features**: Full-text search, dark mode, mobile responsive, deploy previews
