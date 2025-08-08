# 📘 Scenario API Documentation

## 🧩 Version History

- **v1.0**: Initial draft (2017-02-19)
- **v1.1**: Added `additionalText` property (2018-12-14)
- **v1.2**: Deprecated `LIVE` and `PRACTICE` trigger types (2020-10-12)
- **v1.3**: Added scenario list endpoint with `cutOff` and `configId` filters (2020-10-12)
- **v1.4**: Added trigger by scenario code endpoint (2021-10-12)
- **v1.5**: Added `additionalRecipients` attribute to trigger requests (2021-11-08)
- **v1.6**: Fixed typos and formatting (2022-09-13)
- **v1.7**: Added support for terminating alarms and sending updates to active scenarios (2024-11-28)
- **v1.8**: Reworked structure. Added data center location Vienna. Fixed some wordings and typos.
---

## 🎯 Purpose & Access

The **Scenario API** is designed to provide **programmatic access** to the scenario-based alerting capabilities of the safeREACH platform. While scenarios can typically be triggered manually through the safeREACH web interface or mobile apps, this API extends those capabilities by enabling automated and ad hoc interaction with scenarios.

To access the Scenario API, ensure the following:

### ✅ API Credentials

Scenario API access **requires credentials issued by the safeREACH Support Team**.

- These are **not** the same as User API credentials.
- Required fields:
  - `customerId`
  - `username`
  - `password`

> ℹ️ Contact [support@safereach.com](mailto:support@safereach.com) to request credentials.

---

### 🌐 Data Center Selection
| Region   | Base URL                                |
|----------|------------------------------------------|
| Frankfurt| `https://api.safereach.com/blaulicht`    |
| Vienna   | `https://api.safereach.at/blaulicht` |

If you're unsure, ask support which region your account is in.

---

### Fair Use & Rate Limits

API usage is subject to:

- The **Fair Use Policy**
- **Account-specific** rate limits

Contact support if you anticipate high traffic or require an increase in limits.

---


### Key Use Cases

- **Automated Scenario Triggering**  
  Integrate with internal systems (e.g., monitoring, facility management, emergency response platforms) to trigger predefined scenarios automatically when certain conditions are met.

- **Dynamic Recipient Assignment**  
  Add or override recipients on the fly using the `additionalRecipients` parameter, allowing flexibility depending on the current operational context.

- **Custom User Messages**  
  Attach contextual information to alerts via the `additionalText` field to better inform recipients during critical events.

- **Centralized Orchestration**  
  Use the API to build your own alert management dashboard or integrate scenario control into existing IT or crisis management systems.

- **Programmatic Termination and Updates**  
  Send follow-up updates or terminate scenarios based on external signals (e.g., incident resolved, escalation completed).

---

## 🧱 Models

### `ScenarioConfigData`

```json
{
  "scenarioConfigId": "32849abcdef23343",
  "customerId": "500027",
  "name": "Szenario 1"
}
```

### `ScenarioPreviewData`

```json
{
  "scenarioId": "123-ABC-asdfqwerasdf",
  "scenarioConfigId": "123-ABC-asdfqwerasdf-CONFIG",
  "authorId": "author1",
  "authorName": "Max Mustermann",
  "creationDate": "2018-12-13T14:56:53.016Z",
  "startDate": "2018-12-13T14:56:53.016Z",
  "endDate": null
}
```

### `ScenarioData`

```json
{
  "scenarioId": "123-ABC-asdfqwerasdf",
  "scenarioConfigId": "123-ABC-asdfqwerasdf-CONFIG",
  "authorId": "author1",
  "authorName": "Max Mustermann",
  "usersAlertedCount": 5,
  "creationDate": "2018-12-13T14:56:53.016Z",
  "startDate": "2018-12-13T14:56:53.016Z",
  "endDate": null,
  "alarms": [
    // <AlarmData>
  ]
}
```

### `RecipientConfiguration`
- type: one of `"MSISDN"`, `"GROUP"`, `"CASCADE"`
- target: depending on `type`, either a group or cascade ID, or a telephone number with country prefix ([E. 164 format](https://en.wikipedia.org/wiki/E.164))

```json
{
  "type": "MSISDN",
  "target": "+4367712345678"
}
```

```json
{
  "type": "GROUP",
  "target": "G1"
}
```

### `EndAlarmType`

- `SILENT`: a simple notification sound (similar to receiving a text message)  
- `LOUD`: regular alarm sound (full duration of chosen alarm sound)

---

## 🔌 Endpoints

All endpoints use `Content-Type: application/json` and UTF-8 encoding.

---

### List Scenarios

**POST** `/api/alarm/v1/scenario/list`

**Parameters:**

| Name               | Type            | Required | Default | Description                                          |
|--------------------|-----------------|----------|---------|------------------------------------------------------|
| username           | string          | Yes      | –       | API username                                         |
| password           | string          | Yes      | –       | API password                                         |
| customerId         | string          | Yes      | –       | Customer ID                                          |
| startedOrEndedAfter| ISO 8601 string | No       |         | Filter for scenarios started or ended after this date |
| scenarioConfigIds  | List<string>    | No       |         | Filter for specific scenario configurations          |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "startedOrEndedAfter": "2020-10-12T09:18:53.740Z",
  "scenarioConfigIds": [
    "0123-ABC-asdfqwerasdf-CONFIG1",
    "0123-ABC-asdfqwerasdf-CONFIG2"
  ]
}
```

Response:

HTTP 200 OK

```jsonc
{
  "result": "OK",
  "description": "",
  "startedOrEndedAfter": "2020-10-12T09:18:53.740Z",
  "startedOrEndedBefore": "2020-10-12T09:48:54.068Z",
  "scenarioConfigIds": [
    "0123-ABC-asdfqwerasdf"
  ],
  "scenarioPreviews": {
    // <ScenarioPreviewData>
  }
}
```

The value of `startedOrEndedBefore` can be used as `startedOrEndedAfter` for the next request.


---

### Query a Scenario

**POST** `/api/alarm/v1/scenario/query`

**Parameters:**

| Name        | Type   | Required | Default | Description     |
|-------------|--------|----------|---------|-----------------|
| username    | string | Yes      | –       | API username    |
| password    | string | Yes      | –       | API password    |
| customerId  | string | Yes      | –       | Customer ID     |
| scenarioId  | string | Yes      | –       | Scenario ID     |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "scenarioId": "123-ABC-asdfqwerasdf"
}
```

Response:

HTTP 200 OK

```jsonc
{
  "result": "OK",
  "description": "",
  "scenarioData": {
    // <ScenarioData>
  }
}
```

---


### List Scenario Configurations

**POST** `/api/alarm/v1/scenario/config/list`

**Parameters:**

| Name         | Type         | Required | Default | Description                        |
|--------------|--------------|----------|---------|------------------------------------|
| username     | string       | Yes      | –       | API username                       |
| password     | string       | Yes      | –       | API password                       |
| customerIds  | List<string> | Yes      | –       | List of customer IDs               |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerIds": [
    "500027"
  ]
}
```

Response:

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "",
  "configs": [
    // <ScenarioConfigData>
  ]
}
```

