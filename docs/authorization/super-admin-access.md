## Super Administrator access requirements (Auth service)

This document lists places in the Authifi service where **Super Administrator** privileges are required **or treated as a privileged bypass**.

### What “system admin” means in code

- **Boolean used everywhere**: `UserInfoAuthorization.isSystemAdmin`
- **Where it comes from**: request middleware binds `UserInfoBindings.USER_AUTHORIZATION` and sets `isSystemAdmin` via `TenantAuthorizationService.isSystemAdmin({ userId })`.
  - Source: `packages/auth/src/providers/auth-middleware.provider.ts` (`bindAuthorizationContext`)

### Legacy Code References

The Authifi service codebase uses these identifiers (unchanged for backward compatibility):

- **Role constant:** `ROLE_AUTH_SYSTEM_ADMIN` (refers to Super Administrator role)
- **Authorization check:** `UserInfoAuthorization.isSystemAdmin` (checks Super Administrator status)
- **Group name:** `systemAdmins` (group containing Super Administrators)
- **Service method:** `TenantAuthorizationService.isSystemAdmin()` (verifies Super Administrator)

These identifiers remain in code for backward compatibility but refer to Super Administrators. Throughout this document, "system admin" refers to what is now called "Super Administrator" in user-facing documentation to avoid confusion with infrastructure system administrators.

### Legend

- **SUA-only**: request is rejected unless `isSystemAdmin === true` (Super Administrator).
- **SUA-or-scope**: Super Administrator is allowed, but non-super-admins can proceed with a specific elevated scope.
- **SUA-or-tenant-admin**: Super Administrator is allowed, but tenant admins can also proceed.
- **Conditional-SUA-only**: becomes SUA-only only under a config/feature-flag condition.
- **SUA-bypass**: Super Administrators are exempt from a limit/constraint that applies to non-super-admins.

### Table of contents

