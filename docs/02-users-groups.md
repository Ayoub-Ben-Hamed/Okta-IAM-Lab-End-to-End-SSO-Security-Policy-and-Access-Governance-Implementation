# Phase 2: Users & Groups Simulation

## Overview

In this phase, I populated the directory with simulated enterprise personas, evaluated manual user management, configured inbound profile mastering from BambooHR, and automated group memberships using Okta Expression Language.

## Manual User Lifecycle Simulation

I started by provisioning user accounts manually to examine the standard onboarding sequence and end-user activation workflow.

Prior to creating new accounts, the directory contained only the initial administrator profile.

![Directory People Before User Add](../images/02-users-groups/directory-people-before-user-add.png)

I opened **Directory > People > Add Person** to create an initial user profile with standard attributes (First name, Last name, Username, Primary email).

![Add Person Form](../images/02-users-groups/add-person-form.png)

Once submitted, the user was added to the directory in a staged/pending state awaiting account activation.

![Directory People User Created](../images/02-users-groups/directory-people-user-created.png)

From the end-user perspective, account activation required three steps:

1. Logging in using the temporary credentials issued by the administrator, followed immediately by setting a permanent password.

![User Activation Set Password](../images/02-users-groups/user-activation-set-password.png)

2. Enrolling an initial multi-factor authenticator, such as Okta Verify on a mobile device.

![User Activation MFA Enrollment](../images/02-users-groups/user-activation-mfa-enrollment.png)

3. Completing authentication and landing on the Okta End-User Dashboard.

![End-User Dashboard Access](../images/02-users-groups/end-user-dashboard-first-access.png)

Following this process, I populated additional test users across different business roles.

![Simulated Users List](../images/02-users-groups/directory-people-simulated-users-list.png)

### Manual Population of Custom Attributes

Because custom schema attributes do not appear on the standard user creation modal, setting them manually requires opening each profile individually:

1. Navigate to **Directory > People** and select the user.
2. Click the **Profile** tab, then select **Edit**.

![User Profile Tab Edit](../images/02-users-groups/user-profile-tab-edit.png)

3. Populate the custom fields (`Department`, `Employee Type`) at the bottom of the profile form and save.

![Custom Attributes Manual Entry](../images/02-users-groups/custom-attributes-manual-entry.png)

While CSV bulk import is supported for batch updates, manual attribute entry remains prone to human error and data drift.

![CSV Import Dialog](../images/02-users-groups/directory-people-csv-import-dialog.png)

## Inbound Profile Mastering with BambooHR

To replicate an enterprise architecture where an HR system serves as the single source of truth, I integrated BambooHR as an authoritative profile master.

### API Integration Setup

I accessed the BambooHR tenant settings to generate an API key for Okta connectivity.

![BambooHR API Keys Menu](../images/02-users-groups/bamboohr-api-keys-menu.png)

I generated a dedicated API key named `Okta`.

![Generate API Key](../images/02-users-groups/bamboohr-generate-api-key.png)

The key was generated and copied securely.

![API Key Created](../images/02-users-groups/bamboohr-api-key-created.png)

In the Okta Admin Console, I browsed the App Integration Catalog for the pre-built BambooHR connector.

![App Catalog Search BambooHR](../images/02-users-groups/app-catalog-search-bamboohr.png)

I added the integration to the tenant.

![Add BambooHR Integration](../images/02-users-groups/app-catalog-add-bamboohr.png)

I configured the application labels and visibility options under General Settings.

![BambooHR General Settings](../images/02-users-groups/bamboohr-general-settings.png)

I configured the sign-on mode.

![BambooHR Sign-On Options](../images/02-users-groups/bamboohr-sign-on-options.png)

I noted the tenant subdomain (`isimsfax`) from the BambooHR account settings.

![BambooHR Account Domain Info](../images/02-users-groups/bamboohr-account-domain-info.png)

In this lab configuration, the integration was established through Okta API provisioning rather than direct app installation in the trial tenant.

![BambooHR Trial Notice](../images/02-users-groups/bamboohr-trial-settings-notice.png)

In Okta, I opened the **Provisioning** tab of the BambooHR application.

![BambooHR Provisioning Tab](../images/02-users-groups/bamboohr-provisioning-tab.png)

I selected **Configure API Integration**, checked **Enable API integration**, and authenticated with the BambooHR tenant to link credentials.

![Enable API Integration](../images/02-users-groups/bamboohr-enable-api-integration.png)

### Sourcing & Attribute Mapping

With the API connection validated, I configured profile mastering:

