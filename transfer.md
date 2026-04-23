# Transfer Endpoints

This document provides detailed information about the transfer endpoints in the AMove API. A `Transfer` is a cloud-to-cloud (C2C) copy operation between two cloud accounts the current user can access. The lifecycle is:

1. **Calculate cost** — estimate the USD cost before running the transfer.
2. **Transfer** — queue a new transfer; status progresses `Created → Running → Completed` (or `Error` / `Canceled`).
3. **Cancel / Retry / Download Failed Files** — manage an in-flight or completed transfer.

Progress updates are streamed in real time over the `/api/rm` SignalR hub; this REST surface lets you orchestrate transfers and query their state.

## Endpoints

1. [Get All Transfers](#get-all-transfers)
2. [Transfer](#transfer)
3. [Calculate Cost](#calculate-cost)
4. [Cancel Transfer](#cancel-transfer)
5. [Retry Transfer](#retry-transfer)
6. [Download Failed Files](#download-failed-files)


## Get All Transfers

Returns every transfer initiated by any user in the current account, excluding transfers still in the `Created` state (i.e. queued but never picked up).

- **URL**: `/api/v1/transfer/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "RequestDate" | Field to sort by |
| descending | boolean | true | Sort direction |
| type | integer | 15 | Bitmask of `TransferType` values to include. |

`type` is the bitwise OR of `TransferType`:

| Value | Type |
|---|---|
| 1 | Transfer — an ad-hoc cloud-to-cloud copy |
| 2 | Sync — a scheduled sync cycle |
| 4 | SyncInitTransfer — the initial population step of a new sync |
| 8 | ManualTransfer — a manually triggered transfer inside a sync |
| 15 | All of the above (default) |

### Response

Returns a `DTOCollection<Transfer>`. Each `Transfer` includes:

- `sourceCloudAccountId`, `sourceBucket`, `sourcePath`, `sourceRegion` and the matching `destination*` fields
- `transferStatus` — one of `None(0)`, `Created(1)`, `Running(2)`, `Completed(4)`, `Canceled(8)`, `Error(16)`, `Canceling(32)`, `Pausing(64)`, `Paused(128)`
- `total`, `transferred`, `failed`, `skipped`, `deleted` — object counts
- `totalSize`, `transferredSize`, plus `totalSizeString` / `transferredSizeString` for human-readable values
- `percent`, `averageSpeed` (bytes/sec), `transferredCost` (USD)
- `requestDate`, `startDate`, `endDate`

Embedded `sourceCloudAccount` and `destinationCloudAccount` objects have their credential fields masked.


## Transfer

Queues a new cloud-to-cloud transfer. The request is accepted immediately and persisted as a `Transfer` record with status `Created`; the background dispatcher picks it up, streams progress updates over SignalR, and finalizes the record when the operation ends.

- **URL**: `/api/v1/transfer/transfer`
- **Method**: POST
- **Auth Required**: Yes — `ProviderAdmin`, `AccountAdmin`, `DesktopAdmin`, or `DesktopCreativeUser`. (Regular `AccountUser` and `DesktopStandardUser` cannot initiate transfers.)

### Request Body

```json
{
  "sourceCloudAccountId": "string (uuid)",
  "sourceBucket": "string",
  "sourceBucketId": "string",
  "sourcePath": "string",
  "sourceId": "string",
  "destinationCloudAccountId": "string (uuid)",
  "destinationBucket": "string",
  "destinationBucketId": "string",
  "destinationPath": "string",
  "destinationId": "string",
  "allowSkip": true,
  "keepSourceTree": false
}
```

- `sourceCloudAccountId` / `destinationCloudAccountId` — ids of the source and destination cloud accounts. Both must be visible to the caller.
- `sourceBucket` / `destinationBucket` — bucket or container name on each side.
- `sourceBucketId` / `destinationBucketId` — provider-specific bucket ids (used by Dropbox, Box, OneDrive, Google Drive). Empty for S3-compatible providers.
- `sourcePath` / `destinationPath` — object key prefix on each side. A trailing `/` selects a folder; an exact key selects a single object.
- `sourceId` / `destinationId` — provider-specific object ids (used by non-S3 providers). Empty for S3-compatible providers.
- `allowSkip` — when `true`, the transfer skips objects whose newer copy already exists at the destination.
- `keepSourceTree` — when `true`, the source directory structure is preserved under `destinationPath`; when `false`, objects are flattened.

### Response

A `200 OK` status with no body. Poll [Get All Transfers](#get-all-transfers) or subscribe to the SignalR hub to observe progress.


## Calculate Cost

Estimates the USD cost of a transfer without running it. The cost factors in per-provider egress pricing and the total byte volume the transfer would move.

- **URL**: `/api/v1/transfer/calculate_cost`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

Same shape as the [Transfer](#transfer) request body.

### Response

Returns a single decimal value — the estimated cost in USD.

```json
0.42
```


## Cancel Transfer

Requests cancellation of a transfer that is currently in `Running` state. The transfer transitions through `Canceling` and ends in `Canceled`.

- **URL**: `/api/v1/transfer/cancel_transfer`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)"
}
```

- `id` — id of the `Transfer` to cancel. Must belong to a user in the caller's Amove account.

### Response

A `200 OK` status with no body on success.


## Retry Transfer

Creates a new transfer that re-processes only the failed files from a previous run. The new transfer is linked to the original via the `retryBatchTransferID` field.

- **URL**: `/api/v1/transfer/retry_transfer`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)"
}
```

- `id` — id of the `Transfer` whose failures should be retried.

### Response

A `200 OK` status with no body on success.


## Download Failed Files

Generates a signed URL for a CSV report listing the files that failed during the specified transfer, along with their reasons. The URL is short-lived and can be handed off to a browser for direct download.

- **URL**: `/api/v1/transfer/download_failed_files`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)"
}
```

- `id` — id of the `Transfer` to report on.

### Response

Returns a single string — the signed download URL.

```json
"https://..."
```


## Sample Code

### Calculate cost, then run a transfer

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
JWT = "YOUR_JWT"

payload = {
    "sourceCloudAccountId": "00000000-0000-0000-0000-000000000000",
    "sourceBucket": "source-bucket",
    "sourcePath": "/photos/",
    "destinationCloudAccountId": "11111111-1111-1111-1111-111111111111",
    "destinationBucket": "dest-bucket",
    "destinationPath": "/backup/photos/",
    "allowSkip": True,
    "keepSourceTree": True,
}

cost = requests.post(
    f"{BASE}/api/v1/transfer/calculate_cost",
    headers={"Authorization": f"Bearer {JWT}"},
    json=payload,
).json()
print(f"Estimated cost: ${cost}")

requests.post(
    f"{BASE}/api/v1/transfer/transfer",
    headers={"Authorization": f"Bearer {JWT}"},
    json=payload,
)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const BASE = "https://api.amove.io";
const JWT = "YOUR_JWT";

const payload = {
  sourceCloudAccountId: "00000000-0000-0000-0000-000000000000",
  sourceBucket: "source-bucket",
  sourcePath: "/photos/",
  destinationCloudAccountId: "11111111-1111-1111-1111-111111111111",
  destinationBucket: "dest-bucket",
  destinationPath: "/backup/photos/",
  allowSkip: true,
  keepSourceTree: true
};

const headers = {
  "Authorization": `Bearer ${JWT}`,
  "Content-Type": "application/json"
};

const cost = await fetch(`${BASE}/api/v1/transfer/calculate_cost`, {
  method: "POST", headers, body: JSON.stringify(payload)
}).then(r => r.json());
console.log("Estimated cost:", cost);

await fetch(`${BASE}/api/v1/transfer/transfer`, {
  method: "POST", headers, body: JSON.stringify(payload)
});
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Net.Http.Json;

using var client = new HttpClient { BaseAddress = new Uri("https://api.amove.io/") };
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var payload = new
{
    sourceCloudAccountId = Guid.Parse("00000000-0000-0000-0000-000000000000"),
    sourceBucket = "source-bucket",
    sourcePath = "/photos/",
    destinationCloudAccountId = Guid.Parse("11111111-1111-1111-1111-111111111111"),
    destinationBucket = "dest-bucket",
    destinationPath = "/backup/photos/",
    allowSkip = true,
    keepSourceTree = true
};

decimal cost = await (await client.PostAsJsonAsync("api/v1/transfer/calculate_cost", payload))
    .Content.ReadFromJsonAsync<decimal>();
Console.WriteLine($"Estimated cost: ${cost}");

await client.PostAsJsonAsync("api/v1/transfer/transfer", payload);
```

</details>


### Cancel an in-flight transfer

<details>
<summary>Python</summary>

```python
import requests

requests.post(
    "https://api.amove.io/api/v1/transfer/cancel_transfer",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json={"id": "00000000-0000-0000-0000-000000000000"},
)
```

</details>


### Retry failed files and download the failure report

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
JWT = "YOUR_JWT"
transfer_id = "00000000-0000-0000-0000-000000000000"

requests.post(
    f"{BASE}/api/v1/transfer/retry_transfer",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"id": transfer_id},
)

download_url = requests.post(
    f"{BASE}/api/v1/transfer/download_failed_files",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"id": transfer_id},
).json()
print(download_url)
```

</details>


For error handling, see [Error Model](errors.md).
