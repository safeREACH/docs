# 📘 User API Documentation

<details>
  <summary><h2>🧩 Version History</h2></summary>

- **v2.0**: Initial Draft (2018-09-30)
- **v2.1**: Small API adjustments (2019-03-01)
- **v2.2**: Add `merge` flag for import requests (2019-08-20)
- **v2.3**: Rename `customerOrGroupId` to `customerId` in data objects (2020-03-05)
- **v2.4**: Add optional `deleteOnlyExternal` flag (2020-04-09)
- **v2.5**: Fix `useExternalId` → `externalId` (2020-04-14)
- **v2.6**: Add `channels` attribute to `PublicRecipientData` (2021-10-01)
- **v2.7**: Document deletion endpoints (2022-04-11)
- **v2.8**: Add unique constraint for `msisdns` and `emails` (2022-10-18)
- **v2.9**: Add `recipientsToDelete` and `groupsToDelete` (2023-07-11)
- **v2.10**: Add `functions` to recipient import (2024-01-15)
- **v2.11**: Rewrite and restructuring (2025-08-07)
- **v2.12**: Add `X-API-Key` authentication option (2025-08-23)

</details>

---

## 🎯 Purpose & Access

The **User Management API** provides secure, programmatic access to manage recipients, groups, and functions on the safeREACH platform. It enables automation for user lifecycle management, group assignment, and metadata synchronization. For detailed examples see the [Common Use Cases & Examples](./api-user-examples.md) document.

### 🔐 Authentication

All endpoints of the User API support two authentication mechanisms.

> Note: The customerOrGroupId parameter must always be provided:
> 
> - For **JSON-based import endpoints**: in the request body as `customerOrGroupId`
> - For **GET, DELETE, or import (CSV) endpoints**: in the header `X-CustomerId`

### **Option 1: API Key Authentication**

Use an API key provided by the safeREACH support team.

| Header | Required | Description |
| --- | --- | --- |
| `X-API-Key` | ✅ | API key provided by support |
| `X-CustomerId` | ✅ | Customer ID or Group ID for **GET, DELETE, or import (CSV) endpoints** |
| `Content-Type` | ✅ | `application/json; charset=utf-8` |


**Notes:**
- This method **works for all endpoints** except:  
  - `GET /api/public/v1/recipient/{customerOrGroupId}/export`  
  - `GET /api/public/v1/group/{customerOrGroupId}/export`  
- For those two export endpoints, you must continue using **Option 2** (Admin Account Authentication).  
- API keys are customer-specific and should be stored securely.

---

### **Option 2: Admin Account Authentication**
Use a **dedicated admin account** with the `API user` permission provided by the safeREACH support team.


**For JSON-based endpoints (e.g., recipient or group import via JSON)**

For endpoints that accept JSON payloads, provide credentials in the **request body**:

- `customerOrGroupId`:
    - For most customers, this is your **Customer ID** (e.g., `"500027"`).
    - For customers with multi-tenant or hierarchical account structures, this may be a **Group ID** (i.e., a parent customer that owns multiple sub-customers).
    - ⚠️ **Do not confuse this with recipient group IDs** such as `"G1"` — they are unrelated.
- `username`: the **username** of your admin API user
- `password`: the **password** of your admin API user

**For CSV-based or non-body endpoints (e.g., CSV imports, exports, deletion)**

For CSV-based or HTTP `GET` / `DELETE` endpoints, credentials must be provided via **HTTP headers**:

| Header | Required | Description |
| --- | --- | --- |
| `X-CustomerId` | ✅ | Your **Customer ID** or **Group ID** (same as `customerOrGroupId`) |
| `X-Username` | ✅ | The username of the admin account with `API user` permission |
| `X-Password` | ✅ | The password for the API user |
| `Content-Type` | ✅ | Must be `text/csv` for CSV imports or `application/json` for JSON requests |
| `Accept` | optional | Only required for **export** endpoints. Use `application/json` or `text/csv` to specify format |

> 🔐 Important:
> 
> - The admin account must explicitly have the `API user` permission assigned.
> - API credentials for the **User API** are **not compatible** with the **Scenario API**.
> - Only one customer or group context can be targeted per request.

---


## 🌐 Data Center Selection

| Region   | Base URL                                |
|----------|------------------------------------------|
| Frankfurt| `https://api.safereach.com/blaulicht`    |
| Vienna   | `https://api.safereach.at/blaulicht` |

If you're unsure, ask support which region your account is in.

---

## 🧱 Data Models

### RecipientData

