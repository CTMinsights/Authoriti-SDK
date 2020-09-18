## Authorit REST API

### Customer Management APIs ###

All customer management methods require authentication. Authentication requires passing a Bearer token in the Authorization header. The login API grants the bearer token.

Login

```curl
POST /api/1/auth/login
{
    email: "string",
    password: "string"
}

Possible response:

1. 200 Ok

{
    token: "string",
    first_name: "string",
    last_name: "string",
    profile_image_url: "string",
    id: "string",
    role: "string",
    portals: [{
        customer_name: "string",
        customer_subdomain: "string"
    }]
}

2. 400 Bad Request
{
    "message": "Invalid email and/or password. Please try again."
}
```

Create customer

Creating a customer is a two step process. The GET Areas API should be called to get the IDs of the Area which the customer will have enabled. Finally the create customer API should be called.

1. Get Areas

```curl
GET /api/1/areas

Response:

{
    id: "uuid",
    name: "string",
    group: "integer"
}
```

2. Create customer

| Parameter                | Description                                                                                                    |
|--------------------------|----------------------------------------------------------------------------------------------------------------|
| customer_name            | The name of the customer                                                                                       |
| customer_subdomain       | The subdomain at which the portal will be available                                                            |
| customer_logo            | The customer logo                                                                                              |
| allow_non_email_username | If set to true, the portals username can be non-email strings, otherwise usernames must be valid email strings |
| theme                    | Must be 6 character RGB values. Describes the theme of the portal                                              |
| domains                  | If allow_non_email_username is set to false, possible usernames can only have domains from this array          |
| areas                    | The areas to enable for this customer. Must be UUID's returned from GET Areas API                              |
| admin_email              | The admin of this portal. This email will get a validation email                                               |


```curl
POST /api/1/customers
{
    "customer_name": "string",
    "customer_subdomain": "string",
    "customer_logo": "string",
    "allow_non_email_username": "string",
    "admin_email": "string",
    "theme": {
        "navigation_bar_color": "string",
        "theme_color": "string",
        "primary_color": "string",
        "warning_color": "string",
        "danger_color": "string",
        "success_color": "string"
    },
    "domain": ["string"],
    "areas": ["uuid"]
}

```

Edit Customer

If the areas enabled for the customer needs to be edited, then GET Areas API must be called to get the Area IDs. A customer may be edited with the following API call, where the {{subdomain}} in the URL is the subdomain of the customer being edited. The body parameters are the same as that of create customer API call.

```curl
PUT /api/1/customers/{{subdomain}}
{
    "customer_name": "string",
    "customer_subdomain": "string",
    "customer_logo": "string",
    "allow_non_email_username": "string",
    "admin_email": "string",
    "theme": {
        "navigation_bar_color": "string",
        "theme_color": "string",
        "primary_color": "string",
        "warning_color": "string",
        "danger_color": "string",
        "success_color": "string"
    },
    "domain": ["string"],
    "areas": ["uuid"]
}
```

Delete Customer

A customer may be deleted with the following API call. {{subdomain}} in the query path is the subdomain of the customer being deleted.

```curl
DELETE /api/1/customers/{{suddomain}}
```

### Customer Management APIs ###

**Authentication**

All customer APIs require the subdomain of the customer to be in the query path. All methods require authentication. Authentication requires passing a Bearer token in the Authorization header. The login API grants the bearer token.

**Login API**

```curl
POST /api/1/portal/{{subdomain}}/auth/login
{
    username: "string",
    password: "string"
}

Response:

{
    token: "string"
}
```

**Invitation Code API**

Creating and Editing Invitation Codes require Purpose (Area) Ids. The portal details API may be used to get list of purposes enabled for this customer

```curl
GET /api/1/portal/{{subdomain}}/areas

Response
[{
    "id": "uuid",
    "name": "string"
}]
```

| Parameter    | Description                                                                                                                    |
|--------------|--------------------------------------------------------------------------------------------------------------------------------|
| code         | The invitation code                                                                                                            |
| expires_at   | A date time string indicating the expiration time                                                                              |
| requires_dlv | A boolean variable indicating whether using this invitation code would require the user to perform Driver's License Validation |
| purposes     | Array of area (purpose) Ids                                                                                                    |

1. Create Invitation Code

To create an invitation code the following API may be used

```curl
POST /api/1/portal{{subdomain}}/invites
{
    code: "string",
    expires_at: "string",
    requires_dlv: "boolean",
    purposes: ["uuid"]
}
```

2. Update Invitation Code

Updating Invitation Code requires the ID of the invitation code being updated

```curl
PUT /api/1/portal/{{subdomain}}/invites/{{invitation_code_id}}
{
    expires_at: "string",
    requires_dlv: "boolean",
    purposes: ["uuid"]
}
```

3. Delete Invitation Code

Deleting Invitation Code requires the ID of the invitation code being deleted

```curl
DELETE /api/1/portal/{{subdomain}}/invites/{{invitation_code_id}}
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
 POST api/1/portal/{{subdomain}}/bulk-upload
 {
     csv: "link to csv"
 }
 ```

 ### Validation API ###


 The validation API is hosted in AWS as a lambda function and sits behind AWS API Gateway. All validation requests should be invoked using the following URL

 Validation Request URL: https://5t1bndpu88.execute-api.us-east-1.amazonaws.com

 Each purpose takes it's own payload (CTI - Companion Transmitted Information), but all the purposes require the following parameters

 | Parameter          | Description                                                                                                                                                |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| accountId          | A SHA256 hash of the account id, lowercased and stripped of all non-alphanumeric characters and space.                                                     |
| passcode           | The 10 digit Permission Code                                                                                                                               |
| customerImportOnly | A flag indicating whether to validate non customer imported users. If set to true, only users who were uploaded by the calling customer would be uploaded. |

To authenticate validation requests, the license key must be sent as Authorization header in the following format

Authorization: AUTHORITI license_key

The response of validation requests is the following object

```json
{
    "passed": "boolean",
    "message": "string"
}
```

 1. Validating "Manage Account" purpose

 ```curl
 POST /dev/api/v1/passcode/validate
 {
     accountId: "string",
     customerImportOnly: "boolean",
     passcode: "string",
     datatypes: ["string"],
     schema: "integer",
 }

 datatypes is a string of datatypes against which the Permission Code (passcode) is being validated.
 ```

 2. Validating "Transfer Funds" purpose

 ```curl
POST /dev/api/v1/passcode/validate
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

 3. Validating "Escrow" purpose

 ```curl
 POST /dev/api/v1/passcode/validate
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

 4. Validating "eHealth-Certificate" purpose

  ```curl
 POST /dev/api/v1/passcode/validate
{
    accountId: "string",
    customerImportOnly: "boolean",
    passcode: "string",
    schema: "integer",
    secret: "string"
}

secret: SHA256 hash of the Patient's ID Number stripped of all non-alphanumeric characters and lowercased before hashing.

```

5. Validating "Insurance Claim" and "Share Personal Information" purposes

  ```curl
 POST /dev/api/v1/passcode/validate
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

