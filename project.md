# Project Endpoints

This document provides detailed information about the Project-related endpoints in the AMove API. These endpoints allow you to manage projects and their associated resources.

## Endpoints

1. [Get All Projects](#get-all-projects)
2. [Insert Project](#insert-project)
3. [Update Project](#update-project)
4. [Delete Project](#delete-project)
5. [Get Assigned Shared Cloud Drives](#get-assigned-shared-cloud-drives)
6. [Get Assigned Projects](#get-assigned-projects)
7. [Assign Shared Cloud Drive](#assign-shared-cloud-drive)
8. [Unassign Shared Cloud Drive](#unassign-shared-cloud-drive)

## Get All Projects

Retrieves the list of projects defined in the system.

- **URL**: `/api/v1/project/get_all`
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
| name | string | - | Project name to filter by |

### Response

```json
{
  "data": [
    {
      "id": "string (uuid)",
      "accountId": "string (uuid)",
      "name": "string",
      "description": "string",
      "active": "boolean",
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

## Insert Project

Insert a new project.

- **URL**: `/api/v1/project/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "accountId": "string (uuid)",
  "name": "string",
  "description": "string",
  "active": "boolean",
  "deleted": "boolean"
}
```

### Response

Returns the created project object.

## Update Project

Update an existing project.

- **URL**: `/api/v1/project/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "name": "string",
  "description": "string",
  "active": "boolean",
  "deleted": "boolean"
}
```

### Response

Returns the updated project object.

## Delete Project

Delete an existing project.

- **URL**: `/api/v1/project/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of project to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Get Assigned Shared Cloud Drives

Retrieves the list of shared cloud drives assigned to a project.

- **URL**: `/api/v1/project/get_assigned_sharedclouddrives`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| projectId | string (uuid) | - | Project ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedCloudDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |

### Response

```json
{
  "data": [
    {
      "sharedCloudDrive": {
        "id": "string (uuid)",
        "name": "string",
        "prefix": "string",
        "storageName": "string"
      },
      "project": {
        "id": "string (uuid)",
        "name": "string",
        "description": "string"
      },
      "projectSharedCloudDrive": {
        "id": "string (uuid)",
        "projectId": "string (uuid)",
        "sharedCloudDriveId": "string (uuid)"
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

## Get Assigned Projects

Retrieves the list of projects assigned to a shared cloud drive.

- **URL**: `/api/v1/project/get_assigned_projects`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sharedCloudDriveId | string (uuid) | - | Shared Cloud Drive ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |

### Response

```json
{
  "data": [
    {
      "sharedCloudDrive": {
        "id": "string (uuid)",
        "name": "string",
        "prefix": "string",
        "storageName": "string"
      },
      "project": {
        "id": "string (uuid)",
        "name": "string",
        "description": "string"
      },
      "projectSharedCloudDrive": {
        "id": "string (uuid)",
        "projectId": "string (uuid)",
        "sharedCloudDriveId": "string (uuid)"
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

## Assign Shared Cloud Drive

Assign a shared cloud drive to a project.

- **URL**: `/api/v1/project/assign_sharedclouddrive`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
[
  {
    "id": "string (uuid)",
    "projectId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)"
  }
]
```

### Response

A successful assignment returns a `200 OK` status with no body.

## Unassign Shared Cloud Drive

Unassign a shared cloud drive from a project.

- **URL**: `/api/v1/project/unassign_sharedclouddrive`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the assignment to delete |

### Response

A successful unassignment returns a `200 OK` status with no body.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get All Projects

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/project/get_all"
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
    projects = response.json()
    for project in projects['data']:
        print(f"Project: {project['name']}, Active: {project['active']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/project/get_all?page=1&pagesize=10&sortfield=Name&descending=false', {
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
  data.data.forEach(project => {
    console.log(`Project: ${project.name}, Active: ${project.active}`);
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

            var response = await client.GetAsync("api/v1/project/get_all?page=1&pagesize=10&sortfield=Name&descending=false");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var projects = JObject.Parse(content);
                foreach (var project in projects["data"])
                {
                    Console.WriteLine($"Project: {project["name"]}, Active: {project["active"]}");
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