| Field | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `id` | `string (UUIDv4)` | No | Leave empty for new records or when using `externalId`. Required if updating by `id`. | — |
| `externalId` | `string` | No | Required if using `externalId`-based imports. Must be unique within the customer context. | — |
| `customerId` | `string` | Yes | Customer ID to which the user belongs. | — |
| `msisdn` | `string` (E.164) | Yes | Must be globally unique across safeREACH. Example: `+4366412345678` | — |
| `givenname` | `string` | Yes | First name of the recipient. | — |
| `surname` | `string` | Yes | Last name of the recipient. | — |
| `email` | `string` | No | Optional email address. Must be unique across safeREACH. | — |
| `comment` | `string` | No | Free text comment field (e.g. department, role, etc.) | — |
| `groups` | `RecipientGroupParticipationData[]` | Yes | List of group assignments. Can be empty. | `[]` |
| `functions` | `FunctionParticipationData[]` | No | Optional list of functions assigned to the user. | `null` |
| `channels` | `string[]` | No | Optional list of preferred channels (`SMS`, `PUSH`, `EMAIL`, `VOICE`) | `null` |

---

### RecipientBaseData

| Field | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `id` | `string (UUIDv4)` | No | Internal ID. Required when deleting by `id`. | — |
| `externalId` | `string` | No | External ID. Required when deleting by `externalId`. | — |
| `customerId` | `string` | Yes | The customer ID or `"*"` for group-level operations across child customers. | — |

---

### RecipientGroupParticipationData

| Field | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `groupId` | string | Yes | Group identifier (e.g., `"G1"`) | — |

---

### FunctionParticipationData

| Field | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `functionCode` | string | Yes | Function code (e.g., `"F1"` refers to Display order in the UI and has to be unique) | — |

---

### GroupData

| Field | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `id` | `string (UUIDv4)` | No | Internal ID. Leave empty for new group creation. | — |
| `externalId` | `string` | No | Optional external ID, useful for integrations. | — |
| `customerId` | `string` | Yes | Customer ID owning the group. | — |
| `groupId` | `string` | Yes | Group ID (e.g., `G1` to `G999999`). Cannot be changed after creation. Must start with `G` and a number. | — |
| `name` | `string` | Yes | Group name (e.g., `"Operations"`, `"IT"`) | — |

---

### FunctionData

| Field | Type | Required | Description | Default |
| --- | --- | --- | --- | --- |
| `id` | `string (UUIDv4)` | No | Internal ID. Leave empty for new function creation. | — |
| `externalId` | `string` | No | Optional external ID. Required only if using external-based updates. | — |
| `customerId` | `string` | Yes | Customer ID owning the function. | — |
| `functionCode` | `string` | Yes | Code starting with `F` and followed by a number (e.g., `F1`, `F999999`). Immutable after creation. Refers to Display order in UI and has to be unique. | — |
| `name` | `string` | Yes | Human-readable function name (e.g., `"IT Admin"`, `"First Aid Officer"`) | — |

---

## 🔌 Endpoints

If not specified differently, all endpoints require header `Content-Type: application/json; charset=utf-8`.

---

### Import Recipients — JSON

**`POST /api/public/v1/recipient/import`**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `dryRun` | Boolean | Optional | false | Only simulate changes |
| `externalId` | Boolean | Optional | false | Use external IDs |
| `partial` | Boolean | Optional | false | Don’t delete missing records |
| `merge` | Boolean | Optional | false | Merge by MSISDN, requires `externalId` |
| `deleteOnlyExternal` | Boolean | Optional | false | Only delete records with `externalId` |
| `recipientsToDelete` | List | Optional | null | List of `externalId` |
| `groupsToDelete` | List | Optional | null | List of `groupId`s |
| `recipients` | List | ✅ Yes | — | List of `RecipientData` objects |

**Example**

Request:

```json
{
  "customerOrGroupId": "500027",
  "dryRun": true,
  "externalId": false,
  "merge": false,
  "recipients": [
    {
      "id": "",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+4366412345678",
      "givenname": "Max",
      "surname": "Mustermann",
      "email": "max@example.com",
      "groups": [
        { "groupId": "G1" }
      ],
      "functions": [
        { "functionCode": "F1" }
      ],
      "channels": ["PUSH", "EMAIL"]
    }
  ]
}
```

Response:

```json
{
  "result": "OK",
  "created": 1,
  "updated": 0,
  "deleted": 0,
  "merged": 0,
  "request": {
    "dryRun": true,
    "externalId": false,
    "merge": false}
}
```

---

### Export Recipients / Groups

**`GET /api/public/v1/recipient/{customerOrGroupId}/export`**

**`GET /api/public/v1/group/{customerOrGroupId}/export`**

**Headers**:

- `X-Username`, `X-Password`
- `Accept`: `application/json` or `text/csv`

**JSON Example**

```json
{
  "result": "OK",
  "recipients": [ { ...RecipientData } ]
}
```

**CSV Example**

```csv
id;externalId;customerId;givenname;surname;msisdn;email;comment;G1;G2
UUIDv4;;500027;Jane;Doe;+4366412345678;jane@example.com;;1;0
```

---

### Delete Recipients

**`DELETE /api/public/v1/recipient`**

**Request Body**:

