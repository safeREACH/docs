# 📋 Common Use Cases - User Management API

This section shows typical ways customers use the User API — from **business goals** to **exact API calls**.

It’s structured so you can:

1. Understand **when and why** to use a pattern.
2. Quickly find the **right endpoint and parameters**.
3. Copy & adapt **ready-to-use request examples**.

## 📄 Quick Reference: Use Cases → Endpoints → Parameters

| # | Use Case | Endpoint(s) | Key Parameters / Headers | Tips |
| --- | --- | --- | --- | --- |
| **1** | **[Initial User & Group Onboarding](#initial-user--group-onboarding)** | `POST /api/public/v1/group/importPOST /api/public/v1/functions/{customerId}/importPOST /api/public/v1/recipient/import` | **Body:**`customerOrGroupId`, `username`, `password`,`dryRun`, `externalId`, `groups` / `functions` / `recipients` arrays | Import groups/functions **before** recipients. Use `dryRun=true` to validate. Assign `groups` and `functions` in recipient imports. |
| **2** | **[Daily / Scheduled Sync with External System](#daily--scheduled-sync-with-external-system)** | `POST /api/public/v1/recipient/import` | **Body:**`externalId=truepartial=truemerge=truerecipients` array | Use stable `externalId`mapping to avoid duplicates. Set `partial=true` to remove missing users automatically. Automate via cron or integration job. |
| **3** | **[One-off Bulk Updates](#one-off-bulk-updates)** | `POST /api/public/v1/recipient/import` | **Body:**`externalId=truemerge=truedryRun=truerecipients` array | Ideal for department renames, adding functions, updating emails. Use `merge=true` to avoid overwriting unrelated fields. |
| **4** | **[Selective User or Group Deletion](#selective-user-or-group-deletion)** | `DELETE /api/public/v1/recipient`**or** `POST /api/public/v1/recipient/import` with `recipientsToDelete` / `groupsToDelete` | **Body:**`externalId=truedeleteOnlyExternal=truedryRun=truerecipientsToDelete` / `groupsToDelete` arrays | Use `deleteOnlyExternal=true`to avoid deleting manually added entries. Always preview with `dryRun=true`. |
| **5** | **[Export for Auditing or Reporting](#export-for-auditing-or-reporting)** | `GET /api/public/v1/recipient/{customerOrGroupId}/exportGET /api/public/v1/group/{customerOrGroupId}/export` | **Headers:**`X-CustomerId`, `X-Username`, `X-Password`,`Accept: application/json` or `text/csv` | Use JSON for integration into BI tools, CSV for Excel/manual review. Export regularly for compliance audits. |

## **1. Initial User & Group Onboarding**

When first setting up safeREACH, you can bulk-import all recipients, groups, and functions in one or more API calls.

This is ideal for **migrating from spreadsheets** or **loading data from an HR export**.

- **What it does:** Creates your full user base, group structure, and functional roles in safeREACH in a single operation.
- **When to use:** During first-time setup or when creating a new safeREACH environment.
- **Example:** Import 500 employees, assign them to their departments (`G1 = Operations`, `G2 = IT`), and tag 20 people with the `F1 = First Aid Officer` function.
- **Key tips:**
    - Use `dryRun = true` to preview results before committing.
    - Import groups and functions before importing recipients that reference them.

**Example Requests:**

**A) Create Groups**

```bash
curl -X POST "<BASE_URL>/api/public/v1/group/import" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "customerOrGroupId": "<CUSTOMER_ID>",
    "username": "<API_USER>",
    "password": "<API_PASS>",
    "dryRun": true,
    "externalId": false,
    "groups": [
      { "id": "", "externalId": "", "customerId": "<CUSTOMER_ID>", "groupId": "G1", "name": "Operations" },
      { "id": "", "externalId": "", "customerId": "<CUSTOMER_ID>", "groupId": "G2", "name": "IT" }
    ]
  }'

```

**B) Create Functions**

```bash
curl -X POST "<BASE_URL>/api/public/v1/functions/<CUSTOMER_ID>/import" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "customerOrGroupId": "<CUSTOMER_ID>",
    "username": "<API_USER>",
    "password": "<API_PASS>",
    "dryRun": true,
    "externalId": false,
    "functions": [
      { "id": "", "externalId": "", "customerId": "<CUSTOMER_ID>", "functionCode": "F1", "name": "First Aid Officer" },
      { "id": "", "externalId": "", "customerId": "<CUSTOMER_ID>", "functionCode": "F2", "name": "Crisis Manager" }
    ]
  }'

```

**C) Import Recipients**

```bash
curl -X POST "<BASE_URL>/api/public/v1/recipient/import" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "customerOrGroupId": "<CUSTOMER_ID>",
    "username": "<API_USER>",
    "password": "<API_PASS>",
    "dryRun": true,
    "externalId": false,
    "merge": false,
    "recipients": [
      {
        "id": "",
        "externalId": "",
        "customerId": "<CUSTOMER_ID>",
        "msisdn": "+4366412345678",
        "givenname": "Max",
        "surname": "Mustermann",
        "email": "max@example.com",
        "groups": [ { "groupId": "G1" } ],
        "functions": [ { "functionCode": "F1" } ],
        "channels": ["PUSH", "EMAIL"]
      }
    ]
  }'

```

---

## **2. Daily or Scheduled Sync with an External System**

Automate safeREACH to stay in sync with a master data source such as **Active Directory, LDAP, or an HR system**.

- **What it does:** Adds new staff, updates existing user details, and removes users who no longer exist in the source system.
- **When to use:** To keep data accurate without manual updates.
- **Example:** Every night, a script queries the HR API for all active employees and calls `/recipient/import` with `externalId = true` and `partial = true`.
- **Key tips:**
    - Use `externalId` for stable cross-system matching.
    - Use `merge = true` if matching by MSISDN.

**Example Request:**

```bash
curl -X POST "<BASE_URL>/api/public/v1/recipient/import" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "customerOrGroupId": "<CUSTOMER_ID>",
    "username": "<API_USER>",
    "password": "<API_PASS>",
    "dryRun": false,
    "externalId": true,
    "partial": true,
    "merge": true,
    "recipients": [
      {
        "externalId": "HR-123",
        "customerId": "<CUSTOMER_ID>",
        "msisdn": "+4366412345678",
        "givenname": "Jane",
        "surname": "Doe",
        "email": "jane.doe@example.com",
        "groups": [ { "groupId": "G1" } ]
      }
    ]
  }'

```

---

## **3. One-off Bulk Updates**

Change multiple recipients at once — for example, after reorganising departments or updating titles.

- **What it does:** Updates existing recipient records without editing each one manually.
- **When to use:** For structural or contact detail changes affecting many users at the same time.
- **Example:** Update 120 recipients’ department comments and add them to a new function.
- **Key tips:**
    - Use `merge = true` to avoid overwriting unrelated fields.
    - Always run with `dryRun = true` first.

**Example Request:**

```bash
curl -X POST "<BASE_URL>/api/public/v1/recipient/import" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "customerOrGroupId": "<CUSTOMER_ID>",
    "username": "<API_USER>",
    "password": "<API_PASS>",
    "dryRun": true,
    "externalId": true,
    "merge": true,
    "recipients": [
      {
        "externalId": "HR-123",
        "customerId": "<CUSTOMER_ID>",
        "msisdn": "+4366412345678",
        "givenname": "Jane",
        "surname": "Doe",
        "comment": "Dept: Operations",
        "functions": [ { "functionCode": "F3" } ]
      }
    ]
  }'

```

---

## **4. Selective User or Group Deletion**

Remove specific recipients or groups that are no longer needed.

- **What it does:** Deletes targeted records from safeREACH.
- **When to use:** During cleanup after staff departures or contract expirations.
- **Example:** Remove all contractors listed in an `externalId` list.
- **Key tips:**
    - Use `recipientsToDelete` or `groupsToDelete` for targeted cleanup.
    - Set `dryRun = true` to preview the result.
    - Use `deleteOnlyExternal = true` to avoid touching manual entries.

**Example Request:**

```bash
curl -X POST "<BASE_URL>/api/public/v1/recipient/import" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "customerOrGroupId": "<CUSTOMER_ID>",
    "username": "<API_USER>",
    "password": "<API_PASS>",
    "dryRun": true,
    "externalId": true,
    "deleteOnlyExternal": true,
    "recipientsToDelete": ["EXT-123", "EXT-456"],
    "recipients": []
  }'

```

---

## **5. Export for Auditing or Reporting**

Download the current state of recipients or groups for **compliance checks, backup, or integration with BI tools**.

- **What it does:** Produces a complete list of recipients or groups in CSV or JSON format.
- **When to use:** For quarterly audits, verifying group memberships, or feeding external systems.
- **Example:** Export all recipients in JSON and feed the data into an audit dashboard.
- **Key tips:**
    - Use `Accept: application/json` for integrations.
    - Use `Accept: text/csv` for Excel/manual review.

**Example Request (JSON Export):**

```bash
curl -X GET "<BASE_URL>/api/public/v1/recipient/<CUSTOMER_ID>/export" \
  -H "X-CustomerId: <CUSTOMER_ID>" \
  -H "X-Username: <API_USER>" \
  -H "X-Password: <API_PASS>" \
  -H "Accept: application/json"

```

**Example Request (CSV Export):**

```bash
curl -X GET "<BASE_URL>/api/public/v1/group/<CUSTOMER_ID>/export" \
  -H "X-CustomerId: <CUSTOMER_ID>" \
  -H "X-Username: <API_USER>" \
  -H "X-Password: <API_PASS>" \
  -H "Accept: text/csv"

```