# Scenario API

## Version

- v1.0: Initial draft (2017-02-19)
- v1.1: Added `additionalText` property (2018-12-14)
- v1.2: Deprecated `LIVE` and `PRACTICE` trigger types (2020-10-12)
- v1.3: Added new scenario list list endpoint with `cutOff` and `configId` filters (2020-10-12)

## General

### Encoding

UTF-8 encoding shall be used at all times.

### Staging base URL

https://api-staging.safereach.com/blaulicht

### Live base URL

https://api.safereach.com/blaulicht

### Usage

The API extends the possibility for triggering scenarios in addition to the [safeREACH Cockpit](https://cockpit.safereach.com/) and [safeREACH App](https://safereach.com/en/emergency-notification-system/alert-app/).

API requests are limited by the Fair Use Policy.

### Authentication

For API usage your `customerOrGroupId`, an automatic alarm trigger user `username` and `password` are needed. Credentials for the user and scenario api cannot be shared.

### Models

#### ScenarioConfigData

- scenarioConfigId: Die ID einer Konfiguration
- customerId: Die Kundennummer zu der dieser Alarm gehört
- name: Der Name einer Konfiguration

**Example:**

```json
{
  "scenarioConfigId": "32849abcdef23343",
  "customerId": "500027",
  "name": "Szenario 1"
}
```

#### ScenarioPreviewData

- scenarioId
- scenarioConfigId
- authorId
- authorName
- creationDate
- startDate
- endDate

**Example:**

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

#### ScenarioData

- scenarioId
- scenarioConfigId
- authorId
- authorName
- usersAlertedCount
- creationDate
- startDate
- endDate
- alarms: List of alarm objects

**Example:**

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
    AlarmData
  ]
}
```

## List scenario configurations

_**/api/alarm/v1/scenario/config/list**_

**Method:** POST
**Header:** `Content-Type: application/json`

#### Request payload

- username: string - mandatory
- password: string - mandatory
- customerIds: List<string> - mandatory

Example:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerIds": [
    "500027"
  ]
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "descirption": "",
  "configs": [
    <ScenarioConfigData>
  ]
}
```

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

## Trigger a scenario via its configId

_**/api/alarm/v1/scenario/trigger**_

**Method:** POST  
**Header:** `Content-Type: application/json`

- username: string - mandatory
- password: string - mandatory
- customerId: string - mandatory
- scenarioConfigId: string - mandatory
- additionalText: string - optional

#### Example request

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "scenarioConfigId": "32849abcdef23343",
  "additionalText": "Zusatzinfo vom User"
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "descirption": "",
  "scenarioId": "123-ABC-asdfqwerasdf"
}
```

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

## Trigger a scenario via its code

_**/api/alarm/v1/scenario/trigger/code**_

**Method:** POST  
**Header:** `Content-Type: application/json`

- username: string - mandatory
- password: string - mandatory
- customerId: string - mandatory
- scenarioConfigId: string - mandatory
- additionalText: string - optional

#### Example request

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "code": "V1",
  "additionalText": "Zusatzinfo vom User"
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "descirption": "",
  "scenarioId": "123-ABC-asdfqwerasdf"
}
```

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

## Query a scenario

_**/api/alarm/v1/scenario/query**_

**Method:** POST
**Header:** `Content-Type: application/json`

- username: string - mandatory
- password: string - mandatory
- customerId: string - mandatory
- scenarioId: string - mandatory

#### Example request

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "scenarioId": "123-ABC-asdfqwerasdf"
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "descirption": "",
  "scenarioData": {
    ScenarioData
  }
}
```

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

## List scenarios

_**/api/alarm/v1/scenario/list**_

**Method:** POST
**Header:** `Content-Type: application/json`

- username: string - mandatory
- password: string - mandatory
- customerId: string - mandatory
- startedOrEndedAfter: date - optional
- scenarioConfigIds: List<string> - optional

#### Example request

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

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "descirption": "",
  "startedOrEndedAfter": "2020-10-12T09:18:53.740Z",
  "startedOrEndedBefore": "2020-10-12T09:48:54.068Z",
  "scenarioConfigIds": [
    "0123-ABC-asdfqwerasdf"
  ],
  "scenarioPreviews": {
    <ScenarioPreviewData>
  }
}
```

The value of `startedOrEndedBefore` should be used as `startedOrEndedAfter` for the next request.

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

