# Storage Endpoints

This document provides detailed information about the Storage-related endpoints in the AMove API. These endpoints allow you to manage storage keys, buckets, and their associated settings.

## Endpoints

1. [Get Storage Keys](#get-storage-keys)
2. [Create Storage Key](#create-storage-key)
3. [Delete Storage Key](#delete-storage-key)
4. [Create Bucket](#create-bucket)
5. [Get Bucket Status](#get-bucket-status)
6. [Update Bucket](#update-bucket)
7. [Delete Bucket](#delete-bucket)

## Get Storage Keys

Retrieves all storage keys for the current user.

- **URL**: `/api/v1/storage/get_storage_key`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | The page number for pagination |
| pagesize | integer | 50 | The number of items per page |
| sortfield | string | "CreateDate" | The field to sort the results by |
| descending | boolean | true | Whether to sort in descending order |

### Response

```json
{
  "data": [
    {
      "id": "string (uuid)",
      "userId": "string (uuid)",
      "accountId": "string (uuid)",
      "name": "string",
      "accessKey": "string",
      "description": "string",
      "region": "string",
      "storageDn": "string",
      "storageTier": "integer (enum)",
      "createDate": "string (date-time)"
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

## Create Storage Key

Creates a new storage key.

- **URL**: `/api/v1/storage/create_storage_key`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "name": "string",
  "region": "string",
  "storageDn": "string",
  "permission": "integer (enum)",
  "allBuckets": "boolean",
  "selectedBuckets": [
    "string"
  ],
  "cloudAccountId": "string (uuid)"
}
```

### Response

```json
{
  "id": "string (uuid)",
  "userId": "string (uuid)",
  "accountId": "string (uuid)",
  "name": "string",
  "accessKey": "string",
  "description": "string",
  "region": "string",
  "storageDn": "string",
  "storageTier": "integer (enum)",
  "createDate": "string (date-time)",
  "secretKey": "string"
}
```

## Delete Storage Key

Deletes a storage key.

- **URL**: `/api/v1/storage/delete_storage_key`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The ID of the storage key to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Create Bucket

Creates a new bucket in the current IDrive user's account.

- **URL**: `/api/v1/storage/create_bucket`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "bucketName": "string",
  "isPublic": "boolean",
  "isEncrypted": "boolean",
  "versioningEnabled": "boolean",
  "objectLockEnabled": "boolean"
}
```

### Response

A successful creation returns a `200 OK` status with no body.

## Get Bucket Status

Retrieves the status of a bucket.

- **URL**: `/api/v1/storage/bucket_status`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "bucketName": "string"
}
```

### Response

```json
{
  "versioningEnabled": "boolean",
  "encryptionEnabled": "boolean",
  "isPublic": "boolean",
  "objectLockEnabled": "boolean"
}
```

## Update Bucket

Updates a bucket in the current IDrive user's account.

- **URL**: `/api/v1/storage/update_bucket`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "bucketName": "string",
  "isPublic": "boolean",
  "isEncrypted": "boolean",
  "versioningEnabled": "boolean"
}
```

### Response

A successful update returns a `200 OK` status with no body.

## Delete Bucket

Deletes a bucket from the current IDrive user's account.

- **URL**: `/api/v1/storage/delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | The ID of the cloud account |
| bucketName | string | The name of the bucket to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get Storage Keys

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/storage/get_storage_key"
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
    storage_keys = response.json()
    for key in storage_keys['data']:
        print(f"Storage Key: {key['name']}, Access Key: {key['accessKey']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/storage/get_storage_key?page=1&pagesize=10&sortfield=CreateDate&descending=true', {
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
  data.data.forEach(key => {
    console.log(`Storage Key: ${key.name}, Access Key: ${key.accessKey}`);
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

            var response = await client.GetAsync("api/v1/storage/get_storage_key?page=1&pagesize=10&sortfield=CreateDate&descending=true");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var storageKeys = JObject.Parse(content);
                foreach (var key in storageKeys["data"])
                {
                    Console.WriteLine($"Storage Key: {key["name"]}, Access Key: {key["accessKey"]}");
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

