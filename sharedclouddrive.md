# SharedCloudDrive Endpoints

This document provides detailed information about the SharedCloudDrive-related endpoints in the AMove API. These endpoints allow you to manage shared cloud drives within the system.

## Endpoints

1. [Get All Shared Cloud Drives](#get-all-shared-cloud-drives)
2. [Insert Shared Cloud Drive](#insert-shared-cloud-drive)
3. [Update Shared Cloud Drive](#update-shared-cloud-drive)
4. [Delete Shared Cloud Drive](#delete-shared-cloud-drive)
5. [Get Shared Cloud Drive](#get-shared-cloud-drive)

## Get All Shared Cloud Drives

Retrieves the list of shared cloud drives defined in the system.

- **URL**: `/api/v1/sharedclouddrive/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| deleted | boolean | false | When true, return deleted records in the result |
| name | string | - | Shared cloud drive name to filter by |

### Response

```json
{
  "data": [
    {
      "id": "string (uuid)",
      "accountId": "string (uuid)",
      "creatorUserId": "string (uuid)",
      "cloudAccountId": "string (uuid)",
      "active": "boolean",
      "name": "string",
      "prefix": "string",
      "prefixId": "string",
      "storageId": "string",
      "storageName": "string",
      "objectsPerFolder": "integer",
      "allowDelete": "boolean",
      "localCacheEncrypted": "boolean",
      "cloudObjectsEncrypted": "boolean",
      "driveType": "integer (enum)",
      "syncType": "integer (enum)",
      "deleted": "boolean",
      "cloudAccount": {
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

## Insert Shared Cloud Drive

Insert a new shared cloud drive.

- **URL**: `/api/v1/sharedclouddrive/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "accountId": "string (uuid)",
  "creatorUserId": "string (uuid)",
  "cloudAccountId": "string (uuid)",
  "active": "boolean",
  "name": "string",
  "prefix": "string",
  "prefixId": "string",
  "storageId": "string",
  "storageName": "string",
  "objectsPerFolder": "integer",
  "allowDelete": "boolean",
  "localCacheEncrypted": "boolean",
  "cloudObjectsEncrypted": "boolean",
  "driveType": "integer (enum)",
  "syncType": "integer (enum)"
}
```

### Response

Returns the created shared cloud drive object.

## Update Shared Cloud Drive

Update an existing shared cloud drive.

- **URL**: `/api/v1/sharedclouddrive/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "creatorUserId": "string (uuid)",
  "cloudAccountId": "string (uuid)",
  "active": "boolean",
  "name": "string",
  "prefix": "string",
  "prefixId": "string",
  "storageId": "string",
  "storageName": "string",
  "objectsPerFolder": "integer",
  "allowDelete": "boolean",
  "localCacheEncrypted": "boolean",
  "cloudObjectsEncrypted": "boolean",
  "driveType": "integer (enum)",
  "syncType": "integer (enum)",
  "deleted": "boolean"
}
```

### Response

Returns the updated shared cloud drive object.

## Delete Shared Cloud Drive

Delete an existing shared cloud drive.

- **URL**: `/api/v1/sharedclouddrive/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of shared cloud drive to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Get Shared Cloud Drive

Retrieves the shared cloud drive permissions for the current user.

- **URL**: `/api/v1/sharedclouddrive/get_shared_cloud_drive`
- **Method**: GET
- **Auth Required**: Yes

### Response

```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (enum)",
    "sharedCloudDrive": {
      // SharedCloudDrive object
    }
  }
]
```

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get All Shared Cloud Drives

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/sharedclouddrive/get_all"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
params = {
    "page": 1,
    "pagesize": 10,
    "sortfield": "Name",
    "descending": False
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    shared_drives = response.json()
    for drive in shared_drives['data']:
        print(f"Shared Drive: {drive['name']}, Active: {drive['active']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/sharedclouddrive/get_all?page=1&pagesize=10&sortfield=Name&descending=false', {
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
  data.data.forEach(drive => {
    console.log(`Shared Drive: ${drive.name}, Active: ${drive.active}`);
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

            var response = await client.GetAsync("api/v1/sharedclouddrive/get_all?page=1&pagesize=10&sortfield=Name&descending=false");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var sharedDrives = JObject.Parse(content);
                foreach (var drive in sharedDrives["data"])
                {
                    Console.WriteLine($"Shared Drive: {drive["name"]}, Active: {drive["active"]}");
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

