---
name: authifi-oauth-concepts
description: Understand Authifi OAuth 2.0 and OIDC concepts as documented, without calling live product APIs.
---

# Authifi OAuth and OIDC Concepts

Use this skill when explaining or planning integration with Authifi as an authorization server. This covers documented product behavior, not live API calls on the docs domain.

## Authifi as authorization server

Authifi acts as an OAuth 2.0 authorization server and OpenID Connect provider. Each tenant can define user groups, assign OAuth clients, and restrict access by group membership.

## Key documentation

- OAuth client authorization: `/authorization/authorization/`
- SSO integration and client setup: `/guides/sso-integration-guide/`
- Admin roles and API scopes: `/authorization/admin-roles/`
- NHE delegated tokens for LLM agents: `/guides/nhe-delegated-tokens/`

## Discovery on product deployments

OAuth and OIDC discovery endpoints are published per tenant on Authifi product deployments, not on the documentation site. Typical paths include:

- `/.well-known/openid-configuration`
- Tenant-scoped authorize and token endpoints under `/_api/auth/<tenant>/`

Refer to the SSO Integration guide for issuer configuration, client registration, redirect URIs, PKCE, and supported grant types.

## Agent delegation

For short-lived tokens delegated to automated agents or LLM pipelines, read the NHE Delegated Tokens guide. It covers RFC 8693 token exchange, actor tokens, and tenant-level configuration.

## Important distinction

The documentation site at authifi.pages.dev is public and unauthenticated. Do not expect OAuth metadata, token endpoints, or protected resources on this domain.