1. Under **Provisioning > To Okta**, I clicked **Edit** on **Profile & Lifecycle Sourcing**.

![Provisioning To Okta Navigation](../images/02-users-groups/bamboohr-provisioning-to-okta.png)

![Edit Profile Sourcing](../images/02-users-groups/bamboohr-profile-lifecycle-sourcing-edit.png)

2. I checked **Allow BambooHR to source Okta users**. This setting establishes BambooHR as the authoritative master over the Okta directory.

![Allow BambooHR to Source Users](../images/02-users-groups/bamboohr-allow-sourcing-okta-users.png)

3. I navigated to **Directory > Profile Editor > BambooHR** to review the inbound profile schema.

![BambooHR Profile in Profile Editor](../images/02-users-groups/profile-editor-bamboohr-profile.png)

4. I mapped BambooHR attributes to the custom Universal Directory attributes created in Phase 1:
   - BambooHR `Department` mapped directly to Okta `department`.
   - BambooHR `Employment Status` mapped to Okta `employeeType`.

![Map Department Attribute](../images/02-users-groups/bamboohr-mapping-department.png)

![Map Employee Type Attribute](../images/02-users-groups/bamboohr-mapping-employeetype.png)

5. Under **User Creation & Matching**, I configured rules to handle newly discovered profiles and match existing accounts.

![User Creation Matching Navigation](../images/02-users-groups/bamboohr-user-creation-matching-nav.png)

![User Creation Matching Rules](../images/02-users-groups/bamboohr-user-creation-matching-rules.png)

### Import Execution & Mastering Verification

I navigated to the **Import** tab and initiated an import by clicking **Import Now**.

![Import Now](../images/02-users-groups/bamboohr-import-now.png)

To verify the synchronization pipeline end-to-end, I created a new record directly in BambooHR: an employee named Yara Haddad with Department set to "IT" and Employment Status set to "Full-Time".

![BambooHR Employee Personal Info](../images/02-users-groups/bamboohr-new-employee-personal-info.png)

![BambooHR Employee Department](../images/02-users-groups/bamboohr-new-employee-job-department.png)

![BambooHR Employee Status Saved](../images/02-users-groups/bamboohr-new-employee-status-saved.png)

I re-ran the import in Okta. The system detected Yara Haddad as a new user, matched the identity, and auto-activated the account based on matching rules.

![Imported User Auto-Activate](../images/02-users-groups/bamboohr-imported-user-auto-activate.png)

When viewing Yara Haddad under **Directory > People**, both `department` and `employeeType` were populated directly from BambooHR. Because BambooHR is the designated profile master, these attributes are read-only in Okta. Any manual modifications attempted in the Okta console are either restricted or overwritten on subsequent import cycles.

![Profile Sourced Attributes](../images/02-users-groups/directory-people-profile-sourced-attributes.png)

## Dynamic Group Automation

Managing group memberships manually becomes unworkable as organizations scale. To automate access based on directory data, I implemented dynamic Group Rules powered by Okta Expression Language (OEL).

### Dynamic Group: IT-All

I created a new group named `IT-All`.

![Add IT-All Group](../images/02-users-groups/directory-groups-add-it-all.png)

I navigated to **Directory > Groups > Rules** and selected **Add Group Rule**.

![Add Group Rule](../images/02-users-groups/group-rules-add-rule-button.png)

I configured the rule parameters:
- **Name**: `Auto-Assign-IT`
- **Condition**: Select users where condition is true: `user.department=="IT"`
- **Action**: Assign to group `IT-All`

![Group Rule Condition](../images/02-users-groups/group-rule-auto-assign-it-condition.png)

New rules are created in an inactive state by default. I activated the rule.

![Activate Rule Prompt](../images/02-users-groups/group-rule-activate-prompt.png)

Once active, Okta evaluates the condition against all existing user profiles and re-evaluates automatically whenever a user's department changes.

![Active Rules List](../images/02-users-groups/group-rules-active-rules-list.png)

I confirmed that users whose department attribute equaled "IT" were immediately added to `IT-All`.

### Static Group: IAM-Lab-Admins

For contrast, I created a static group named `IAM-Lab-Admins` without any attached rules. Members were added manually.

This comparison highlights the difference between attribute-driven governance and manual maintenance:
- Dynamic groups (`IT-All`) automatically mirror organizational state changes with zero manual overhead.
- Static groups (`IAM-Lab-Admins`) require explicit administrator updates whenever personnel change, introducing access sprawl risks when members depart.
