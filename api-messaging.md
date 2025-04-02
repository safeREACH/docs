# Messaging API

## Version

- v1.0: Initial draft (2021-11-25)
- v1.1: Error response definition and sending mechanism (2022-07-18)
- v1.2: Replace SAFE_REACH channel name with PUSH (2022-07-22)
- v1.3: Add new field `type` to `MessageRequest` (2022-07-25)
- v1.4: Add new field `title` to `MessageRequest` (2022-07-25)
- v1.5: Add new field `language` to `MessageRequest` (2022-11-07)
- v1.6: Remove `fr`, and `it` from `language` option and add additional information for `language` field (2022-11-28)
- v1.7: Add properties for voice calls, `retries`, `timeout`, `delay` (2023-07-13)

## General

### Encoding

UTF-8 encoding shall be used at all times.

### Live base URL

https://api.safereach.com/blaulicht

### Usage

The API is a simple transactional messaging API.

API requests are limited by the Fair Use Policy and configured account limits.

### Authentication

For API usage your `customerId`, an automatic alarm trigger user `username` and `password` are needed.

### Models

#### MessageTarget

- channel[]: List of `SMS`, `EMAIL`, `VOICE`, `PUSH`
- msisdn: target phone number (mandatory for `SMS` and `VOICE`)
- email: target email address (mandatory for `EMAIL`)

> NOTE: PUSH is the channel via you can notify targets in the safeREACH app.

#### Sending mechanism

The channel array holds all communication channels which are available for the specific target. `safeREACH` will
send a message via all specified channels. In case the target is not reached via the defined channels, an error
is returned (see ErrorResponse).

**Example:**

```json
{
  "channel": [
    "SMS",
    "VOICE",
    "EMAIL"
  ],
  "msisdn": "+43123456789",
  "email": "target@acme.com"
}
```

## Send message

_**/api/messaging/v1/send**_

**Method:** POST
**Header:** `Content-Type: application/json`

#### Request payload

- username: string - mandatory
- password: string - mandatory
- customerId: string - mandatory
- message: string - mandatory
- type: string **(possible values: info, alarm)** - optional (when at least one of the `MessageTarget` objects has `PUSH` as channel type it
needs to be specified)
- title: string - optional (when at least one of the `MessageTarget` objects has `PUSH` as channel type it
  needs to be specified)
- language: enum - optional (valid values are: en, de)
- retries: number - optional, default is 2 (number of retries if user does not answer voice call)
- timeout: number (in seconds) - optional, default is 60 seconds (number of seconds to wait until user answer voice call)
- delay: number (in seconds) - optional, default is 300 seconds (number of seconds to wait until the retry voice call is made)

`NOTE:` The language property is *optional*. If it is not set, the language for the given customer is used. The current
version can only be used properly, if the specified targets are added in the safeREACH platform. Other than that, it is 
sufficient to specify `PUSH` as channel.

Example:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "message": "Hello\n\nYou got some✉️",
  "language": "en",
  "targets": [
    MessageTarget
  ]
}
```

#### Response

The response is a json object containing the following properties:

- result: string - OK, NOK, PARTIAL
  - OK: all targets were notified
  - NOK: no targets were notified
  - PARTIAL: only a subset of targets was notified
- errors: [ErrorResponse] - array of ErrorResponse objects (empty in case there are no errors)

HTTP 200 OK

```json
{
  "result": "OK",
  "errors": []
}
```

The following errors can occur:

- HTTP 400 Bad Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

**Response in case no target could be notified:**
```json
{
  "result": "NOK",
  "errors": [
    ErrorResponse
  ]
}
```

or

**Response in case a subset of targets could not be notified:**
```json
{
  "result": "PARTIAL",
  "errors": [
    ErrorResponse
  ]
}
```

#### Error Response

The error response is a json object containing the following properties:

- message: string - error description
- channel: string - channel which was not able to be used to notify target
- target: string - msisdn or email of the target which was not able to be reached
- status: integer - status code for the specific message target
  - 404 - in case the target was not found
  - 409 - in case a target could not be notified by any of the specified channels

> Each channel for a specific target will be returned as separate Error response object.

#### Error Response - Example

```json
{
  "message": "Could not notify target because channel SAFE_REACH was not defined for msisdn.",
  "channel": "SAFE_REACH",
  "target": "+43123456789",
  "status": 409
}
```

or 

```json
{
  "message": "Target with the specified msisdn was not found.",
  "channel": null,
  "target": "+43123456789",
  "status": 404
}
```
