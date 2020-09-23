## Create customer

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

## Edit Customer

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

## Delete Customer

A customer may be deleted with the following API call. {{customerId}} in the query path is the customer id of the customer being deleted.

```curl
DELETE /api/v1/customers/{{customerId}}
```
