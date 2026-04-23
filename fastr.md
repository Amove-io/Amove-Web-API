# Fastr Endpoints

This document provides detailed information about the Fastr-related endpoints in the Amove API. Fastr is Amove's high-performance file-serving product, built on QUIC, that customers install on their own machines and register here so other users in the organization can discover and use them.

## Endpoints

1. [Get All Fastr Servers](#get-all-fastr-servers)
2. [Insert Fastr Server](#insert-fastr-server)
3. [Manage Share](#manage-share)
4. [Delete Fastr Server](#delete-fastr-server)


## Get All Fastr Servers

Returns every Fastr server visible to the current user. The caller always sees the servers they own; `DesktopAdmin` callers additionally see servers owned by other users in the same account that are flagged as shared.

- **URL**: `/api/v1/fastr/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a `DTOCollection<FastrServer>` sorted by `Name`. Results are limited to active, non-deleted servers. The `username` and `password` fields are always obfuscated (`abc*****` / `**********`) before the response is returned.

```json
{
  "data": [
    {
      "id": "00000000-0000-0000-0000-000000000000",
      "accountId": "11111111-1111-1111-1111-111111111111",
      "userId": "22222222-2222-2222-2222-222222222222",
      "name": "Office Fastr",
      "serviceUrl": "fastr.example.com",
      "port": "29124",
      "username": "adm*****",
      "password": "**********",
      "active": true,
      "shared": true,
      "dedicated": true
    }
  ],
  "total": 1
}
```


## Insert Fastr Server

Registers a Fastr server under the current user's account. The server's `AccountId` and `UserId` are taken from the JWT, and `Dedicated` is forced to `true`.

- **URL**: `/api/v1/fastr/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "name": "string",
  "serviceUrl": "string",
  "port": "string",
  "username": "string",
  "password": "string",
  "shared": "boolean"
}
```

- `name` — display name of the server (max 100 characters).
- `serviceUrl` — hostname or IP of the Fastr server (max 100 characters).
- `port` — port number as a string (max 10 characters).
- `username` / `password` — optional credentials used when the Fastr server enforces HTTP basic auth.
- `shared` — when `true`, other `DesktopAdmin` users in the same account will see this server via `get_all`.

### Response

Returns the newly-created `FastrServer` object with its `username` and `password` fields obfuscated.


## Manage Share

Toggles the `shared` flag on an existing Fastr server. Only the user who originally registered the server may change its sharing state.

- **URL**: `/api/v1/fastr/manage_share`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "shared": "boolean"
}
```

### Response

Returns `200 OK` with no body when successful. If the caller is not the owner of the server, the endpoint returns `400 Bad Request` with the message `You are not authorized to update this fastr server`.


## Delete Fastr Server

Soft-deletes a Fastr server owned by the current user's account.

- **URL**: `/api/v1/fastr/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the Fastr server to delete |

### Response

Returns `true` when the delete succeeded. If no active server with that ID exists under the caller's account, the endpoint returns a `499` with `NOT_FOUND`.


## Sample Code

### Register a new Fastr server and list your servers

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
HEADERS = {"Authorization": "Bearer YOUR_JWT"}

# Register a new Fastr server
new_server = requests.post(
    f"{BASE}/api/v1/fastr/insert",
    headers=HEADERS,
    json={
        "name": "Office Fastr",
        "serviceUrl": "fastr.example.com",
        "port": "29124",
        "username": "admin",
        "password": "EXAMPLE_PASSWORD",
        "shared": True,
    },
).json()

# List all Fastr servers visible to the current user
servers = requests.get(f"{BASE}/api/v1/fastr/get_all", headers=HEADERS).json()
print(servers)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const BASE = "https://api.amove.io";
const headers = {
  "Authorization": "Bearer YOUR_JWT",
  "Content-Type": "application/json"
};

await fetch(`${BASE}/api/v1/fastr/insert`, {
  method: "POST",
  headers,
  body: JSON.stringify({
    name: "Office Fastr",
    serviceUrl: "fastr.example.com",
    port: "29124",
    username: "admin",
    password: "EXAMPLE_PASSWORD",
    shared: true
  })
});

const servers = await fetch(`${BASE}/api/v1/fastr/get_all`, { headers })
  .then(r => r.json());
console.log(servers);
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

await client.PostAsJsonAsync("api/v1/fastr/insert", new {
    name = "Office Fastr",
    serviceUrl = "fastr.example.com",
    port = "29124",
    username = "admin",
    password = "EXAMPLE_PASSWORD",
    shared = true
});

string servers = await client.GetStringAsync("api/v1/fastr/get_all");
Console.WriteLine(servers);
```

</details>

### Toggle sharing and delete a server

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
HEADERS = {"Authorization": "Bearer YOUR_JWT"}

server_id = "00000000-0000-0000-0000-000000000000"

# Share the server with other DesktopAdmins in the account
requests.put(
    f"{BASE}/api/v1/fastr/manage_share",
    headers=HEADERS,
    json={"id": server_id, "shared": True},
)

# Later, delete it
requests.delete(
    f"{BASE}/api/v1/fastr/delete",
    headers=HEADERS,
    params={"id": server_id},
)
```

</details>

For error handling, see [Error Model](errors.md).
