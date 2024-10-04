# SSO (Single Sign-On) Endpoints

This document provides detailed information about the SSO-related endpoints in the AMove API. These endpoints allow you to manage and interact with various Single Sign-On providers.

## Endpoints

1. [Get SSO URL](#get-sso-url)
2. [Authenticate](#authenticate)
3. [SSO URL Import User](#sso-url-import-user)
4. [Get User User Groups](#get-user-user-groups)
5. [Setup Okta SSO](#setup-okta-sso)
6. [Get Okta SSO](#get-okta-sso)
7. [Update Okta SSO](#update-okta-sso)
8. [Delete Okta SSO](#delete-okta-sso)
9. [SAML ACS](#saml-acs)
10. [Setup SAML SSO](#setup-saml-sso)
11. [Get SAML SSO](#get-saml-sso)
12. [Update SAML SSO](#update-saml-sso)
13. [Delete SAML SSO](#delete-saml-sso)
14. [Setup Azure AD SSO](#setup-azure-ad-sso)
15. [Get Azure AD SSO](#get-azure-ad-sso)
16. [Update Azure AD SSO](#update-azure-ad-sso)
17. [Delete Azure AD SSO](#delete-azure-ad-sso)

## Get SSO URL

Retrieves the SSO URL for authentication.

- **URL**: `/api/v1/sso/sso_url`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "callbackUrl": "string",
  "username": "string"
}
```

### Response

```json
{
  "url": "string",
  "provider": "integer (enum)"
}
```

## Authenticate

Authenticates a user using the SSO Authorization Code.

- **URL**: `/api/v1/sso/authenticate`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "identifier": "string",
  "authorizationCode": "string",
  "callbackUrl": "string"
}
```

### Response

Returns a string (likely an authentication token).

## SSO URL Import User

Imports a user using SSO URL.

- **URL**: `/api/v1/sso/sso_url_import_user`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "callbackUrl": "string",
  "username": "string"
}
```

### Response

```json
{
  "url": "string",
  "provider": "integer (enum)"
}
```

## Get User User Groups

Retrieves user groups for a user.

- **URL**: `/api/v1/sso/get_user_usergroups`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| authorizationCode | string | The authorization code |
| callbackUrl | string | The callback URL |

### Response

```json
[
  {
    "user": {
      // User object
    },
    "userGroups": [
      // UserGroup objects
    ]
  }
]
```

## Setup Okta SSO

Sets up Okta SSO configuration.

- **URL**: `/api/v1/sso/setup_okta`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "clientId": "string",
  "clientSecret": "string",
  "openIdURL": "string",
  "active": "boolean"
}
```

### Response

Returns the created Okta SSO configuration object.

## Get Okta SSO

Retrieves the Okta SSO configuration.

- **URL**: `/api/v1/sso/get_okta`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns the Okta SSO configuration object.

## Update Okta SSO

Updates the Okta SSO configuration.

- **URL**: `/api/v1/sso/update_okta`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "clientId": "string",
  "clientSecret": "string",
  "openIdURL": "string",
  "active": "boolean"
}
```

### Response

Returns the updated Okta SSO configuration object.

## Delete Okta SSO

Deletes the Okta SSO configuration.

- **URL**: `/api/v1/sso/delete_okta`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

A successful deletion returns a `200 OK` status with no body.

## SAML ACS

SAML SSO Assertion handler. It will redirect to the Frontend callback URL.

- **URL**: `/api/v1/sso/saml_acs`
- **Method**: POST
- **Auth Required**: No

### Response

Redirects to the frontend callback URL.

## Setup SAML SSO

Sets up SAML SSO configuration.

- **URL**: `/api/v1/sso/setup_saml`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "certificate": "string",
  "spEntityId": "string",
  "idPSSOURL": "string",
  "active": "boolean"
}
```

### Response

Returns the created SAML SSO configuration object.

## Get SAML SSO

Retrieves the SAML SSO configuration.

- **URL**: `/api/v1/sso/get_saml`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns the SAML SSO configuration object.

## Update SAML SSO

Updates the SAML SSO configuration.

- **URL**: `/api/v1/sso/update_saml`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "certificate": "string",
  "spEntityId": "string",
  "idPSSOURL": "string",
  "active": "boolean"
}
```

### Response

Returns the updated SAML SSO configuration object.

## Delete SAML SSO

Deletes the SAML SSO configuration.

- **URL**: `/api/v1/sso/delete_saml`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

A successful deletion returns a `200 OK` status with no body.

## Setup Azure AD SSO

Sets up Azure AD SSO configuration.

- **URL**: `/api/v1/sso/setup_entraId`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "clientId": "string",
  "clientSecret": "string",
  "openIdURL": "string",
  "active": "boolean"
}
```

### Response

Returns the created Azure AD SSO configuration object.

## Get Azure AD SSO

Retrieves the Azure AD SSO configuration.

- **URL**: `/api/v1/sso/get_entraId`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns the Azure AD SSO configuration object.

## Update Azure AD SSO

Updates the Azure AD SSO configuration.

- **URL**: `/api/v1/sso/update_entraId`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "clientId": "string",
  "clientSecret": "string",
  "openIdURL": "string",
  "active": "boolean"
}
```

### Response

Returns the updated Azure AD SSO configuration object.

## Delete Azure AD SSO

Deletes the Azure AD SSO configuration.

- **URL**: `/api/v1/sso/delete_entraId`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

A successful deletion returns a `200 OK` status with no body.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get SSO URL

<details>
<summary>Python</summary>

```python
import requests
import json

url = "https://api.amove.com/api/v1/sso/sso_url"
headers = {
    "Content-Type": "application/json"
}
data = {
    "callbackUrl": "https://your-app.com/callback",
    "username": "user@example.com"
}

response = requests.post(url, headers=headers, data=json.dumps(data))

if response.status_code == 200:
    sso_data = response.json()
    print(f"SSO URL: {sso_data['url']}")
    print(f"Provider: {sso_data['provider']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const data = {
  callbackUrl: 'https://your-app.com/callback',
  username: 'user@example.com'
};

fetch('https://api.amove.com/api/v1/sso/sso_url', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
})
.then(response => {
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
})
.then(ssoData => {
  console.log(`SSO URL: ${ssoData.url}`);
  console.log(`Provider: ${ssoData.provider}`);
})
.catch(error => {
  console.error('Error:', error);
});
```

</details>

<details>
<summary>C#</summary>

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

class Program
{
    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        {
            var data = new
            {
                callbackUrl = "https://your-app.com/callback",
                username = "user@example.com"
            };

            var content = new StringContent(JsonConvert.SerializeObject(data), Encoding.UTF8, "application/json");
            var response = await client.PostAsync("https://api.amove.com/api/v1/sso/sso_url", content);

            if (response.IsSuccessStatusCode)
            {
                var responseContent = await response.Content.ReadAsStringAsync();
                var ssoData = JObject.Parse(responseContent);
                Console.WriteLine($"SSO URL: {ssoData["url"]}");
                Console.WriteLine($"Provider: {ssoData["provider"]}");
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
                Console.WriteLine(await response.Content.ReadAsStringAsync());
            }
        }
    }
}
```

</details>

For more detailed examples and usage of other endpoints, please refer to our [Examples Directory](examples/README.md).

