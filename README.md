# Authifi Documentation

Public documentation for the Authifi identity and access management platform.

**Live Site**: [https://authifi.pages.dev](https://authifi.pages.dev)

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
https://abc123.authifi.pages.dev
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

## Agent Readiness

This site publishes static assets for AI agent discovery and crawl policy control.

### Static assets (in repo)

| Path | Purpose |
|------|---------|
| `/robots.txt` | Crawl rules, AI bot entries, Content Signals |
| `/_headers` | Cloudflare Link headers and Content-Type overrides |
| `/.well-known/api-catalog` | RFC 9727 documentation catalog |
| `/auth.md` | Agent access instructions |
| `/.well-known/agent-skills/index.json` | Agent skills discovery index |

The MkDocs hook at `docs/hooks/agent_assets.py` generates `sitemap.xml`, copies static agent files, and updates skill SHA-256 digests on each build.

### Markdown for Agents (Cloudflare dashboard)

Markdown content negotiation requires a Cloudflare zone setting. It is not implemented in application code.

1. Log in to the [Cloudflare dashboard](https://dash.cloudflare.com)
2. Select the zone for `authifi.pages.dev` (or your custom docs domain)
3. Open **AI Crawl Control**
4. Enable **Markdown for Agents**

Requires Pro, Business, or Enterprise. Alternatively, enable via API:

```bash
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/{zone_id}/settings/content_converter" \
  --header "Authorization: Bearer {api_token}" \
  --header "Content-Type: application/json" \
  --data '{"value":"on"}'
```

Do not enable conflicting robots.txt management in AI Crawl Control while `/robots.txt` is maintained in this repository.

### Verification

After deploy:

```bash
curl -sI https://authifi.pages.dev/robots.txt
curl -sI https://authifi.pages.dev/
curl -sH "Accept: application/linkset+json" https://authifi.pages.dev/.well-known/api-catalog
curl -sI https://authifi.pages.dev/auth.md
curl -s https://authifi.pages.dev/.well-known/agent-skills/index.json
curl -sI https://authifi.pages.dev/ -H "Accept: text/markdown"
```

Run a full scan:

```bash
curl -s -X POST https://isitagentready.com/api/scan \
  -H "Content-Type: application/json" \
  -d '{"url":"https://authifi.pages.dev"}'
```

## Project Structure

```
authifi-docs/
├── .changeset/              # Changesets configuration
├── docs/                    # Documentation source files
│   ├── index.md            # Home page
│   ├── robots.txt          # Crawl and AI bot policy
│   ├── _headers            # Cloudflare Pages header rules
│   ├── auth.md             # Agent access instructions
│   ├── .well-known/        # Agent discovery endpoints
│   ├── hooks/              # MkDocs build hooks
│   ├── javascripts/        # Client-side scripts (WebMCP)
│   ├── authorization/      # Authorization concepts
│   ├── guides/             # Administrator guides
│   └── security/           # Security documentation
├── mkdocs.yml              # MkDocs configuration
├── package.json            # Node.js dependencies (for changesets)
├── requirements.txt        # Python dependencies
├── CONTRIBUTING.md         # Contributor guide
└── README.md               # This file
```

## Versioning with Changesets

This project uses [Changesets](https://github.com/changesets/changesets) to manage versioning and generate changelogs.

### Adding a Changeset

When you make changes that should be noted in the changelog, add a changeset:

```bash
npm run changeset:add
```

This will prompt you to:
1. Select the type of change (major, minor, or patch)
2. Write a summary of the change

The changeset file is committed with your PR and consumed when releasing.

### Version Types

- **patch**: Bug fixes, typo corrections, small updates
- **minor**: New documentation pages, significant content additions
- **major**: Major restructuring, breaking changes to URL structure

### Releasing (Automated)

When PRs with changesets are merged to `main`, a GitHub Action automatically:

1. Detects pending changesets
2. Applies version bumps directly to `package.json` and `CHANGELOG.md`
3. Commits and pushes the changes

No manual intervention required.

To manually release (if needed):

```bash
npm run version
```

### Checking Status

To see pending changesets:

```bash
npm run changeset:status
```

## Technology Stack

- **Static Site Generator**: [MkDocs](https://www.mkdocs.org/)
- **Theme**: [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- **Hosting**: [Cloudflare Pages](https://pages.cloudflare.com/)
- **Features**: Full-text search, dark mode, mobile responsive, deploy previews
