# Shared Cloud Drive Endpoints

This document provides detailed information about the shared-cloud-drive endpoints in the Amove API. A shared cloud drive links an account and a backing `CloudAccount`/bucket so multiple users can mount it as a virtual drive or local path.

## Endpoints

1. [Get All Shared Cloud Drives](#get-all-shared-cloud-drives)
2. [Get All Shared Cloud Drives With Details](#get-all-shared-cloud-drives-with-details)
3. [Insert Shared Cloud Drive](#insert-shared-cloud-drive)
4. [Update Shared Cloud Drive](#update-shared-cloud-drive)
5. [Delete Shared Cloud Drive](#delete-shared-cloud-drive)
6. [Get Shared Cloud Drives For Current User](#get-shared-cloud-drives-for-current-user)


## Get All Shared Cloud Drives

Returns a paginated list of shared cloud drives in the current user's account. Each row includes its backing `CloudAccount`, with access credentials obfuscated (masked) before return.

- **URL**: `/api/v1/sharedclouddrive/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| deleted | boolean | false | When `true`, return soft-deleted drives instead of active ones |
| name | string | null | Optional case-insensitive substring filter on drive name |

### Response

Returns a `DTOCollection<SharedCloudDrive>`:

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "accountId": "00000000-0000-0000-0000-000000000000",
      "creatorUserId": "00000000-0000-0000-0000-000000000000",
      "cloudAccountId": "00000000-0000-0000-0000-000000000000",
      "active": true,
      "name": "team-drive",
      "prefix": "team/",
      "prefixId": "",
      "storageId": "",
      "storageName": "team-bucket",
      "objectsPerFolder": 10000,
      "allowDelete": false,
      "localCacheEncrypted": false,
      "cloudObjectsEncrypted": false,
      "driveType": 1,
      "syncType": 2,
      "deleted": false,
      "cloudAccount": { }
    }
  ],
  "total": 1
}
```

`driveType` values: `1 = VirtualDrive`, `2 = LocalPath`.
`syncType` values: `1 = Stream`, `2 = Mirror`.


## Get All Shared Cloud Drives With Details

Same filters and paging as [Get All Shared Cloud Drives](#get-all-shared-cloud-drives), but each drive is returned with its assigned projects, users, and user groups inline — avoiding the per-drive follow-up calls otherwise needed to resolve these relationships. `CloudAccount` credentials are still obfuscated on return.

- **URL**: `/api/v1/sharedclouddrive/get_all_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| deleted | boolean | false | When `true`, return soft-deleted drives |
| name | string | null | Optional case-insensitive substring filter on drive name |

### Response

Returns a `DTOCollection<SharedCloudDriveWithDetailsDTO>`. Each item contains the base drive fields plus three collections:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "team-drive",
  "cloudAccount": { },
  "projectsData": [
    { "project": { }, "projectSharedCloudDrive": { } }
  ],
  "usersData": [
    { "user": { }, "permission": { } }
  ],
  "groupsData": [
    { "userGroup": { }, "permission": { } }
  ]
}
```


## Insert Shared Cloud Drive

Creates a new shared cloud drive. The server overrides `accountId` and `creatorUserId` on the request body with the caller's values.

- **URL**: `/api/v1/sharedclouddrive/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "00000000-0000-0000-0000-000000000000",
  "active": true,
  "name": "string (max 50)",
  "prefix": "string",
  "prefixId": "string",
  "storageId": "string",
  "storageName": "string",
  "objectsPerFolder": 10000,
  "allowDelete": false,
  "localCacheEncrypted": false,
  "cloudObjectsEncrypted": false,
  "driveType": 1,
  "syncType": 2
}
```

- `driveType` — `1` VirtualDrive, `2` LocalPath.
- `syncType` — `1` Stream (cloud-only with explicit offline selection), `2` Mirror (cloud + full local copy).
- `storageName` — the bucket name in the backing cloud.
- `prefix` — optional folder prefix within the bucket.
- `allowDelete` — when `true`, local deletions propagate to the cloud.

### Response

Returns the newly-created `SharedCloudDrive` object.


## Update Shared Cloud Drive

Updates an existing shared cloud drive. `accountId` and `creatorUserId` are always reset to the caller's values.

- **URL**: `/api/v1/sharedclouddrive/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

A full `SharedCloudDrive` object including the `id` of the record to update.

### Response

Returns the updated `SharedCloudDrive`.


## Delete Shared Cloud Drive

Soft-deletes a shared cloud drive and cascades cleanup of its related permissions and project attachments: every `UserSharedCloudDrivePermission`, `UserGroupSharedCloudDrivePermission`, and `ProjectSharedCloudDrive` referencing this drive is deleted before the drive itself is soft-deleted.

- **URL**: `/api/v1/sharedclouddrive/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the shared cloud drive to delete |

### Response

A `200 OK` status with no body on success.


## Get Shared Cloud Drives For Current User

Returns every shared cloud drive the current user can reach, whether permission is granted directly on the drive, through a user group, through a project, or through a project a user group is a member of. Duplicate drives across permission paths are de-duplicated. Each returned permission includes the fully-populated `SharedCloudDrive` with its decrypted `CloudAccount`.

- **URL**: `/api/v1/sharedclouddrive/get_shared_cloud_drive`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a `List<UserSharedCloudDrivePermission>`:

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "userId": "00000000-0000-0000-0000-000000000000",
    "sharedCloudDriveId": "00000000-0000-0000-0000-000000000000",
    "permissionType": 0,
    "sharedCloudDrive": { }
  }
]
```


## Sample Code

### Create a shared cloud drive

<details>
<summary>Python</summary>

```python
import requests

requests.post(
    "https://api.amove.io/api/v1/sharedclouddrive/insert",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json={
        "cloudAccountId": "00000000-0000-0000-0000-000000000000",
        "active": True,
        "name": "team-drive",
        "storageName": "team-bucket",
        "prefix": "team/",
        "objectsPerFolder": 10000,
        "allowDelete": False,
        "driveType": 1,
        "syncType": 2
    },
)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
await fetch("https://api.amove.io/api/v1/sharedclouddrive/insert", {
  method: "POST",
  headers: {
    "Authorization": "Bearer YOUR_JWT",
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    cloudAccountId: "00000000-0000-0000-0000-000000000000",
    active: true,
    name: "team-drive",
    storageName: "team-bucket",
    prefix: "team/",
    objectsPerFolder: 10000,
    allowDelete: false,
    driveType: 1,
    syncType: 2
  })
});
```

</details>

### List drives accessible to the current user

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var res = await client.GetAsync(
    "https://api.amove.io/api/v1/sharedclouddrive/get_shared_cloud_drive");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
