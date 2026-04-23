# User Group Endpoints

This document provides detailed information about the user-group endpoints in the AMove API. A user group is an account-scoped collection of users that can be granted permissions on projects and shared cloud drives collectively.

## Endpoints

1. [Get All User Groups](#get-all-user-groups)
2. [Get All User Groups With Details](#get-all-user-groups-with-details)
3. [Insert User Group](#insert-user-group)
4. [Update User Group](#update-user-group)
5. [Delete User Group](#delete-user-group)
6. [Get Assigned Users](#get-assigned-users)
7. [Get Assigned User Groups](#get-assigned-user-groups)
8. [Assign Users](#assign-users)
9. [Unassign User](#unassign-user)
10. [Import Users](#import-users)


## Get All User Groups

Returns a paginated list of user groups in the current user's account.

- **URL**: `/api/v1/usergroup/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| deleted | boolean | false | When `true`, return soft-deleted groups instead of active ones |
| name | string | null | Optional case-insensitive substring filter on group name |

### Response

Returns a `DTOCollection<UserGroup>`:

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "accountId": "00000000-0000-0000-0000-000000000000",
      "name": "Engineering",
      "description": "Engineering department",
      "active": true,
      "deleted": false
    }
  ],
  "total": 1
}
```


## Get All User Groups With Details

Same filters and paging as [Get All User Groups](#get-all-user-groups), but each group is returned with its assigned projects, member users, and shared cloud drives inline.

- **URL**: `/api/v1/usergroup/get_all_with_details`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| deleted | boolean | false | When `true`, return soft-deleted groups |
| name | string | null | Optional case-insensitive substring filter on group name |

### Response

Returns a `DTOCollection<UserGroupWithDetailsDTO>`. Each item:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "Engineering",
  "description": "Engineering department",
  "projectsData": [
    { "project": { }, "permission": { } }
  ],
  "usersData": [
    { "user": { }, "userUserGroup": { } }
  ],
  "drivesData": [
    { "sharedCloudDrive": { }, "permission": { } }
  ]
}
```


## Insert User Group

Creates a new user group under the current user's account. The server overrides `accountId` with the caller's account id.

- **URL**: `/api/v1/usergroup/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "name": "string (max 100)",
  "description": "string",
  "active": true
}
```

### Response

Returns the newly-created `UserGroup` object.


## Update User Group

Updates an existing user group. `accountId` is always reset to the caller's account.

- **URL**: `/api/v1/usergroup/update`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

A full `UserGroup` object including the `id` of the record to update.

### Response

Returns the updated `UserGroup`.


## Delete User Group

Soft-deletes a user group. Because `UserGroup` implements `ILogicalDeleteableEntity`, the record is marked `Deleted = true` rather than physically removed.

- **URL**: `/api/v1/usergroup/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the user group to delete |

### Response

A `200 OK` status with no body on success.


## Get Assigned Users

Lists the users in a given user group. Soft-deleted users are excluded.

- **URL**: `/api/v1/usergroup/get_assigned_users`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userGroupId | string (uuid) | — | User group whose members to list |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction |
| username | string | null | Optional case-insensitive substring filter on username |

### Response

Returns a `DTOCollection<UserUserGroupDTO>`. Each item contains the full `User` and the joining `UserUserGroup` record.


## Get Assigned User Groups

Lists the user groups a given user belongs to. Soft-deleted groups are excluded.

- **URL**: `/api/v1/usergroup/get_assigned_usergroups`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userId | string (uuid) | — | User whose group memberships to list |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction |
| name | string | null | Optional case-insensitive substring filter on group name |

### Response

Returns a `DTOCollection<UserUserGroupDTO>`. Each item contains the full `UserGroup` and the joining `UserUserGroup` record.


## Assign Users

Adds one or more users to user groups. Existing `(UserId, UserGroupId)` pairs are skipped silently.

- **URL**: `/api/v1/usergroup/assign_users`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

An array of `UserUserGroup` entries:

```json
[
  {
    "userId": "00000000-0000-0000-0000-000000000000",
    "userGroupId": "00000000-0000-0000-0000-000000000000"
  }
]
```

### Response

A `200 OK` status with no body on success.


## Unassign User

Removes a single user-to-group membership.

- **URL**: `/api/v1/usergroup/unassign_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the `UserUserGroup` record to delete |

### Response

A `200 OK` status with no body on success.


## Import Users

Bulk-imports users into user groups. For each entry: if the user already exists in the caller's account it is reused; if the username exists in a different account the entry is skipped; otherwise a new user is created and the caller's billing subscription is extended to include the new seat (subscription status must be `Active`). Each named user group is created if absent, and the user is assigned to it. Group and user creation are additive; existing assignments are left intact.

- **URL**: `/api/v1/usergroup/import_users`
- **Method**: POST
- **Auth Required**: Yes

If the current user has no active billing subscription, the endpoint returns HTTP `402 Payment Required` before doing any work.

### Request Body

An array of `UserUserGroupRelation`:

```json
[
  {
    "user": {
      "username": "user@example.com",
      "email": "user@example.com",
      "userType": 64
    },
    "userGroups": [
      { "name": "Engineering" }
    ]
  }
]
```

- `user.userType` — `UserType` flag: `16` DesktopAdmin, `32` DesktopCreativeUser, `64` DesktopStandardUser.
- `userGroups` — list of groups to join (matched case-insensitively by `name`; created when missing).

### Response

A `200 OK` status with no body on success.


## Sample Code

### Create a group and import users into it

<details>
<summary>Python</summary>

```python
import requests

JWT = "YOUR_JWT"
BASE = "https://api.amove.io"

group = requests.post(
    f"{BASE}/api/v1/usergroup/insert",
    headers={"Authorization": f"Bearer {JWT}"},
    json={"name": "Engineering", "description": "Engineering department", "active": True},
).json()

requests.post(
    f"{BASE}/api/v1/usergroup/import_users",
    headers={"Authorization": f"Bearer {JWT}"},
    json=[{
        "user": {
            "username": "user@example.com",
            "email": "user@example.com",
            "userType": 64
        },
        "userGroups": [{"name": "Engineering"}]
    }],
)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const JWT = "YOUR_JWT";
const BASE = "https://api.amove.io";

const group = await fetch(`${BASE}/api/v1/usergroup/insert`, {
  method: "POST",
  headers: { "Authorization": `Bearer ${JWT}`, "Content-Type": "application/json" },
  body: JSON.stringify({ name: "Engineering", description: "Engineering department", active: true })
}).then(r => r.json());

await fetch(`${BASE}/api/v1/usergroup/import_users`, {
  method: "POST",
  headers: { "Authorization": `Bearer ${JWT}`, "Content-Type": "application/json" },
  body: JSON.stringify([{
    user: { username: "user@example.com", email: "user@example.com", userType: 64 },
    userGroups: [{ name: "Engineering" }]
  }])
});
```

</details>

### Assign existing users to an existing group

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Net.Http.Json;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var payload = new[]
{
    new
    {
        userId = Guid.Parse("00000000-0000-0000-0000-000000000000"),
        userGroupId = Guid.Parse("00000000-0000-0000-0000-000000000000")
    }
};

await client.PostAsJsonAsync("https://api.amove.io/api/v1/usergroup/assign_users", payload);
```

</details>


For error handling, see [Error Model](errors.md).
