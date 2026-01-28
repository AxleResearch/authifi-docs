# Authifi Service - Recommended Secure Configuration

This document provides comprehensive security guidance for configuring and operating the Authifi service in accordance with FedRAMP Recommended Secure Configuration requirements and security best practices.

> **See Also**: For a structured guide covering administrative account lifecycle (setup, configuration, operation, decommissioning) and security settings reference tables, refer to the [Security Admin Guide](./security-admin-guide.md).

## Table of Contents

- [Document Information](#document-information)
- [Terminology](#terminology)
- [Executive Summary](#executive-summary)
- [Administrative Account Types](#administrative-account-types)
- [Top-Level Administrative Accounts](#top-level-administrative-accounts)
- [Privileged Accounts](#privileged-accounts)
- [Security Settings Reference](#security-settings-reference)
- [Secure Configuration Baseline](#secure-configuration-baseline)
- [Security Best Practices](#security-best-practices)
- [Compliance and Audit](#compliance-and-audit)

---

## Document Information

**Document Version**: 1.0
**Last Updated**: 2026-01-28  
**Applies To**: Authifi Service 10.x.x and later
**Compliance**: FedRAMP Rev5 Recommended Secure Configuration  
**Classification**: Public

**Related Documentation**:

- [Super Admin Access Requirements](../authorization/super-admin-access.md)
- [Admin Roles Overview](../authorization/admin-roles.md)
- [Tenant Administrator Guide](../guides/tenant-admin-guide.md)
- [Users and Groups Management Guide](../guides/users-groups-admin-guide.md)
- [Monitoring and Logging Guide](../guides/monitoring-guide.md)

---

## Terminology

**Important Distinction:** This document uses specific terminology to distinguish between infrastructure and application administration:

### Super Administrator

- **Application-level administrator** with full control over the Authifi service
- Highest privilege level within the Authifi application
- Manages all tenants and platform-wide configurations
- Cross-tenant access and platform-level security controls
- **Code identifiers:** `Auth System Admin` role (constant: `DEFAULT_ROLE.SYSTEM_ADMIN`), `systemAdmins` group

### Infrastructure Administrator

- **Not covered in this document**
- Manages underlying servers, containers, Kubernetes, and operating systems
- Deploys and maintains Authifi service infrastructure
- OS-level system administration
- See deployment documentation for infrastructure requirements

### Tenant Administrator

- Manages a single tenant within Authifi
- No cross-tenant access
- Cannot access super administrator functions

**Throughout this document, "Super Administrator" refers exclusively to the Authifi application administrator role.**

---

## Executive Summary

The Authifi service provides enterprise identity and access management with a hierarchical administrative model. This document describes:

- **Top-level administrative accounts** (Super Administrators): Full platform control, cross-tenant management
- **Privileged accounts** (Tenant Administrators): Tenant-scoped administrative access
- **Scoped admin accounts**: Delegated permissions for specific resources

**Key Security Features**:

- Multi-factor authentication (MFA) enforcement for administrative accounts
- Role-based access control (RBAC) with least privilege
- Comprehensive audit logging of all administrative actions
- Session management with configurable lifetimes
- Encryption at rest and in transit
- API-based configuration management

**Secure Defaults**:

- MFA required for all administrative functions
- Short session lifetimes for privileged contexts
- Audit logging enabled by default
- HTTPS/TLS enforced for all communications

---

## Administrative Account Types

### Account Hierarchy

The Authifi service implements three levels of administrative access:

```
┌─────────────────────────────────────┐
│   Super Administrators              │  <- Top-Level Administrative Accounts
│   (Authifi System Admin role)          │
│   - Platform-wide control           │
│   - All tenant access               │
│   - License management              │
└─────────────────────────────────────┘
              │
              ↓
┌─────────────────────────────────────┐
│   Tenant Administrators             │  <- Privileged Accounts
│   (Tenant Admin Group Members)      │
│   - Single tenant control           │
│   - All resources within tenant     │
│   - Cannot access other tenants     │
└─────────────────────────────────────┘
              │
              ↓
┌─────────────────────────────────────┐
│   Scoped Administrators             │  <- Privileged Accounts
│   (ADMIN_SCOPE.* Permissions)       │
│   - Specific resource control       │
│   - Limited to assigned scopes      │
│   - Cannot modify tenant settings   │
└─────────────────────────────────────┘
```

### Terminology

**Top-Level Administrative Accounts** (referred to as "Super Administrators" in Authifi):

- Accounts with `Auth System Admin` role (constant: `DEFAULT_ROLE.SYSTEM_ADMIN`)
- Full control over all platform features and all tenants
- Can create/delete tenants, manage licenses, access all tenant data
- Bypass tenant-level permission checks

**Privileged Accounts** (referred to as "Tenant Administrators" and "Scoped Admins" in Authifi):

- Tenant Administrators: Members of tenant admin groups or users with tenant-admin-capable permissions
- Scoped Administrators: Users with `ADMIN_SCOPE.*` permissions for specific resources
- Limited to assigned tenant or resource scope
- Cannot access super administrator-level features

**Standard Accounts**:

- Regular users with no administrative privileges
- Access controlled by roles and permissions within their tenant
- Cannot modify security settings or administrative configurations

---

## Top-Level Administrative Accounts

### Super Administrator Role

**Reference**: See [Super Admin Access Requirements](../authorization/super-admin-access.md) for complete list of super-admin-only operations.

Super Administrators have unrestricted access to the Authifi platform and can:

**Platform Management**:

- Create, modify, and delete tenants
- Manage platform licenses and quotas
- Access all tenant data and configurations
- View and export cross-tenant audit logs
- Configure system-wide settings

**Security Operations**:

- Manage identity provider trust relationships
- Configure authentication assurance levels (AAL)
- Set multi-factor authentication (MFA) policies
- Manage encryption keys and certificates
- Access secret management across all tenants

**Administrative Functions**:

- Create and manage super administrators (via `systemAdmins` group in system configuration - legacy name)
- Grant temporary super admin access with expiration
- Execute system maintenance jobs
- Access SSH terminal for advanced operations
- Override tenant-level security restrictions
- Manage JWKS key rotation

### Creating Super Administrators

**Prerequisites**:

- Existing super administrator account required to perform assignment
- User account must already exist in the platform
- User must authenticate via a trusted identity provider

**Process**:

Super administrator assignment is controlled through system configuration, not through the standard UI workflow.

1. **Add User to systemAdmins Group**:
   - Existing super admin accesses system configuration
   - Adds target user account to the `systemAdmins` group (legacy group name)
   - This action automatically:
     - Grants `Auth System Admin` role
     - Triggers security alerts to all existing super admins
     - Creates detailed audit log entry
     - Provides platform-wide access

2. **Temporary Super Admin Access** (Optional):
   - For time-limited super admin needs (e.g., incident response, audits)
   - Existing super admin can grant temporary membership to `systemAdmins` group
   - Specify expiration date/time
   - Access automatically revoked after expiration
   - All actions during temporary access are logged with temporary flag

3. **Verify MFA Enrollment**:
   - Confirm user has MFA enabled (TOTP or WebAuthn)
   - Enforce MFA requirement at identity provider level
   - Document MFA device registration
   - WebAuthn/FIDO2 strongly recommended for super admins

4. **Security Alerts**:
   - All existing super admins receive alert notification
   - Alert includes: user email, granting admin, timestamp, temporary/permanent
   - Provides 24-hour window for review/challenge
   - Unexplained grants should be investigated immediately

**Security Checklist**:

- [ ] User authenticates via trusted identity provider
- [ ] MFA enabled and enrolled (TOTP or WebAuthn preferred)
- [ ] User identity verified through out-of-band channel
- [ ] Business justification documented
- [ ] All existing super admins notified
- [ ] User acknowledged security responsibilities
- [ ] Backup contact information recorded
- [ ] Temporary access expiration set (if applicable)

### Super Administrator Security Settings

**Authentication**:

- **Identity Provider**: Must use trusted identity provider (`isTrusted: true`)
- **MFA Requirement**: Mandatory for all super admin actions
- **MFA Type**: WebAuthn/FIDO2 strongly recommended; TOTP minimum
- **Session Lifetime**: Recommend 8 hours maximum (configurable per tenant)
- **Idle Session Timeout**: Recommend 1 hour (configurable per tenant)

**Authorization**:

- **Role Assignment**: Explicit `Auth System Admin` role required
- **Bypass Mechanism**: Super admins bypass tenant-level permission checks
- **Scope Validation**: Not applicable (full platform access)

**Audit and Monitoring**:

- **Audit Logging**: All actions logged in tenant audit logs
- **Log Retention**: Configurable per tenant (recommend 365+ days for super admin actions)
- **Export Capability**: Super admins can export audit logs (JSON/CSV)
- **Monitoring**: Regular review of super admin actions required

### Decommissioning Super Administrators

**Process**:

1. **Immediate Actions**:
   - Existing super admin removes user from `systemAdmins` group via system configuration
   - This action automatically:
     - Removes `Auth System Admin` role
     - Triggers security alerts to all remaining super admins
     - Creates audit log entry with reason
   - Revoke all active sessions immediately
   - Revoke all active OAuth grants
   - Document decommission reason and approvals

2. **Audit Review**:
   - Export audit logs for departing admin's actions (last 90 days minimum)
   - Review for unauthorized or suspicious activity
   - Pay special attention to:
     - Recent privilege escalations
     - Secret access or modifications
     - Tenant or IdP configuration changes
     - Unusual after-hours activity
   - Document findings in security log
   - Escalate any suspicious findings to security team

3. **Secret Rotation** (if applicable):
   - Rotate shared secrets accessed by departing admin
   - Update API keys and service account credentials
   - Regenerate encryption keys if compromise suspected
   - Review all secrets accessed in last 30 days (audit logs)
   - Priority rotation for sensitive credentials

4. **Account Retention**:
   - Maintain user account for audit trail purposes
   - Account remains disabled but not deleted
   - Remove from all other privileged groups
   - Retain for compliance period (7 years for SOC 2)

5. **Security Notification**:
   - All remaining super admins notified of removal
   - Security team notified if emergency removal
   - Document in change management system

### Super Administrator Security Implications

**Privilege Escalation Risk**:

- Super admins can grant themselves additional roles in any tenant
- Can modify identity provider trust settings
- Can bypass MFA requirements for other users (not recommended)
- **Mitigation**: Regular audit of super admin role assignments, separation of duties

**Data Access**:

- Full read access to all tenant data including:
  - User personal information
  - Authentication credentials (hashed)
  - Audit logs and activity
  - Application configurations
- **Mitigation**: Enforce least privilege, limit number of super admins, comprehensive logging

**Configuration Changes**:

- Can modify tenant security settings globally
- Can enable/disable security features
- Can change authentication flows
- **Mitigation**: Change management process, approval requirements, audit review

**Key Management**:

- Access to encryption keys and key rotation
- Can regenerate tenant JWKS keys
- Manage secret encryption keys
- **Mitigation**: Multi-person integrity for key operations, hardware security module (HSM) integration

---

## Privileged Accounts

### Tenant Administrator Role

**Reference**: See [Admin Roles Overview](../authorization/admin-roles.md) for complete role descriptions and [Users and Groups Management Guide](../guides/users-groups-admin-guide.md) for management procedures.

Tenant Administrators have full administrative access within a single tenant and can:

**Tenant Management**:

- Configure tenant settings (branding, session timeouts, MFA policies)
- Manage login and landing page customization
- Configure allowed origins (CORS)
- Set up email and SMTP settings

**User and Group Management**:

- Create, modify, and delete users within tenant
- Manage user groups and memberships
- Assign roles and permissions to users
- Configure user MFA requirements
- Import/export user data

**Application Management**:

- Create and configure applications (OIDC/SAML clients)
- Manage application secrets and certificates
- Configure application roles and permissions
- Set up API resource servers

**Identity Provider Management**:

- Add and configure identity providers (OAuth, OIDC, SAML, LDAP)
- Cannot mark identity providers as "trusted" (super admin only)
- Configure claims mapping and attribute release
- Test identity provider connections

**Access Control**:

- Manage roles and permissions
- Configure RBAC policies
- Set up access request workflows
- Approve access requests

**Monitoring and Reporting**:

- View audit logs for tenant
- Monitor login events and failures
- Review job logs
- Export reports and logs

### Scoped Administrator Role

**Reference**: See [Access Requests Guide](../guides/access-requests-guide.md) for delegated administration with UMRS roles.

Scoped Administrators have permissions limited to specific resources:

**Scope-Based Permissions**:

- `ADMIN_SCOPE.users`: User management only
- `ADMIN_SCOPE.groups`: Group management only
- `ADMIN_SCOPE.clients`: Application management only
- `ADMIN_SCOPE.providers`: Identity provider management only

**Delegated Administration (UMRS)**:

- User-Managed Role System (UMRS) enables resource-level delegation
- Designated managers can grant specific roles without full admin access
- Supports project-based or team-based access control
- Time-limited grants with expiration

### Creating Privileged Accounts

**Tenant Administrators**:

1. **Via Group Membership** (Recommended):
   - Create or identify tenant admin group
   - Add user to admin group
   - Group membership grants tenant admin capabilities
   - Automatically applies to all tenant resources

2. **Via Permission Assignment**:
   - Assign tenant-admin-capable permissions directly to user
   - Requires specific permission scopes (e.g., `auth.admin.*`)
   - More granular but harder to maintain

**Scoped Administrators**:

1. **Direct Scope Assignment**:
   - Assign specific `ADMIN_SCOPE.*` permissions to user
   - User gains admin capabilities only for that resource type
   - Example: Assign `ADMIN_SCOPE.users` for user management only

2. **UMRS Role Grant**:
   - Create UMRS role scoped to specific resource
   - Designate manager group
   - Managers grant role to users as needed
   - Supports time-limited access

### Privileged Account Security Settings

**Authentication**:

- **Identity Provider**: Trusted identity provider recommended but not required
- **MFA Requirement**: Strongly recommended, can be enforced at tenant level
- **Session Lifetime**: Default 8 hours, configurable per tenant
- **Idle Session Timeout**: Default 1 hour, configurable per tenant
- **Refresh Token**: Configurable lifetime (default 30 days)

**Authorization**:

- **Permission Model**: Role-based access control (RBAC)
- **Scope Enforcement**: API-level scope validation
- **Middleware Override**: Tenant admins can bypass certain scope errors (delegated tenants only)
- **Audit Trail**: All administrative actions logged

**Tenant-Level Security Settings**:

Tenant administrators can configure these security settings within their tenant:

1. **Multi-Factor Authentication**:
   - Enable Admin MFA: Require MFA for all tenant admin actions
   - Always Require MFA When Set: Enforce MFA for users with MFA enrolled
   - Enable TOTP: Enable TOTP MFA (also controls reset capabilities)
   - TOTP Suspension Threshold: Failed TOTP attempts before suspension
   - TOTP Lockout Threshold: Failed TOTP attempts before lockout
   - TOTP Suspension Period: Duration of TOTP suspension

2. **Session Management**:
   - Session Lifetime: Maximum authenticated session duration
   - Idle Session Lifetime: Inactivity timeout
   - Refresh Token Lifetime: Refresh token validity period

3. **Login Security**:
   - Login page customization (disclaimer, warnings)
   - Background images and branding
   - Do not skip login page: Force login page display

4. **User Security**:
   - User alert settings: Email notifications for login events
   - Account expiration policies
   - Password policy enforcement (via identity provider)

5. **Application Security**:
   - Allowed origins (CORS): Restrict application origins
   - Allow-list enabled: Enforce allowed origins
   - Discovery mode: Automatic origin discovery
   - Client authentication methods (client_secret, private_key_jwt, etc.)

**Reference**: See [Tenant Administrator Guide](../guides/tenant-admin-guide.md) for complete configuration details.

### Decommissioning Privileged Accounts

**Tenant Administrators**:

1. **Remove from Admin Group**:
   - Remove user from tenant admin group
   - User immediately loses tenant admin capabilities
   - Existing sessions remain valid until expiration

2. **Revoke Sessions**:
   - Navigate to Monitoring > Sessions
   - Filter by user
   - Revoke all active sessions
   - User must re-authenticate with reduced privileges

3. **Audit Review**:
   - Export audit logs for departing admin (last 90 days)
   - Review administrative actions
   - Document any suspicious activity

**Scoped Administrators**:

1. **Remove Scope Assignments**:
   - Edit user permissions
   - Remove `ADMIN_SCOPE.*` permissions
   - Save changes

2. **Revoke UMRS Grants**:
   - Navigate to Access Requests > Access Grants
   - Filter by user
   - Revoke all UMRS role assignments

### Privileged Account Security Implications

**Tenant Data Access**:

- Full read/write access to all tenant data
- Can export user information
- Can view authentication logs
- **Mitigation**: Regular audit reviews, data classification policies

**Identity Provider Configuration**:

- Can add new identity providers
- Can modify claims mapping
- Cannot mark IdPs as "trusted" (super admin only)
- **Mitigation**: Review IdP changes, alert on new IdP additions

**Application Management**:

- Create applications with any redirect URIs
- Generate client secrets
- Configure token lifetimes
- **Mitigation**: Validate redirect URIs, monitor client creation, rotate secrets regularly

**User Impersonation Risk**:

- Can create users with any email address
- Can reset user passwords (via IdP)
- Can modify user roles and permissions
- **Mitigation**: Email verification, separate admin accounts, audit logging

---

## Security Settings Reference

### Authentication Settings

#### Multi-Factor Authentication (MFA)

**System-Level Settings** (Super Admin Only):

- **Identity Provider MFA Type**: Configure MFA method at IdP level (TOTP, WebAuthn, Email, SMS)
- **AAL Override**: Set Authentication Assurance Level (AAL1, AAL2, AAL3) per identity provider
- **Trusted Identity Provider**: Mark IdP as trusted (enables higher privileges)

**Tenant-Level Settings** (Tenant Admin):

- **Enable Admin MFA**: Require MFA for all tenant admin actions
- **Always Require MFA When Set**: Enforce MFA for users with MFA enrolled
- **TOTP Suspension Threshold**: Failed TOTP attempts before suspension (default: 5)
- **TOTP Lockout Threshold**: Failed TOTP attempts before lockout (default: 10)
- **TOTP Suspension Period**: Duration of TOTP suspension in minutes (default: 2)
- **Enable TOTP**: Enable TOTP MFA, also controls reset capabilities (`enableTotp`)

**User-Level Settings**:

- **MFA Enrollment**: Users can enroll TOTP or WebAuthn devices
- **Multiple Devices**: Users can register multiple WebAuthn devices (backup)
- **Recovery Codes**: Not currently supported (use TOTP reset instead)

**Security Implications**:

- MFA significantly reduces account compromise risk
- WebAuthn/FIDO2 provides strongest protection (phishing-resistant)
- TOTP is acceptable minimum, SMS/Email not recommended
- Strike count balances security with usability (too low = lockouts, too high = brute force risk)

#### Session Management

**Tenant-Level Settings**:

- **Session Lifetime**: Maximum authenticated session duration (default: 8 hours)
  - Recommend shorter for admin contexts (2-4 hours)
  - Balance security with usability
- **Idle Session Lifetime**: Inactivity timeout (default: 1 hour)
  - Recommend 15-30 minutes for high-security tenants
  - Prevents abandoned session exploitation
- **Refresh Token Lifetime**: Long-lived token validity (default: 30 days)
  - Allows applications to refresh access without re-authentication
  - Recommend shorter for sensitive applications (7-14 days)

**Security Implications**:

- Shorter session lifetimes reduce exposure window for stolen sessions
- Idle timeouts protect against unattended workstation attacks
- Refresh tokens enable convenience but extend access duration
- Revoked sessions do not affect refresh tokens (must revoke grants separately)

#### Identity Provider Security

**Super Admin Configuration**:

- **Is Trusted**: Enables elevated privileges for users from this IdP
  - Only enable for enterprise-managed, well-secured IdPs
  - Trusted IdPs can be used for admin accounts
  - Non-trusted IdPs have restricted access
- **AAL Override**: Set authentication assurance level
  - AAL1: Single-factor authentication
  - AAL2: Multi-factor authentication
  - AAL3: Hardware-based cryptographic authentication
- **MFA Type**: Specify MFA method for this IdP
  - Informs authorization decisions
  - Should match actual IdP configuration

**Tenant Admin Configuration**:

- **Claims Mapping**: Map IdP claims to user attributes
  - Use claims scripting for complex transformations
  - Validate mapped claims (don't trust blindly)
- **Scopes**: Request appropriate scopes from IdP
  - Minimize data requested (privacy principle)
  - Request only necessary user information

**Security Implications**:

- Compromised IdP = compromised Authifi tenant
- Misconfigured claims mapping can grant unintended access
- Claims scripting vulnerabilities can be exploited
- Always validate IdP authenticity before trusting

### Authorization Settings

#### Role-Based Access Control (RBAC)

**Client Roles** (Application-specific):

- Assigned to users or groups
- Included in access tokens for application consumption
- Can be hierarchical (role inheritance)

**API Roles** (Resource server-specific):

- Control access to API resources
- Support fine-grained permissions
- Composable (user can have multiple API roles)

**Permission Model**:

- Permissions are granular capabilities (e.g., `users.create`, `clients.update`)
- Roles are collections of permissions
- Users/groups can have roles or direct permission assignments

**Scope-Based Access Control**:

- OAuth scopes control API access
- Applications request scopes, users grant consent
- Scope enforcement at API gateway layer

**Security Implications**:

- Least privilege: Grant minimum necessary roles/permissions
- Regular review: Audit role assignments quarterly
- Separation of duties: Prevent conflicting role combinations
- Inheritance complexity: Document role hierarchies clearly

#### Delegated Administration (UMRS)

**User-Managed Role System**:

- Enables non-admin users to grant specific roles
- Manager group controls who can grant role
- Supports time-limited grants with expiration
- Resource-scoped (not tenant-wide)

**Configuration**:

- **UMRS Role**: Defines what access is granted
- **Manager Group**: Who can grant this role
- **Resource Server**: What API/resource this controls
- **Allow Extension Requests**: Users can request to extend grants

**Security Implications**:

- Delegation introduces additional attack surface
- Manager group compromise = unauthorized grants
- Monitor manager actions via audit logs
- Limit UMRS to appropriate use cases (projects, teams)

### Data Protection Settings

#### Encryption

**In Transit**:

- TLS 1.2+ required for all API communications
- HTTPS enforced for web UI
- Certificate validation required
- Strong cipher suites only

**At Rest**:

- Database encryption via database provider
- Secrets encrypted with tenant-specific keys
- JWKS private keys encrypted
- Session data encrypted in cache

**Key Management**:

- Tenant-specific encryption keys for secrets
- JWKS key rotation capability
- Key rotation procedures documented
- HSM support for high-security deployments

#### Secret Management

**Secret Types**:

- **Sensitive**: Write-only, cannot be retrieved (recommended for credentials)
- **Variable**: Readable, encrypted at rest (for non-sensitive config)

**Secret Configuration**:

- **Name**: Unique identifier
- **Expiration**: Recommend 90-180 days for credentials
- **Provider**: Local (default) or external (AWS Secrets Manager, etc.)
- **System Shared**: Available to all tenants (super admin only)

**Security Implications**:

- Sensitive secrets provide strongest protection (one-way write)
- Expired secrets continue working (manual rotation required)
- Secret access logged in audit trail
- Compromise requires immediate rotation

**Reference**: See [Resources and Tools Guide](../guides/resources-tools-guide.md) for secret management procedures.

### Monitoring and Audit Settings

#### Audit Logging

**What is Logged**:

- All administrative actions (create, update, delete)
- Authentication events (login, logout, MFA)
- Authorization decisions (grants, denials)
- Configuration changes (tenants, apps, IdPs)
- Secret access and modifications
- **Super admin group changes** (additions/removals with automatic security alerts)
- Temporary access grants and expirations

**Audit Log Fields**:

- **Action**: Type of change
- **Target**: Resource affected
- **Actor**: User who performed action
- **Timestamp**: When action occurred
- **Old Entity**: State before change
- **New Entity**: State after change
- **Comments**: Admin-provided justification

**Retention**:

- Configurable per tenant (default: 90 days)
- Recommend 365+ days for compliance
- Trim jobs can automatically clean up old logs
- Export logs before trimming for long-term retention

**Security Implications**:

- Audit logs are critical for forensics and compliance
- Protect audit logs from tampering (append-only)
- Regular review (weekly for high-privilege actions)
- Alert on suspicious patterns (bulk deletions, after-hours admin)

**Reference**: See [Monitoring and Logging Guide](../guides/monitoring-guide.md) for audit log usage.

#### Event Logging

**Login Events**:

- Successful logins
- Failed login attempts
- MFA verification (success/failure)
- Password resets
- Account lockouts

**Event Fields**:

- Email, username, IP address, user agent
- Client (application) name
- Identity provider used
- Timestamp

**Security Monitoring**:

- Failed login detection (brute force)
- Unusual access patterns (geography, time)
- Multiple concurrent sessions
- Rapid password changes

**Security Alerts** (Automatic):

- **Super Admin Changes**: Triggered when user added to or removed from `systemAdmins` group
  - All existing super admins receive immediate notification
  - Includes: user email, action (add/remove), granting admin, timestamp, temporary flag
  - 24-hour review window for challenge
- **Privileged Action Alerts**: Configurable alerts for high-risk operations
  - Identity provider trust changes
  - Tenant deletion or suspension
  - Bulk user modifications
  - Secret key rotation

**Reference**: See [Monitoring and Logging Guide](../guides/monitoring-guide.md) for event log usage.

---

## Secure Configuration Baseline

### Default Security Settings

The Authifi service provisions new tenants with the following secure defaults:

**Authentication**:

- Session Lifetime: 8 hours
- Idle Session Timeout: 1 hour
- Refresh Token Lifetime: 30 days
- Enable Admin MFA: false (recommend enabling immediately)
- Always Require MFA When Set: true
- TOTP Suspension Threshold: 5
- TOTP Lockout Threshold: 10
- TOTP Suspension Period: 2 minutes

**Authorization**:

- Allowed Origins: Empty (must be explicitly configured)
- Allow-list Enabled: true
- Discovery Mode: false

**Audit**:

- Audit Logging: Enabled
- Log Retention: 90 days (recommend increasing)
- Event Logging: Enabled

**Email**:

- Use Default Email Settings: true
- Email Branding: Default tenant branding

### Recommended Secure Configuration

**For Production Tenants**:

1. **Authentication**:
   - Enable Admin MFA: true
   - Always Require MFA When Set: true
   - Session Lifetime: 4-8 hours (depending on security requirements)
   - Idle Session Timeout: 30-60 minutes
   - Refresh Token Lifetime: 7-14 days (reduce for sensitive applications)

2. **Identity Providers**:
   - Use trusted identity providers for admin accounts
   - Enable MFA at identity provider level
   - Configure AAL2 or AAL3 for sensitive applications
   - Validate claims mapping carefully

3. **Authorization**:
   - Apply least privilege principle
   - Regular role/permission audits (quarterly)
   - Document role assignments and justifications
   - Use UMRS for delegated administration where appropriate

4. **Monitoring**:
   - Audit Log Retention: 365+ days
   - Enable user alert settings for suspicious activity
   - Regular review of admin actions (weekly)
   - Export audit logs for long-term storage

5. **Applications**:
   - Use PKCE for all OAuth flows (when supported)
   - Require client authentication for confidential clients
   - Set short access token lifetimes (15-60 minutes)
   - Validate redirect URIs strictly
   - Rotate client secrets quarterly

6. **Secrets**:
   - Use Sensitive type for all credentials
   - Set expiration (90-180 days)
   - Rotate immediately on compromise
   - Limit secret access to necessary jobs only

### Configuration Comparison

**API Endpoint**: `GET /auth/admin/tenants/{tenantId}`

Returns the tenant configuration including security-relevant settings:

```json
{
  "id": "example-tenant",
  "sessionLifetime": 28800,
  "idleSessionLifetime": 3600,
  "refreshTokenLifetime": 2592000,
  "enableAdminMfa": true,
  "requireMfaWhenSet": true,
  "totpSuspensionThreshold": 5,
  "totpLockoutThreshold": 10,
  "totpSuspensionPeriod": 2,
  "allowListEnabled": true,
  "discoveryMode": false,
  "customTrimJobSettings": {
    "ageLimit": 365
  }
}
```

**Comparison Process**:

1. Retrieve current configuration via API
2. Compare against recommended baseline
3. Identify deviations
4. Assess security implications of deviations
5. Remediate or document exceptions

### Configuration Export

**Tenant Configuration Export**:

- Navigate to Tenant > Settings > Export
- Exports complete tenant configuration as JSON
- Includes all security settings, applications, IdPs, roles, permissions
- Can be imported to another tenant or version controlled

**Audit Log Export**:

- Navigate to Monitoring > Audit Logs > Export Audit Logs
- Choose JSON (machine-readable) or CSV (human-readable)
- Specify record limit (up to 1,000,000)
- Automated exports via API:
  - `GET /auth/admin/tenants/{tenantId}/audit/export/csv`
  - `GET /auth/admin/tenants/{tenantId}/audit/export/json`

**Event Log Export**:

- Navigate to Monitoring > Event Logs > Export Login Events
- Available formats: JSON, CSV
- API endpoint: `GET /loginEvent/file/export` (tenant-scoped)

### API-Based Configuration Management

**Security Configuration API**:

All security settings are accessible via REST API:

**Authentication**:

```
GET    /auth/admin/tenants/{tenantId}
PUT    /auth/admin/tenants/{tenantId}
PATCH  /auth/admin/tenants/{tenantId}
```

**Identity Providers**:

```
GET    /auth/admin/tenants/{tenantId}/identity-providers
POST   /auth/admin/tenants/{tenantId}/identity-providers
PUT    /auth/admin/tenants/{tenantId}/identity-providers/{idpId}
DELETE /auth/admin/tenants/{tenantId}/identity-providers/{idpId}
```

**Applications**:

```
GET    /auth/admin/tenants/{tenantId}/clients
POST   /auth/admin/tenants/{tenantId}/clients
PUT    /auth/admin/tenants/{tenantId}/clients/{clientId}
DELETE /auth/admin/tenants/{tenantId}/clients/{clientId}
```

**Users and Groups**:

```
GET    /auth/admin/tenants/{tenantId}/users
POST   /auth/admin/tenants/{tenantId}/users
PUT    /auth/admin/tenants/{tenantId}/users/{userId}
DELETE /auth/admin/tenants/{tenantId}/users/{userId}
```

**Complete API Documentation**: Available at `/auth/docs` (Swagger/OpenAPI)

---

## Security Best Practices

### Administrative Account Security

**Super Administrators**:

1. **Minimize Count**:
   - Limit super admin accounts to 2-5 individuals
   - Document business justification for each
   - Review quarterly and remove unnecessary accounts
   - Use temporary super admin access for short-term needs

2. **Dedicated Accounts**:
   - Separate admin accounts from day-to-day user accounts
   - Use naming convention (e.g., `admin-firstname.lastname@domain`)
   - Never share admin credentials
   - Individual accountability required

3. **Strong Authentication**:
   - Require trusted identity provider
   - Enforce hardware-based MFA (WebAuthn/FIDO2)
   - Minimum: TOTP with backup device
   - Never use SMS or email as MFA

4. **Session Security**:
   - Use short session lifetimes (2-4 hours)
   - Always log out when done
   - Never save admin passwords in browser
   - Use password manager for admin credentials

5. **Access from Secure Locations**:
   - Use company-managed devices only
   - Avoid public WiFi for admin access
   - Consider IP allowlist for admin access
   - Use VPN when accessing remotely

6. **Security Alert Monitoring**:
   - All super admins must monitor security alerts
   - Respond to `systemAdmins` group change alerts within 24 hours
   - Challenge unexplained admin additions immediately
   - Maintain current contact information for alerts

7. **Change Management**:
   - Document all super admin additions/removals
   - Require approval from at least one other super admin
   - Use temporary access for auditors and contractors
   - Set clear expiration dates for temporary access

**Tenant Administrators**:

1. **Group-Based Assignment**:
   - Use admin groups instead of direct permission assignments
   - Easier to audit and manage
   - Consistent permissions across admins

2. **Least Privilege**:
   - Use scoped admin roles where possible
   - Don't grant tenant admin for single-resource management
   - Consider UMRS for delegated administration

3. **Regular Reviews**:
   - Audit admin group membership quarterly
   - Remove departing employees immediately
   - Review admin actions in audit logs monthly

4. **MFA Enforcement**:
   - Enable "Enable Admin MFA" tenant setting
   - Require MFA enrollment for all admins
   - Monitor for MFA bypass attempts

### Identity Provider Security

1. **Trusted Identity Providers**:
   - Only mark IdPs as "trusted" if:
     - IdP is enterprise-managed and secured
     - IdP enforces strong authentication
     - IdP has comprehensive audit logging
     - IdP vendor is reputable
   - Never trust public IdPs (Google, Facebook) for admin access

2. **Claims Validation**:
   - Validate all mapped claims
   - Don't trust user-supplied claims for authorization
   - Use claims scripting defensively
   - Sanitize inputs in claims scripts

3. **MFA at IdP Level**:
   - Configure MFA at identity provider
   - Don't rely solely on Authifi service MFA
   - Prefer IdP-enforced MFA for admins

4. **Regular Testing**:
   - Test IdP connections regularly
   - Monitor for authentication failures
   - Alert on IdP configuration changes

### Application Security

1. **Client Authentication**:
   - Use `private_key_jwt` for confidential clients (most secure)
   - `client_secret_post` is acceptable minimum
   - Never use `none` for production applications
   - Rotate client secrets quarterly

2. **Redirect URI Validation**:
   - Explicitly list all valid redirect URIs
   - Use exact matches (no wildcards)
   - HTTPS required for production
   - Monitor for redirect URI changes

3. **Token Lifetimes**:
   - Access tokens: 15-60 minutes
   - Refresh tokens: 7-14 days (sensitive apps), 30 days (standard)
   - ID tokens: Same as access tokens
   - Shorter lifetimes reduce exposure window

4. **PKCE Enforcement**:
   - Require PKCE for all OAuth flows
   - Prevents authorization code interception
   - Mandatory for SPAs and mobile apps

### Secret Management

1. **Secret Classification**:
   - Always use Sensitive type for:
     - Passwords
     - API keys and tokens
     - Private keys
     - Database credentials
   - Use Variable type only for non-sensitive config

2. **Expiration Policies**:
   - Set expiration for all secrets (90-180 days)
   - Shorter for high-privilege secrets (30-90 days)
   - Calendar reminders for rotation
   - Automate rotation where possible

3. **Access Control**:
   - Limit secret access to necessary jobs only
   - Don't expose secrets to users
   - Audit secret access regularly
   - Alert on secret modifications

4. **Rotation Procedures**:
   - Rotate immediately on compromise
   - Test after rotation (verify applications still work)
   - Document rotation procedures
   - Maintain rotation audit trail

### Monitoring and Incident Response

1. **Continuous Monitoring**:
   - Review audit logs weekly (admin actions)
   - Monitor event logs daily (failed logins)
   - Set up alerts for:
     - Multiple failed logins
     - New admin account creation
     - Identity provider changes
     - After-hours admin activity

2. **Incident Response**:
   - Document incident response procedures
   - Include Authifi service in IR playbooks
   - Know how to:
     - Revoke sessions immediately
     - Remove admin privileges
     - Export audit logs
     - Rotate secrets
     - Lock accounts

3. **Regular Reviews**:
   - **Daily**: Failed login monitoring
   - **Weekly**: Admin action audit
   - **Monthly**: Application and IdP configuration review
   - **Quarterly**: Comprehensive access review (all admins, roles, permissions)
   - **Annually**: Security posture assessment

4. **Audit Log Protection**:
   - Export logs regularly (weekly)
   - Store exports securely off-platform
   - Long retention period (7 years for SOC 2)
   - Protect from tampering (write-once storage)

---

## Compliance and Audit

### FedRAMP Compliance

This document addresses the following FedRAMP Recommended Secure Configuration requirements:

**FRR-RSC-01**: Top-Level Administrative Accounts Guidance

- Covered in: [Top-Level Administrative Accounts](#top-level-administrative-accounts)
- Addresses: Access, configuration, operation, and decommission procedures

**FRR-RSC-02**: Top-Level Administrative Accounts Security Settings

- Covered in: [Super Administrator Security Settings](#super-administrator-security-settings)
- Addresses: All super admin-only security settings and their implications

**FRR-RSC-03**: Privileged Accounts Security Settings

- Covered in: [Privileged Account Security Settings](#privileged-account-security-settings)
- Addresses: Tenant admin and scoped admin settings and implications

**FRR-RSC-04**: Secure Defaults on Provisioning

- Covered in: [Secure Configuration Baseline](#secure-configuration-baseline)
- All security settings default to secure values on tenant creation

**FRR-RSC-05**: Comparison Capability

- Covered in: [Configuration Comparison](#configuration-comparison)
- API endpoint provides current configuration for comparison

**FRR-RSC-06**: Export Capability

- Covered in: [Configuration Export](#configuration-export)
- Tenant config export via UI and API in JSON format

**FRR-RSC-07**: API Capability

- Covered in: [API-Based Configuration Management](#api-based-configuration-management)
- Complete REST API for all security settings

**FRR-RSC-08**: Machine-Readable Guidance

- This document is version controlled and can be published in markdown format
- Security configuration available via API in JSON format

**FRR-RSC-09**: Publish Guidance

- This document is publicly available in the Authifi Documentation repository at: https://authifi.pages.dev/security/recommended-secure-configuration

**FRR-RSC-10**: Versioning and Release History

- Document version tracked in Git
- Security setting changes tracked in audit logs
- Release notes document security-related changes

### Audit Checklists

**Super Administrator Audit** (Quarterly):

- [ ] List all members of `systemAdmins` group
- [ ] Verify business justification for each permanent member
- [ ] Review all temporary super admin grants and expirations
- [ ] Confirm MFA enrollment (TOTP or WebAuthn)
- [ ] Review super admin actions in last 90 days
- [ ] Check for suspicious activity
- [ ] Verify no unauthorized privilege escalations
- [ ] Review all `systemAdmins` group change alerts
- [ ] Confirm all additions were properly approved and documented
- [ ] Verify security alert system is functioning
- [ ] Confirm all super admins are receiving and responding to alerts
- [ ] Confirm compliance with security policies
- [ ] Document findings and remediation

**Tenant Administrator Audit** (Quarterly):

- [ ] List all tenant admin group memberships
- [ ] Verify admin assignments are current
- [ ] Check for departing employees still in admin groups
- [ ] Review tenant admin actions in last 90 days
- [ ] Verify MFA enrollment for admins
- [ ] Confirm no excessive permission grants
- [ ] Review UMRS manager assignments
- [ ] Document findings

**Identity Provider Audit** (Monthly):

- [ ] List all configured identity providers
- [ ] Verify "trusted" flag is appropriate for each
- [ ] Check MFA configuration at IdP level
- [ ] Review claims mapping and scripting
- [ ] Test IdP connections
- [ ] Monitor for authentication failures
- [ ] Review IdP change history
- [ ] Document any issues

**Application Audit** (Monthly):

- [ ] List all applications (clients)
- [ ] Review redirect URIs for validity
- [ ] Check for weak client authentication
- [ ] Verify token lifetimes are appropriate
- [ ] Review grant types enabled
- [ ] Check for unused applications
- [ ] Review client secret rotation dates
- [ ] Document findings

**Secret Audit** (Monthly):

- [ ] List all secrets
- [ ] Check expiration dates
- [ ] Identify secrets nearing expiration
- [ ] Plan rotation for expiring secrets
- [ ] Review secret access logs
- [ ] Verify secret types (Sensitive vs Variable)
- [ ] Check for unused secrets
- [ ] Document rotation actions

### Compliance Evidence

**For SOC 2 / FedRAMP Audits**:

1. **Access Control** (AC):
   - Admin role assignments
   - MFA enrollment records
   - Session management configuration
   - Least privilege evidence

2. **Audit and Accountability** (AU):
   - Audit log exports (comprehensive)
   - Event log exports (login activity)
   - Admin action reviews
   - Incident investigation logs

3. **Configuration Management** (CM):
   - Baseline configuration documentation
   - Configuration change logs
   - Security setting exports
   - Change approval records

4. **Identification and Authentication** (IA):
   - Identity provider configurations
   - MFA enforcement evidence
   - Authentication policy documentation
   - Trusted IdP justifications

5. **System and Communications Protection** (SC):
   - TLS/HTTPS enforcement
   - Encryption at rest configuration
   - Key management procedures
   - Secret rotation records

---

## Document Maintenance

**Review Schedule**:

- Quarterly: Technical accuracy review
- Annually: Comprehensive security review
- As needed: After major product changes or security incidents

**Change History**:

| Version | Date       | Changes         | Author                  |
| ------- | ---------- | --------------- | ----------------------- |
| 1.0     | 2025-12-12 | Initial release | Authifi Documentation Team |

**Feedback**:

- Submit documentation feedback via: [GitHub Issues](https://github.com/AxleResearch/auth-monorepo/issues)
- Report security concerns via: security@axleinfo.com
- Request clarifications via: support@axleinfo.com

---

## Additional Resources

**Internal Documentation**:

- [Super Admin Access Requirements](../authorization/super-admin-access.md) - Complete list of system-admin-only operations
- [Admin Roles Overview](../authorization/admin-roles.md) - Role hierarchy and enforcement
- [Tenant Administrator Guide](../guides/tenant-admin-guide.md) - Tenant settings configuration
- [SSO Integration Guide](../guides/sso-integration-guide.md) - Application and IdP setup
- [Users and Groups Management Guide](../guides/users-groups-admin-guide.md) - User/group management
- [Access Requests Guide](../guides/access-requests-guide.md) - Delegated administration and UMRS
- [Monitoring and Logging Guide](../guides/monitoring-guide.md) - Audit logs and security monitoring
- [Resources and Tools Guide](../guides/resources-tools-guide.md) - Templates, secrets, and jobs

**External Resources**:

- FedRAMP Recommended Secure Configuration: https://www.fedramp.gov/docs/rev5/recommended-secure-configuration/
- NIST SP 800-63B: Digital Identity Guidelines (Authentication)
- OWASP Authentication Cheat Sheet
- OAuth 2.0 Security Best Current Practice (BCP)

**API Documentation**:

- Authifi API Reference: `/auth/docs` (Swagger/OpenAPI)
- Available at runtime on Authifi service instance

---

**Classification**: Public  
**Distribution**: Unlimited  
**Compliance**: FedRAMP Rev5 Recommended Secure Configuration
