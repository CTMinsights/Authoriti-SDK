## Create customer

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

## Edit Customer

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

## Delete Customer

A customer may be deleted with the following API call. {{subdomain}} in the query path is the subdomain of the customer being deleted.

```curl
DELETE /api/1/customers/{{suddomain}}
```
