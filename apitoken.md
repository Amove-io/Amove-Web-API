# API Token Endpoints

This document provides detailed information about the API-token endpoints in the AMove API. API tokens are long-lived JWT tokens a user can mint against their own account for programmatic access without going through the interactive login flow.

## Endpoints

1. [Get All API Tokens](#get-all-api-tokens)
2. [Insert API Token](#insert-api-token)
3. [Delete API Token](#delete-api-token)


## Get All API Tokens

Returns every API token that belongs to the current user.

- **URL**: `/api/v1/apitoken/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction |

### Response

Returns an array of `ApiToken` objects owned by the current user.

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "userId": "00000000-0000-0000-0000-000000000000",
    "sessionId": "00000000-0000-0000-0000-000000000000",
    "title": "ci-pipeline",
    "token": "eyJhbGciOi...",
    "encryptionKey": "",
    "isEncrypted": false,
    "expirationDate": "2027-01-01T00:00:00Z",
    "createDate": "2026-01-01T00:00:00Z"
  }
]
```


## Insert API Token

Creates a new API token for the current user. The server mints a JWT bound to the user's identity and tied to a new authenticator session whose timeout is derived from `expirationDate`. The returned `token` is the bearer value to use on subsequent calls to the Main API.

- **URL**: `/api/v1/apitoken/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "title": "string",
  "isEncrypted": "boolean",
  "encryptionKey": "string",
  "expirationDate": "string (date-time)"
}
```

- `title` — caller-supplied label for the token.
- `isEncrypted` — when `true`, the issued JWT payload is encrypted; `encryptionKey` must also be set.
- `expirationDate` — UTC date and time at which the token expires. The server derives the authenticator session and JWT timeouts from the interval between "now" and this value.

### Response

Returns the created `ApiToken` object, including the newly-issued `token` (JWT string) and server-generated `id`, `sessionId`, and `createDate`.


## Delete API Token

Revokes an API token. The underlying authenticator session is killed and the token record is removed, so any client still holding the token receives `401 Unauthorized` on the next call.

- **URL**: `/api/v1/apitoken/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The `id` of the `ApiToken` to delete. Must belong to the current user; tokens owned by other users are rejected. |

### Response

A `200 OK` status with no body on success.


## Sample Code

### Create and use an API token

<details>
<summary>Python</summary>

```python
import requests

JWT = "YOUR_JWT"
BASE = "https://api.amove.io"

# Create a long-lived API token (1 year)
from datetime import datetime, timedelta, timezone
expiration = (datetime.now(timezone.utc) + timedelta(days=365)).isoformat()

created = requests.post(
    f"{BASE}/api/v1/apitoken/insert",
    headers={"Authorization": f"Bearer {JWT}"},
    json={
        "title": "ci-pipeline",
        "isEncrypted": False,
        "expirationDate": expiration,
    },
).json()

api_token = created["token"]

# Use the API token as a bearer on subsequent calls
profile = requests.get(
    f"{BASE}/api/v1/user/userinfo",
    headers={"Authorization": f"Bearer {api_token}"},
).json()
print(profile)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const JWT = "YOUR_JWT";
const BASE = "https://api.amove.io";

const expiration = new Date(Date.now() + 365 * 24 * 60 * 60 * 1000).toISOString();

const created = await fetch(`${BASE}/api/v1/apitoken/insert`, {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${JWT}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    title: "ci-pipeline",
    isEncrypted: false,
    expirationDate: expiration
  })
}).then(r => r.json());

const apiToken = created.token;

const profile = await fetch(`${BASE}/api/v1/user/userinfo`, {
  headers: { "Authorization": `Bearer ${apiToken}` }
}).then(r => r.json());
console.log(profile);
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Text.Json;

using var client = new HttpClient { BaseAddress = new Uri("https://api.amove.io/") };
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var body = new
{
    title = "ci-pipeline",
    isEncrypted = false,
    expirationDate = DateTime.UtcNow.AddDays(365)
};

var created = await (await client.PostAsJsonAsync("api/v1/apitoken/insert", body))
    .Content.ReadFromJsonAsync<JsonElement>();
string apiToken = created.GetProperty("token").GetString();

Console.WriteLine(apiToken);
```

</details>

### Delete an API token

<details>
<summary>Python</summary>

```python
import requests

requests.delete(
    "https://api.amove.io/api/v1/apitoken/delete",
    params={"id": "00000000-0000-0000-0000-000000000000"},
    headers={"Authorization": "Bearer YOUR_JWT"},
)
```

</details>


For error handling, see [Error Model](errors.md).
