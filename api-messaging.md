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

> `SAFEREACH` will automatically determine the best message channel which will be the app in most cases

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

HTTP 200 OK

```json
{
  "result": "OK",
  "descirption": ""
}
```

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions
