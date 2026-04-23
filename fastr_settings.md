# Fastr Settings Endpoints

This document provides detailed information about the Fastr settings endpoints in the AMove API. A `FastrSettings` record configures how Fastr transfers data for one `(user, cloudAccount)` pair — bandwidth limits, checksum verification, and collision handling.

A user may have at most one settings record per cloud account; the database enforces a unique index on `(UserId, CloudAccountId)`.

The controller exposes two classes of endpoints:

- **Self-service** endpoints (`insert`, `update`, `delete`, `get_all`) operate on the caller's own settings.
- **Admin** variants (`insert_for_user`, `update_for_user`, `delete_for_user`, `get_all/{userId}`) let an admin manage settings for another user in the same account. These endpoints return `403 Forbidden` unless the caller has one of the admin roles `ProviderAdmin`, `AccountAdmin`, or `DesktopAdmin`.

## Endpoints

1. [Get All Settings](#get-all-settings)
2. [Get All Settings For User](#get-all-settings-for-user)
3. [Get Settings By Cloud Account](#get-settings-by-cloud-account)
4. [Insert Settings](#insert-settings)
5. [Insert Settings For User](#insert-settings-for-user)
6. [Update Settings](#update-settings)
7. [Update Settings For User](#update-settings-for-user)
8. [Delete Settings](#delete-settings)
9. [Delete Settings For User](#delete-settings-for-user)


## Get All Settings

Returns settings records the caller can see. Admin callers (`ProviderAdmin`, `AccountAdmin`, `DesktopAdmin`) see every settings record in their account; non-admin callers see only their own.

- **URL**: `/api/v1/fastrsettings/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a `DTOCollection<FastrSettings>` sorted by `CloudAccountId`.

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "cloudAccountId": "11111111-1111-1111-1111-111111111111",
      "userId": "22222222-2222-2222-2222-222222222222",
      "accountId": "33333333-3333-3333-3333-333333333333",
      "maxUploadBytesPerSecond": 0,
      "maxDownloadBytesPerSecond": 0,
      "maxServerToServerBytesPerSecond": 0,
      "verifyChecksum": true,
      "fileExistsAction": 0,
      "active": true
    }
  ],
  "total": 1
}
```


## Get All Settings For User

Returns every settings record owned by a specific user within the caller's account. Admin only.

- **URL**: `/api/v1/fastrsettings/get_all/{userId}`
- **Method**: GET
- **Auth Required**: Yes (`ProviderAdmin`, `AccountAdmin`, or `DesktopAdmin`)

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| userId | string (uuid) | The user whose settings you want to list |

### Response

Returns a `DTOCollection<FastrSettings>`. Non-admin callers receive `403 Forbidden`.


## Get Settings By Cloud Account

Returns the active settings record for a given cloud account, if any exists.

- **URL**: `/api/v1/fastrsettings/get_by_cloud_account/{cloudAccountId}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | The cloud account ID |

### Response

Returns a single `FastrSettings` object, or a `499` with `NOT_FOUND` when no active settings record exists for the cloud account.


## Insert Settings

Creates or updates settings for the current user and a given cloud account. If a record already exists for `(currentUser, cloudAccountId)`, the endpoint updates it in place rather than creating a duplicate.

- **URL**: `/api/v1/fastrsettings/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "maxUploadBytesPerSecond": "integer",
  "maxDownloadBytesPerSecond": "integer",
  "maxServerToServerBytesPerSecond": "integer",
  "verifyChecksum": "boolean",
  "fileExistsAction": "integer",
  "active": "boolean"
}
```

- `cloudAccountId` — the cloud account these settings apply to.
- `maxUploadBytesPerSecond`, `maxDownloadBytesPerSecond`, `maxServerToServerBytesPerSecond` — bandwidth limits in bytes per second. Use `0` for unlimited.
- `verifyChecksum` — if `true`, Fastr performs a SHA-256 check after each transfer.
- `fileExistsAction` — how to handle an existing destination file. See [File Exists Action Values](#file-exists-action-values).
- `active` — set `false` to disable this record without deleting it.

The `userId` and `accountId` fields, if present in the body, are ignored — they are always filled from the JWT.

### Response

Returns the created or updated `FastrSettings` object.


## Insert Settings For User

Creates or updates settings for another user's cloud account. Admin only.

- **URL**: `/api/v1/fastrsettings/insert_for_user`
- **Method**: POST
- **Auth Required**: Yes (`ProviderAdmin`, `AccountAdmin`, or `DesktopAdmin`)

### Request Body

Same shape as [Insert Settings](#insert-settings), plus a required `userId`:

```json
{
  "userId": "string (uuid)",
  "cloudAccountId": "string (uuid)",
  "maxUploadBytesPerSecond": "integer",
  "maxDownloadBytesPerSecond": "integer",
  "maxServerToServerBytesPerSecond": "integer",
  "verifyChecksum": "boolean",
  "fileExistsAction": "integer",
  "active": "boolean"
}
```

- `userId` — must be a non-empty GUID; the endpoint returns `400 Bad Request` if it is missing or empty.
- `accountId` — ignored; always set to the caller's account from the JWT.

### Response

Returns the created or updated `FastrSettings` object. Non-admin callers receive `403 Forbidden`.


## Update Settings

Updates an existing settings record owned by the current user. Only the mutable fields (`MaxUploadBytesPerSecond`, `MaxDownloadBytesPerSecond`, `MaxServerToServerBytesPerSecond`, `VerifyChecksum`, `FileExistsAction`, `Active`) are applied; `UserId`, `AccountId`, and `CloudAccountId` cannot be changed by this endpoint.

- **URL**: `/api/v1/fastrsettings/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "maxUploadBytesPerSecond": "integer",
  "maxDownloadBytesPerSecond": "integer",
  "maxServerToServerBytesPerSecond": "integer",
  "verifyChecksum": "boolean",
  "fileExistsAction": "integer",
  "active": "boolean"
}
```

### Response

Returns the updated `FastrSettings` object. Returns `499` with `NOT_FOUND` when the record does not exist; returns `403 Forbidden` when the record is owned by another user.


## Update Settings For User

Updates any settings record in the caller's account, regardless of which user owns it. Admin only. Applies the same set of mutable fields as [Update Settings](#update-settings).

- **URL**: `/api/v1/fastrsettings/update_for_user`
- **Method**: PUT
- **Auth Required**: Yes (`ProviderAdmin`, `AccountAdmin`, or `DesktopAdmin`)

### Request Body

Same shape as [Update Settings](#update-settings).

### Response

Returns the updated `FastrSettings` object. Returns `403 Forbidden` when the caller is not an admin or the record belongs to a different account; returns `499` with `NOT_FOUND` when the record does not exist.


## Delete Settings

Permanently removes a settings record owned by the current user.

- **URL**: `/api/v1/fastrsettings/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the settings record to delete |

### Response

Returns `200 OK` with no body. Returns `499` with `NOT_FOUND` when the record does not exist; returns `403 Forbidden` when the record is owned by another user.


## Delete Settings For User

Permanently removes any settings record within the caller's account. Admin only.

- **URL**: `/api/v1/fastrsettings/delete_for_user`
- **Method**: DELETE
- **Auth Required**: Yes (`ProviderAdmin`, `AccountAdmin`, or `DesktopAdmin`)

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the settings record to delete |

### Response

Returns `200 OK` with no body. Returns `403 Forbidden` when the caller is not an admin or the record belongs to a different account; returns `499` with `NOT_FOUND` when the record does not exist.


## File Exists Action Values

The `fileExistsAction` field controls how Fastr handles a destination file that already exists:

| Value | Meaning |
|---|---|
| 0 | Overwrite — replace the existing file |
| 1 | SkipIfSame — skip when the source and destination match |
| 2 | KeepBoth — write the new file under a non-colliding name |


## Sample Code

### Create and update your own Fastr settings

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
HEADERS = {"Authorization": "Bearer YOUR_JWT"}

cloud_account_id = "11111111-1111-1111-1111-111111111111"

# Create or upsert your own settings for a cloud account
settings = requests.post(
    f"{BASE}/api/v1/fastrsettings/insert",
    headers=HEADERS,
    json={
        "cloudAccountId": cloud_account_id,
        "maxUploadBytesPerSecond": 50_000_000,
        "maxDownloadBytesPerSecond": 0,
        "maxServerToServerBytesPerSecond": 100_000_000,
        "verifyChecksum": True,
        "fileExistsAction": 1,
        "active": True,
    },
).json()

# Later, raise the upload cap
requests.put(
    f"{BASE}/api/v1/fastrsettings/update",
    headers=HEADERS,
    json={
        "id": settings["id"],
        "maxUploadBytesPerSecond": 100_000_000,
        "maxDownloadBytesPerSecond": 0,
        "maxServerToServerBytesPerSecond": 100_000_000,
        "verifyChecksum": True,
        "fileExistsAction": 1,
        "active": True,
    },
)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const BASE = "https://api.amove.io";
const headers = {
  "Authorization": "Bearer YOUR_JWT",
  "Content-Type": "application/json"
};

const settings = await fetch(`${BASE}/api/v1/fastrsettings/insert`, {
  method: "POST",
  headers,
  body: JSON.stringify({
    cloudAccountId: "11111111-1111-1111-1111-111111111111",
    maxUploadBytesPerSecond: 50_000_000,
    maxDownloadBytesPerSecond: 0,
    maxServerToServerBytesPerSecond: 100_000_000,
    verifyChecksum: true,
    fileExistsAction: 1,
    active: true
  })
}).then(r => r.json());
console.log("Settings id:", settings.id);
```

</details>

### Admin: manage another user's Fastr settings

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
HEADERS = {"Authorization": "Bearer YOUR_ADMIN_JWT"}

target_user_id = "22222222-2222-2222-2222-222222222222"
cloud_account_id = "11111111-1111-1111-1111-111111111111"

# List a user's settings
user_settings = requests.get(
    f"{BASE}/api/v1/fastrsettings/get_all/{target_user_id}",
    headers=HEADERS,
).json()

# Set bandwidth caps for a user
requests.post(
    f"{BASE}/api/v1/fastrsettings/insert_for_user",
    headers=HEADERS,
    json={
        "userId": target_user_id,
        "cloudAccountId": cloud_account_id,
        "maxUploadBytesPerSecond": 25_000_000,
        "maxDownloadBytesPerSecond": 25_000_000,
        "maxServerToServerBytesPerSecond": 0,
        "verifyChecksum": True,
        "fileExistsAction": 0,
        "active": True,
    },
)
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Net.Http.Json;

using var client = new HttpClient { BaseAddress = new Uri("https://api.amove.io/") };
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_ADMIN_JWT");

var targetUserId = Guid.Parse("22222222-2222-2222-2222-222222222222");
var cloudAccountId = Guid.Parse("11111111-1111-1111-1111-111111111111");

await client.PostAsJsonAsync("api/v1/fastrsettings/insert_for_user", new {
    userId = targetUserId,
    cloudAccountId,
    maxUploadBytesPerSecond = 25_000_000L,
    maxDownloadBytesPerSecond = 25_000_000L,
    maxServerToServerBytesPerSecond = 0L,
    verifyChecksum = true,
    fileExistsAction = 0,
    active = true
});
```

</details>

For error handling, see [Error Model](errors.md).
