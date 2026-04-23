# Authentication

This document describes the Amove Authentication API. All authentication endpoints run on a dedicated host (`https://auth.amove.io`), separate from the Main API (`https://api.amove.io`).

## Overview

Amove authentication uses a **two-step request-token handshake**:

1. Request a short-lived **request token** (60-second TTL).
2. Use the request token in the body of your `login` call to obtain a **JWT**.

If the account has multi-factor authentication enabled, the `login` response returns a TOTP **challenge** and an encrypted **session** instead of a JWT. Complete the challenge via `mfa_respond` to receive the JWT.

Once you have a JWT, include it on every call to the Main API:

```
Authorization: Bearer <your_jwt_here>
```

When the JWT expires, repeat the login flow to obtain a new one.

Every authentication endpoint requires a freshly-minted request token; a request token is one-shot and is consumed by the first successful call that uses it.

## Endpoints

1. [Request Token](#request-token)
2. [Login](#login)
3. [MFA Respond](#mfa-respond)
4. [Logout](#logout)
5. [Forgot Password](#forgot-password)
6. [Reset Password](#reset-password)

## Request Token

Requests a new temporary token used as part of a subsequent authentication call.

- **URL**: `/api/authentication/request_token`
- **Method**: POST
- **Auth Required**: No
- **Host**: `https://auth.amove.io`

### Request Body

None.

### Response

A plain string containing a request token, valid for **60 seconds**. The token is consumed the first time it is used in another authentication call.

```
"d3c7b8e5-a0f2-4e50-b7a1-EXAMPLEONLY"
```

## Login

Authenticates a user and returns either a JWT or an MFA challenge.

- **URL**: `/api/authentication/login`
- **Method**: POST
- **Auth Required**: No (request token required in body)
- **Host**: `https://auth.amove.io`

### Request Body

```json
{
  "requestToken": "string",
  "username": "string",
  "password": "string",
  "type": "integer (UserType)"
}
```

- `requestToken` — obtained from [Request Token](#request-token).
- `type` — integer matching the user role to authenticate as (see [UserType values](#usertype-values) below).

### Response

```json
{
  "challenge": "integer (ChallengeType)",
  "jwtToken": "string",
  "session": "string"
}
```

- If `challenge == 0` (None): `jwtToken` is populated; use it as a bearer token.
- If `challenge == 1` (TOTP): `session` is populated; complete the challenge via [MFA Respond](#mfa-respond).

If authentication fails, the response is `401 Unauthorized`.

## MFA Respond

Completes an MFA challenge with a TOTP code from the user's authenticator app.

- **URL**: `/api/authentication/mfa_respond`
- **Method**: POST
- **Auth Required**: No (request token required in body)
- **Host**: `https://auth.amove.io`

### Request Body

```json
{
  "requestToken": "string",
  "userCode": "string",
  "session": "string"
}
```

- `requestToken` — **freshly requested**; the request token used for `login` is already consumed.
- `userCode` — current 6-digit TOTP code from the user's authenticator app.
- `session` — the encrypted session string returned by `login` when a TOTP challenge was issued.

### Response

```json
{
  "challenge": 0,
  "jwtToken": "string"
}
```

## Logout

Terminates a login session.

- **URL**: `/api/authentication/logout`
- **Method**: POST
- **Auth Required**: No (request token required in body)
- **Host**: `https://auth.amove.io`

### Request Body

```json
{
  "requestToken": "string",
  "jwtToken": "string"
}
```

### Response

A successful logout returns a `200 OK` status with no body.

## Forgot Password

Initiates a password reset. If the username exists, a password-reset email is sent; if it does not, the endpoint still returns `200` (to prevent account enumeration).

- **URL**: `/api/authentication/forgot_password`
- **Method**: POST
- **Auth Required**: No (request token required in body)
- **Host**: `https://auth.amove.io`

### Request Body

```json
{
  "requestToken": "string",
  "username": "string"
}
```

### Response

A `200 OK` status with no body. The reset email contains a token to use with [Reset Password](#reset-password).

## Reset Password

Sets a new password using a reset token from the forgot-password email.

- **URL**: `/api/authentication/reset_password`
- **Method**: POST
- **Auth Required**: No (request token required in body)
- **Host**: `https://auth.amove.io`

### Request Body

```json
{
  "requestToken": "string",
  "resetToken": "string",
  "password": "string"
}
```

### Response

A `200 OK` status with no body.

## UserType Values

The `type` field in login requests uses the following integer values:

| Value | Name | Description |
|---|---|---|
| 1 | ProviderAdmin | Provider administrator |
| 2 | ResellerAdmin | Reseller administrator |
| 4 | AccountAdmin | Account administrator |
| 8 | AccountUser | Regular account user |
| 16 | DesktopAdmin | Desktop administrator |
| 32 | DesktopCreativeUser | Desktop creative-tier user |
| 64 | DesktopStandardUser | Desktop standard user |

## Error Responses

Errors from authentication endpoints follow the conventions in [errors.md](errors.md), with a few specific cases:

- `401 Unauthorized` — invalid username or password, or the `type` in the request does not match the user's actual role.
- `499` with error code `TOKEN` — request token expired or already used.
- `499` with error code `TOTP` — MFA challenge failed (wrong code or stale session).
- `499` with error code `AUTH` — authentication subsystem rejected the request.

## Sample Code

### Full login flow (no MFA)

<details>
<summary>Python</summary>

```python
import requests

AUTH_HOST = "https://auth.amove.io"

# Step 1: request a request token
request_token = requests.post(f"{AUTH_HOST}/api/authentication/request_token").json()

# Step 2: login
login = requests.post(f"{AUTH_HOST}/api/authentication/login", json={
    "requestToken": request_token,
    "username": "user@example.com",
    "password": "YOUR_PASSWORD",
    "type": 8,  # AccountUser
}).json()

if login["challenge"] == 0:
    jwt = login["jwtToken"]
    print("JWT:", jwt)
else:
    print("MFA required, session:", login["session"])
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const AUTH_HOST = "https://auth.amove.io";

const requestToken = await fetch(`${AUTH_HOST}/api/authentication/request_token`, {
  method: "POST"
}).then(r => r.json());

const login = await fetch(`${AUTH_HOST}/api/authentication/login`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    requestToken: requestToken,
    username: "user@example.com",
    password: "YOUR_PASSWORD",
    type: 8
  })
}).then(r => r.json());

if (login.challenge === 0) {
  console.log("JWT:", login.jwtToken);
} else {
  console.log("MFA required, session:", login.session);
}
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Text.Json;

using var client = new HttpClient { BaseAddress = new Uri("https://auth.amove.io/") };

string requestToken = await (await client.PostAsync("api/authentication/request_token", null))
    .Content.ReadAsStringAsync();
requestToken = requestToken.Trim('"');

var login = await client.PostAsJsonAsync("api/authentication/login", new {
    requestToken,
    username = "user@example.com",
    password = "YOUR_PASSWORD",
    type = 8
});

var body = await login.Content.ReadFromJsonAsync<JsonElement>();
if (body.GetProperty("challenge").GetInt32() == 0)
{
    Console.WriteLine("JWT: " + body.GetProperty("jwtToken").GetString());
}
else
{
    Console.WriteLine("MFA required, session: " + body.GetProperty("session").GetString());
}
```

</details>

### Login flow with MFA (TOTP)

<details>
<summary>Python</summary>

```python
import requests

AUTH_HOST = "https://auth.amove.io"

# Step 1: login
rt = requests.post(f"{AUTH_HOST}/api/authentication/request_token").json()
login = requests.post(f"{AUTH_HOST}/api/authentication/login", json={
    "requestToken": rt,
    "username": "user@example.com",
    "password": "YOUR_PASSWORD",
    "type": 8,
}).json()

if login["challenge"] == 1:
    user_code = input("Enter TOTP code from your authenticator: ")

    # Step 2: fresh request token for mfa_respond
    rt2 = requests.post(f"{AUTH_HOST}/api/authentication/request_token").json()

    # Step 3: complete the challenge
    final = requests.post(f"{AUTH_HOST}/api/authentication/mfa_respond", json={
        "requestToken": rt2,
        "userCode": user_code,
        "session": login["session"],
    }).json()
    print("JWT:", final["jwtToken"])
```

</details>

### Using the JWT against the Main API

<details>
<summary>Python</summary>

```python
import requests

user_info = requests.get(
    "https://api.amove.io/api/v1/user/userinfo",
    headers={"Authorization": f"Bearer {jwt}"},
).json()
print(user_info)
```

</details>
