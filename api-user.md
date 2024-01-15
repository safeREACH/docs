# User API

## Version

- v2.0: Initial Draft (2018-09-30)
- v2.1: Small API adjustments and (2019-03-01)
- v2.2: Add merge flag for import requests (2019-08-20)
- v2.3: Rename `customerOrGroupId` to `customerId` in data objects (2020-03-05)
- v2.4: Add optional `deleteOnlyExternal` flag for import requests (2020-04-09)
- v2.5: Fix typo: `useExternalId` should be named `externalId` (2020-04-14)
- v2.6: Add optional `channels` attribute to `PublicRecipientData` to restrict / configure notification channels
(2021-10-01)
- v2.7: Add documentation for recipient deletion endpoints (2022-04-11)
- v2.8: Add unique constraint for msisdns and email addresses (2022-10-18)
- v2.9: Add `recipientsToDelete` and `groupsToDelete` for explicit deletion of users (2023-07-11)
- v2.10: Add `functions` to recipient import and `functions` import (2024-01-15)

## General

### Encoding

UTF-8 encoding shall be used at all times.

### Staging base URL

https://api-staging.safereach.com/blaulicht

### Live base URL

https://api.safereach.com/blaulicht

### Usage

The API extends the possibility for managing users and groups on
the [safeREACH web platform](https://start.safereach.com/).

API requests are limited by the Fair Use Policy.

### Authentication

For API usage your `customerOrGroupId`, an API user `username` and `password` are needed. Credentials for the user and
scenario api cannot be shared.

### Use cases / flags

The purpose of the API is to export recipients and their assigned groups to external data sources for comparison. The
identification of a recipient / group is done via a UUID version 4, referenced as `<UUIDv4>`.

- recipient and group identification via an `externalId` (_VARCHAR(255)_)
- a `dryRun` flag, which allows data-set comparison and pre-import validation
- a `partial` flag, which allows partial imports (no deletion of unreferenced elements)
- a `merge` flag, which allows an initial merge with existing data (makes migration a lot easier)
- a `deleteOnlyExternal` flag, which will only delete recipients with an externalId if `partial` is `false` and
  externalIds are used
- a `recipientsToDelete` list of users `external ids` to delete
- a `groupsToDelete` list of `groups` to delete

### Models

#### RecipientData

- id: string - mandatory - `<UUIDv4>` - for new records and for external id usage leave empty
- externalId: string - optional - for id usage leave empty; mandatory for external id usage
- customerId: string - mandatory
- msisdn: string - mandatory - phone number with country code prefix in
  the [ISO E.164](https://en.wikipedia.org/wiki/E.164) format: `+4366412345678` - has to be unique across all recipients (including existing ones)
- givenname: string - mandatory - first name
- surname: string - mandatory - last name
- email: string - optional - e-mail address, has to be unique across all recipients (including existing ones)
- comment: string- optional - e.g. division in organisation or other additional information
- groups: list of objects of the type `RecipientGroupParticipationData` - mandatory can be an empty list
- channels: list of `Channel` - optional - restricted / configured notification channels
- functions: list of objects of type `FunctionParticipationData` - optional (by default null)

#### RecipientBaseData

- id: string - optional - `<UUIDv4>` - mandatory for id usage
- externalId: string - optional - mandatory for external id usage
- customerId: string - mandatory - can be set to "*" if every available customer id should be targeted

#### RecipientGroupParticipationData

- groupId: string - mandatory e.g. "G1"

#### FunctionParticipationData

- functionCode: string - mandatory e.g. "F1"

#### Channel

> If now automatic channel restriction is preferred it is recommended to leave the channels attribute `null`.
> Submitting and empty list is handled the same as `null`.
> `SMS` and `VOICE` are only sent if the safeREACH app is not installed or not reachable.

- `SMS`
- `PUSH`
- `VOICE`
- `EMAIL`

#### GroupData

- id: string - mandatory - `<UUIDv4>` - for new records or external id usage leave empty
- externalId: string - optional - for new records or id usage leave empty
- customerId: string - mandatory
- groupId: string - mandatory - group Id - the groupId has to start with a `G` followed by an int between G0 and
  G999999 - The groupId can't be changed once it was created - only the name can be updated
- name: string - mandatory - name of the group

#### FunctionData

- id: string - mandatory - `<UUIDv4>` - for new records or external id usage leave empty
- externalId: string - optional - for new records or id usage leave empty
- customerId: string - optional
- functionCode: string  - mandatory - the functionCode has to start with a `F` followed by an int between F0 and
  F999999 - The functionCode can't be changed once it was created - only the name can be updated
- name: string - mandatory

### Import recipients - JSON

_**/api/public/v1/recipient/import**_

With an HTTP POST request with the header: `Content-Type: application/json` recipients can be imported.

- dryRun: boolean - optional - default `false`; defines if only a data-set comparison should be made (true) or if data
  should also be imported (false)
- externalId: boolean - optional - default `false`; enables the use of externalIds or safeREACH UUIDs
- partial: boolean - optional - default `false`; defines if records missing in the import data should be deleted from
  the existing data (false)
- merge: boolean - optional - default `false`; defines if existing entries should be merged based on the recipients
  msisdn. The `externalId` is mandatory and will be added to the recipient. Current group assignment will not be
  deleted, but can be merged with new groups. Also, the comment will not be overwritten.
- deleteOnlyExternal: boolean - optional - default `false`; defines if only recipients with an `externalId` should be
  considered for deletion
- recipients: list of objects of the type `RecipientData` - recipients
- recipientsToDelete: list of strings (list of `externalId`) - optional - default `null` - only working when `partial` is `true` and `merge` is `false`
- groupsToDelete: list of strings (list of `groupId` - see description of `groupId` above in `GroupData`) - optional - default `null` - only working when `partial` is `true` and `merge` is `false`

#### Example request

```json
{
  "customerOrGroupId": "500027",
  "username": "import",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": false,
  "partial": false,
  "merge": false,
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+436641234567890",
      "givenname": "Max",
      "surname": "Mustermann",
      "email": null,
      "comment": "Division 1",
      "groups": [
        {
          "groupId": "G1"
        },
        {
          "groupId": "G2"
        }
      ],
      "functions": [
        {
          "functionCode": "F1"
        },
        {
          "functionCode": "F2"
        }
      ]
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+436761234567890",
      "givenname": "Martina",
      "surname": "Musterfrau",
      "email": "martina.musterfrau@example.com",
      "comment": "Division 2",
      "groups": [],
      "channels": [
        "PUSH",
        "EMAIL",
        "SMS",
        "VOICE"
      ]
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+491901234567890",
      "givenname": "Erwin",
      "surname": "Email",
      "email": "erwin.email@example.com",
      "comment": "Only receives emails",
      "groups": [],
      "channels": [
        "EMAIL"
      ]
    }
  ]
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "description": null,
  "created": 1,
  "updated": 2,
  "deleted": 3,
  "merged": 0,
  "request": {
    "dryRun": true,
    "externalId": false,
    "partial": false,
    "merge": false
  }
}
```

The following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions
- HTTP 409 Conflict: input data conflicting with current data set; see the description of the response for more
  information

## Import recipients - CSV

_**/api/public/v1/recipient/import**_

The following query parameters can be set:

- dryRun: boolean - optional - default `false`
- externalId: boolean - optional - default `false`
- partial: boolean - optional - default `false`
- merge: boolean - optional - default `false`
- deleteOnlyExternal: boolean - optional - default `false`

The following headers have to be set:

- `Content-Type: text/csv`
- `X-CustomerId: 500027`
- `X-Username: myUser`
- `X-Password: mySuperSecretPwd`

The following columns have to be separated by `;`

- id: string - mandatory - `<UUIDv4>` - for new records or external id usage leave empty
- externalId: string - optional - for new records or id usage leave empty
- customerId: string - mandatory
- givenname: string - mandatory
- surname: string - mandatory
- msisdn: string - mandatory - has to be unique across all recipients (including existing ones)
- email: string - optional - has to be unique across all recipients (including existing ones)
- comment: string - optional - e.g. division ()

After "comment" group participation is listed by `groupId`.
`1` indicates group participation. `0` indicates no participation.

> As lines are separated by a newline, which will not be escaped, it is not possible to import values that contain a `;` or `\n`.

#### Example request

```csv
id;externalId;customerId;givenname;surname;msisdn;email;comment;G1;G2;G3 
<UUIDv4>;;500027;Max;Mustermann;+4366412345678;;;1;1;0 
<UUIDv4>;;500027;Martina;Musterfrau;+4367612345678;martina.musterfrau@example.com;;0;0;0
```

## Import groups - JSON

_**/api/public/v1/group/import**_

With an HTTP POST request with the header: `Content-Type: application/json` groups can be imported.

- customerOrGroupId: string - mandatory - customerOrGroupId
- username: string - mandatory - username
- password: string - mandatory - password
- dryRun: boolean - optional - default `false`
- externalId: boolean - optional - default `false`
- partial: boolean - optional - default `false`
- groups: List of objects of type GroupData - groups

#### Example request

```json
{
  "customerOrGroupId": "500027",
  "username": "import",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": false,
  "partial": false,
  "merge": false,
  "groups": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "groupId": "G1",
      "name": "All employees"
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "groupId": "G2",
      "name": "Crises managment"
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "groupId": "G3",
      "name": "IT"
    }
  ]
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "description": null,
  "created": 1,
  "updated": 2,
  "deleted": 3,
  "merged": 0,
  "request": {
    "dryRun": true,
    "externalId": false,
    "partial": false,
    "merge": false
  }
}
```

Following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions
- HTTP 409 Conflict: input data conflicting with current data set; see the description of the response for more
  information

## Import groups - CSV

_**/api/public/v1/group/import**_

The following query parameters can be set:

- dryRun: boolean - optional - default `false`
- externalId: boolean - optional - default `false`
- partial: boolean - optional - default `false`
- merge: boolean - optional - default `false`
- deleteOnlyExternal: boolean - optional - default `false`

The following headers have to be set:

- `Content-Type: text/csv`
- `X-Username: myUser`
- `X-Password: mySuperSecretPwd`

The following columns need to be separated by `;`

- id: string - mandatory - `<UUIDv4>` - for new records or external id usage leave empty
- externalId: string - optional - for new records or id usage leave empty
- customerId
- groupId
- name

#### Example request

```csv
id;exteranlId;customerId;groupId;name
<UUIDv4>;externalId;500027;G1;Sirenenalarm
<UUIDv4>;externalId;500027;G2;Stiller Alar
<UUIDv4>;externalId;500027;G3;Alle Kameraden
```

## Import functions - JSON

_**/api/public/v1/functions/{customerId}/import**_

With an HTTP POST request with the header `Content-Type: application/json` functions can be imported.

- customerOrGroupId: string - mandatory - customerOrGroupId
- username: string - mandatory - username
- password: string - mandatory - password
- dryRun: boolean - optional - default `false`
- externalId: boolean - optional - default `false`
- partial: boolean - optional - default `false`
- functions: List of objects of type FunctionData - groups

#### Example request

```json
{
  "customerOrGroupId": "500027",
  "username": "import",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": false,
  "partial": false,
  "merge": false,
  "functions": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "functionCode": "F1",
      "name": "Management"
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "groupId": "F2",
      "name": "Crises"
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "groupId": "F3",
      "name": "IT"
    }
  ]
}
```

#### Response

HTTP 200 OK

```json
{
  "result": "OK",
  "description": null,
  "created": 3,
  "updated": 0,
  "deleted": 0,
  "merged": 0,
  "request": {
    "dryRun": true,
    "externalId": false,
    "partial": false,
    "merge": false
  }
}
```

Following errors can occur:

- HTTP 400 BAD Request: malformed JSON request received
- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions
- HTTP 409 Conflict: input data conflicting with current data set; see the description of the response for more
  information


## Export recipients - CSV / JSON

_**/api/public/v1/group/{{customerOrGroupId}}/export**_

or

_**/api/public/v1/recipient/{{customerOrGroupId}}/export**_

The following headers have to be set:

- `X-Username: myUser`
- `X-Password: mySuperSecretPwd`
- `Accept: text/csv` | `Accept: application/json`

Groups and recipients can be exported via JSON or CSV.

### response

HTTP 200 OK

```json
{
  "result": "OK",
  "description": null,
  "recipients": [
    <RecipientData>
  ]
}
```

or

```json
{
  "result": "OK",
  "description": null,
  "groups": [
    <GroupData>
  ]
}
```

The CSV response will look like the import CSV examples.

The following errors can occur:

- HTTP 401 Unauthorized: invalid credentials
- HTTP 403 Forbidden: missing permissions

## Delete recipients

`DELETE` _**/api/public/v1/recipient**_

Delete multiple recipients by their internal or external ID or by groupId. You can use the `dryRun` flag to test a
deletion and preview which recipients would be deleted for a given request.

- customerOrGroupId: string - mandatory - customerOrGroupId
- username: string - mandatory - username
- password: string - mandatory - password
- dryRun: boolean - mandatory; defines if the query should only return possibly deleted data (true)
  or if that data should also be deleted (false)
- externalId: boolean - mandatory; enables the use of externalIds for deletion
- recipients: list of objects of the type `RecipientBaseData` - recipients to be deleted

### Delete a single recipient by internal id

#### Example request (dry run)

```json
{
  "customerOrGroupId": "500027",
  "username": "myUserName",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": false,
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027"
    }
  ]
}
```

#### Response (dry run)

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "OK",
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+436761234567890",
      "givenname": "John",
      "surname": "Doe",
      "email": "john.doe@example.com",
      "comment": "",
      "groups": [],
      "channels": [
        "SMS",
        "EMAIL"
      ]
    }
  ],
  "dryRun": true
}
```

### Delete multiple recipients by internal id

#### Example request (dry run)

```json
{
  "customerOrGroupId": "500027",
  "username": "myUserName",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": true,
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027"
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027"
    },
    ...
  ]
}
```

#### Response (dry run)

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "OK",
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+436761234567890",
      "givenname": "John",
      "surname": "Doe",
      "email": "john.doe@example.com",
      "comment": "",
      "groups": [],
      "channels": [
        "SMS",
        "EMAIL"
      ]
    },
    {
      "id": "<UUIDv4>",
      "externalId": "",
      "customerId": "500027",
      "msisdn": "+436760987654321",
      "givenname": "Jane",
      "surname": "Doe",
      "email": "jane.doe@example.com",
      "comment": "",
      "groups": [],
      "channels": [
        "SMS",
        "EMAIL"
      ]
    }
  ],
  "dryRun": true
}
```

