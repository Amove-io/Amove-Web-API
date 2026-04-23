# Storage Endpoints

This document provides detailed information about the storage endpoints in the AMove API. These endpoints manage access keys and buckets on the AMove-native IDrive-backed internal storage.

## Endpoints

1. [Get Storage Keys](#get-storage-keys)
2. [Create Storage Key](#create-storage-key)
3. [Delete Storage Key](#delete-storage-key)
4. [Create Bucket](#create-bucket)
5. [Get Bucket Status](#get-bucket-status)
6. [Update Bucket](#update-bucket)
7. [Delete Bucket](#delete-bucket)


## Get Storage Keys

Returns all storage access keys owned by the current user in their account.

- **URL**: `/api/v1/storage/get_storage_key`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction |

### Response

Returns a `DTOCollection<StorageApiKey>`:

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "userId": "00000000-0000-0000-0000-000000000000",
      "accountId": "00000000-0000-0000-0000-000000000000",
      "name": "ci-key",
      "accessKey": "EXAMPLE_ACCESS_KEY",
      "description": "Write access on selected buckets",
      "region": "us-east-1",
      "storageDn": "s3.example.com",
      "storageTier": 0,
      "createDate": "2026-01-01T00:00:00Z"
    }
  ],
  "total": 1
}
```

The secret key is never returned by this endpoint; it is only visible once in the [Create Storage Key](#create-storage-key) response.


## Create Storage Key

Provisions a new access key on the backing IDrive storage and stores it against the caller's account. The email sent to IDrive is derived from the caller's email plus the `StorageTier` of the selected cloud account (e.g., `user+Tier2@example.com`).

- **URL**: `/api/v1/storage/create_storage_key`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "name": "string",
  "region": "string",
  "storageDn": "string",
  "permission": "integer (StoragePermission)",
  "allBuckets": true,
  "selectedBuckets": ["bucket1", "bucket2"],
  "cloudAccountId": "00000000-0000-0000-0000-000000000000"
}
```

- `permission` — `0` Read, `1` Write, `2` ReadWrite.
- `allBuckets` — when `true`, `selectedBuckets` is ignored and the key is scoped to every bucket on the storage.
- `cloudAccountId` — the `CloudAccount` whose storage tier this key is being issued on.

### Response

Returns a `ShowStorageApiKey` object — a `StorageApiKey` with the one-time `secretKey` included:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "userId": "00000000-0000-0000-0000-000000000000",
  "accountId": "00000000-0000-0000-0000-000000000000",
  "name": "ci-key",
  "accessKey": "EXAMPLE_ACCESS_KEY",
  "secretKey": "SECRET_EXAMPLE_ONLY",
  "description": "Write access on selected buckets",
  "region": "us-east-1",
  "storageDn": "s3.example.com",
  "createDate": "2026-01-01T00:00:00Z"
}
```

Store the `secretKey` immediately — it is not recoverable after this call.


## Delete Storage Key

Revokes a storage access key on the backing IDrive storage and deletes the local record.

- **URL**: `/api/v1/storage/delete_storage_key`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the `StorageApiKey` to delete |

### Response

A `200 OK` status with no body on success.


## Create Bucket

Creates a new bucket on the internal storage behind the specified `CloudAccount`. When `versioningEnabled` or `isEncrypted` are set, the server issues follow-up calls to enable versioning and server-side encryption. The `cloudAccount` must be an internal-storage account in the caller's account and must be active.

- **URL**: `/api/v1/storage/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "bucketName": "string",
  "isPublic": false,
  "isEncrypted": false,
  "versioningEnabled": false,
  "objectLockEnabled": false
}
```

### Response

A `200 OK` status with no body on success.


## Get Bucket Status

Reports whether a bucket has versioning, encryption, public access, and object-lock enabled.

- **URL**: `/api/v1/storage/bucket_status`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "bucketName": "string"
}
```

### Response

```json
{
  "versioningEnabled": false,
  "encryptionEnabled": false,
  "isPublic": false,
  "objectLockEnabled": false
}
```


## Update Bucket

Updates the versioning and encryption flags on an existing bucket. Only flags that differ from the current bucket status trigger an underlying change on the storage.

- **URL**: `/api/v1/storage/update_bucket`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "bucketName": "string",
  "isPublic": false,
  "isEncrypted": false,
  "versioningEnabled": false
}
```

### Response

A `200 OK` status with no body on success.


## Delete Bucket

Deletes a bucket from the internal storage behind the specified `CloudAccount`.

- **URL**: `/api/v1/storage/delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | Internal-storage cloud account |
| bucketName | string | The name of the bucket to delete |

### Response

A `200 OK` status with no body on success.


## Sample Code

### Create a storage key and a bucket

<details>
<summary>Python</summary>

```python
import requests

JWT = "YOUR_JWT"
BASE = "https://api.amove.io"

key = requests.post(
    f"{BASE}/api/v1/storage/create_storage_key",
    headers={"Authorization": f"Bearer {JWT}"},
    json={
        "name": "ci-key",
        "region": "us-east-1",
        "storageDn": "s3.example.com",
        "permission": 2,
        "allBuckets": False,
        "selectedBuckets": ["team-bucket"],
        "cloudAccountId": "00000000-0000-0000-0000-000000000000"
    },
).json()
print("Save this secret now:", key["secretKey"])

requests.post(
    f"{BASE}/api/v1/storage/create_bucket",
    headers={"Authorization": f"Bearer {JWT}"},
    json={
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "bucketName": "team-bucket",
        "isEncrypted": True,
        "versioningEnabled": True,
        "objectLockEnabled": False
    },
)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const JWT = "YOUR_JWT";
const BASE = "https://api.amove.io";

const key = await fetch(`${BASE}/api/v1/storage/create_storage_key`, {
  method: "POST",
  headers: { "Authorization": `Bearer ${JWT}`, "Content-Type": "application/json" },
  body: JSON.stringify({
    name: "ci-key",
    region: "us-east-1",
    storageDn: "s3.example.com",
    permission: 2,
    allBuckets: false,
    selectedBuckets: ["team-bucket"],
    cloudAccountId: "00000000-0000-0000-0000-000000000000"
  })
}).then(r => r.json());
console.log("Save this secret now:", key.secretKey);
```

</details>

### Inspect and update a bucket's flags

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Text.Json;

using var client = new HttpClient { BaseAddress = new Uri("https://api.amove.io/") };
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var status = await (await client.PostAsJsonAsync("api/v1/storage/bucket_status", new
{
    cloudAccountId = Guid.Parse("00000000-0000-0000-0000-000000000000"),
    bucketName = "team-bucket"
})).Content.ReadFromJsonAsync<JsonElement>();

Console.WriteLine(status);

await client.PutAsJsonAsync("api/v1/storage/update_bucket", new
{
    cloudAccountId = Guid.Parse("00000000-0000-0000-0000-000000000000"),
    bucketName = "team-bucket",
    isEncrypted = true,
    versioningEnabled = true
});
```

</details>


For error handling, see [Error Model](errors.md).
