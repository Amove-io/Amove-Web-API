# Desktop Endpoints

This document describes the `/api/v1/desktop/...` endpoints. These are the hosted Web-API counterparts of the desktop client's local operations: they handle desktop signup and Google SSO, server-side cloud storage operations used by the desktop UI (bucket listings, presigned URLs, folder management), billing lookups, log-forwarding settings, and the OAuth 2.0 flows that connect Dropbox, OneDrive, Google Drive, and Box accounts.

Unless otherwise noted, every endpoint requires an `Authorization: Bearer <jwt>` header; the few signup-related endpoints are anonymous and instead carry a short-lived request token in the body (see [Authentication](authentication.md)).

## Endpoints

### Sign-up and social authentication

1. [Sign Up](#sign-up)
2. [Confirm Sign Up](#confirm-sign-up)
3. [Resend Confirmation Email](#resend-confirmation-email)
4. [Google Sign Up](#google-sign-up)
5. [Google Sign In](#google-sign-in)

### Cloud accounts

6. [Insert Cloud Account](#insert-cloud-account)
7. [Delete Cloud Account](#delete-cloud-account)
8. [Refresh Token](#refresh-token)

### Storage operations

9. [List Buckets](#list-buckets)
10. [List Objects](#list-objects)
11. [Create Folder](#create-folder)
12. [Delete Object](#delete-object)
13. [Generate Download URL](#generate-download-url)
14. [Generate Upload URL](#generate-upload-url)
15. [Send Download URL](#send-download-url)

### Subscription

16. [Billing Status](#billing-status)
17. [Active Subscriptions](#active-subscriptions)

### Log-forwarding settings

18. [Setup Account Log Setting](#setup-account-log-setting)
19. [Get Account Log Setting](#get-account-log-setting)
20. [Update Account Log Setting](#update-account-log-setting)
21. [Delete Account Log Setting](#delete-account-log-setting)

### OAuth flows

22. [Generate Dropbox Authorization URL](#generate-dropbox-authorization-url)
23. [Dropbox Authorization Callback](#dropbox-authorization-callback)
24. [Generate OneDrive Authorization URL](#generate-onedrive-authorization-url)
25. [OneDrive Authorization Callback](#onedrive-authorization-callback)
26. [Generate Google Drive Authorization URL](#generate-google-drive-authorization-url)
27. [Google Drive Authorization Callback](#google-drive-authorization-callback)
28. [Generate Box Authorization URL](#generate-box-authorization-url)
29. [Box Authorization Callback](#box-authorization-callback)


## Sign Up

Registers a new desktop user and sends an activation code to their email. Sign-up consumes a request token from [Request Token](authentication.md#request-token).

- **URL**: `/api/v1/desktop/signup`
- **Method**: POST
- **Auth Required**: No (request token required in body)

### Request Body

```json
{
  "requestToken": "string",
  "name": "string",
  "email": "string",
  "password": "string"
}
```

- `name` — display name, up to 50 characters.
- `email` — valid email address, up to 50 characters. Used as the login identifier.
- `password` — user password.

### Response

A `200 OK` status with no body. The caller must then confirm via [Confirm Sign Up](#confirm-sign-up) using the token emailed to the user.


## Confirm Sign Up

Completes the sign-up flow by exchanging the emailed activation code for a JWT.

- **URL**: `/api/v1/desktop/confirm_signup`
- **Method**: POST
- **Auth Required**: No (request token required in body)

### Request Body

```json
{
  "requestToken": "string",
  "token": "string"
}
```

- `token` — confirmation token received in the signup email.

### Response

Returns a plain string containing the newly-issued JWT.

```
"eyJhbGciOi..."
```

Returns `404 Not Found` if the confirmation token is unknown.


## Resend Confirmation Email

Extends the TTL of a pending activation token and re-sends the confirmation email.

- **URL**: `/api/v1/desktop/resend_confirmation_email`
- **Method**: POST
- **Auth Required**: No (request token required in body)

### Request Body

```json
{
  "requestToken": "string",
  "email": "string"
}
```

### Response

A `200 OK` status with no body. Returns `404 Not Found` if no pending activation exists for the given email.


## Google Sign Up

Registers a new user from a Google ID token. The server verifies the ID token with Google, creates an Amove account keyed to the Google-verified email, and provisions a demo shared cloud drive for the new user.

- **URL**: `/api/v1/desktop/google_signup`
- **Method**: POST
- **Auth Required**: No (request token required in body)

### Request Body

```json
{
  "requestToken": "string",
  "idToken": "string"
}
```

- `idToken` — Google OAuth ID token obtained via the Google Sign-In SDK on the client.

### Response

A `200 OK` status with no body.


## Google Sign In

Authenticates an existing user from a Google ID token and returns a JWT.

- **URL**: `/api/v1/desktop/google_signin`
- **Method**: POST
- **Auth Required**: No (request token required in body)

### Request Body

```json
{
  "requestToken": "string",
  "idToken": "string"
}
```

### Response

Returns a plain string containing the JWT.

```
"eyJhbGciOi..."
```


## Insert Cloud Account

Creates a new cloud account owned by the current user and subscribes the account owner to the appropriate connection feature (free up to the plan's maximum, paid beyond it). The `internalStorage` flag on the request is honored here — set it to `false` for user-provided cloud accounts.

- **URL**: `/api/v1/desktop/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudType": "integer (CloudProvider)",
  "name": "string",
  "accessKey": "string",
  "secretKey": "string",
  "credentialsData": "string",
  "serviceUrl": "string",
  "shared": false
}
```

See [Cloud Account — Insert](cloudaccount.md#insert-cloud-account) for the full field reference, including the `cloudType` enum and which fields are required for each provider.

### Response

Returns the created `CloudAccount` object with credentials masked.


## Delete Cloud Account

Soft-deletes a cloud account owned by the current user's Amove account.

- **URL**: `/api/v1/desktop/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The `id` of the cloud account to delete. |

### Response

Returns `true` on success. Returns a `499` validation error if the cloud account does not exist or is not visible to the caller.


## Refresh Token

For cloud providers whose credentials expire (Box, Google Drive, OneDrive), refreshes the access token and returns the updated `CloudAccount`. For providers without expiring credentials, returns the existing cloud account unchanged. The operation is thread-safe; concurrent calls against the same cloud account coalesce.

- **URL**: `/api/v1/desktop/refresh_token`
- **Method**: POST
- **Auth Required**: No (but intended for clients already holding a valid session)

### Request Body

```json
{
  "cloudAccountId": "string (uuid)"
}
```

### Response

Returns the updated `CloudAccount` object.


## List Buckets

Retrieves the list of buckets or top-level containers for the specified cloud account.

- **URL**: `/api/v1/desktop/list_buckets`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "includeRegion": true
}
```

### Response

Returns an `ICloudStorageCollection`. See [Cloud Account — List Buckets](cloudaccount.md#list-buckets) for the detailed shape.


## List Objects

Retrieves the list of objects under a given path inside a bucket.

- **URL**: `/api/v1/desktop/list_objects`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "bucketName": "string",
  "path": "/",
  "continuationToken": "",
  "count": 1000
}
```

See [Cloud Account — List Objects](cloudaccount.md#list-objects) for field descriptions and pagination semantics.

### Response

Returns an `ICloudStorageObjectCollection`.


## Create Folder

Creates a folder (or the equivalent zero-byte key or provider-native folder object) inside a bucket.

- **URL**: `/api/v1/desktop/create_folder`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "path": "string",
  "folderName": "string"
}
```

- `storageName` — bucket or container name.
- `id` — provider-specific parent folder id, for non-S3 providers. Empty for S3-compatible providers.
- `path` — path (key prefix) under which the folder is created.
- `folderName` — the new folder's name.

### Response

A `200 OK` status with no body on success.


## Delete Object

Deletes a single object from a bucket.

- **URL**: `/api/v1/desktop/delete_object`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string"
}
```

- `storageName` — bucket or container name.
- `key` — object key (or provider-specific object identifier) to delete.
- `id` — provider-specific object id, for non-S3 providers.

### Response

A `200 OK` status with no body on success.


## Generate Download URL

Produces a time-limited signed URL that lets a client download an object directly from the underlying cloud provider.

- **URL**: `/api/v1/desktop/generate_download_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": 24,
  "forceToDownload": false
}
```

- `storageName` — bucket or container name.
- `key` — object key.
- `expireHours` — lifetime of the signed URL in hours. Defaults to `24`.
- `forceToDownload` — when `true`, the signed URL's `Content-Disposition` is set to `attachment` so browsers download rather than render inline.

### Response

Returns a string containing the signed URL.

```json
"https://s3.example.com/my-bucket/photos/cat.jpg?..."
```


## Generate Upload URL

Produces a time-limited signed URL that lets a client upload directly to the underlying cloud provider.

- **URL**: `/api/v1/desktop/generate_upload_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": 24,
  "contentType": "application/octet-stream"
}
```

- `expireHours` — lifetime of the signed URL in hours. Defaults to `24`.
- `contentType` — the exact `Content-Type` header the client must use when uploading to the URL.

### Response

Returns a string containing the signed URL.


## Send Download URL

Generates a download link for an object and emails it to one or more recipients. The recipient opens a viewer page on the Amove website that exposes both a preview and a download link.

- **URL**: `/api/v1/desktop/send_download_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "email": "string",
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": 24,
  "note": "string"
}
```

- `email` — recipient email address.
- `note` — optional free-text note included in the email body.

### Response

A `200 OK` status with no body on success.


## Billing Status

Returns the current user's primary subscription.

- **URL**: `/api/v1/desktop/billing_status`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a `UserSubscription` object describing the active plan, status, payment method, trial state, and renewal dates.


## Active Subscriptions

Returns every active subscription the current user holds (plan, connection, storage, etc.).

- **URL**: `/api/v1/desktop/active_subscriptions`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns an array of `UserSubscription` objects.


## Setup Account Log Setting

Creates the Splunk log-forwarding configuration for the current user's Amove account. Only one `AccountLogSetting` record exists per account; a second call returns a duplicate error.

- **URL**: `/api/v1/desktop/setup_accountlogsetting`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "token": "string",
  "serverAddress": "string",
  "servicePort": "string",
  "active": true,
  "provider": 1
}
```

- `token` — HEC token for the target log provider.
- `serverAddress` — hostname or IP of the log receiver.
- `servicePort` — receiver port, as a string (max 10 chars).
- `active` — when `false`, the agent stops forwarding but the configuration is retained.
- `provider` — log provider enum. Currently only `1` (Splunk) is supported.

### Response

Returns the created `AccountLogSetting` object.


## Get Account Log Setting

Returns the log-forwarding configuration for the current account, or `null` if none is set.

- **URL**: `/api/v1/desktop/get_accountlogsetting`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a single `AccountLogSetting` object or `null`.


## Update Account Log Setting

Updates the log-forwarding configuration for the current account.

- **URL**: `/api/v1/desktop/update_accountlogsetting`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

Same shape as [Setup Account Log Setting](#setup-account-log-setting), with the additional `id` field identifying the record.

### Response

Returns the updated `AccountLogSetting` object.


## Delete Account Log Setting

Removes the log-forwarding configuration for the current account.

- **URL**: `/api/v1/desktop/delete_accountlogsetting`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

A `200 OK` status with no body on success.


## Generate Dropbox Authorization URL

Builds the Dropbox OAuth 2.0 authorization URL the user should be redirected to. The server constructs a signed `state` parameter binding the flow to the caller's user id, the desired cloud-account name, and the share flag.

- **URL**: `/api/v1/desktop/generate_dropbox_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "redirectURL": "string",
  "cloudName": "string",
  "shared": false
}
```

- `redirectURL` — OAuth redirect URI registered with your Dropbox app. Must match exactly.
- `cloudName` — name for the resulting `CloudAccount` record on successful connection.
- `shared` — whether the resulting cloud account is shared across the caller's Amove account.

### Response

Returns a string containing the Dropbox authorization URL; send the user to it to complete consent.


## Dropbox Authorization Callback

Exchanges the Dropbox OAuth authorization code for access and refresh tokens, then stores them as a new `CloudAccount` of type `Dropbox (128)` and runs the same connection-subscription bookkeeping as [Insert Cloud Account](#insert-cloud-account).

- **URL**: `/api/v1/desktop/dropbox_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "redirectURL": "string"
}
```

- `code` — authorization code returned by Dropbox at the redirect URI.
- `state` — the `state` parameter returned by Dropbox; validated against the original signed state.
- `redirectURL` — the same URI passed to [Generate Dropbox Authorization URL](#generate-dropbox-authorization-url).

### Response

Returns the created `CloudAccount` object with credentials masked. Rejects the request with a `400 Bad Request` if state validation fails.


## Generate OneDrive Authorization URL

Builds the Microsoft OneDrive (Azure AD v2.0) OAuth authorization URL with scopes `Files.ReadWrite.All offline_access`. The server constructs a signed `state` parameter binding the flow to the caller.

- **URL**: `/api/v1/desktop/generate_onedrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "redirectURL": "string",
  "cloudName": "string",
  "shared": false
}
```

### Response

Returns a string containing the OneDrive authorization URL.


## OneDrive Authorization Callback

Exchanges the OneDrive OAuth authorization code for access and refresh tokens, stores them as a new `CloudAccount` of type `OneDrive (512)`, and runs the connection-subscription bookkeeping.

- **URL**: `/api/v1/desktop/onedrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "error": "string",
  "errorDescription": "string",
  "redirectURL": "string"
}
```

- `error` / `errorDescription` — populated by Microsoft when the user cancels or consent fails. When present, the server returns `401 Unauthorized` with the details.

### Response

Returns the created `CloudAccount` object with credentials masked.


## Generate Google Drive Authorization URL

Builds the Google Drive OAuth authorization URL with scope `https://www.googleapis.com/auth/drive`. The server constructs a signed `state` parameter binding the flow to the caller.

- **URL**: `/api/v1/desktop/generate_googledrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "redirectURL": "string",
  "cloudName": "string",
  "shared": false
}
```

### Response

Returns a string containing the Google Drive authorization URL.


## Google Drive Authorization Callback

Exchanges the Google Drive OAuth authorization code for access and refresh tokens, stores them as a new `CloudAccount` of type `GoogleDrive (1024)`, and runs the connection-subscription bookkeeping.

- **URL**: `/api/v1/desktop/googledrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "error": "string",
  "errorDescription": "string",
  "redirectURL": "string"
}
```

### Response

Returns the created `CloudAccount` object with credentials masked.


## Generate Box Authorization URL

Builds the Box OAuth 2.0 authorization URL. The server constructs a signed `state` parameter binding the flow to the caller.

- **URL**: `/api/v1/desktop/generate_box_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "redirectURL": "string",
  "cloudName": "string",
  "shared": false
}
```

### Response

Returns a string containing the Box authorization URL.


## Box Authorization Callback

Exchanges the Box OAuth authorization code for access and refresh tokens, stores them as a new `CloudAccount` of type `Box (256)`, and runs the connection-subscription bookkeeping.

- **URL**: `/api/v1/desktop/box_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "redirectURL": "string"
}
```

### Response

Returns the created `CloudAccount` object with credentials masked.


## Sample Code

### Desktop signup flow

<details>
<summary>Python</summary>

```python
import requests

AUTH = "https://auth.amove.io"
API = "https://api.amove.io"

# Step 1: get a request token from the auth service
rt = requests.post(f"{AUTH}/api/authentication/request_token").json()

# Step 2: start the signup
requests.post(f"{API}/api/v1/desktop/signup", json={
    "requestToken": rt,
    "name": "Jane Doe",
    "email": "user@example.com",
    "password": "YOUR_PASSWORD",
})

# User receives a confirmation email; treat its token as `CODE`.
rt2 = requests.post(f"{AUTH}/api/authentication/request_token").json()
jwt = requests.post(f"{API}/api/v1/desktop/confirm_signup", json={
    "requestToken": rt2,
    "token": "CODE",
}).json()
print("JWT:", jwt)
```

</details>


### Generate and use a presigned download URL

<details>
<summary>Python</summary>

```python
import requests

url = requests.post(
    "https://api.amove.io/api/v1/desktop/generate_download_url",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json={
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "storageName": "my-bucket",
        "key": "photos/cat.jpg",
        "expireHours": 4,
        "forceToDownload": True,
    },
).json()

print(url)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/desktop/generate_download_url", {
  method: "POST",
  headers: {
    "Authorization": "Bearer YOUR_JWT",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    cloudAccountId: "00000000-0000-0000-0000-000000000000",
    storageName: "my-bucket",
    key: "photos/cat.jpg",
    expireHours: 4,
    forceToDownload: true
  })
});
console.log(await res.json());
```

</details>


### Connect a Dropbox account end-to-end

<details>
<summary>Python</summary>

```python
import requests, webbrowser

BASE = "https://api.amove.io"
JWT = "YOUR_JWT"
REDIRECT = "https://your-app.example.com/oauth/dropbox"

auth_url = requests.post(
    f"{BASE}/api/v1/desktop/generate_dropbox_authorization_url",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"redirectURL": REDIRECT, "cloudName": "My Dropbox", "shared": False},
).json()

webbrowser.open(auth_url)  # user completes consent, lands on REDIRECT with ?code=...&state=...

# When your callback handler receives `code` and `state`, POST them back:
account = requests.post(
    f"{BASE}/api/v1/desktop/dropbox_authorization_callback",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"code": "CODE_FROM_REDIRECT", "state": "STATE_FROM_REDIRECT", "redirectURL": REDIRECT},
).json()

print(account["id"])
```

</details>


For error handling, see [Error Model](errors.md).
