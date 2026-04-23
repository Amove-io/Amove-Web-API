# SSO Endpoints

This document provides detailed information about the single-sign-on (SSO) endpoints in the AMove API. AMove supports three external identity providers per account:

- **Okta** — OIDC authorization-code flow
- **Entra ID** (formerly Azure AD) — OIDC authorization-code flow
- **SAML** — HTTP-POST SAML 2.0 via an ACS endpoint

A few endpoints on this controller are part of the login flow and are anonymous (`Auth Required: No`):

- [Get SSO URL](#get-sso-url) — build the identity-provider redirect URL given a username
- [Authenticate](#authenticate) — exchange an authorization code for an AMove JWT
- [SAML ACS](#saml-acs) — the SAML Assertion Consumer Service callback

The remaining endpoints are authenticated and are used by an account administrator to configure, update, and delete the SSO provider for the account.

## Endpoints

### Login Flow (Anonymous)

1. [Get SSO URL](#get-sso-url)
2. [Authenticate](#authenticate)
3. [SAML ACS](#saml-acs)

### Import Users

4. [Get SSO URL For Import User](#get-sso-url-for-import-user)
5. [Get User User Groups](#get-user-user-groups)

### Okta Configuration

6. [Setup Okta](#setup-okta)
7. [Get Okta](#get-okta)
8. [Update Okta](#update-okta)
9. [Delete Okta](#delete-okta)

### SAML Configuration

10. [Setup SAML](#setup-saml)
11. [Get SAML](#get-saml)
12. [Update SAML](#update-saml)
13. [Delete SAML](#delete-saml)

### Entra ID Configuration

14. [Setup Entra ID](#setup-entra-id)
15. [Get Entra ID](#get-entra-id)
16. [Update Entra ID](#update-entra-id)
17. [Delete Entra ID](#delete-entra-id)


## AuthProvider Values

Several endpoints in this controller return or key off the `AuthProvider` enum:

| Value | Name |
|---|---|
| 0 | AWSCognito |
| 1 | Google |
| 2 | EntraID |
| 4 | Okta |
| 8 | Saml |
| 16 | External |


## Get SSO URL

Given a username, resolves the account's configured SSO provider and returns a provider-specific authorization URL. The client redirects the user's browser to this URL to start the external login.

- **URL**: `/api/v1/sso/sso_url`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "username": "string",
  "callbackUrl": "string"
}
```

- `username` — the AMove username (email) of the user attempting to log in. Used to look up which account they belong to and therefore which SSO config to apply.
- `callbackUrl` — the URL of your application that the identity provider will redirect the user to after authentication.

### Response

```json
{
  "url": "string",
  "provider": "integer (AuthProvider)"
}
```

The `provider` field indicates which flow was selected — `4` (Okta), `8` (SAML), or `2` (EntraID). For Okta and Entra ID, the client receives an authorization code on the callback URL to exchange via [Authenticate](#authenticate). For SAML, the identity provider posts a response directly to [SAML ACS](#saml-acs).


## Authenticate

Exchanges an OIDC authorization code for an AMove JWT. Used for Okta and Entra ID flows (SAML users authenticate via [SAML ACS](#saml-acs) and do not call this endpoint).

- **URL**: `/api/v1/sso/authenticate`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "identifier": "string (uuid)",
  "authorizationCode": "string",
  "callbackUrl": "string"
}
```

- `identifier` — the AMove user id for the user whose authentication is being completed.
- `authorizationCode` — the OIDC `code` returned by the identity provider to your callback URL.
- `callbackUrl` — the same callback URL supplied to [Get SSO URL](#get-sso-url) (used for OIDC redirect-URI verification at the IdP).

### Response

A plain string containing the AMove JWT to be presented as `Authorization: Bearer <jwt>` on subsequent calls.

```
"EXAMPLE_JWT_VALUE"
```


## SAML ACS

SAML Assertion Consumer Service endpoint. The identity provider POSTs the SAML response here; the server validates the assertion against the stored X.509 certificate, issues an AMove JWT, and redirects the user's browser back to the client application's callback URL with the JWT as a query-string parameter.

- **URL**: `/api/v1/sso/saml_acs`
- **Method**: POST
- **Auth Required**: No
- **Content-Type**: `application/x-www-form-urlencoded`

### Request Body (Form)

| Field | Type | Description |
|-------|------|-------------|
| SAMLResponse | string | Base64-encoded SAML response from the identity provider. |
| RelayState | string | Base64-encoded JSON with `AccountID`, `Username`, and `CallbackUrl`. |

### Response

A `302 Found` redirect to `{callbackUrl}?jwt=<JWT>`. The client application parses the JWT out of the query string and uses it as the bearer token. If the SAML response fails signature validation or the account is not recognized, the server returns `400 Bad Request`.


## Get SSO URL For Import User

Generates an Okta authorization URL scoped for importing users and user groups from the identity provider into the account. This is the administrator-facing counterpart to [Get SSO URL](#get-sso-url).

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

`username` is ignored by this endpoint — the currently authenticated user is used instead. `callbackUrl` is the application URL the identity provider will redirect to.

### Response

```json
{
  "url": "string",
  "provider": 4
}
```


## Get User User Groups

Returns the users and user groups available for import from the account's configured identity provider (Okta or Entra ID). This is called after the administrator completes the Okta/Entra ID flow initiated by [Get SSO URL For Import User](#get-sso-url-for-import-user); for Entra ID, no authorization code is required (Graph API is queried using the configured app credentials).

- **URL**: `/api/v1/sso/get_user_usergroups`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| authorizationCode | string | OIDC authorization code from the import-user flow (Okta only). |
| callbackUrl | string | The same callback URL used when requesting the import URL (Okta only). |

### Response

```json
[
  {
    "user": {
      "id": "string (uuid)",
      "username": "string",
      "email": "string",
      "firstname": "string",
      "lastname": "string"
    },
    "userGroups": [
      { "id": "string (uuid)", "name": "string" }
    ]
  }
]
```


## Setup Okta

Stores the Okta SSO configuration for the authenticated user's account. The account id on the request body is always overridden server-side with the caller's account.

- **URL**: `/api/v1/sso/setup_okta`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "clientId": "string",
  "clientSecret": "string",
  "openIdURL": "string",
  "active": "boolean"
}
```

### Response

Returns the stored `OktaSSO` record.


## Get Okta

Returns the Okta SSO configuration for the authenticated user's account, if any.

- **URL**: `/api/v1/sso/get_okta`
- **Method**: GET
- **Auth Required**: Yes

### Response

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


## Update Okta

Updates the Okta SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/update_okta`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Same schema as [Setup Okta](#setup-okta), including the record `id`.

### Response

Returns the updated `OktaSSO` record.


## Delete Okta

Deletes the Okta SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/delete_okta`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

`200 OK` with an empty body.


## Setup SAML

Stores the SAML SSO configuration for the authenticated user's account. The account id on the request body is always overridden server-side with the caller's account.

- **URL**: `/api/v1/sso/setup_saml`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "certificate": "string",
  "spEntityId": "string",
  "idPSSOURL": "string",
  "active": "boolean"
}
```

- `certificate` — the identity provider's X.509 certificate (PEM text).
- `spEntityId` — the AMove service-provider entity id configured at the IdP.
- `idPSSOURL` — the identity provider's SSO endpoint URL.

### Response

Returns the stored `SamlSSO` record.


## Get SAML

Returns the SAML SSO configuration for the authenticated user's account, if any.

- **URL**: `/api/v1/sso/get_saml`
- **Method**: GET
- **Auth Required**: Yes

### Response

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


## Update SAML

Updates the SAML SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/update_saml`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Same schema as [Setup SAML](#setup-saml), including the record `id`.

### Response

Returns the updated `SamlSSO` record.


## Delete SAML

Deletes the SAML SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/delete_saml`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

`200 OK` with an empty body.


## Setup Entra ID

Stores the Entra ID (Azure AD) SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/setup_entraId`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "clientId": "string",
  "clientSecret": "string",
  "openIdURL": "string",
  "active": "boolean"
}
```

### Response

Returns the stored `EntraIDSSO` record.


## Get Entra ID

Returns the Entra ID SSO configuration for the authenticated user's account, if any.

- **URL**: `/api/v1/sso/get_entraId`
- **Method**: GET
- **Auth Required**: Yes

### Response

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


## Update Entra ID

Updates the Entra ID SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/update_entraId`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Same schema as [Setup Entra ID](#setup-entra-id), including the record `id`.

### Response

Returns the updated `EntraIDSSO` record.


## Delete Entra ID

Deletes the Entra ID SSO configuration for the authenticated user's account.

- **URL**: `/api/v1/sso/delete_entraId`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

`200 OK` with an empty body.


## Sample Code

### Start an SSO login

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/sso/sso_url",
    json={
        "username": "user@example.com",
        "callbackUrl": "https://app.example.com/sso/callback"
    }
)
data = response.json()
print("Redirect the browser to:", data["url"])
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/sso/sso_url", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    username: "user@example.com",
    callbackUrl: "https://app.example.com/sso/callback"
  })
});
const { url, provider } = await res.json();
window.location = url;
```

</details>

### Complete the OIDC callback (Okta or Entra ID)

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/sso/authenticate",
    json={
        "identifier": "00000000-0000-0000-0000-000000000000",
        "authorizationCode": "EXAMPLE_AUTH_CODE",
        "callbackUrl": "https://app.example.com/sso/callback"
    }
)
jwt = response.json()
print("JWT:", jwt)
```

</details>

### Configure Okta for the current account

<details>
<summary>C#</summary>

```csharp
using System.Net.Http.Json;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "EXAMPLE_TOKEN");

object body = new
{
    clientId = "YOUR_OKTA_CLIENT_ID",
    clientSecret = "YOUR_OKTA_CLIENT_SECRET",
    openIdURL = "https://your-org.okta.com/oauth2/default",
    active = true
};

HttpResponseMessage res = await client.PostAsJsonAsync(
    "https://api.amove.io/api/v1/sso/setup_okta",
    body);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