- [Tenants](#tenants)
- [Users & groups](#users-groups)
- [Apps / clients](#apps-clients)
- [Identity providers](#identity-providers)
- [Roles & permissions (privileged RBAC entities)](#roles-permissions-privileged-rbac-entities)
- [Access requests](#access-requests)
- [Secrets](#secrets)
- [Templates (system templates)](#templates-system-templates)
- [Platform / system APIs](#platform-system-apis)

---

## Tenants

- **Create tenant**: `POST /auth/admin/tenants`
  - **Conditional-SA-only** when `auth.defaults.restrictTenantCreation === true`
  - Enforcement: `packages/auth/src/controllers/tenant.controller.mts`

- **Create tenant with shared identity providers**: `POST /auth/admin/tenants/createWithProviders`
  - **Conditional-SA-only** when `auth.defaults.restrictTenantCreation === true`
  - Enforcement: `packages/auth/src/controllers/tenant.controller.mts`

- **Export tenant data**:
  - `GET /auth/admin/tenants/{tenantId}/export`
  - `GET /auth/admin/tenants/{tenantId}/export/stream`
  - **SA-or-tenant-admin** (requires “admin” as determined by `TenantAuthorizationService.isAdmin`, which treats system admins as privileged)
  - Enforcement: `packages/auth/src/controllers/tenant.controller.mts`

- **Update a tenant certificate name**
  - **SA-or-tenant-admin**
  - Enforcement: `packages/auth/src/controllers/tenant-certificates.controller.ts`

- **Create tenant beyond license `maxTenants`**
  - **SA-bypass** (system admins bypass the max-tenant limit)
  - Enforcement: `packages/auth/src/services/tenant-life-cycle.service.ts`

---

## Users & groups

- **Modify a tenant user’s `licenseId`**: `PATCH /auth/admin/tenants/{tenantId}/users/{id}`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/user.controller.ts`

- **Add users to the system-admin group**
  - `PUT /auth/admin/tenants/{tenantId}/groups/{groupId}/users/rel/{userId}`
  - `PUT /auth/admin/tenants/{tenantId}/groups/{groupId}/users/assign/bulk`
  - **SA-only** when the target group is `auth.defaults.systemAdminGroup`
  - Enforcement: `packages/auth/src/controllers/tenant-group.controller.ts`

- **Remove config-defined system admins from the system-admin group**
  - `DELETE /auth/admin/tenants/{tenantId}/groups/{groupId}/users/rel/{userId}`
  - **Not allowed** for users listed in `auth.system.administrators` (even by system admins)
  - Enforcement: `packages/auth/src/controllers/tenant-group.controller.ts`

- **Create/update/delete privileged groups** (groups marked `isPrivileged`, including names under the `admin::` namespace)
  - **SA-or-scope** (`ADMIN_SCOPE.ADMIN_PERMISSIONS_UPDATE`)
  - Enforcement: `packages/auth/src/controllers/tenant-group.controller.ts`

- **Invite users to a privileged group via data-change requests**
  - `POST /auth/admin/tenants/{tenantId}/data-change-request` (when `groupId` points at a privileged group)
  - **SA-or-scope** (`ADMIN_SCOPE.ADMIN_PERMISSIONS_UPDATE`)
  - Enforcement: `packages/auth/src/components/data-change-requests/controllers/data-change-request.controller.mts`

- **Tenant user expiration job excludes system admins**
  - **SA-bypass** (system admins are not expired by this job)
  - Enforcement: `packages/auth/src/components/bullmq-job/jobs/tenant-user-profile-expiration.mts`

---

## Apps / clients

- **Create clients beyond license `maxClients`**
  - **SA-bypass** (system admins bypass the max-client limit)
  - Enforcement: `packages/auth/src/controllers/tenant-client.controller.ts`

- **Update SAML access scripts** (claims mapping script, claims authorization script, user-info customization script)
  - **SA-or-scope** (non-system-admins must have the relevant elevated scopes, e.g. `ADMIN_SCOPE.UPDATE_ACCESS_SCRIPTS` and/or `ADMIN_SCOPE.UPDATE_CLIENTS`)
  - Enforcement: `packages/auth/src/controllers/tenant-client.controller.ts`

---

## Identity providers

- **Set or change `aal_override`**
  - Applies to:
    - `POST /auth/admin/tenants/{tenantId}/identity-providers`
    - `PATCH /auth/admin/tenants/{tenantId}/identity-providers/{id}`
    - `PUT /auth/admin/tenants/{tenantId}/identity-providers/{id}`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Create a trusted identity provider (`isTrusted: true`)**
  - `POST /auth/admin/tenants/{tenantId}/identity-providers`
  - **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT`)
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Change verification status (`isTrusted`) on an existing identity provider**
  - `PATCH/PUT /auth/admin/tenants/{tenantId}/identity-providers/{id}` (when `isTrusted` changes)
  - **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT`)
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Edit a verified/trusted identity provider**
  - `PATCH/PUT /auth/admin/tenants/{tenantId}/identity-providers/{id}` when the existing provider is already trusted
  - **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT`)
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Set or change IdP `mfaType`**
  - Applies to create and update/replace endpoints.
  - **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT`), with additional tenant-admin logic depending on whether the IdP is trusted.
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Set identity provider claims mapping script** (`config.scripts.mapUserProfile`)
  - **SA-or-scope** (`ADMIN_SCOPE.IDENTITY_PROVIDER_CLAIMS_SCRIPTING`) in the non-system-admin path
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Configure secondary unique attributes** (`config.secondaryUniqueAttributes`)
  - **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT` is treated as a privileged bypass; otherwise blocked)
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Manage “generic OAuth2” identity providers** (type `OAUTH2`)
  - **SA-or-scope** (`ADMIN_SCOPE.TRUSTED_PROVIDER_EDIT` is treated as a privileged bypass; otherwise `ADMIN_SCOPE.IDENTITY_PROVIDER_CLAIMS_SCRIPTING` is required)
  - Enforcement: `packages/auth/src/controllers/tenant-identity-provider.controller.mts`

- **Identity provider secret fields in API responses** (unmasked)
  - **SA-or-scope** (`ADMIN_SCOPE.IDENTITY_PROVIDER_SECRETS_LIST`)
  - Enforcement: `packages/auth/src/repositories/identity-provider.repository.mts` (repository masks secrets unless privileged)

---

## Roles & permissions (privileged RBAC entities)

- **Create/update/delete privileged roles**
  - Enforcement:
    - `packages/auth/src/controllers/roles/tenant-client-role.controller.ts`
    - `packages/auth/src/controllers/roles/tenant-access-role.controller.ts`
  - **SA-or-scope** (`ADMIN_SCOPE.ADMIN_PERMISSIONS_UPDATE`)
  - Extra restriction: default tenant-admin role (`DEFAULT_ROLE.TENANT_ADMIN`) has additional protections against rename/removal.

- **Create/update/delete privileged permissions**
  - Enforcement:
    - `packages/auth/src/controllers/tenant-permission.controller.ts`
    - `packages/auth/src/controllers/tenant-resource-server-permission.controller.ts`
  - **SA-or-scope** (`ADMIN_SCOPE.ADMIN_PERMISSIONS_UPDATE`)

---

## Access requests

- **Approve an access request even if not in the approval chain** (system admin override)
  - **SA-bypass** (system admins can approve even if not otherwise an authorized approver)
  - Enforcement: `packages/auth/src/components/access-requests/services/request-approval.service.mts`

- **Update a TOTP Reset request template**
  - `PUT /auth/admin/tenants/{tenantId}/requestable-accesses/{id}` (when `accessType === TOTP_RESET`)
  - **SA-or-tenant-admin**
  - Enforcement: `packages/auth/src/components/access-requests/controllers/requestable-access.controller.mts`

- **Delete the default-tenant TOTP Reset request template**
  - `DELETE /auth/admin/tenants/{tenantId}/requestable-accesses/{id}`
  - **Conditional-SA-only** when deleting a `TOTP_RESET` template in the **default tenant**
  - Enforcement: `packages/auth/src/components/access-requests/controllers/requestable-access.controller.mts`

---

## Secrets

- **Read plaintext for a system-shared tenant secret**
  - `GET /auth/admin/tenants/{tenantId}/secrets/{secretId}` (when the secret is shared at `SharedLevel.System`)
  - **SA-only**
  - Enforcement: `packages/auth/src/components/secret-manager/controllers/tenant-secret-manager.controller.mts`
  - Note: secrets marked `SecretWriteOnly` cannot be read.

- **Backfill users’ TOTP secrets to JWE**
  - `POST /auth/admin/secrets/backfill-totp-secrets-jwe`
  - **SA-only**
  - Enforcement: `packages/auth/src/components/secret-manager/controllers/user-totp-secret.controller.mts`

---

## Templates (system templates)

- **Revert an HTML template to system default**
  - `POST /html-templates/{id}/revert-to-system-default`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/html-template-history.controller.ts`

- **Create or update a system HTML template** (when `isSystemTemplate === true`)
  - `POST /auth/admin/tenants/{tenantId}/html-template`
  - `PUT /auth/admin/tenants/{tenantId}/html-template/{id}`
  - **SA-or-scope** (`ADMIN_SCOPE.UPDATE_SYSTEM_TEMPLATES`)
  - Enforcement: `packages/auth/src/components/html-template/html-template.controller.mts`

- **Create or update a system email template** (when `isSystemTemplate === true`)
  - `POST /auth/admin/tenants/{tenantId}/email-template`
  - `PUT /auth/admin/tenants/{tenantId}/email-template/{id}`
  - **SA-or-scope** (`ADMIN_SCOPE.UPDATE_SYSTEM_TEMPLATES`)
  - Enforcement: `packages/auth/src/components/email-template/email-template.controller.mts`

---

## Platform / system APIs

### Licensing

- **Create a license**: `POST /auth/admin/licenses`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/license.controller.ts`

- **Update a license**: `PATCH /auth/admin/licenses/{licenseId}`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/license.controller.ts`

- **Delete a license**: `DELETE /auth/admin/licenses/{licenseId}`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/license.controller.ts`

- **Assign a license to a user**: `PUT /auth/admin/licenses/{licenseId}/rel/{userId}`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/license.controller.ts`

### Issuers

- **Create/update/delete issuer**
  - `POST /auth/admin/issuers`
  - `PATCH /auth/admin/issuers`
  - `PATCH /auth/admin/issuers/{id}`
  - `PUT /auth/admin/issuers/{id}`
  - `DELETE /auth/admin/issuers/{id}`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/issuer.controller.ts`

### Allowed origins (global CORS allow-list)

- **List global allowed origins**: `GET /auth/admin/allowed-origins/global`
- **Refresh allowed-origins cache**: `POST /auth/admin/allowed-origins/refresh-cache`
- **Get allow-list enabled state**: `GET /auth/admin/allowed-origins/allowlist-enabled`
- **Set allow-list enabled state**: `PATCH /auth/admin/allowed-origins/allowlist-enabled`
- **Get discovery mode state**: `GET /auth/admin/allowed-origins/discovery-mode`
- **Set discovery mode state**: `PATCH /auth/admin/allowed-origins/discovery-mode`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/allowed-origin.controller.ts`

- **Create a global allowed origin**: `POST /auth/admin/tenants/{tenantId}/allowed-origins` with body `{ isGlobal: true }`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/allowed-origin.controller.ts`

- **List global origins via tenant endpoint**: `GET /auth/admin/tenants/{tenantId}/allowed-origins` with `filter.where.isGlobal === true`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/allowed-origin.controller.ts`

- **Toggle global status on an origin**: `PATCH /auth/admin/tenants/{tenantId}/allowed-origins/{id}` when request body includes `isGlobal`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/allowed-origin.controller.ts`

### System notifications

- **Set a notification as global**: `POST /auth/admin/tenants/{tenantId}/system-notification` with `isGlobal: true`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/system-notification.controller.ts`

- **Change global notification settings**: `PATCH /auth/admin/tenants/{tenantId}/system-notification/{id}` when request changes `isGlobal`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/system-notification.controller.ts`

### Audit

- **Restore audit logs**: `POST /auth/admin/tenants/{tenantId}/audit/restore`
  - **SA-only**
  - Enforcement: `packages/auth/src/controllers/audit.controller.ts`

- **Audit chain verification / last hash**
  - `GET /audit-chain/verify`
  - `GET /audit-chain/verify-checkpoints`
  - `GET /audit-chain/verify-last-week`
  - `GET /audit-chain/verify-last-month`
  - `GET /audit-chain/last-hash`
  - **SA-or-client-credentials** (allowed if system admin OR client-credentials grant)
  - Enforcement: `packages/auth/src/controllers/audit-log-chain.controller.ts`

### Jobs

- **Create/update/delete a job**
  - `POST /auth/admin/tenants/{tenantId}/job`
  - `PUT /auth/admin/tenants/{tenantId}/job/{id}`
  - `DELETE /auth/admin/tenants/{tenantId}/job/{id}`
  - **SA-only**
  - Enforcement: `packages/auth/src/components/job/controllers/job.controller.mts`

### SSH manager

- **Create or update SSH access requests (user SSH keys)**
  - `POST /auth/tenants/{tenantId}/user/ssh/ssh-request`
  - `PUT /auth/tenants/{tenantId}/user/ssh/ssh-request`
  - **SA-or-scope** (`ADMIN_SCOPE.USER_SSH_SECRET`)
  - Enforcement: `packages/auth/src/components/ssh-manager/controllers/ssh-manager.controller.mts`

### Uploads

- **Upload files while uploads are disabled**
  - **Conditional-SA-only** when `auth.uploads.enabled === false` and `auth.uploads.systemAdminsOnly === true`
  - Enforcement: `packages/auth/src/services/upload-authorization.service.ts`

### Landing pages

- **Edit landing pages** (wherever `LandingPageAuthorizationService.checkEditAuthorization(...)` is used)
  - **Conditional-SA-only** when `auth.landingPage.systemAdminsEnabled === true` and `auth.landingPage.tenantAdminsEnabled === false`
  - Enforcement: `packages/auth/src/services/landing-page-authorization.service.ts`
