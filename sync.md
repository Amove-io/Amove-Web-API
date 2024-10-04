# Sync Endpoints

This document provides detailed information about the Sync-related endpoints in the AMove API. These endpoints allow you to manage synchronization tasks between cloud storage accounts.

## Endpoints

1. [Get All Sync Entities](#get-all-sync-entities)
2. [Get Sync Jobs](#get-sync-jobs)
3. [Get Sync Entity](#get-sync-entity)
4. [Insert Sync Entity](#insert-sync-entity)
5. [Activate/Deactivate Sync Entity](#activatedeactivate-sync-entity)
6. [Delete Sync Entity](#delete-sync-entity)

## Get All Sync Entities

Retrieves the list of sync entities associated with the current user's account.

- **URL**: `/api/v1/sync/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction; descending: true |

### Response

```json
{
  "data": [
    {
      "id": "string (uuid)",
      "userId": "string (uuid)",
      "sourceCloudAccountId": "string (uuid)",
      "sourceBucket": "string",
      "sourceRegion": "string",
      "destinationCloudAccountId": "string (uuid)",
      "destinationBucket": "string",
      "destinationRegion": "string",
      "allowDelete": "boolean",
      "active": "boolean",
      "autoDeactive": "boolean",
      "deleted": "boolean",
      "createDate": "string (date-time)",
      "user": {
        // User object
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

## Get Sync Jobs

Retrieves the list of sync jobs associated with the current user's account.

- **URL**: `/api/v1/sync/get_jobs`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Info.CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction; descending: true |

### Response

```json
{
  "data": [
    {
      "lastError": "string",
      "lastCycle": {
        // SyncCycle object
      },
      "info": {
        // SyncInfo object
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

## Get Sync Entity

Retrieves a sync entity by the specified id.

- **URL**: `/api/v1/sync/get`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Sync entity id |

### Response

Returns a `SyncInfo` object.

## Insert Sync Entity

Creates a sync entity.

- **URL**: `/api/v1/sync/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "userId": "string (uuid)",
  "sourceCloudAccountId": "string (uuid)",
  "sourceBucket": "string",
  "sourceRegion": "string",
  "destinationCloudAccountId": "string (uuid)",
  "destinationBucket": "string",
  "destinationRegion": "string",
  "allowDelete": "boolean",
  "active": "boolean",
  "autoDeactive": "boolean"
}
```

### Response

A successful creation returns a `200 OK` status with no body.

## Activate/Deactivate Sync Entity

Activates or deactivates a sync entity.

- **URL**: `/api/v1/sync/activate`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "active": "boolean"
}
```

### Response

A successful activation/deactivation returns a `200 OK` status with no body.

## Delete Sync Entity

Deletes a sync entity.

- **URL**: `/api/v1/sync/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Sync entity id |

### Response

Returns a boolean indicating success or failure.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get All Sync Entities

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/sync/get_all"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
params = {
    "page": 1,
    "pagesize": 10,
    "sortfield": "CreateDate",
    "descending": True
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    sync_entities = response.json()
    for entity in sync_entities['data']:
        print(f"Sync Entity: {entity['id']}, Source: {entity['sourceBucket']}, Destination: {entity['destinationBucket']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/sync/get_all?page=1&pagesize=10&sortfield=CreateDate&descending=true', {
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
  data.data.forEach(entity => {
    console.log(`Sync Entity: ${entity.id}, Source: ${entity.sourceBucket}, Destination: ${entity.destinationBucket}`);
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

            var response = await client.GetAsync("api/v1/sync/get_all?page=1&pagesize=10&sortfield=CreateDate&descending=true");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var syncEntities = JObject.Parse(content);
                foreach (var entity in syncEntities["data"])
                {
                    Console.WriteLine($"Sync Entity: {entity["id"]}, Source: {entity["sourceBucket"]}, Destination: {entity["destinationBucket"]}");
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

