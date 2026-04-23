# Project Endpoints

This document provides detailed information about the Project-related endpoints in the Amove API. Projects are account-scoped groupings of shared cloud drives, users, and user groups.

## Endpoints

1. [Get All Projects](#get-all-projects)
2. [Get All Projects With Details](#get-all-projects-with-details)
3. [Insert Project](#insert-project)
4. [Update Project](#update-project)
5. [Delete Project](#delete-project)
6. [Get Assigned Shared Cloud Drives](#get-assigned-shared-cloud-drives)
7. [Get Assigned Projects](#get-assigned-projects)
8. [Assign Shared Cloud Drive](#assign-shared-cloud-drive)
9. [Unassign Shared Cloud Drive](#unassign-shared-cloud-drive)


## Get All Projects

Returns a paginated list of projects in the current user's account. The caller can optionally include soft-deleted records and filter by a case-insensitive substring match on the project name.

- **URL**: `/api/v1/project/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| deleted | boolean | false | When `true`, return soft-deleted projects instead of active ones |
| name | string | null | Optional case-insensitive substring filter on project name |

### Response

Returns a `DTOCollection<Project>`. Each project has:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "accountId": "00000000-0000-0000-0000-000000000000",
  "name": "Marketing",
  "description": "Marketing team shared assets",
  "active": true,
  "deleted": false
}
```


## Get All Projects With Details

Same filtering and paging as [Get All Projects](#get-all-projects), but each project is returned with its assigned users, user groups, and shared cloud drives inlined in a single response — avoiding the per-project follow-up calls otherwise needed to resolve these relationships.

- **URL**: `/api/v1/project/get_all_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| deleted | boolean | false | When `true`, return soft-deleted projects instead of active ones |
| name | string | null | Optional case-insensitive substring filter on project name |

### Response

Returns a `DTOCollection<ProjectWithDetailsDTO>`. Each item carries the base project fields plus three collections:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "Marketing",
  "description": "Marketing team shared assets",
  "usersData": [
    { "user": { }, "permission": { } }
  ],
  "groupsData": [
    { "userGroup": { }, "permission": { } }
  ],
  "drivesData": [
    { "sharedCloudDrive": { }, "projectSharedCloudDrive": { } }
  ]
}
```


## Insert Project

Creates a new project under the current user's account. The `accountId` on the request body is ignored and overwritten with the caller's account id.

- **URL**: `/api/v1/project/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "name": "string (max 100)",
  "description": "string",
  "active": "boolean"
}
```

### Response

Returns the newly-created `Project` object, including its server-generated `id`.


## Update Project

Updates an existing project. The project's `accountId` is always reset to the caller's account id on the server.

- **URL**: `/api/v1/project/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "active": true
}
```

### Response

Returns the updated `Project`.


## Delete Project

Soft-deletes a project. Because `Project` implements `ILogicalDeleteableEntity`, the record is marked `Deleted = true` rather than physically removed.

- **URL**: `/api/v1/project/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the project to delete |

### Response

A `200 OK` status with no body on success.


## Get Assigned Shared Cloud Drives

Lists the shared cloud drives currently attached to a given project.

- **URL**: `/api/v1/project/get_assigned_sharedclouddrives`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| projectId | string (uuid) | — | The project whose attached drives to list |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "SharedCloudDrive.Name" | Field to sort by |
| descending | boolean | false | Sort direction |

### Response

Returns a `DTOCollection<SharedCloudDriveProjectDTO>`. Each item contains the full `SharedCloudDrive` and the joining `ProjectSharedCloudDrive` record.


## Get Assigned Projects

Lists the projects currently attached to a given shared cloud drive.

- **URL**: `/api/v1/project/get_assigned_projects`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| sharedCloudDriveId | string (uuid) | — | The shared drive whose attached projects to list |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Project.Name" | Field to sort by |
| descending | boolean | false | Sort direction |

### Response

Returns a `DTOCollection<SharedCloudDriveProjectDTO>`, each item containing the full `Project` and the joining `ProjectSharedCloudDrive` record.


## Assign Shared Cloud Drive

Attaches one or more shared cloud drives to projects. The server rejects duplicate `(ProjectId, SharedCloudDriveId)` pairs silently — any pair that already exists is skipped.

- **URL**: `/api/v1/project/assign_sharedclouddrive`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

An array of `ProjectSharedCloudDrive` entries:

```json
[
  {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "sharedCloudDriveId": "00000000-0000-0000-0000-000000000000"
  }
]
```

### Response

A `200 OK` status with no body on success.


## Unassign Shared Cloud Drive

Removes an attachment between a project and a shared cloud drive.

- **URL**: `/api/v1/project/unassign_sharedclouddrive`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the `ProjectSharedCloudDrive` record to delete |

### Response

A `200 OK` status with no body on success.


## Sample Code

### Create a project and list it

<details>
<summary>Python</summary>

```python
import requests

JWT = "YOUR_JWT"
BASE = "https://api.amove.io"

created = requests.post(
    f"{BASE}/api/v1/project/insert",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"name": "Marketing", "description": "Marketing team assets", "active": True},
).json()

projects = requests.get(
    f"{BASE}/api/v1/project/get_all",
    headers={"Authorization": f"Bearer {JWT}"},
    params={"page": 1, "pagesize": 50},
).json()
print(projects["data"])
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const JWT = "YOUR_JWT";
const BASE = "https://api.amove.io";

await fetch(`${BASE}/api/v1/project/insert`, {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${JWT}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({ name: "Marketing", description: "Marketing team assets", active: true })
});

const list = await fetch(`${BASE}/api/v1/project/get_all?page=1&pagesize=50`, {
  headers: { "Authorization": `Bearer ${JWT}` }
}).then(r => r.json());
console.log(list.data);
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

await client.PostAsJsonAsync("api/v1/project/insert", new
{
    name = "Marketing",
    description = "Marketing team assets",
    active = true
});

var res = await client.GetAsync("api/v1/project/get_all?page=1&pagesize=50");
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

</details>

### Attach a shared cloud drive to a project

<details>
<summary>Python</summary>

```python
import requests

requests.post(
    "https://api.amove.io/api/v1/project/assign_sharedclouddrive",
    headers={"Authorization": "Bearer YOUR_JWT"},
    json=[{
        "projectId": "00000000-0000-0000-0000-000000000000",
        "sharedCloudDriveId": "00000000-0000-0000-0000-000000000000"
    }],
)
```

</details>


For error handling, see [Error Model](errors.md).
