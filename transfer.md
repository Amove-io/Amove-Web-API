# Transfer Endpoints

This document provides detailed information about the Transfer-related endpoints in the AMove API. These endpoints allow you to manage and monitor data transfer operations between cloud storage accounts.

## Endpoints

1. [Get All Transfers](#get-all-transfers)
2. [Initiate Transfer](#initiate-transfer)
3. [Calculate Transfer Cost](#calculate-transfer-cost)
4. [Cancel Transfer](#cancel-transfer)
5. [Retry Transfer](#retry-transfer)
6. [Download Failed Files](#download-failed-files)

## Get All Transfers

Retrieves the list of transfers associated with the current user's account.

- **URL**: `/api/v1/transfer/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "RequestDate" | Field to sort by |
| descending | boolean | true | Sort direction; descending: true |
| type | integer | 15 | Transfer type |

### Response

```json
{
  "data": [
    {
      "id": "string (uuid)",
      "userId": "string (uuid)",
      "syncCycleId": "string (uuid)",
      "sourceCloudAccountId": "string (uuid)",
      "sourceBucket": "string",
      "sourceRegion": "string",
      "sourcePath": "string",
      "destinationCloudAccountId": "string (uuid)",
      "destinationBucket": "string",
      "destinationRegion": "string",
      "destinationPath": "string",
      "requestDate": "string (date-time)",
      "startDate": "string (date-time)",
      "endDate": "string (date-time)",
      "transferStatus": "integer (enum)",
      "transferType": "integer (enum)",
      "allowSkip": "boolean",
      "allowDelete": "boolean",
      "total": "integer",
      "failed": "integer",
      "deleted": "integer",
      "skipped": "integer",
      "skippedSize": "integer",
      "transferred": "integer",
      "totalSize": "integer",
      "totalSizeString": "string",
      "transferredSize": "integer",
      "transferredSizeString": "string",
      "percent": "number",
      "averageSpeed": "number",
      "transferredCost": "number",
      "retryBatchTransferID": "string (uuid)",
      "syncCycle": {
        // SyncCycle object
      },
      "sourceCloudAccount": {
        // CloudAccount object
      },
      "destinationCloudAccount": {
        // CloudAccount object
      }
    }
  ],
  "total": "integer",
  "options": {
    "pageSize": "integer",
    "page": "integer",
    "sort": [
      {
        "field": "string",
        "descending": "boolean"
      }
    ]
  }
}
```

## Initiate Transfer

Sends a transfer request to initiate a new data transfer.

- **URL**: `/api/v1/transfer/transfer`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "sourceCloudAccountId": "string (uuid)",
  "sourceBucket": "string",
  "sourcePath": "string",
  "destinationCloudAccountId": "string (uuid)",
  "destinationBucket": "string",
  "destinationPath": "string",
  "allowSkip": "boolean"
}
```

### Response

A successful initiation returns a `200 OK` status with no body.

## Calculate Transfer Cost

Calculates the cost of a transfer; returns the cost in USD.

- **URL**: `/api/v1/transfer/calculate_cost`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "sourceCloudAccountId": "string (uuid)",
  "sourceBucket": "string",
  "sourcePath": "string",
  "destinationCloudAccountId": "string (uuid)",
  "destinationBucket": "string",
  "destinationPath": "string",
  "allowSkip": "boolean"
}
```

### Response

Returns a number representing the calculated cost in USD.

## Cancel Transfer

Sends a cancel request to a running transfer.

- **URL**: `/api/v1/transfer/cancel_transfer`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)"
}
```

### Response

A successful cancellation returns a `200 OK` status with no body.

## Retry Transfer

Sends a retry (failed files) request to a running transfer.

- **URL**: `/api/v1/transfer/retry_transfer`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)"
}
```

### Response

A successful retry initiation returns a `200 OK` status with no body.

## Download Failed Files

Sends a request to download failed files from a transfer.

- **URL**: `/api/v1/transfer/download_failed_files`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)"
}
```

### Response

Returns a string containing the URL to download the failed files.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get All Transfers

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/transfer/get_all"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
params = {
    "page": 1,
    "pagesize": 10,
    "sortfield": "RequestDate",
    "descending": True,
    "type": 15
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    transfers = response.json()
    for transfer in transfers['data']:
        print(f"Transfer ID: {transfer['id']}, Status: {transfer['transferStatus']}, Progress: {transfer['percent']}%")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/transfer/get_all?page=1&pagesize=10&sortfield=RequestDate&descending=true&type=15', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN_HERE'
  }
})
.then(response => {
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
})
.then(data => {
  data.data.forEach(transfer => {
    console.log(`Transfer ID: ${transfer.id}, Status: ${transfer.transferStatus}, Progress: ${transfer.percent}%`);
  });
})
.catch(error => {
  console.error('Error:', error);
});
```

</details>

<details>
<summary>C#</summary>

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Newtonsoft.Json.Linq;

class Program
{
    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        {
            client.BaseAddress = new Uri("https://api.amove.com/");
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", "YOUR_TOKEN_HERE");

            var response = await client.GetAsync("api/v1/transfer/get_all?page=1&pagesize=10&sortfield=RequestDate&descending=true&type=15");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var transfers = JObject.Parse(content);
                foreach (var transfer in transfers["data"])
                {
                    Console.WriteLine($"Transfer ID: {transfer["id"]}, Status: {transfer["transferStatus"]}, Progress: {transfer["percent"]}%");
                }
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
            }
        }
    }
}
```

</details>

For more detailed examples and usage of other endpoints, please refer to our [Examples Directory](examples/README.md).

