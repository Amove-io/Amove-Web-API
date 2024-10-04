# UsersPermission Endpoints

This document provides detailed information about the UsersPermission-related endpoints in the AMove API. These endpoints allow you to manage permissions for users and user groups on projects and shared cloud drives.

## Endpoints

1. [Users Project Permission](#users-project-permission)
2. [Get Users Assigned to Project](#get-users-assigned-to-project)
3. [Get Projects Assigned to User](#get-projects-assigned-to-user)
4. [Users Shared Cloud Drive Permission](#users-shared-cloud-drive-permission)
5. [Get Users Assigned to Shared Cloud Drive](#get-users-assigned-to-shared-cloud-drive)
6. [Get Shared Cloud Drives Assigned to User](#get-shared-cloud-drives-assigned-to-user)
7. [User Groups Project Permission](#user-groups-project-permission)
8. [Get User Groups Assigned to Project](#get-user-groups-assigned-to-project)
9. [Get Projects Assigned to User Group](#get-projects-assigned-to-user-group)
10. [User Group Shared Cloud Drive Permission](#user-group-shared-cloud-drive-permission)
11. [Get User Groups Assigned to Shared Cloud Drive](#get-user-groups-assigned-to-shared-cloud-drive)
12. [Get Shared Cloud Drives Assigned to User Group](#get-shared-cloud-drives-assigned-to-user-group)

## Users Project Permission

Apply, edit, or delete permissions of a user for a project.

- **URL**: `/api/v1/permission/users_project_permission`
- **Methods**: POST, PUT, DELETE
- **Auth Required**: Yes

### Request Body (POST, PUT)

```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (enum)"
  }
]
```

### Query Parameters (DELETE)

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the permission to delete |

### Response

A successful operation returns a `200 OK` status with no body.

## Get Users Assigned to Project

Retrieves the list of users assigned to a project.

- **URL**: `/api/v1/permission/get_users_assigned_to_project`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| projectId | string (uuid) | - | Project ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| username | string | - | Username to filter by |

### Response

Returns a collection of UserProjectPermissionDTO objects.

## Get Projects Assigned to User

Retrieves the list of projects assigned to a user.

- **URL**: `/api/v1/permission/get_projects_assigned_to_user`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userId | string (uuid) | - | User ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | Project name to filter by |

### Response

Returns a collection of UserProjectPermissionDTO objects.

## Users Shared Cloud Drive Permission

Apply, edit, or delete permissions of a user for a shared cloud drive.

- **URL**: `/api/v1/permission/users_sharedclouddrive_permision`
- **Methods**: POST, PUT, DELETE
- **Auth Required**: Yes

### Request Body (POST, PUT)

```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (enum)"
  }
]
```

### Query Parameters (DELETE)

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the permission to delete |

### Response

A successful operation returns a `200 OK` status with no body.

## Get Users Assigned to Shared Cloud Drive

Retrieves the list of users assigned to a shared cloud drive.

- **URL**: `/api/v1/permission/get_users_assigned_to_sharedclouddrive`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sharedCloudDriveId | string (uuid) | - | Shared cloud drive ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| username | string | - | Username to filter by |

### Response

Returns a collection of UserSharedCloudDrivePermissionDTO objects.

## Get Shared Cloud Drives Assigned to User

Retrieves the list of shared cloud drives assigned to a user.

- **URL**: `/api/v1/permission/get_sharedclouddrive_assigned_to_user`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userId | string (uuid) | - | User ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedClouDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | Shared cloud drive name to filter by |

### Response

Returns a collection of UserSharedCloudDrivePermissionDTO objects.

## User Groups Project Permission

Apply, edit, or delete permissions of a user group for a project.

- **URL**: `/api/v1/permission/usergroups_project_permission`
- **Methods**: POST, PUT, DELETE
- **Auth Required**: Yes

### Request Body (POST, PUT)

```json
[
  {
    "id": "string (uuid)",
    "userGroupId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (enum)"
  }
]
```

### Query Parameters (DELETE)

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the permission to delete |

### Response

A successful operation returns a `200 OK` status with no body.

## Get User Groups Assigned to Project

Retrieves the list of user groups assigned to a project.

- **URL**: `/api/v1/permission/get_usergroups_assigned_to_project`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| projectId | string (uuid) | - | Project ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | User group name to filter by |

### Response

Returns a collection of UserGroupProjectPermissionDTO objects.

## Get Projects Assigned to User Group

Retrieves the list of projects assigned to a user group.

- **URL**: `/api/v1/permission/get_projects_assigned_to_usergroup`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userGroupId | string (uuid) | - | User group ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | Project name to filter by |

### Response

Returns a collection of UserGroupProjectPermissionDTO objects.

## User Group Shared Cloud Drive Permission

Apply, edit, or delete permissions of a user group for a shared cloud drive.

- **URL**: `/api/v1/permission/usergroup_sharedclouddrive_permission`
- **Methods**: POST, PUT, DELETE
- **Auth Required**: Yes

### Request Body (POST, PUT)

```json
[
  {
    "id": "string (uuid)",
    "userGroupId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (enum)"
  }
]
```

### Query Parameters (DELETE)

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the permission to delete |

### Response

A successful operation returns a `200 OK` status with no body.

## Get User Groups Assigned to Shared Cloud Drive

Retrieves the list of user groups assigned to a shared cloud drive.

- **URL**: `/api/v1/permission/get_usergroups_assigned_to_sharedclouddrive`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sharedCloudDrive | string (uuid) | - | Shared cloud drive ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | User group name to filter by |

### Response

Returns a collection of UserGroupSharedCloudDrivePermissionDTO objects.

## Get Shared Cloud Drives Assigned to User Group

Retrieves the list of shared cloud drives assigned to a user group.

- **URL**: `/api/v1/permission/get_sharedclouddrive_assigned_to_usergroup`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userGroupId | string (uuid) | - | User group ID |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedCloudDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | Shared cloud drive name to filter by |

### Response

Returns a collection of UserGroupSharedCloudDrivePermissionDTO objects.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get Users Assigned to Project

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/permission/get_users_assigned_to_project"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
params = {
    "projectId": "PROJECT_ID_HERE",
    "page": 1,
    "pagesize": 10,
    "sortfield": "User.Username",
    "descending": False
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    assigned_users = response.json()
    for user in assigned_users['data']:
        print(f"User: {user['user']['username']}, Permission: {user['permission']['permissionType']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/permission/get_users_assigned_to_project?projectId=PROJECT_ID_HERE&page=1&pagesize=10&sortfield=User.Username&descending=false', {
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
  data.data.forEach(user => {
    console.log(`User: ${user.user.username}, Permission: ${user.permission.permissionType}`);
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

            var response = await client.GetAsync("api/v1/permission/get_users_assigned_to_project?projectId=PROJECT_ID_HERE&page=1&pagesize=10&sortfield=User.Username&descending=false");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var assignedUsers = JObject.Parse(content);
                foreach (var user in assignedUsers["data"])
                {
                    Console.WriteLine($"User: {user["user"]["username"]}, Permission: {user["permission"]["permissionType"]}");
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

