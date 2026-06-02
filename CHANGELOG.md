# authifi-docs

## 1.2.0

### Minor Changes

- 22d4dc6: Add Authifi Identity Broker feature list page covering all 27 feature categories including authentication protocols, MFA, RBAC, NHE delegation tokens, GA4GH Passport, FedRAMP High compliance, and standards compliance.
- ea66ae4: Add NHE delegated tokens guide for LLM agents and automated pipelines
- 810813f: Add agent-readiness static assets, WebMCP tools, and Cloudflare header rules for AI discovery.

### Patch Changes

- 4d585ba: Add GitHub Actions workflow to automate changeset releases
- dbaba76: Update release workflow to apply version bumps automatically
- 451e591: Clarify App Roles vs API Roles terminology and add access_roles scope documentation
- 2b09d92: Fix release workflow failing on protected main branch by using the changesets/action to open a "Version Packages" PR instead of pushing version bumps directly to main.
- 7a98237: Sync feature list with upstream accuracy corrections: scope MFA enforcement and standards claims to what is implemented, correct emergency-MFA-disable and session-extension behavior, reflect tenant/app-level IdP MFA settings, and add notes for delegated-scope and privileged-entity items tracked in LSA-9041/LSA-9042.
- 284bc6f: Correct feature list accuracy: clarify FedRAMP High authorization is as a supporting service under the Palantir Federal Cloud Service (PFCS-SS, FR2315464863), remove unsupported SAML Artifact binding, reframe recertification as access lifecycle management, correct CLI capabilities, and remove unimplemented notification preferences and session device tracking claims.

## 1.1.0

### Minor Changes

- 65a716d: Add changesets for versioning and changelog management
