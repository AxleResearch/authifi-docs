## Admin roles and privileged access (Authifi service)

This document describes how the Authifi service models “admin” privileges and where those privileges come from:

- **System admins** (platform-wide)
- **Tenant admins** (tenant-wide)
- **Scoped admins** (users granted elevated `ADMIN_SCOPE.*` permissions)

It complements (not replaces) the detailed operation inventory in `packages/auth/docs/authorization/super-admin-access.md`.

---

### Terminology Note

This document uses "system admin" to refer to the technical implementation in code. In user-facing documentation, this role is called **"Super Administrator"** to distinguish it from infrastructure system administrators who manage servers and operating systems. The code identifiers (`ROLE_AUTH_SYSTEM_ADMIN`, `systemAdmins` group, `isSystemAdmin` checks) remain unchanged for backward compatibility.

---

### Table of contents

- [System admins](#system-admins)
- [Tenant admins](#tenant-admins)
- [Scoped admins (admin scopes)](#scoped-admins-admin-scopes)
- [How these roles are enforced](#how-these-roles-are-enforced)
- [Common patterns and examples](#common-patterns-and-examples)
- [Securing admin accounts (best practices)](#securing-admin-accounts-best-practices)

---

## System admins

**What it means**

- A request-scoped boolean used throughout the codebase: `UserInfoAuthorization.isSystemAdmin`.

**How it is determined**

- Bound in request middleware (`UserInfoBindings.USER_AUTHORIZATION`) and set via `TenantAuthorizationService.isSystemAdmin({ userId })`.
- Sources:
  - `packages/auth/src/providers/auth-middleware.provider.ts` (`bindAuthorizationContext`)
  - `packages/auth/src/services/is-tenant-admin.service.ts` (`TenantAuthorizationService.isSystemAdmin`)

**What it enables**

- System admins can perform platform-wide privileged operations and are often treated as a bypass for limits and checks.
- Some actions are **still blocked even for system admins** (e.g., certain protections around config-defined system admins).
- See `packages/auth/docs/authorization/super-admin-access.md` for a comprehensive list of SUA-only/SUA-or-scope operations.

---

## Tenant admins

**What it means**

- Tenant admins are treated as “full admin within a tenant” for most tenant-scoped APIs.

**How it is determined**

- `TenantAuthorizationService.isAdmin({ tenantId, userId })` checks tenant admin membership (configured via `auth.defaults.adminGroup`).
- Source: `packages/auth/src/services/is-tenant-admin.service.ts` (`TenantAuthorizationService.isAdmin`)

**What it enables**

- **Within a tenant**, tenant admins are generally able to perform tenant admin actions, except where an operation is explicitly restricted to system admins or to an elevated admin scope.
- Tenant admins are also used in **cross-tenant authorization** via “trusted tenants” delegation (see below).

---

## Scoped admins (admin scopes)

**What it means**

- “Scoped admins” are users who are not necessarily system admins, but have elevated permissions via `ADMIN_SCOPE.*` scopes (checked with `userAuthorization.checkUserHasScope(...)`).

**How it is determined**

- Scopes come from the user’s permissions/roles and are evaluated at request time via `UserInfoAuthorization.checkUserHasScope(...)`.
- Sources:
  - `packages/auth/src/keys.js` / `UserInfoAuthorization`
  - Controllers and repositories that explicitly gate behavior by `ADMIN_SCOPE.*`

**What it enables**

- These scopes commonly grant access to operations that would otherwise be **system-admin-only** (or allow edits on “privileged” entities).
- Examples of elevated admin scopes used in Auth:
  - `ADMIN_SCOPE.ADMIN_PERMISSIONS_UPDATE` (privileged RBAC entities: admin groups/roles/permissions)
  - `ADMIN_SCOPE.UPDATE_SYSTEM_TEMPLATES` (system templates)
  - `ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT` (trusted/verified IdP restrictions, some IdP type restrictions)
  - `ADMIN_SCOPE.IDENTITY_PROVIDER_CLAIMS_SCRIPTING` (IdP claims mapping scripting)
  - `ADMIN_SCOPE.UPDATE_ACCESS_SCRIPTS` / `ADMIN_SCOPE.UPDATE_CLIENTS` (client scripting / client updates)
  - `ADMIN_SCOPE.IDENTITY_PROVIDER_SECRETS_LIST` (unmask IdP secrets in responses)
  - `ADMIN_SCOPE.USER_SSH_SECRET` (SSH key operations)

For the exact places these are enforced, see the relevant sections in `packages/auth/docs/authorization/super-admin-access.md`.

---

## How these roles are enforced

### 1) Controller-level and repository-level checks

- Many endpoints explicitly check `isSystemAdmin`, `isAdmin(...)`, and/or specific `ADMIN_SCOPE.*` permissions.
- Example patterns:
  - **SA-only**: `if (!userAuthorization.isSystemAdmin) throw Forbidden(...)`
  - **SA-or-scope**: allow system admin, otherwise require an `ADMIN_SCOPE.*`
  - **SA-or-tenant-admin**: allow system admin, otherwise allow tenant admins

### 2) Middleware “admin override” for scope failures

- A large portion of `/auth/admin/tenants/{tenantId}/...` endpoints are primarily protected by **OAuth scopes**.
- When authentication fails due to `Insufficient scope`, the request middleware can allow certain admin users to proceed:
  - Tenant admin of the **target tenant** (`{tenantId}`), or
  - Tenant admin of the **original token tenant** _and_ the target tenant is a **trusted tenant**.
- This bypass is not applied to **Client Credentials** tokens.
- Source: `packages/auth/src/providers/auth-middleware.provider.ts`

This is why many “admin” APIs don’t contain an explicit “tenant admin required” check: tenant admins may still be able to use them even when scopes are missing.

### 3) Trusted tenants delegation

- The service supports delegating admin access across tenants using “trusted tenants” relationships.
- Used by:
  - `packages/auth/src/controllers/trusted-tenant.controller.ts`
  - Authifi middleware and audience logic when evaluating tenant-scoped requests.

---

## Common patterns and examples

- **Tenant admins can do “anything in the tenant”, except what is SA-only or scope-gated**
  - Practically, tenant admins can perform most tenant administration, but some operations remain reserved for:
    - **System admins** (SA-only), or
    - **Scoped admins** (SA-or-scope), especially around privileged RBAC entities and sensitive scripting/secrets.

- **Privileged RBAC entities** (admin groups/roles/permissions)
  - Commonly enforced as **SA-or-scope** (`ADMIN_SCOPE.ADMIN_PERMISSIONS_UPDATE`).

- **Trusted identity provider controls**
  - Commonly enforced as **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT`), with additional tenant-admin-specific logic for some IdP fields.

- **Monitoring / reporting APIs**
  - Often scope-gated (e.g. audit/log scopes, protected health scope), with tenant-admin access potentially enabled via the middleware “admin override”.

---

## Securing admin accounts (best practices)

- **Prefer “trusted” identity providers for admin identities**
  - Restrict admin sign-in to identity providers that are verified/trusted and centrally managed.
  - Avoid granting admin access to accounts authenticated via ad-hoc or weakly verified providers.

- **Enforce MFA for all admin-capable accounts**
  - Require phishing-resistant MFA where possible (e.g. WebAuthn/FIDO2), at minimum TOTP.
  - Ensure “break-glass” admin accounts are also MFA-protected (with tightly controlled recovery).

- **Apply least privilege and minimize standing access**
  - Keep the number of **system admins** small.
  - Prefer **scoped admin access** (`ADMIN_SCOPE.*`) over system-admin where possible.
  - Remove unused admin scopes; time-bound or approval-gated elevation where feasible.

- **Separate admin and day-to-day user accounts**
  - Use dedicated admin accounts for privileged actions.
  - Do not browse email/drive/chat from privileged accounts; reduce exposure to phishing and token theft.

- **Review admin access regularly**
  - Periodically review:
    - system admin assignments,
    - tenant admin group membership,
    - users holding `ADMIN_SCOPE.*` privileges,
    - “trusted tenant” delegations.
  - Treat emergency additions as temporary and require follow-up removal/review.

- **Monitor and audit admin activity**
  - Audit logs should be routinely reviewed for privileged actions (RBAC changes, secrets access, IdP changes, template/script changes, tenant exports).
  - Alert on anomalous admin behavior (new admin grants, repeated denied attempts, unusual login patterns, cross-tenant access via delegation).

- **Harden sessions and tokens**
  - Shorter session lifetimes for admin contexts; revoke sessions promptly on role removal.
  - Rotate credentials/secrets used by admin automation; avoid long-lived tokens.

- **Have an incident-ready process**
  - Document how to revoke admin access quickly (including removing scopes, disabling accounts, revoking sessions).
  - Validate restore/recovery paths for audit logs and configuration (and restrict restore operations appropriately).
