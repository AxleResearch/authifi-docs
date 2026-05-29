# Authifi Service - Security Documentation

## Overview

This directory contains security documentation for the Authifi service, including guidance for securely configuring and operating administrative accounts in accordance with FedRAMP Recommended Secure Configuration requirements.

---

## Security Admin Guide

**Document**: [Security Admin Guide](./security-admin-guide.md)

The Security Admin Guide provides comprehensive guidance for setting up, configuring, operating, and decommissioning administrative accounts within the Authifi service.

### Administrative Account Types Covered

| Account Type              | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| **Super Administrators**  | Top-level administrative accounts with platform-wide control |
| **Tenant Administrators** | Privileged accounts with full control within a single tenant |
| **Delegated Admins**      | Users with specific elevated permissions for particular resources |

### Key Topics

- Administrative account role definitions (permissions, actions, operations)
- Account lifecycle procedures (setup, MFA requirements, configuration, decommissioning)
- Security settings reference tables with recommended values
- FedRAMP compliance mapping

---

## Encryption Key Security and Backup

**Document**: [Encryption Key Security and Backup](./encryption-key-security.md)

Comprehensive documentation of Authifi's encryption key architecture and backup procedures:

- Envelope encryption key hierarchy (DEKs and KEKs)
- KEK provider modes: `none`, `local` (SecureCrypto), and `vault-transit` (HashiCorp Vault Transit)
- Cryptographic details (AES-256-CBC, scrypt, HMAC)
- DEK migration services (backfill and unwrap)
- Shamir's Secret Sharing in the Super Admin Console for key backup and recovery
- Key ceremony procedures and disaster recovery scenarios

---

## Recommended Secure Configuration

**Document**: [Recommended Secure Configuration](./recommended-secure-configuration.md)

Detailed security configuration guidance covering:

- Authentication and session management settings
- Identity provider security configuration
- Role-based access control (RBAC) settings
- Data protection and encryption
- Monitoring and audit configuration
- Security best practices
- Compliance checklists

---

## Quick Reference

### For Super Administrators

- [Super Administrator Role Definition](./security-admin-guide.md#super-administrators-top-level-administrative-accounts)
- [Super Administrator Lifecycle](./security-admin-guide.md#super-administrator-lifecycle)
- [Super Administrator Security Settings](./security-admin-guide.md#super-administrator-security-settings-reference)

### For Tenant Administrators

- [Tenant Administrator Role Definition](./security-admin-guide.md#tenant-administrators-privileged-accounts)
- [Tenant Administrator Lifecycle](./security-admin-guide.md#tenant-administrator-lifecycle)
- [Tenant Administrator Security Settings](./security-admin-guide.md#tenant-administrator-security-settings-reference)

### For Delegated Admins

- [Delegated Admin Role Definition](./security-admin-guide.md#delegated-admins)
- [Admin Scope Permissions](./security-admin-guide.md#admin-scope-capabilities)
- [UMRS Delegated Administration](./security-admin-guide.md#user-managed-role-system-umrs)

---

## Related Documentation

- [Tenant Administrator Guide](../guides/tenant-admin-guide.md) - UI-focused tenant settings guide
- [Super Admin Access Requirements](../authorization/super-admin-access.md) - Complete list of super-admin-only operations
- [Admin Roles Overview](../authorization/admin-roles.md) - Technical role implementation details

---

**Classification**: Public  
**Last Updated**: 2026-01-22
