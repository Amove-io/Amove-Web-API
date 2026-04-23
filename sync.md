# Sync Endpoints

This document provides detailed information about the sync endpoints in the AMove API. A sync entity replicates objects from a source cloud bucket to a destination cloud bucket on a repeating schedule. Access is restricted to `ProviderAdmin`, `AccountAdmin`, and `AccountUser`.

## Endpoints

1. [Get All Syncs](#get-all-syncs)
2. [Get Sync Jobs](#get-sync-jobs)
3. [Get Sync](#get-sync)
4. [Insert Sync](#insert-sync)
5. [Activate Sync](#activate-sync)
6. [Delete Sync](#delete-sync)


## Get All Syncs

Returns every sync entity owned by users in the current user's account. Cloud-account credentials on the source and destination are obfuscated before return.

- **URL**: `/api/v1/sync/get_all`
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

Returns a `DTOCollection<SyncInfo>`:

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "userId": "00000000-0000-0000-0000-000000000000",
      "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
      "sourceBucket": "source-bucket",
      "sourceRegion": "us-east-1",
      "destinationCloudAccountId": "00000000-0000-0000-0000-000000000000",
      "destinationBucket": "destination-bucket",
      "destinationRegion": "us-west-2",
      "allowDelete": false,
      "active": true,
      "deleted": false,
      "createDate": "2026-01-01T00:00:00Z",
      "sourceCloudAccount": { },
      "destinationCloudAccount": { }
    }
  ],
  "total": 1
}
```


## Get Sync Jobs

Returns every sync entity in the account alongside its most recent cycle and the last error (if any).

- **URL**: `/api/v1/sync/get_jobs`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Info.CreateDate" | Field to sort by (properties of the inner `Info` are prefixed with `Info.`) |
| descending | boolean | true | Sort direction |

### Response

Returns a `DTOCollection<SyncInfoJob>`. Each item:

```json
{
  "info": { },
  "lastCycle": { },
  "lastError": ""
}
```

Cloud-account credentials on the nested `info` are obfuscated before return.


## Get Sync

Retrieves a single sync entity by id. The sync must belong to a user in the caller's account; otherwise `NOT_FOUND` is returned.

- **URL**: `/api/v1/sync/get`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the sync entity |

### Response

Returns the `SyncInfo` object (see [Get All Syncs](#get-all-syncs) for shape).


## Insert Sync

Creates a new sync entity. The server:

- Stamps `userId` from the caller and `createDate` to the current UTC time.
- Validates that the source and destination cloud providers are permitted by the server configuration.
- Rejects same-account + same-bucket source/destination, and rejects a second sync on the same source bucket.
- Calculates source and destination regions from the underlying cloud providers and enforces any transfer-cost rules.
- Enables storage logging on the source cloud when the provider supports it; if not supported, the insert is rejected with error code `SYNC`.
- Raises the `sync.create` event for email notification.

- **URL**: `/api/v1/sync/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "sourceBucket": "source-bucket",
  "destinationCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "destinationBucket": "destination-bucket",
  "allowDelete": false,
  "active": true
}
```

- `allowDelete` — when `true`, objects deleted from the source bucket are also deleted in the destination on the next cycle.

### Response

A `200 OK` status with no body on success.


## Activate Sync

Activates or deactivates a sync entity.

- **URL**: `/api/v1/sync/activate`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "active": true
}
```

### Response

A `200 OK` status with no body on success. If the sync does not belong to the caller's account, `NOT_FOUND` is returned.


## Delete Sync

Soft-deletes a sync entity and disables storage logging on its source bucket if the provider supports it. Because `SyncInfo` implements `ILogicalDeleteableEntity`, the record is marked `Deleted = true` rather than physically removed.

- **URL**: `/api/v1/sync/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the sync entity to delete |

### Response

Returns a boolean indicating whether at least one record was deleted.


## Sample Code

### Create and activate a sync

<details>
<summary>Python</summary>

```python
import requests

JWT = "YOUR_JWT"
BASE = "https://api.amove.io"

sync = requests.post(
    f"{BASE}/api/v1/sync/insert",
    headers={"Authorization": f"Bearer {JWT}"},
    json={
        "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
        "sourceBucket": "source-bucket",
        "destinationCloudAccountId": "00000000-0000-0000-0000-000000000000",
        "destinationBucket": "destination-bucket",
        "allowDelete": False,
        "active": True
    },
)

requests.post(
    f"{BASE}/api/v1/sync/activate",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"id": "00000000-0000-0000-0000-000000000000", "active": True},
)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const JWT = "YOUR_JWT";
const BASE = "https://api.amove.io";

await fetch(`${BASE}/api/v1/sync/insert`, {
  method: "POST",
  headers: { "Authorization": `Bearer ${JWT}`, "Content-Type": "application/json" },
  body: JSON.stringify({
    sourceCloudAccountId: "00000000-0000-0000-0000-000000000000",
    sourceBucket: "source-bucket",
    destinationCloudAccountId: "00000000-0000-0000-0000-000000000000",
    destinationBucket: "destination-bucket",
    allowDelete: false,
    active: true
  })
});
```

</details>

### List sync jobs with their last cycle

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var res = await client.GetAsync(
    "https://api.amove.io/api/v1/sync/get_jobs?page=1&pagesize=50");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
