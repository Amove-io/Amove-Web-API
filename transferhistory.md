# Transfer History Endpoints

This document provides detailed information about the transfer-history endpoints in the Amove API. A `TransferHistory` record tracks a single transfer job (direct transfer, cloud drive upload/download, sync cycle, or Fastr transfer) with aggregated progress, timing, and throughput.

Access is restricted to `ProviderAdmin`, `AccountAdmin`, `AccountUser`, `DesktopAdmin`, `DesktopCreativeUser`, and `DesktopStandardUser`.

## Endpoints

1. [Get All Transfer History](#get-all-transfer-history)
2. [Get Trends](#get-trends)
3. [Get By Name](#get-by-name)
4. [Insert Transfer History](#insert-transfer-history)
5. [Update Transfer History Status](#update-transfer-history-status)
6. [Delete Transfer History](#delete-transfer-history)


## Get All Transfer History

Returns transfer-history records. Admin users (`ProviderAdmin`, `AccountAdmin`, `DesktopAdmin`) see every transfer in their account; other users see only transfers they initiated. Each row is enriched with the initiating user's email and username and the source/destination cloud-account names.

- **URL**: `/api/v1/transferhistory/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "StartDate" | Field to sort by |
| descending | boolean | true | Sort direction |
| origin | integer (enum) | 0 | Filter by `TransferHistoryOrigin`: `0` = any, `1` Direct, `2` Drive, `4` Sync, `8` FastrLocal, `16` FastrServerToServer |
| status | integer (enum) | 0 | Filter by `TransferStatus`: `0` = any, `1` Created, `2` Running, `4` Completed, `8` Canceled, `16` Error, `32` Canceling, `64` Pausing, `128` Paused |
| direction | integer (enum) | 0 | Filter by `TransferHistoryDirection`: `0` = any, `1` Upload, `2` Download |

### Response

Returns a `DTOCollection<TransferHistoryDetail>`. Each item:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "userId": "00000000-0000-0000-0000-000000000000",
  "accountId": "00000000-0000-0000-0000-000000000000",
  "userEmail": "user@example.com",
  "userName": "user@example.com",
  "origin": 2,
  "direction": 1,
  "transferStatus": 4,
  "startDate": "2026-01-01T00:00:00Z",
  "endDate": "2026-01-01T00:10:00Z",
  "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "sourceCloudProvider": 1,
  "sourceBucket": "source-bucket",
  "sourcePath": "folder/file.bin",
  "sourceCloudAccountName": "My AWS",
  "destinationCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "destinationCloudProvider": 8,
  "destinationBucket": "destination-bucket",
  "destinationPath": "archive/file.bin",
  "destinationCloudAccountName": "My S3-Compatible",
  "totalObjects": 100,
  "transferredObjects": 100,
  "failedObjects": 0,
  "skippedObjects": 0,
  "totalSize": 10737418240,
  "transferredSize": 10737418240,
  "totalSizeString": "10.0 GB",
  "transferredSizeString": "10.0 GB",
  "averageSpeed": 17895697.0,
  "errorMessage": null,
  "name": "Nightly archive"
}
```


## Get Trends

Returns per-day aggregated transfer trends for a given backup/sync `name` over the last `days` days, scoped to the caller's account. Useful for rendering charts.

- **URL**: `/api/v1/transferhistory/get_trends`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| name | string | — | Name to filter by (matches the `Name` field on `TransferHistory`) |
| days | integer | 30 | Number of trailing days to include |

### Response

Returns a `List<TransferTrendDay>`:

```json
[
  {
    "date": "2026-04-01T00:00:00Z",
    "transferCount": 12,
    "completedVolume": 107374182400,
    "speedSum": 150000000.0,
    "speedCount": 10,
    "completedCount": 10,
    "failedCount": 1,
    "canceledCount": 1
  }
]
```

- `completedVolume` — sum of `transferredSize` for completed transfers that day.
- `speedSum` / `speedCount` — sum and count of `averageSpeed` for completed transfers with non-zero speed. The client computes the average as `speedSum / speedCount`.


## Get By Name

Returns transfer-history records filtered by backup/sync `name`, restricted to `Sync`-origin transfers. Admin users see all account transfers; other users see only their own.

- **URL**: `/api/v1/transferhistory/get_by_name`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| name | string | — | Name to filter by |
| status | integer (enum) | 0 | Filter by `TransferStatus` (see [Get All Transfer History](#get-all-transfer-history)) |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "StartDate" | Field to sort by |
| descending | boolean | true | Sort direction |

### Response

Returns a `DTOCollection<TransferHistoryDetail>` (same shape as [Get All Transfer History](#get-all-transfer-history)).


## Insert Transfer History

Creates a new transfer-history record. The server stamps `userId` and `accountId` from the caller, defaults `startDate` to now when unset, and defaults `transferStatus` to `Created` when unset.

- **URL**: `/api/v1/transferhistory/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "destinationCloudAccountId": "00000000-0000-0000-0000-000000000000",
  "sourceCloudProvider": 1,
  "destinationCloudProvider": 8,
  "sourceBucket": "source-bucket",
  "sourcePath": "folder/",
  "destinationBucket": "destination-bucket",
  "destinationPath": "archive/",
  "origin": 1,
  "direction": 1,
  "transferStatus": 1,
  "startDate": "2026-01-01T00:00:00Z",
  "totalObjects": 0,
  "totalSize": 0,
  "name": "Nightly archive"
}
```

- `origin`, `direction`, `transferStatus` — see enum tables in [Get All Transfer History](#get-all-transfer-history).
- `sourceCloudProvider` / `destinationCloudProvider` — `CloudProvider` flag: `1` AWS, `2` Wasabi, `4` Azure, `8` S3Generic, `16` Google, `32` Frame, `64` Cloudflare, `128` Dropbox, `256` Box, `512` OneDrive, `1024` GoogleDrive, `2048` LocalHttp, `4096` FastrServer, `8192` Atomosphere.

### Response

Returns the newly-created `TransferHistory` object.


## Update Transfer History Status

Updates the mutable progress fields on an existing transfer-history record. Only the caller who originally created the record can update it — otherwise `Forbid` is returned.

- **URL**: `/api/v1/transferhistory/update_status`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "transferStatus": 4,
  "endDate": "2026-01-01T00:10:00Z",
  "totalObjects": 100,
  "transferredObjects": 100,
  "failedObjects": 0,
  "skippedObjects": 0,
  "totalSize": 10737418240,
  "transferredSize": 10737418240,
  "averageSpeed": 17895697.0,
  "errorMessage": null
}
```

Only these fields are copied onto the existing record; all other fields on the existing record (source/destination, origin, direction, user, account, etc.) are preserved.

### Response

Returns the updated `TransferHistory`.


## Delete Transfer History

Deletes a transfer-history record. Only the caller who originally created the record can delete it — otherwise `Forbid` is returned.

- **URL**: `/api/v1/transferhistory/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the transfer-history record to delete |

### Response

A `200 OK` status with no body on success.


## Sample Code

### List recent completed transfers

<details>
<summary>Python</summary>

```python
import requests

res = requests.get(
    "https://api.amove.io/api/v1/transferhistory/get_all",
    headers={"Authorization": "Bearer YOUR_JWT"},
    params={"page": 1, "pagesize": 50, "status": 4},  # status 4 = Completed
).json()
for row in res["data"]:
    print(row["name"], row["transferredSizeString"], row["averageSpeed"])
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch(
  "https://api.amove.io/api/v1/transferhistory/get_all?page=1&pagesize=50&status=4",
  { headers: { "Authorization": "Bearer YOUR_JWT" } }
).then(r => r.json());
console.log(res.data);
```

</details>

### Fetch 30-day trend for a named sync

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var res = await client.GetAsync(
    "https://api.amove.io/api/v1/transferhistory/get_trends?name=Nightly%20archive&days=30");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>


For error handling, see [Error Model](errors.md).
