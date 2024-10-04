# UserGroup Endpoints

This document provides detailed information about the UserGroup-related endpoints in the AMove API. These endpoints allow you to manage user groups, assign users to groups, and manage group-related operations.

## Endpoints

1. [Get All User Groups](#get-all-user-groups)
2. [Insert User Group](#insert-user-group)
3. [Update User Group](#update-user-group)
4. [Delete User Group](#delete-user-group)
5. [Get Assigned Users](#get-assigned-users)
6. [Get Assigned User Groups](#get-assigned-user-groups)
7. [Assign Users](#assign-users)
8. [Unassign User](#unassign-user)
9. [Import Users](#import-users)

## Get All User Groups

Retrieves the list of user groups defined in the system.

- **URL**: `/api/v1/usergroup/get_all`
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
| name | string | - | UserGroup name to filter by |

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

## Insert User Group

Insert a new user group.

- **URL**: `/api/v1/usergroup/insert`
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

Returns the created UserGroup object.

## Update User Group

Update an existing user group.

- **URL**: `/api/v1/usergroup/update`
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

Returns the updated UserGroup object.

## Delete User Group

Delete an existing user group.

- **URL**: `/api/v1/usergroup/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of user group to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Get Assigned Users

Retrieves the list of users that are assigned to a user group.

- **URL**: `/api/v1/usergroup/get_assigned_users`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userGroupId | string (uuid) | - | UserGroup id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "User.Username" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| username | string | - | Username to filter by |

### Response

Returns a collection of UserUserGroupDTO objects.

## Get Assigned User Groups

Retrieves the list of user groups that are assigned to a user.

- **URL**: `/api/v1/usergroup/get_assigned_usergroups`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| userId | string (uuid) | - | User id |
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "UserGroup.Name" | Field to sort by |
| descending | boolean | false | Sort direction; descending: true |
| name | string | - | UserGroup name to filter by |

### Response

Returns a collection of UserUserGroupDTO objects.

## Assign Users

Assign users to a user group.

- **URL**: `/api/v1/usergroup/assign_users`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
[
  {
    "id": "string (uuid)",
    "userId": "string (uuid)",
    "userGroupId": "string (uuid)"
  }
]
```

### Response

A successful assignment returns a `200 OK` status with no body.

## Unassign User

Unassign a user from a user group.

- **URL**: `/api/v1/usergroup/unassign_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | Id of the assignment to delete |

### Response

A successful unassignment returns a `200 OK` status with no body.

## Import Users

Import users from a value.

- **URL**: `/api/v1/usergroup/import_users`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
[
  {
    "user": {
      // User object
    },
    "userGroups": [
      // UserGroup objects
    ]
  }
]
```

### Response

A successful import returns a `200 OK` status with no body.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get All User Groups

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/usergroup/get_all"
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
    user_groups = response.json()
    for group in user_groups['data']:
        print(f"User Group: {group['name']}, Active: {group['active']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/usergroup/get_all?page=1&pagesize=10&sortfield=Name&descending=false', {
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
  data.data.forEach(group => {
    console.log(`User Group: ${group.name}, Active: ${group.active}`);
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

            var response = await client.GetAsync("api/v1/usergroup/get_all?page=1&pagesize=10&sortfield=Name&descending=false");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var userGroups = JObject.Parse(content);
                foreach (var group in userGroups["data"])
                {
                    Console.WriteLine($"User Group: {group["name"]}, Active: {group["active"]}");
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

