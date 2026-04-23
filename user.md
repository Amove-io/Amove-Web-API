# User Endpoints

This document provides detailed information about the user-related endpoints in the AMove API. These endpoints cover self-service profile operations, account-owner subscription changes, MFA enrollment, admin-side user invitation and management, and the email-token signup flow.

## Endpoints

1. [Get Signup Info](#get-signup-info)
2. [User Signup](#user-signup)
3. [Get User Info](#get-user-info)
4. [Update User](#update-user)
5. [Update Subscription](#update-subscription)
6. [Reset Password](#reset-password)
7. [Generate MFA Token](#generate-mfa-token)
8. [Set MFA Preference](#set-mfa-preference)
9. [Get All Users](#get-all-users)
10. [Get All Users With Details](#get-all-users-with-details)
11. [Insert User](#insert-user)
12. [Edit User](#edit-user)
13. [Delete User](#delete-user)
14. [Resend User Email](#resend-user-email)
15. [Package Inquiry](#package-inquiry)


## Get Signup Info

Resolves a signup invitation token emailed to a new user and returns the pending user record it is bound to. Used by the signup landing page to pre-fill the form and confirm the token is still valid.

- **URL**: `/api/v1/user/signupinfo`
- **Method**: GET
- **Auth Required**: No

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | Signup token received by email. |

### Response

Returns the pending `User` record associated with the token. If the token is invalid or expired, the response is a `499` with error code `TOKEN`.


## User Signup

Finalizes a user signup using the token from the invitation email. Sets the first name, last name, and password on the pending user, flips the account to active, and completes provisioning with the identity provider.

- **URL**: `/api/v1/user/signup`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "firstname": "string",
  "lastname": "string",
  "password": "string",
  "token": "string"
}
```

### Response

Returns the finalized `User` object.


## Get User Info

Returns the profile of the currently authenticated user, enriched with account metadata (account id, name, subscription type, migration flag, and authentication provider).

- **URL**: `/api/v1/user/userinfo`
- **Method**: GET
- **Auth Required**: Yes

### Response

```json
{
  "userId": "string (uuid)",
  "username": "string",
  "firstname": "string",
  "lastname": "string",
  "userType": "integer (UserType)",
  "mfa": "boolean",
  "migration": "boolean",
  "owner": "boolean",
  "accountId": "string (uuid)",
  "accountName": "string",
  "subscriptionType": "integer (AccountSubscriptionType)",
  "authProvider": "integer (AuthProvider)"
}
```


## Update User

Updates the authenticated user's first name, last name, and account display name.

- **URL**: `/api/v1/user/update_user`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "accountName": "string",
  "firstName": "string",
  "lastName": "string"
}
```

### Response

Returns the updated `UserInfo` object (same schema as [Get User Info](#get-user-info)).


## Update Subscription

Changes the billing cadence of the account's subscription (monthly or yearly). Only the account owner may call this; any other user receives `400 Bad Request`.

- **URL**: `/api/v1/user/update_subscription`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "subscriptionType": "integer (AccountSubscriptionType)"
}
```

`subscriptionType` values: `1` = Monthly, `2` = Yearly.

### Response

`200 OK` with an empty body.


## Reset Password

Resets the authenticated user's password. The caller must supply both the current password and the new password.

- **URL**: `/api/v1/user/reset_password`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "password": "string",
  "currentPassword": "string"
}
```

### Response

`200 OK` with an empty body.


## Generate MFA Token

Requests a shared secret key for associating a software-based TOTP authenticator (e.g., Google Authenticator, Authy, 1Password) with the authenticated user. The returned string is the TOTP seed the client displays as a QR code or formats into an `otpauth://` URI.

- **URL**: `/api/v1/user/generate_mfa_token`
- **Method**: POST
- **Auth Required**: Yes

### Response

A plain string containing the TOTP secret. Pass this to the authenticator app and then confirm with [Set MFA Preference](#set-mfa-preference).


## Set MFA Preference

Enables or disables software-token MFA for the authenticated user. When enabling, the caller must include a current TOTP code proving the authenticator app is correctly configured with the secret issued by [Generate MFA Token](#generate-mfa-token).

- **URL**: `/api/v1/user/set_mfa`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "enabled": "boolean",
  "userCode": "string"
}
```

- `enabled` — `true` to enable MFA, `false` to disable.
- `userCode` — the current 6-digit TOTP code from the authenticator app (required when enabling; ignored when disabling).

### Response

`200 OK` with an empty body.


## Get All Users

Returns a paginated list of users in the authenticated user's account. Supports filters on status, type, and username.

- **URL**: `/api/v1/user/get_all_users`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction |
| deleted | boolean | false | When true, includes deleted records |
| userStatus | integer (flags) | `Active \| Inactive \| Pending` | Filter by user status |
| userType | integer (flags) | `All` | Filter by user type |
| username | string | null | Case-insensitive substring match on username |

### Response

Returns a `DTOCollection<User>`.


## Get All Users With Details

Same listing semantics as [Get All Users](#get-all-users), but each row also includes the user's groups, project permissions, and shared-cloud-drive permissions inline to eliminate the N+1 fetch pattern.

- **URL**: `/api/v1/user/get_all_users_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

Same as [Get All Users](#get-all-users).

### Response

Returns a `DTOCollection<UserWithDetailsDTO>` where each element looks like:

```json
{
  "id": "string (uuid)",
  "email": "string",
  "username": "string",
  "firstname": "string",
  "lastname": "string",
  "userType": "integer (UserType)",
  "status": "integer (UserStatus)",
  "groups": [
    {
      "user": { },
      "userGroup": { },
      "userUserGroup": { }
    }
  ],
  "projectsData": [
    {
      "user": { },
      "project": { },
      "permission": { }
    }
  ],
  "drivesData": [
    {
      "user": { },
      "sharedClouDrive": { },
      "permission": { }
    }
  ]
}
```


## Insert User

Creates a pending user in the caller's account, generates a signup token with a configurable expiration window, and emails the invitation. The endpoint also records a billing/subscription line for the invited user based on the account's current package limits. The username must be a valid email address.

- **URL**: `/api/v1/user/insert_user`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "username": "string (email)",
  "email": "string (email)",
  "firstname": "string",
  "lastname": "string",
  "userType": "integer (UserType)"
}
```

### Response

Returns the newly-created pending `User` (status `Pending`) with a signup token already emailed.


## Edit User

Updates an existing user's type. Owner users cannot have their type changed.

- **URL**: `/api/v1/user/edit_user`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "userType": "integer (UserType)"
}
```

### Response

Returns the updated `User` object.


## Delete User

Soft-deletes a user in the caller's account and unsubscribes any associated invited-user billing line.

- **URL**: `/api/v1/user/delete_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The user id to delete. |

### Response

`200 OK` with an empty body.


## Resend User Email

Resends the signup invitation email for a pending user, regenerating the signup token in the process. If no pending user matches the supplied username, the endpoint returns `400 Bad Request`.

- **URL**: `/api/v1/user/resend_user_email`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "username": "string"
}
```

### Response

`200 OK` with an empty body.


## Package Inquiry

Returns the remaining capacity on the account's current product package — how many more admin, creative, and standard user slots, cloud connections, and other entitlements the account may consume. When capacity has been reached, the corresponding field will be `0`.

- **URL**: `/api/v1/user/package_inquiry`
- **Method**: GET
- **Auth Required**: Yes

### Response

```json
{
  "adminUsers": "integer",
  "creativeUsers": "integer",
  "standardUsers": "integer",
  "storage": "integer",
  "connections": "integer",
  "projects": "integer",
  "teams": "integer",
  "drives": "integer",
  "syncs": "integer",
  "logs": "boolean",
  "sso": "boolean"
}
```


## Sample Code

### Get the current user's profile

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "https://api.amove.io/api/v1/user/userinfo",
    headers={"Authorization": "Bearer EXAMPLE_TOKEN"}
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/user/userinfo", {
  headers: { "Authorization": "Bearer EXAMPLE_TOKEN" }
});
console.log(await res.json());
```

</details>

<details>
<summary>C#</summary>

```csharp
using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "EXAMPLE_TOKEN");

HttpResponseMessage res = await client.GetAsync("https://api.amove.io/api/v1/user/userinfo");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>

### Invite a new user (admin)

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/user/insert_user",
    headers={"Authorization": "Bearer EXAMPLE_TOKEN"},
    json={
        "username": "newuser@example.com",
        "email": "newuser@example.com",
        "firstname": "New",
        "lastname": "User",
        "userType": 32
    }
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/user/insert_user", {
  method: "POST",
  headers: {
    "Authorization": "Bearer EXAMPLE_TOKEN",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    username: "newuser@example.com",
    email: "newuser@example.com",
    firstname: "New",
    lastname: "User",
    userType: 32
  })
});
console.log(await res.json());
```

</details>

### Finalize signup from the invitation email

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/user/signup",
    json={
        "firstname": "New",
        "lastname": "User",
        "password": "CHOSEN_PASSWORD",
        "token": "EXAMPLE_TOKEN"
    }
)
print(response.json())
```

</details>


For error handling, see [Error Model](errors.md).