### Delete a single recipient by external id

#### Example request (dry run)

```json
{
  "customerOrGroupId": "500027",
  "username": "myUserName",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": true,
  "recipients": [
    {
      "id": "",
      "externalId": "ABC-123",
      "customerId": "500027"
    }
  ]
}
```

#### Response (dry run)

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "OK",
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "ABC-123",
      "customerId": "500027",
      "msisdn": "+436761234567890",
      "givenname": "John",
      "surname": "Doe",
      "email": "john.doe@example.com",
      "comment": "",
      "groups": [],
      "channels": [
        "SMS",
        "EMAIL"
      ]
    }
  ],
  "dryRun": true
}
```

### Delete multiple recipients by the same external id

#### Example request (dry run)

```json
{
  "customerOrGroupId": "500",
  "username": "myUserName",
  "password": "mySuperSecretPwd",
  "dryRun": true,
  "externalId": true,
  "recipients": [
    {
      "id": "",
      "externalId": "ABC-123",
      "customerId": "*"
    }
  ]
}
```

#### Response (dry run)

HTTP 200 OK

```json
{
  "result": "OK",
  "description": "OK",
  "recipients": [
    {
      "id": "<UUIDv4>",
      "externalId": "ABC-123",
      "customerId": "500027",
      "msisdn": "+436761234567890",
      "givenname": "John",
      "surname": "Doe",
      "email": "john.doe@example.com",
      "comment": "",
      "groups": [],
      "channels": [
        "SMS",
        "EMAIL"
      ]
    },
    {
      "id": "<UUIDv4>",
      "externalId": "ABC-123",
      "customerId": "500654",
      "msisdn": "+436761234567890",
      "givenname": "John",
      "surname": "Doe",
      "email": "john.doe@example.com",
      "comment": "",
      "groups": [],
      "channels": [
        "SMS"
      ]
    }
  ],
  "dryRun": true
}
```

### Errors

If the API user does not have access to all the targeted recipients no recipients will be deleted and the following
response will be sent:

```json
{
  "result": "NOK",
  "description": "You don't have access to all requested recipients! No recipients were deleted.",
  "recipients": null,
  "dryRun": false
}
```