| Parameter | Required | Description |
| --- | --- | --- |
| `dryRun` | ✅ Yes | Preview vs delete |
| `externalId` | ✅ Yes | Use externalId matching |
| `recipients` | ✅ Yes | List of `RecipientBaseData` |

**Example**

Request:

```json
{
  "customerOrGroupId": "500027",
  "dryRun": true,
  "externalId": true,
  "recipients": [
    { "id": "", "externalId": "EXT-123", "customerId": "500027" }
  ]
}
```

Response:

```json
{
  "result": "OK",
  "description": "OK",
  "recipients": [
    {
      "msisdn": "+4366412345678",
      "givenname": "John",
      "surname": "Doe"
    }
  ],
  "dryRun": true}
```

---

### Import Groups / Functions

**`POST /api/public/v1/group/import`**

**`POST /api/public/v1/functions/{customerId}/import`**

- Accepts same flags as `/recipient/import`
- `groups` and `functions` are lists of `GroupData` / `FunctionData`

---

### CSV Import (Recipients or Groups)

- `Content-Type: text/csv`
- `X-CustomerId`, `X-API-Key` or (`X-Username`, `X-Password`) headers
- Use `;` as separator
- `1` = member of group, `0` = not member
- `customerOrGroupId` is always passed in `X-CustomerId`
- Import options can be added as query parameters:
    - `dryRun`
    - `externalId`
    - `partial`
    - `merge`
    - `deleteOnlyExternal`

**CSV example for recipients:**


```http
POST /blaulicht/api/public/v1/recipient/import?dryRun=true&merge=false HTTP/1.1
Host: api.safereach.com
X-API-Key: your-api-key-here
X-CustomerId: 500027
Content-Type: application/json; charset=utf-8

id;externalId;customerId;givenname;surname;msisdn;email;comment;G1;G2
;;500027;Max;Mustermann;+4366412345678;max@example.com;Ops;1;0
```

---

## 🧪 Tips

- Always start with a `dryRun = true` request before executing changes
- Use `merge` for safe migrations from legacy systems
- Assign functions for richer role-based logic
- Avoid mixing `id` and `externalId` logic within the same request

---

## 🐞 Known Issues

**Recipient Groups with `externalId` become System Groups**

Creating recipient groups via `POST /blaulicht/api/public/v1/group/import` with `externalId` set results in **system groups** that cannot be edited or deleted. This limits management flexibility and may cause clutter in group lists.

**Workaround:** Omit `externalId` when creating groups unless system group behavior is desired.

---

## ❗Error Handling

All endpoints in the User API respond with standardized HTTP status codes and JSON-formatted error messages. This section outlines possible errors, their causes, and how to resolve them.

### Authentication & Authorization Errors

| Status Code | Meaning | Cause | Resolution |
| --- | --- | --- | --- |
| `401 Unauthorized` | Invalid credentials | The `username` or `password` is incorrect, or the account is locked. | Double-check credentials. Ensure the account is active and has the correct permissions. |
| `403 Forbidden` | Access denied | The API user lacks the necessary permission (e.g., missing `API user`permission). | Assign the `API user` permission in the admin interface. |

---

### Request Errors

| Status Code | Meaning | Cause | Resolution |
| --- | --- | --- | --- |
| `400 Bad Request` | Malformed or invalid request | JSON or CSV payload is incorrect, fields are missing or misformatted. | Validate the request structure against the documented models and examples. |
| `409 Conflict` | Data conflict | The import would cause duplicate `msisdn` / `email`, or `externalId` conflicts. | Ensure unique identifiers and validate data before import. |

---

### Dry Run-Specific Response

During dry runs (`"dryRun": true`), responses will include a list of recipients or groups that **would be created, updated, or deleted**, but **no actual changes are performed**.

If the `dryRun` flag is omitted or `false`, changes are **committed** and the response will indicate actual database modifications.

---

### Deletion Errors

| Status Code | Meaning | Cause | Resolution |
| --- | --- | --- | --- |
| `403 Forbidden` | Partial access for deletion | User does not have access to all recipients or groups targeted in a bulk deletion. | Restrict deletions to recipients within the allowed `customerOrGroupId`. |
| `409 Conflict` | Cannot delete referenced data | Attempt to delete a group or function still referenced by active users. | Remove user associations first, or use the import options for safe deletion. |

### Examples

**Error Response (Unauthorized)**

```json
{
  "result": "NOK",
  "description": "Invalid username or password."
}
```

**Error Response (Forbidden)**

```json
{
  "result": "NOK",
  "description": "You don't have access to all requested recipients! No recipients were deleted.",
  "recipients": null,
  "dryRun": false}
```

**Error Response (Conflict)**

```json
{
  "result": "NOK",
  "description": "Recipient with MSISDN +4366412345678 already exists.",
  "conflicts": [
    {
      "msisdn": "+4366412345678",
      "email": "existing@example.com"
    }
  ]
}
```
