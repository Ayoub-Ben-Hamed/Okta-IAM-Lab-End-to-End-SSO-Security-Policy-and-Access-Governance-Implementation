# Phase 6: Provisioning & Lifecycle Management Architecture

## Overview

Single Sign-On authenticates identity at the perimeter, but lifecycle provisioning governs whether an account exists within downstream systems and whether access terminates immediately upon offboarding. Leaving accounts un-deprovisioned after role termination represents a primary access governance vulnerability.

## Implementation: SCIM-Provisioning/Deprovisioning Bridge

Rather than testing lifecycle provisioning against a generic, pre-packaged reference application, I implemented automated provisioning and deprovisioning through a dedicated, custom-built project: [SCIM-Provisioning/Deprovisioning Bridge](https://github.com/ayoub-benhamed/SCIM-Provisioning-Deprovisioning-Bridge).

The **SCIM-Provisioning/Deprovisioning Bridge** is a standalone SCIM 2.0 compliant server providing RESTful endpoints (`/Users`, `/Schemas`, `/ServiceProviderConfig`) backed by an independent database schema. Connected to Okta via a SCIM 2.0 integration with HTTP Header Authentication, the server processes real-time JSON payloads sent by Okta's SCIM client to handle account creation, attribute synchronization, and deactivation. By inspecting inbound HTTP requests (POST, GET, PUT, PATCH), the bridge executes lifecycle state transitions programmatically and guarantees audit compliance across target applications.

## Key Architectural Concepts

### SSO versus Lifecycle Management

- **Single Sign-On (SSO)**: Governs authentication entry points. It answers whether a user holds valid credentials to establish a session.
- **Provisioning**: Governs downstream data existence. It guarantees that an authorized identity possesses an active profile, correct entitlements, and assigned roles within target applications before attempting login.
- **Deprovisioning**: Revokes downstream access. It terminates target system accounts immediately when an employee departs or changes roles, preventing orphaned accounts.

### Account Deactivation versus Session Revocation

A comprehensive offboarding strategy on Okta Identity Engine addresses two separate state components:

1. **Identity Deactivation**: Transitioning an account from Active to Deactivated halts future authentication at the Identity Provider and sends SCIM deactivation directives to connected target apps.
2. **Session Revocation (Universal Logout)**: Clearing active sessions revokes currently valid session cookies, access tokens, and refresh tokens. Without session clearing, a revoked user could continue interacting with an existing browser session until token expiration. Combining lifecycle deactivation with immediate session clearing closes this gap completely.
