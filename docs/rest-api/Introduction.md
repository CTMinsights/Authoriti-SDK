## The Authoriti Rest API can be used to create, edit & delete customers, manage Invitation Codes & perform Permission Code validations.


### Authorization

All methods require authentication. Authentication requires passing a Bearer token in the Authorization header. The login API grants the bearer token.

### Login

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
