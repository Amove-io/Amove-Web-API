# Error Model

## Overview

The AMove API uses a non-standard HTTP status code for application-level validation errors: **499**. This cleanly separates application errors from infrastructure errors.

| Status | Meaning |
|---|---|
| 200 OK | Request succeeded. Response body contains the result (when any). |
| 401 Unauthorized | Authentication failed or is missing. From the Authentication API, this also indicates invalid credentials. |
| 499 | Application-level validation error. Response body contains a `ValidationProblemDetails` object with a specific error code. |
| 500 Internal Server Error | An unexpected server error occurred. No body is returned. |

Note: the API does not use the standard 400 / 403 / 404 codes — application-level problems are always surfaced through 499 regardless of their nature.

## HTTP 499 Response Body

Every 499 response includes a JSON body following ASP.NET Core's `ValidationProblemDetails`:

```json
{
  "type": "DUPLICATE",
  "status": 499,
  "errors": {
    "DUPLICATE": [
      "A resource with the same name already exists."
    ]
  }
}
```

- `type` — the application error code.
- `status` — always `499`.
- `errors` — dictionary with a single entry whose key equals `type` and whose value is a string array of messages.

The response also includes an `error-code` HTTP response header duplicating the value in `type`, which is convenient for clients that want to branch on the error without parsing the body.

## Error Codes

| Code | Meaning |
|---|---|
| DEFAULT | Generic validation error |
| AUTH | Authentication failure (bad credentials, user not found, etc.) |
| INACTIVE_USER | User account is not active |
| TOKEN | Token is invalid, expired, or already used |
| SESSION | Session is invalid or expired |
| SESSION_MAXOUT | Too many concurrent sessions for this user |
| SECURITY | Security policy rejected the request |
| ACCESS | Access denied to the requested resource |
| NOT_FOUND | Resource not found |
| COGNITO | Upstream identity provider returned an error |
| TOTP | MFA (time-based one-time password) challenge failed |
| TRANSFER | Transfer operation failed |
| TRANSFER_C2C | Cloud-to-cloud transfer dispatcher error |
| SYNC | Sync operation failed |
| COST | Billing or cost calculation error |
| DUPLICATE | A resource with the same unique field already exists |
| PENDING | Operation is pending and cannot be completed yet |
| DB_DELETE | Database delete failed |
| DB_INSERT | Database insert failed |
| DB_UPDATE | Database update failed |
| DOWNLOAD_OBJECT | Object download failed |
| INVALID_EMAIL | Email address is invalid |
| INTERNAL_ERROR | Internal server error surfaced as a validation error |
| CHECKSUM_MISMATCH | File checksum verification failed |
| FILESYSTEM_ACCESS | Local filesystem access denied |
| BUCKET_LOGGING_SET | Bucket logging could not be enabled |
| BUCKET_LOGGING_DISABLED | Bucket logging is disabled |
| DISPOSED | Underlying resource has been disposed |

## Handling errors in client code

<details>
<summary>Python</summary>

```python
import requests

response = requests.get(
    "https://api.amove.io/api/v1/user/userinfo",
    headers={"Authorization": "Bearer YOUR_JWT"},
)

if response.status_code == 200:
    print(response.json())
elif response.status_code == 499:
    body = response.json()
    code = body["type"]
    message = body["errors"][code][0]
    print(f"Application error {code}: {message}")
elif response.status_code == 401:
    print("Authentication required; refresh your JWT.")
elif response.status_code == 500:
    print("Server error; try again later.")
else:
    print(f"Unexpected HTTP {response.status_code}")
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const res = await fetch("https://api.amove.io/api/v1/user/userinfo", {
  headers: { "Authorization": "Bearer YOUR_JWT" }
});

if (res.status === 200) {
  console.log(await res.json());
} else if (res.status === 499) {
  const body = await res.json();
  const code = body.type;
  const message = body.errors[code][0];
  console.log(`Application error ${code}: ${message}`);
} else if (res.status === 401) {
  console.log("Authentication required.");
} else if (res.status === 500) {
  console.log("Server error; try again later.");
} else {
  console.log(`Unexpected HTTP ${res.status}`);
}
```

</details>

<details>
<summary>C#</summary>

```csharp
using System.Net.Http;
using System.Text.Json;

using var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", "YOUR_JWT");

var res = await client.GetAsync("https://api.amove.io/api/v1/user/userinfo");

if ((int)res.StatusCode == 499)
{
    var body = await res.Content.ReadFromJsonAsync<JsonElement>();
    string code = body.GetProperty("type").GetString();
    string message = body.GetProperty("errors").GetProperty(code)[0].GetString();
    Console.WriteLine($"Application error {code}: {message}");
}
else if (res.IsSuccessStatusCode)
{
    Console.WriteLine(await res.Content.ReadAsStringAsync());
}
else
{
    Console.WriteLine($"HTTP {(int)res.StatusCode}");
}
```

</details>
