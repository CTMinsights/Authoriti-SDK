## Authorit REST API

### Customer Management APIs

All customer management methods require authentication. Authentication requires passing username:password as a base64 encoded string in the authorization header. `Authorization: Basic b64Encode(username:password)`

Create customer

A new customer can be created by calling the Create Customer API

| Parameter        | Description                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------- |
| name             | The name of the customer                                                                    |
| phone            | The phone number of the customer                                                            |
| email            | The email of the customer                                                                   |
| industry         | The industry of the customer                                                                |
| location         | The state in which the customer is located                                                  |
| parent_location  | The country in which the customer is located                                                |
| dataTypes        | Array of data types                                                                         |
| callCenter       | A boolean value indicated whether customers call center has Permission Code support enabled |
| callCenterNumber | Customers call center number                                                                |

```curl
POST /api/v1/customers
{
    "name": "string",
    "phone": "string",
    "email": "string",
    "industry": "number", // a two digit code indicating the industry
    "location": "number", // a two digit code indicating the location
    "parent_location": "USA",
    "dataTypes": ["string"],
    "callCenter": "boolean",
    "callCenterNumber": "string"
}
```

The create customer API returns the following response

```json
{
  "licenseKey": "string",
  "customerId": "uuid"
}
```

This licenseKey must be used when validating Permission Codes for this customer. The customer ID is needed to make edits to the customer.

Edit Customer

A customer can be edited by calling the edit customer API. The body parameters are the same as that of create customer API call.

```curl
POST /api/v1/customers/{{customerId}}
{
    "name": "string",
    "phone": "string",
    "email": "string",
    "industry": "number", // a two digit code indicating the industry
    "location": "number", // a two digit code indicating the location
    "parent_location": "USA",
    "dataTypes": ["string"],
    "callCenter": "boolean",
    "callCenterNumber": "string"
}
```

Delete Customer

A customer may be deleted with the following API call. {{customerId}} in the query path is the customer id of the customer being deleted.

```curl
DELETE /api/v1/customers/{{customerId}}
```

### Customer Management APIs

**Invitation Code API**

1. Get List of Purposes

To get a list of available puropses the following API may be used

```curl
GET /api/v1/customers/purposes/labels
```

2. Create Invitation Code

To create an invitation code the following API may be used

| Parameter | Description                                                                                                                    |
| --------- | ------------------------------------------------------------------------------------------------------------------------------ |
| code      | The invitation code                                                                                                            |
| expiresAt | The expiration time (milliseconds)                                                                                             |
| skipDLV   | A boolean variable indicating whether using this invitation code would require the user to perform Driver's License Validation |
| customer  | Customer UUID                                                                                                                  |

```curl
POST /api/v1/customers/invites
{
    code: "string",
    expiresAt: "number",
    skipDLV: "boolean",
    customer: "string"
}
```

3. Update Invitation Code and associate purposes

Calling update Invitation Code API is required to associate purposes with an invitation code.

```curl
PUT /api/v1/customers/invites/{code}
{
    expiresAt: "number",
    requiresDLV: "string",
    purposes: ["uuid"]
}

// purposes is an array of UUIDs of purposes that should be downloaded with this invite code
```

**Bulk Upload**

To upload users into the system, so they can register using the Authoriti app, a formatted CSV with user credentials needs to be uploaded. The CSV must conform to the following format

```csv
user_id,user_password,account value,account name
```

- user_id: The sha256 hash of the user id
- user_password: The sha256 hash of the password
- account value: The SHA256 of the account value. The account value must be stripped of all non-alphanumeric characters including spaces before hashing
- account name: A string describing the account

Bulk Upload API call

```csv
POST api/v1/upload
{
    csv: "link to csv",
    "license": "customer license key"
}
```

### Validation API

The validation API is hosted in AWS as a lambda function and sits behind AWS API Gateway. All validation requests should be invoked using the following URL

Validation Request URL: `https://w2llo3thfe.execute-api.us-east-1.amazonaws.com`

Each purpose takes it's own payload (CTI - Companion Transmitted Information), but all the purposes require the following parameters

| Parameter          | Description                                                                                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| accountId          | A SHA256 hash of the account id, lowercased and stripped of all non-alphanumeric characters and space.                                                     |
| passcode           | The 10 digit Permission Code                                                                                                                               |
| customerImportOnly | A flag indicating whether to validate non customer imported users. If set to true, only users who were uploaded by the calling customer would be uploaded. |

To authenticate validation requests, the license key must be sent as Authorization header in the following format

`Authorization: AUTHORITI license_key;APIKEY api_key`

The response of validation requests is the following object

```json
{
  "passed": "boolean",
  "message": "string"
}
```

- Validating "Manage Account" purpose

```curl
POST /qa/api/v1/passcode/validate
{
    accountId: "string",
    customerImportOnly: "boolean",
    passcode: "string",
    datatypes: ["string"],
    schema: "integer",
}

datatypes is a string of datatypes against which the Permission Code (passcode) is being validated.
```

- Validating "Transfer Funds" purpose

```curl
POST /qa/api/v1/passcode/validate
{
   accountId: "string",
    customerImportOnly: "boolean",
    passcode: "string",
    schema: "integer",
    c: "string",
    e: "string"
}

c: sha256(payload.accountId.replace(/^0+/, '') + concatenate(payload.routingNumber, payload.accountNumber, payload.amount)
e: sha256(concatenate(payload.routingNumber, payload.accountNumber, payload.amount)).replace(/^0+/, '')
```

- Validating "Escrow" purpose

```curl
POST /qa/api/v1/passcode/validate
{
   accountId: "string",
   customerImportOnly: "boolean",
   passcode: "string",
   schema: "integer",
   c: "string",
   e: "string",
   h: "string"
}

c: sha256(payload.accountId.replace(/^0+/, '') + concatenate(payload.routingNumber, payload.accountNumber, payload.amount)
e: sha256(concatenate(payload.routingNumber, payload.accountNumber, payload.amount)).replace(/^0+/, '')
h: sha256(payload.accountId.replace(/^0+/, '') + concatenate(payload.routingNumber, payload.accountNumber, payload.amount, payload.transactionId)).replace(/^0+/, '')
```

- Validating "eHealth-Certificate" purpose

```curl
POST /qa/api/v1/passcode/validate
{
  accountId: "string",
  customerImportOnly: "boolean",
  passcode: "string",
  schema: "integer",
  secret: "string"
}

secret: SHA256 hash of the Patient's ID Number stripped of all non-alphanumeric characters and lowercased before hashing.

```

- Validating "Insurance Claim" and "Share Personal Information" purposes

```curl
POST /qa/api/v1/passcode/validate
{
  accountId: "string",
  customerImportOnly: "boolean",
  passcode: "string",
  schema: "integer",
  datatypes: ["string"],
  secret: "string"
}

datatypes: array of Claim Types the Permission Code is being validated against
secret: SHA256 hash of the Patient's ID Number stripped of all non-alphanumeric characters and lowercased before hashing.
```
