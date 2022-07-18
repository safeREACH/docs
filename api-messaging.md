# Messaging API

## Version

- v1.0: Initial draft (2021-11-25)

## General

### Encoding

UTF-8 encoding shall be used at all times.

### Staging base URL

https://api-staging.safereach.com/blaulicht

### Live base URL

https://api.safereach.com/blaulicht

### Usage

The API is a simple transactional messaging API.

API requests are limited by the Fair Use Policy and configured account limits.

### Authentication

For API usage your `customerId`, an automatic alarm trigger user `username` and `password` are needed.

### Models

#### MessageTarget

- channel[]: List of `SMS`, `EMAIL`, `VOICE`, `SAFEREACH`
- msisdn: target phone number (mandatory for `SMS` and `VOICE`)
- email: target email address (mandatory for `EMAIL`)

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

Example:

```json
{
  "username": "myUser",
  "password": "mySuperSecretPwd",
  "customerId": "500027",
  "message": "Hello\n\nYou got some✉️",
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
- HTTP 404 Not Found: in case a target could not be found
- HTTP 409 Conflict: in case a target could not be notified by any of the specified channels

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

> Each channel for a specific target will be returned as separate Error response object.

#### Error Response - Example

```json
{
  "message": "Could not notify target because channel SAFEREACH was not defined for msisdn.",
  "channel": "SAFEREACH",
  "target": "+43123456789"
}
```
