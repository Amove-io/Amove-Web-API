# Cloud Account Endpoints

This document provides detailed information about the cloud-account endpoints in the Amove API. A `CloudAccount` represents a user's connection to an external cloud storage provider (AWS, Azure, Wasabi, an S3-compatible service, Dropbox, OneDrive, Google Drive, Box, etc.) or to an internal Amove storage tier backed by IDrive.

Endpoints in this controller fall into two broad groups:

- **Generic cloud-account operations** — list, create, share, delete, and browse any connected cloud account.
- **IDrive-backed Amove Storage operations** — provision storage tiers, create and delete buckets, and query daily usage for storage that Amove hosts on behalf of the user via IDrive.

## Endpoints

1. [Get All Cloud Accounts](#get-all-cloud-accounts)
2. [Get Amove Storage](#get-amove-storage)
3. [Insert Cloud Account](#insert-cloud-account)
4. [Manage Share](#manage-share)
5. [Delete Cloud Account](#delete-cloud-account)
6. [List Buckets](#list-buckets)
7. [List Objects](#list-objects)
8. [IDrive: Add Storage](#idrive-add-storage)
9. [IDrive: Delete Storage](#idrive-delete-storage)
10. [IDrive: Create Bucket](#idrive-create-bucket)
11. [IDrive: Delete Bucket](#idrive-delete-bucket)
12. [IDrive: Regions](#idrive-regions)
13. [IDrive: Get Daily Usage](#idrive-get-daily-usage)


## Get All Cloud Accounts

Returns every active cloud account the current user can see. Desktop administrators and desktop creative users additionally see cloud accounts that are shared within their account; regular users see only the accounts they own.

- **URL**: `/api/v1/cloudaccount/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a `DTOCollection<CloudAccount>`. Sensitive credentials are masked in the response.

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "accountId": "00000000-0000-0000-0000-000000000000",
      "userId": "00000000-0000-0000-0000-000000000000",
      "cloudType": 1,
      "name": "My AWS",
      "serviceUrl": null,
      "active": true,
      "shared": false,
      "internalStorage": false,
      "storageTier": 2
    }
  ],
  "total": 1,
  "options": {
    "pageSize": 0,
    "page": 0,
    "sort": [ { "field": "Name", "descending": false } ]
  }
}
```

`cloudType` values are bit-flag integers from the `CloudProvider` enum:

| Value | Provider |
|---|---|
| 1 | AWS |
| 2 | Wasabi |
| 4 | Azure |
| 8 | S3 Generic (any S3-compatible endpoint) |
| 16 | Google |
| 32 | Frame |
| 64 | Cloudflare |
| 128 | Dropbox |
| 256 | Box |
| 512 | OneDrive |
| 1024 | Google Drive |
| 4096 | Fastr Server |


## Get Amove Storage

Returns the list of Amove-hosted internal storage accounts the current user can access, with the credentials each client needs to connect directly to the storage endpoint.

- **URL**: `/api/v1/cloudaccount/get_amove_storage`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns an array of `AmoveStorage` objects.

```json
[
  {
    "access_key": "string",
    "secret_key": "string",
    "service_url": "https://..."
  }
]
```

If the API token used to authenticate the request was issued with an encryption key (`isEncrypted = true`), the `access_key` and `secret_key` fields are encrypted with that key and must be decrypted client-side before use.


## Insert Cloud Account

Creates a new cloud account owned by the current user. The service validates the supplied credentials against the target cloud provider before persisting the record; invalid credentials cause a `499` validation error.

- **URL**: `/api/v1/cloudaccount/insert`
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
  "shared": "boolean"
}
```

- `cloudType` — integer value from the `CloudProvider` table above.
- `name` — caller-supplied label for the account. Maximum 50 characters.
- `accessKey` / `secretKey` — required for S3-compatible providers (AWS, Wasabi, Azure, S3 Generic, Google, Cloudflare).
- `credentialsData` — JSON-encoded credentials blob used by providers that do not fit the access-key/secret-key model (e.g. Dropbox, OneDrive, Google Drive, Box). See the dedicated OAuth callback endpoints on the `desktop` controller for the automated OAuth flows that populate this field.
- `serviceUrl` — required for `S3Generic` (e.g. `https://s3.example.com`); optional otherwise.
- `shared` — when `true`, the account becomes visible to other users in the same account.

The `accountId`, `userId`, and `internalStorage` fields on any supplied payload are overwritten by the server.

### Response

Returns the created `CloudAccount` object with credentials masked.


## Manage Share

Toggles whether a cloud account is shared with other users in the same Amove account. Only the owner of the cloud account may call this endpoint.

- **URL**: `/api/v1/cloudaccount/manage_share`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "shared": "boolean"
}
```

### Response

A `200 OK` status with no body on success.


## Delete Cloud Account

Soft-deletes a cloud account belonging to the current user's Amove account. The underlying record is flagged as deleted but is not physically removed.

- **URL**: `/api/v1/cloudaccount/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The `id` of the cloud account to delete. |

### Response

Returns `true` on success. Returns a `499` validation error with code `Global` and message `"Object not found."` if the cloud account does not exist or is not visible to the caller.


## List Buckets

Retrieves the list of buckets (or top-level containers) from the remote cloud provider for the specified cloud account.

- **URL**: `/api/v1/cloudaccount/list_buckets`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "includeRegion": true
}
```

- `cloudAccountId` — id of the cloud account to query. Must belong to the current user's Amove account.
- `includeRegion` — when `true` (default), each bucket is returned with its region; when `false`, the region field is left unset to save a call per bucket.

### Response

Returns an `ICloudStorageCollection`. Each entry describes a bucket: its name, provider-specific id, region (when requested), and creation timestamp.


## List Objects

Retrieves the list of objects under a given path inside a bucket.

- **URL**: `/api/v1/cloudaccount/list_objects`
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

- `cloudAccountId` — id of the cloud account. Must belong to the current user's Amove account.
- `id` — provider-specific bucket or folder id (used by Dropbox, Box, OneDrive, Google Drive). Empty for S3-compatible providers.
- `bucketName` — target bucket or container name.
- `path` — key prefix to list under. Defaults to `/`.
- `continuationToken` — pagination token from a previous response. Empty for the first page.
- `count` — maximum number of objects to return. Defaults to `1000`.

### Response

Returns an `ICloudStorageObjectCollection` containing an array of objects and a `continuationToken` for the next page.


## IDrive: Add Storage

Provisions a new Amove-hosted storage account for the caller. Behind the scenes a storage bucket is created with the selected IDrive region and tier, an `S3Generic` cloud account is stored in Amove, the user is subscribed to the storage feature, and a welcome email with the credentials is sent.

- **URL**: `/api/v1/cloudaccount/idrive_add_storage`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "regionName": "string",
  "storageTier": "integer (StorageTier)",
  "shared": false,
  "defaultPassword": "string"
}
```

- `regionName` — region code. Call [IDrive: Regions](#idrive-regions) for the list of supported codes. Defaults to `OR` if omitted.
- `storageTier` — integer value from the `StorageTier` enum:

  | Value | Tier |
  |---|---|
  | 1 | Perform |
  | 2 | Global |
  | 4 | Scale |
  | 8 | Archive |

- `shared` — when `true`, the resulting cloud account is shared within the Amove account.
- `defaultPassword` — optional password for the provisioned user. If omitted, the server generates one and includes it in the welcome email.

The server rejects the request with a `499` `Global` error if the caller already has storage in the same region and tier.

### Response

Returns the created `CloudAccount` with credentials masked.


## IDrive: Delete Storage

Deletes an Amove-hosted storage cloud account owned by the current user. If this is the user's last storage in the tier, the underlying IDrive-backed storage account is also decommissioned; otherwise only the Amove-side record is removed.

- **URL**: `/api/v1/cloudaccount/idrive_delete_storage`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The `id` of the cloud account to delete. Must be an internal Amove storage account owned by the caller. |

### Response

Returns `true` on success. Returns a `499` `Global` error with message `"Only internal storage can be deleted"` if the target is a user-connected cloud account rather than an internal Amove storage account.


## IDrive: Create Bucket

Creates a new bucket inside an existing Amove-hosted storage cloud account.

- **URL**: `/api/v1/cloudaccount/idrive_create_bucket`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | Id of the internal Amove storage cloud account. |
| bucketName | string | Name of the bucket to create. |

### Response

A `200 OK` status with no body on success.


## IDrive: Delete Bucket

Deletes a bucket from an Amove-hosted storage cloud account.

- **URL**: `/api/v1/cloudaccount/idrive_delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | Id of the internal Amove storage cloud account. |
| bucketName | string | Name of the bucket to delete. |

### Response

A `200 OK` status with no body on success.


## IDrive: Regions

Returns the list of IDrive regions available for provisioning new Amove-hosted storage accounts.

- **URL**: `/api/v1/cloudaccount/idrive_regions`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns an array of region descriptors.

```json
[
  { "name": "Oregon", "code": "OR" },
  { "name": "Virginia", "code": "VA" }
]
```


## IDrive: Get Daily Usage

Returns the maximum disk usage (in bytes) over the last 24 hours for the caller's Amove-hosted storage. If `storageTier` is `0` (`All`), the response is the sum of the per-tier maximums.

- **URL**: `/api/v1/cloudaccount/idrive_get_daily_usage`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| storageTier | integer | `StorageTier` enum value: `0` = All, `1` = Perform, `2` = Global, `4` = Scale, `8` = Archive. |

### Response

Returns a single floating-point number — usage in bytes.

```json
123456789.0
```

If the user has no Amove-hosted storage in the requested tier, the endpoint returns `0`.


## Sample Code

### List cloud accounts

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "https://api.amove.io/api/v1/cloudaccount/get_all",
    headers={"Authorization": "Bearer YOUR_JWT"},
)
print(response.json())
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/cloudaccount/get_all", {
  headers: { "Authorization": "Bearer YOUR_JWT" }
});
console.log(await res.json());
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var res = await client.GetAsync("https://api.amove.io/api/v1/cloudaccount/get_all");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


### Create an S3-compatible cloud account

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/cloudaccount/insert",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json={
        "cloudType": 8,
        "name": "My S3 Endpoint",
        "accessKey": "EXAMPLE_ACCESS_KEY",
        "secretKey": "EXAMPLE_SECRET",
        "serviceUrl": "https://s3.example.com",
        "shared": False,
    },
)
print(response.json())
```

</details>


### List objects in a bucket

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/cloudaccount/list_objects",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json={
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "bucketName": "my-bucket",
        "path": "/photos/",
        "count": 100,
    },
)
print(response.json())
```

</details>


### Provision Amove-hosted storage

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/cloudaccount/idrive_add_storage",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json={
        "regionName": "OR",
        "storageTier": 2,
        "shared": False,
    },
)
print(response.json())
```

</details>


For error handling, see [Error Model](errors.md).
