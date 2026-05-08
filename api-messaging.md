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
| v1.9 | 2026-03-23 | Removed type PUSH from supported types. |
| v2.0 | 2026-05-06 | Remove `title` and `type` fields; clarify simultaneous multi-channel delivery; add fire-and-forget note; mark `retries`/`timeout`/`delay` as VOICE-only; fix `ErrorResponse.channel` nullable type; add NOK response example; remove incorrect 409 from error table; clarify `errors` behaviour per result type. |

</details>

---

## 🎯 Purpose & Access

The Messaging API allows transactional sending of ad-hoc messages via multiple communication channels (SMS, EMAIL, VOICE). It is designed for flexible, low-latency delivery in critical systems.

The API supports:

- **Simultaneous multi-channel delivery** — all channels in a target's `channel` array are triggered at the same time; order does not determine priority
- **Voice call retries** — configurable retry count, timeout, and delay for unanswered calls
- **Per-target delivery error reporting**

> ⚠️ The API responds immediately after the request is accepted. Delivery outcomes (especially for voice) happen asynchronously and are not reflected in the response.

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
| `channel` | array of enum | Yes | List of one or more of `SMS`, `EMAIL`, `VOICE` |
| `msisdn` | string | Required if `SMS` or `VOICE` is used | Phone number in [E.164 format](https://en.wikipedia.org/wiki/E.164) |
| `email` | string | Required if `EMAIL` is used | Email address of the target |

> **Targeting behaviour:**
> - Targets **do not need to be registered** in the safeREACH system — you can send to any MSISDN or email address.
> - If a target **is** a known recipient in the system, their account settings apply:
>   - **SMS:** the channel must be enabled on their account.
>   - **EMAIL:** an email address must be provided explicitly in the request, and it must match the address stored in their account.

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
| `channel` | `string \| null` | Channel that failed (e.g., `SMS`, `EMAIL`). `null` if the error is not channel-specific (e.g. target not found). |
| `target` | string | Phone number or email address that was not reachable |
| `status` | integer | Status code (`404` for target not found, etc.) |

**Examples:**

```json
{
  "message": "Target with the specified msisdn was not found.",
  "channel": null,
  "target": "+43123456789",
  "status": 404
}
```

---

## 🔌 Endpoints

If not specified differently, all endpoints require header `Content-Type: application/json; charset=utf-8`

---

### Send message — JSON

**`POST /api/messaging/v1/send`**

| Field | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `username` | string | Yes | — | API username |
| `password` | string | Yes | — | API password |
| `customerId` | string | Yes | — | Customer ID |
| `message` | string | Yes | — | Message content (plain text, no HTML). For **EMAIL** targets, the first line is used as the subject and the remainder as the body. For **VOICE** targets, SSML markup is supported for controlling speech and pacing (see note below). |
| `language` | string (enum) | Optional | — | `en`, `de`. If not set, defaults to customer language. |
| `retries` | integer | Optional | `2` | **VOICE only.** Number of retries for unanswered voice calls |
| `timeout` | integer (sec) | Optional | `60` | **VOICE only.** Wait time before a voice call is considered unanswered (in seconds) |
| `delay` | integer (sec) | Optional | `300` | **VOICE only.** Delay between voice call retries (in seconds) |
| `targets` | array of `MessageTarget` | Yes | — | Recipients list |


> 💡 **VOICE — SSML support:** For voice calls, the `message` field accepts [SSML (Speech Synthesis Markup Language)](https://www.w3.org/TR/speech-synthesis/) to control speech and pacing. Wrap the content in `<speak>` tags and use elements like `<break>` to insert pauses:
> ```xml
> <speak>
>
> Attention.
>
> <break time="700ms"/>
>
> This is an important alert.
>
> <break time="600ms"/>
>
> Please follow the instructions in the safeREACH application.
>
> <break time="900ms"/>
>
> I repeat.
>
> <break time="500ms"/>
>
> Please follow the instructions in the safeREACH application.
>
> </speak>
> ```

**Example Request:**


```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "message": "Hello\n\nYou got some✉️",
  "language": "en",
  "retries": 3,
  "timeout": 45,
  "delay": 180,
  "targets": [
    {
      "channel": ["SMS", "EMAIL"],
      "msisdn": "+43123456789",
      "email": "target@acme.com"
    },
    {
      "channel": ["VOICE"],
      "msisdn": "+436641234567"
    }
  ]
}
```

**Example Request (VOICE with SSML):**

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "message": "<speak>\n\nAttention.\n\n<break time=\"700ms\"/>\n\nThis is an important alert.\n\n<break time=\"600ms\"/>\n\nPlease follow the instructions in the safeREACH application.\n\n<break time=\"900ms\"/>\n\nI repeat.\n\n<break time=\"500ms\"/>\n\nPlease follow the instructions in the safeREACH application.\n\n</speak>",
  "language": "en",
  "retries": 2,
  "timeout": 60,
  "delay": 300,
  "targets": [
    {
      "channel": ["VOICE"],
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
| `errors` | array of `ErrorResponse` | Per-target error details. Empty for `OK` and `NOK`; populated for `PARTIAL`. |

**`result` meanings:**

- `OK`: All targets were notified
- `NOK`: None were notified (e.g. all targets unknown). `errors` is empty.
- `PARTIAL`: Some targets failed. `errors` contains details for the failed targets.

---

**Example: Successful Response**

```json
{
  "result": "OK",
  "errors": []
}
```

---

**Example: No Targets Notified**

```json
{
  "result": "NOK",
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

Partial or full delivery failures are not returned as HTTP error codes — they are reflected in the `result` field (`PARTIAL` or `NOK`) within a `200 OK` response.