---

### Trigger by Scenario Config ID

**POST** `/api/alarm/v1/scenario/trigger`

**Parameters:**

| Name               | Type                    | Required | Default | Description                             |
|--------------------|-------------------------|----------|---------|-----------------------------------------|
| username           | string                  | Yes      | –       | API username                            |
| password           | string                  | Yes      | –       | API password                            |
| customerId         | string                  | Yes      | –       | Customer ID                             |
| scenarioConfigId   | string                  | Yes      | –       | Scenario configuration ID               |
| additionalText     | string                  | No       |         | Optional message                        |
| additionalRecipients | List<RecipientConfiguration> | No |         | Additional recipients to be alerted     |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "scenarioConfigId": "32849abcdef23343",
  "additionalText": "Zusatzinfo vom User",
  "additionalRecipients": [
    {
      "type": "MSISDN",
      "target": "+4367712345678"
    }
  ]
}
```

Response:

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "",
  "scenarioId": "123-ABC-asdfqwerasdf"
}
```

---

### Trigger by Scenario Code

**POST** `/api/alarm/v1/scenario/trigger/code`

**Parameters:**

| Name               | Type                    | Required | Default | Description                             |
|--------------------|-------------------------|----------|---------|-----------------------------------------|
| username           | string                  | Yes      | –       | API username                            |
| password           | string                  | Yes      | –       | API password                            |
| customerId         | string                  | Yes      | –       | Customer ID                             |
| code               | string                  | Yes      | –       | Scenario short code                     |
| additionalText     | string                  | No       |         | Optional message                        |
| additionalRecipients | List<RecipientConfiguration> | No |         | Additional recipients to be alerted     |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "code": "V1",
  "additionalText": "Zusatzinfo vom User",
  "additionalRecipients": [
    {
      "type": "MSISDN",
      "target": "+4367712345678"
    }
  ]
}
```

Response:

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "",
  "scenarioId": "123-ABC-asdfqwerasdf"
}
```

---

### Send Update to an Active Scenario

**POST** `/api/alarm/v1/scenario/{scenarioId}/updates`

**Path Variable:** `{scenarioId} - can be extracted from the response when triggering by [scenario config id](#-trigger-by-scenario-config-id) or [scenario code](#-trigger-by-scenario-code), alternatively, can be obtained when [listing scenarios](#-list-scenarios).

**Parameters:**

| Name        | Type   | Required | Default | Description                 |
|-------------|--------|----------|---------|-----------------------------|
| username    | string | Yes      | –       | API username                |
| password    | string | Yes      | –       | API password                |
| customerId  | string | Yes      | –       | Customer ID                 |
| text        | string | Yes      | –       | Update message text         |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "text": "Fire detected near warehouse. Please evacuate."
}
```

Response:

HTTP 200 OK

```json
{
  "result": "OK",
  "description": ""
}
```

---

### Terminate Scenario

**PATCH** `/api/alarm/v1/scenario/{scenarioId}`

**Path Variable:** `{scenarioId}` - can be extracted from the response when triggering by [scenario config id](#-trigger-by-scenario-config-id) or [scenario code](#-trigger-by-scenario-code), alternatively, can be obtained when [listing scenarios](#-list-scenarios).

**Parameters:**

| Name               | Type         | Required | Default | Description                                   |
|--------------------|--------------|----------|---------|-----------------------------------------------|
| username           | string       | Yes      | –       | API username                                  |
| password           | string       | Yes      | –       | API password                                  |
| customerId         | string       | Yes      | –       | Customer ID                                   |
| terminationMessage | string       | No       |         | Message shown on termination                  |
| endAlarmType       | EndAlarmType | No       | SILENT  | Type of sound played when alarm is terminated |


**Example**

Request:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "terminationMessage": "Incident resolved. No further action needed.",
  "endAlarmType": "LOUD"
}
```

Response:

HTTP 200 OK

```json
{
  "result": "OK",
  "description": ""
}
```

---


## ❗ Error Codes

| Code | Description                    |
|------|--------------------------------|
| 400  | Bad Request — malformed JSON   |
| 401  | Unauthorized — invalid login   |
| 403  | Forbidden — insufficient rights |














