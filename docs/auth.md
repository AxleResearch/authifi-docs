# auth.md

Agent access instructions for the Authifi Documentation site.

## Audience

AI agents and automated tooling that read Authifi product documentation.

## Access model

All published documentation pages on this site are public. No registration, API keys, or credentials are required to read documentation content.

Send `Accept: text/markdown` to receive a markdown representation of HTML pages when Markdown for Agents is enabled on the hosting zone.

## Discovery

- API catalog: `/.well-known/api-catalog` (`application/linkset+json`)
- Agent skills index: `/.well-known/agent-skills/index.json`
- Crawl policy: `/robots.txt`

## Authifi product OAuth and OIDC

This documentation domain does not host an OAuth authorization server or protected API endpoints.

Authifi deployments expose OAuth 2.0 and OpenID Connect discovery per tenant on the product domain (for example `/.well-known/openid-configuration` under each tenant path). See the [SSO Integration guide](/guides/sso-integration-guide/) for product OAuth behavior, client configuration, and issuer management.

For agent delegation with short-lived tokens, see [NHE Delegated Tokens](/guides/nhe-delegated-tokens/).

## Content usage

This site declares the following content preferences in `/robots.txt`:

- `ai-train=no`
- `search=yes`
- `ai-input=yes`
