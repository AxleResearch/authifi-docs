---
name: authifi-docs-navigation
description: Navigate the Authifi documentation site information architecture and find relevant guides.
---

# Authifi Documentation Navigation

Use this skill when you need to locate Authifi product documentation on https://authifi.pages.dev.

## Site structure

| Section | Path prefix | Topics |
|---------|-------------|--------|
| Home | `/` | Overview and entry points |
| Authorization | `/authorization/` | OAuth client authorization, admin roles, RBAC, privileged access |
| Guides | `/guides/` | Tenant admin, SSO, access requests, monitoring, NHE tokens |
| Security | `/security/` | Security admin, secure configuration, FedRAMP evidence |
| Feature List | `/feature-list.html` | Full product capability list |

## High-value pages

- OAuth client authorization: `/authorization/authorization/`
- Admin roles: `/authorization/admin-roles/`
- SSO integration: `/guides/sso-integration-guide/`
- NHE delegated tokens for agents: `/guides/nhe-delegated-tokens/`
- Recommended secure configuration: `/security/recommended-secure-configuration/`

## Fetching content

Request pages with `Accept: text/markdown` when the hosting zone supports Markdown for Agents.

Use the site search UI or the `search_docs` WebMCP tool when available in a browser context.

## Discovery endpoints

- API catalog: `/.well-known/api-catalog`
- Agent skills index: `/.well-known/agent-skills/index.json`
- Crawl policy: `/robots.txt`
