# CloudAccount Endpoints

This document provides detailed information about the CloudAccount-related endpoints in the AMove API. These endpoints allow you to manage cloud storage accounts and their associated resources.

## Endpoints

1. [Get All Cloud Accounts](#get-all-cloud-accounts)
2. [Get AMove Storage](#get-amove-storage)
3. [Insert Cloud Account](#insert-cloud-account)
4. [Delete Cloud Account](#delete-cloud-account)
5. [List Buckets](#list-buckets)
6. [List Objects](#list-objects)
7. [IDrive Add Storage](#idrive-add-storage)
8. [IDrive Delete Storage](#idrive-delete-storage)
9. [IDrive Create Bucket](#idrive-create-bucket)
10. [IDrive Delete Bucket](#idrive-delete-bucket)
11. [IDrive Regions](#idrive-regions)

## Get All Cloud Accounts

Retrieves all cloud accounts associated with the current user's account.

- **URL**: `/api/v1/cloudaccount/get_all`
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
      "accountId": "string (uuid)",
      "userId": "string (uuid)",
      "cloudType": "integer (enum)",
      "name": "string",
      "accessKey": "string",
      "secretKey": "string",
      "credentialsData": "string",
      "serviceUrl": "string",
      "active": "boolean",
      "shared": "boolean",
      "internalStorage": "boolean",
      "storageTier": "integer (enum)",
      "deleted": "boolean"
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

## Get AMove Storage

Retrieves the AMove storage information associated with the current user's account.

- **URL**: `/api/v1/cloudaccount/get_amove_storage`
- **Method**: GET
- **Auth Required**: Yes

### Response

```json
[
  {
    "access_key": "string",
    "secret_key": "string",
    "service_url": "string"
  }
]
```

## Insert Cloud Account

Creates a new cloud account for the current user.

- **URL**: `/api/v1/cloudaccount/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "accountId": "string (uuid)",
  "userId": "string (uuid)",
  "cloudType": "integer (enum)",
  "name": "string",
  "accessKey": "string",
  "secretKey": "string",
  "credentialsData": "string",
  "serviceUrl": "string",
  "active": "boolean",
  "shared": "boolean",
  "internalStorage": "boolean",
  "storageTier": "integer (enum)"
}
```

### Response

Returns the created cloud account object.

## Delete Cloud Account

Deletes an existing cloud account.

- **URL**: `/api/v1/cloudaccount/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The ID of the cloud account to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## List Buckets

Retrieves the list of buckets associated with a specified cloud account.

- **URL**: `/api/v1/cloudaccount/list_buckets`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "includeRegion": "boolean"
}
```

### Response

```json
[
  {
    "id": "string",
    "name": "string",
    "region": "string",
    "size": "number",
    "creationDate": "string (date-time)"
  }
]
```

## List Objects

Retrieves the list of objects in a specified bucket.

- **URL**: `/api/v1/cloudaccount/list_objects`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "bucketName": "string",
  "path": "string",
  "continuationToken": "string",
  "count": "integer"
}
```

### Response

```json
{
  "data": [
    {
      "id": "string",
      "name": "string",
      "path": "string",
      "size": "integer",
      "lastModifiedUtc": "string (date-time)",
      "type": "integer (enum)"
    }
  ],
  "continuationToken": "string",
  "bucketName": "string",
  "path": "string"
}
```

## IDrive Add Storage

Adds a new storage to the current IDrive user's account.

- **URL**: `/api/v1/cloudaccount/idrive_add_storage`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "firstname": "string",
  "userEmail": "string",
  "cloudAccountName": "string",
  "bucketTitle": "string",
  "region": "string",
  "defaultPassword": "string",
  "shared": "boolean",
  "storageTier": "integer (enum)"
}
```

### Response

Returns the created cloud account object.

## IDrive Delete Storage

Deletes a storage from the current IDrive user's account.

- **URL**: `/api/v1/cloudaccount/idrive_delete_storage`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The ID of the storage to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## IDrive Create Bucket

Creates a new bucket in the current IDrive user's account.

- **URL**: `/api/v1/cloudaccount/idrive_create_bucket`
- **Method**: POST
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | The ID of the cloud account |
| bucketName | string | The name of the bucket to create |

### Response

A successful creation returns a `200 OK` status with no body.

## IDrive Delete Bucket

Deletes a bucket from the current IDrive user's account.

- **URL**: `/api/v1/cloudaccount/idrive_delete_bucket`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| cloudAccountId | string (uuid) | The ID of the cloud account |
| bucketName | string | The name of the bucket to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## IDrive Regions

Retrieves the list of available regions for IDrive.

- **URL**: `/api/v1/cloudaccount/idrive_regions`
- **Method**: GET
- **Auth Required**: Yes

### Response

```json
[
  {
    "name": "string",
    "code": "string"
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

### Get All Cloud Accounts

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/cloudaccount/get_all"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
params = {
    "page": 1,
    "pagesize": 10
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    cloud_accounts = response.json()
    for account in cloud_accounts['data']:
        print(f"Cloud Account: {account['name']}, Type: {account['cloudType']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/cloudaccount/get_all?page=1&pagesize=10', {
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
  data.data.forEach(account => {
    console.log(`Cloud Account: ${account.name}, Type: ${account.cloudType}`);
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

            var response = await client.GetAsync("api/v1/cloudaccount/get_all?page=1&pagesize=10");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var accounts = JObject.Parse(content);
                foreach (var account in accounts["data"])
                {
                    Console.WriteLine($"Cloud Account: {account["name"]}, Type: {account["cloudType"]}");
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

