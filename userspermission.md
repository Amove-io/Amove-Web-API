# Users Permission Endpoints

This document provides detailed information about the permission assignment endpoints in the Amove API. Permissions attach a user or user group to a project or shared cloud drive and specify the level of access (`Read` or `ReadWrite`).

The 20 endpoints are organized in four CRUD + query groups:

- **User → Project** — grant a single user access to a project
- **User → Shared Cloud Drive** — grant a single user access to a shared cloud drive
- **User Group → Project** — grant an entire group access to a project
- **User Group → Shared Cloud Drive** — grant an entire group access to a shared cloud drive

All endpoints in this document require `Authorization: Bearer <JWT>`.

## Permission Types

The `permissionType` field uses the `ResourcePermissionType` enum:

| Value | Name | Description |
|---|---|---|
| 1 | Read | Read-only access |
| 2 | ReadWrite | Read and write access |

## Endpoints

### User → Project

1. [Add User Project Permission](#add-user-project-permission)
2. [Edit User Project Permission](#edit-user-project-permission)
3. [Delete User Project Permission](#delete-user-project-permission)
4. [Get Users Assigned To Project](#get-users-assigned-to-project)
5. [Get Projects Assigned To User](#get-projects-assigned-to-user)

### User → Shared Cloud Drive

6. [Add User Shared Cloud Drive Permission](#add-user-shared-cloud-drive-permission)
7. [Edit User Shared Cloud Drive Permission](#edit-user-shared-cloud-drive-permission)
8. [Delete User Shared Cloud Drive Permission](#delete-user-shared-cloud-drive-permission)
9. [Get Users Assigned To Shared Cloud Drive](#get-users-assigned-to-shared-cloud-drive)
10. [Get Shared Cloud Drive Assigned To User](#get-shared-cloud-drive-assigned-to-user)

### User Group → Project

11. [Add User Group Project Permission](#add-user-group-project-permission)
12. [Edit User Group Project Permission](#edit-user-group-project-permission)
13. [Delete User Group Project Permission](#delete-user-group-project-permission)
14. [Get User Groups Assigned To Project](#get-user-groups-assigned-to-project)
15. [Get Projects Assigned To User Group](#get-projects-assigned-to-user-group)

### User Group → Shared Cloud Drive

16. [Add User Group Shared Cloud Drive Permission](#add-user-group-shared-cloud-drive-permission)
17. [Edit User Group Shared Cloud Drive Permission](#edit-user-group-shared-cloud-drive-permission)
18. [Delete User Group Shared Cloud Drive Permission](#delete-user-group-shared-cloud-drive-permission)
19. [Get User Groups Assigned To Shared Cloud Drive](#get-user-groups-assigned-to-shared-cloud-drive)
20. [Get Shared Cloud Drive Assigned To User Group](#get-shared-cloud-drive-assigned-to-user-group)


---

## User → Project

### Add User Project Permission

Grants one or more users access to a project. The request body accepts an array so multiple assignments can be created in a single call.

- **URL**: `/api/v1/permission/users_project_permission`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "userId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Edit User Project Permission

Updates one or more existing user-to-project permission records (for example, to change `Read` to `ReadWrite`).

- **URL**: `/api/v1/permission/users_project_permission`
- **Method**: PUT
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Delete User Project Permission

Removes a single user-to-project permission record by its id.

- **URL**: `/api/v1/permission/users_project_permission`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission record id. |

#### Response

`200 OK` with an empty body.


### Get Users Assigned To Project

Returns the users granted access to the given project, paginated.

- **URL**: `/api/v1/permission/get_users_assigned_to_project`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| projectId | string (uuid) | — | Target project id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction |
| username | string | null | Case-insensitive substring match on username |

#### Response

Returns a `DTOCollection<UserProjectPermissionDTO>` where each row has the shape:

```json
{
  "user": { },
  "project": { },
  "permission": {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
}
```


### Get Projects Assigned To User

Returns the projects the given user has access to, paginated.

- **URL**: `/api/v1/permission/get_projects_assigned_to_user`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userId | string (uuid) | — | Target user id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Case-insensitive substring match on project name |

#### Response

Returns a `DTOCollection<UserProjectPermissionDTO>`.


---

## User → Shared Cloud Drive

Note: the route spelling `users_sharedclouddrive_permision` has a single "s" in "permision" to match the existing server route.

### Add User Shared Cloud Drive Permission

Grants one or more users access to a shared cloud drive.

- **URL**: `/api/v1/permission/users_sharedclouddrive_permision`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "userId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Edit User Shared Cloud Drive Permission

Updates one or more existing user-to-shared-cloud-drive permission records.

- **URL**: `/api/v1/permission/users_sharedclouddrive_permision`
- **Method**: PUT
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Delete User Shared Cloud Drive Permission

Removes a single user-to-shared-cloud-drive permission record by its id.

- **URL**: `/api/v1/permission/users_sharedclouddrive_permision`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission record id. |

#### Response

`200 OK` with an empty body.


### Get Users Assigned To Shared Cloud Drive

Returns the users granted access to the given shared cloud drive, paginated.

- **URL**: `/api/v1/permission/get_users_assigned_to_sharedclouddrive`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sharedCloudDriveId | string (uuid) | — | Target shared cloud drive id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction |
| username | string | null | Case-insensitive substring match on username |

#### Response

Returns a `DTOCollection<UserSharedCloudDrivePermissionDTO>` where each row has the shape:

```json
{
  "user": { },
  "sharedClouDrive": { },
  "permission": {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
}
```

### Get Shared Cloud Drive Assigned To User

Returns the shared cloud drives the given user has access to, paginated.

- **URL**: `/api/v1/permission/get_sharedclouddrive_assigned_to_user`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userId | string (uuid) | — | Target user id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedClouDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Case-insensitive substring match on shared cloud drive name |

#### Response

Returns a `DTOCollection<UserSharedCloudDrivePermissionDTO>`.


---

## User Group → Project

### Add User Group Project Permission

Grants one or more user groups access to a project. Every member of a granted group inherits the permission.

- **URL**: `/api/v1/permission/usergroups_project_permission`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "userGroupId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Edit User Group Project Permission

Updates one or more existing user-group-to-project permission records.

- **URL**: `/api/v1/permission/usergroups_project_permission`
- **Method**: PUT
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "id": "string (uuid)",
    "userGroupId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Delete User Group Project Permission

Removes a single user-group-to-project permission record by its id.

- **URL**: `/api/v1/permission/usergroups_project_permission`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission record id. |

#### Response

`200 OK` with an empty body.


### Get User Groups Assigned To Project

Returns the user groups granted access to the given project, paginated.

- **URL**: `/api/v1/permission/get_usergroups_assigned_to_project`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| projectId | string (uuid) | — | Target project id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Case-insensitive substring match on user group name |

#### Response

Returns a `DTOCollection<UserGroupProjectPermissionDTO>` where each row has the shape:

```json
{
  "userGroup": { },
  "project": { },
  "permission": {
    "id": "string (uuid)",
    "userGroupId": "string (uuid)",
    "projectId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
}
```


### Get Projects Assigned To User Group

Returns the projects the given user group has access to, paginated.

- **URL**: `/api/v1/permission/get_projects_assigned_to_usergroup`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userGroupId | string (uuid) | — | Target user group id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Case-insensitive substring match on project name |

#### Response

Returns a `DTOCollection<UserGroupProjectPermissionDTO>`.


---

## User Group → Shared Cloud Drive

### Add User Group Shared Cloud Drive Permission

Grants one or more user groups access to a shared cloud drive.

- **URL**: `/api/v1/permission/usergroup_sharedclouddrive_permission`
- **Method**: POST
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "userGroupId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Edit User Group Shared Cloud Drive Permission

Updates one or more existing user-group-to-shared-cloud-drive permission records.

- **URL**: `/api/v1/permission/usergroup_sharedclouddrive_permission`
- **Method**: PUT
- **Auth Required**: Yes

#### Request Body

```json
[
  {
    "id": "string (uuid)",
    "userGroupId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
]
```

#### Response

`200 OK` with an empty body.


### Delete User Group Shared Cloud Drive Permission

Removes a single user-group-to-shared-cloud-drive permission record by its id.

- **URL**: `/api/v1/permission/usergroup_sharedclouddrive_permission`
- **Method**: DELETE
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Permission record id. |

#### Response

`200 OK` with an empty body.


### Get User Groups Assigned To Shared Cloud Drive

Returns the user groups granted access to the given shared cloud drive, paginated.

- **URL**: `/api/v1/permission/get_usergroups_assigned_to_sharedclouddrive`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sharedCloudDrive | string (uuid) | — | Target shared cloud drive id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Case-insensitive substring match on user group name |

#### Response

Returns a `DTOCollection<UserGroupSharedCloudDrivePermissionDTO>` where each row has the shape:

```json
{
  "userGroup": { },
  "sharedCloudDrive": { },
  "permission": {
    "id": "string (uuid)",
    "userGroupId": "string (uuid)",
    "sharedCloudDriveId": "string (uuid)",
    "permissionType": "integer (ResourcePermissionType)"
  }
}
```


### Get Shared Cloud Drive Assigned To User Group

Returns the shared cloud drives the given user group has access to, paginated.

- **URL**: `/api/v1/permission/get_sharedclouddrive_assigned_to_usergroup`
- **Method**: GET
- **Auth Required**: Yes

#### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userGroupId | string (uuid) | — | Target user group id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedCloudDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Case-insensitive substring match on shared cloud drive name |

#### Response

Returns a `DTOCollection<UserGroupSharedCloudDrivePermissionDTO>`.


## Sample Code

### Grant a user read-write access to a project

<details>
<summary>Python</summary>

```python
import requests

response = requests.post(
    "https://api.amove.io/api/v1/permission/users_project_permission",
    headers={"Authorization": "Bearer EXAMPLE_TOKEN"},
    json=[{
        "userId": "00000000-0000-0000-0000-000000000000",
        "projectId": "11111111-1111-1111-1111-111111111111",
        "permissionType": 2
    }]
)
print(response.status_code)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/permission/users_project_permission", {
  method: "POST",
  headers: {
    "Authorization": "Bearer EXAMPLE_TOKEN",
    "Content-Type": "application/json"
  },
  body: JSON.stringify([{
    userId: "00000000-0000-0000-0000-000000000000",
    projectId: "11111111-1111-1111-1111-111111111111",
    permissionType: 2
  }])
});
console.log(res.status);
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http.Json;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "EXAMPLE_TOKEN");

object[] body = new[]
{
    new
    {
        userId = "00000000-0000-0000-0000-000000000000",
        projectId = "11111111-1111-1111-1111-111111111111",
        permissionType = 2
    }
};

HttpResponseMessage res = await client.PostAsJsonAsync(
    "https://api.amove.io/api/v1/permission/users_project_permission",
    body);
Console.WriteLine((int)res.StatusCode);
```

</details>

### List the users assigned to a project

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "https://api.amove.io/api/v1/permission/get_users_assigned_to_project",
    headers={"Authorization": "Bearer EXAMPLE_TOKEN"},
    params={
        "projectId": "11111111-1111-1111-1111-111111111111",
        "page": 1,
        "pagesize": 50
    }
)
print(response.json())
```

</details>

### Grant a user group access to a shared cloud drive

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/permission/usergroup_sharedclouddrive_permission", {
  method: "POST",
  headers: {
    "Authorization": "Bearer EXAMPLE_TOKEN",
    "Content-Type": "application/json"
  },
  body: JSON.stringify([{
    userGroupId: "22222222-2222-2222-2222-222222222222",
    sharedCloudDriveId: "33333333-3333-3333-3333-333333333333",
    permissionType: 1
  }])
});
console.log(res.status);
```

</details>


For error handling, see [Error Model](errors.md).
