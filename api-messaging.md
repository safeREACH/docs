# 📘 Messaging API Documentation

<details>
  <summary><h2>🧩 Version History</h2></summary>

| Version | Date | Changes |
| --- | --- | --- |
| v1.0 | 2021-11-25 | Initial draft |
| v1.1 | 2022-07-18 | Error response definition and sending mechanism |
| v1.2 | 2022-07-22 | Replaced `SAFE_REACH` channel name with `PUSH` |
| v1.3 | 2022-07-25 | Added `type` field to `MessageRequest` |
| v1.4 | 2022-07-25 | Added `title` field to `MessageRequest` |
| v1.5 | 2022-11-07 | Added `language` field to `MessageRequest` |
| v1.6 | 2022-11-28 | Removed `fr`, `it` from allowed languages; clarified usage of `language` |
| v1.7 | 2023-07-13 | Added `retries`, `timeout`, and `delay` for voice calls |
| v1.8 | 2025-08-08 | Reworked structure and fixed wordings. |

</details>

---

## 🎯 Purpose & Access

---

The Messaging API allows transactional sending of ad-hoc messages via multiple communication channels (SMS, EMAIL, VOICE, PUSH). It is designed for flexible, low-latency delivery in critical systems.

The API supports:

- Multi-channel fallback delivery
- In-app push notifications (safeREACH app)
- Voice call retries
- Language and title control for push messages
- Per-target delivery error reporting

### ✅ API Credentials

Messaging API access **requires credentials issued by the safeREACH Support Team**.

- These are **not** the same as User API credentials.
- Required fields:
    - `customerId`
    - `username`
    - `password`

> ℹ️ Contact support@safereach.com to request credentials.
> 

---

### 🌐 Data Center Selection

Depending on your account region, use the correct base URL:

| Region | Base URL |
| --- | --- |
| Frankfurt | `https://api.safereach.com/blaulicht` |
| Vienna | `https://api.safereach.at/blaulicht` |

> 💡 If you're unsure which data center your account belongs to, please contact safeREACH support.
> 

---

### Fair Use & Rate Limits

API usage is subject to:

- The **Fair Use Policy**
- **Account-specific** rate limits

Contact support if you anticipate high traffic or require an increase in limits.

---

## 🧱 Data Models

### `MessageTarget`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `channel` | array of enum | Yes | List of one or more of `SMS`, `EMAIL`, `VOICE`, `PUSH` |
| `msisdn` | string | Required if `SMS` or `VOICE` is used | Phone number in [E.164 format](https://en.wikipedia.org/wiki/E.164) |
| `email` | string | Required if `EMAIL` is used | Email address of the target |

> PUSH is used to send messages to the safeREACH mobile app. The target must be registered in the system beforehand.
> 

**Example:**

```json
{
  "channel": ["SMS", "VOICE", "EMAIL"],
  "msisdn": "+43123456789",
  "email": "target@acme.com"
}
```

---

### `ErrorResponse`

| Field | Type | Description |
| --- | --- | --- |
| `message` | string | Description of the error |
| `channel` | string | Channel that failed (e.g., `SMS`, `EMAIL`, etc.) |
| `target` | string | Phone number or email address that was not reachable |
| `status` | integer | HTTP-like status code (`404` for not found, `409` for delivery conflict, etc.) |

**Examples:**

```json
{
  "message": "Target with the specified msisdn was not found.",
  "channel": null,
  "target": "+43123456789",
  "status": 404
}
```

```json
{
  "message": "Could not notify target because channel PUSH was not defined.",
  "channel": "PUSH",
  "target": "+43123456789",
  "status": 409
}
```

---

## 🔌 Endpoints

All endpoints use `Content-Type: application/json` and UTF-8 encoding.

---

### Send message — JSON

**`POST /api/messaging/v1/send`**

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `username` | string | Yes | — | API username |
| `password` | string | Yes | — | API password |
| `customerId` | string | Yes | — | Customer ID |
| `message` | string | Yes | — | Message content (plain text, no HTML) |
| `type` | string | Optional | — | One of `info`, `alarm`. Required if any target includes `PUSH`. |
| `title` | string | Optional | — | Push title shown in app. Required if `PUSH` is used. |
| `language` | string (enum) | Optional | — | `en`, `de`. If not set, defaults to customer language. |
| `retries` | integer | Optional | `2` | Number of retries for unanswered voice calls |
| `timeout` | integer (sec) | Optional | `60` | Wait time for voice call response (in seconds) |
| `delay` | integer (sec) | Optional | `300` | Delay before voice call retry (in seconds) |
| `targets` | array of `MessageTarget` | Yes | — | Recipients list |


**Example Request:**


```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "message": "Hello\n\nYou got some✉️",
  "type": "info",
  "title": "New Notification",
  "language": "en",
  "retries": 3,
  "timeout": 45,
  "delay": 180,
  "targets": [
    {
      "channel": ["SMS", "VOICE", "EMAIL"],
      "msisdn": "+43123456789",
      "email": "target@acme.com"
    },
    {
      "channel": ["PUSH"],
      "msisdn": "+436641234567"
    }
  ]
}
```

---

#### Response

| Field | Type | Description |
| --- | --- | --- |
| `result` | string | One of `OK`, `NOK`, `PARTIAL` |
| `errors` | array of `ErrorResponse` | Empty if all messages succeeded |

**`result` meanings:**

- `OK`: All targets were notified
- `NOK`: None were notified
- `PARTIAL`: Some targets failed

---

**Example: Successful Response**

```json
{
  "result": "OK",
  "errors": []
}
```

---

**Example: Partial Success**

```json
{
  "result": "PARTIAL",
  "errors": [
    {
      "message": "Target with the specified msisdn was not found.",
      "channel": null,
      "target": "+43123456789",
      "status": 404
    }
  ]
}
```

---

## ❗Error Handling

The following errors can be returned from the API:

| HTTP Code | Meaning | Description |
| --- | --- | --- |
| 400 | Bad Request | Malformed JSON or invalid data format |
| 401 | Unauthorized | Invalid username or password |
| 403 | Forbidden | Missing permissions or invalid customer context |
| 404 | Not Found | Target was not found in the system |
| 409 | Conflict | Unable to deliver message via all channels |