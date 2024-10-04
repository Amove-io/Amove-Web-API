# Provisioning Endpoint

This document provides detailed information about the Provisioning-related endpoint in the AMove API. This endpoint is used for creating new users in the system.

## Endpoint

1. [Create User](#create-user)

## Create User

Creates a new user in the system.

- **URL**: `/api/v1/provision/create_user`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "username": "string"
}
```

### Response

Returns a string (likely a user identifier or token) upon successful user creation.

### Response Codes

- `200 OK`: User successfully created
- `400 Bad Request`: Invalid input
- `401 Unauthorized`: Authentication failed
- `403 Forbidden`: Not authorized to create users
- `500 Internal Server Error`: Server error

## Error Responses

The endpoint may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served. The exact error should be explained in the error payload.
- `401 Unauthorized`: The request requires user authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Create User

<details>
<summary>Python</summary>

```python
import requests
import json

url = "https://api.amove.com/api/v1/provision/create_user"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE",
    "Content-Type": "application/json"
}
data = {
    "username": "new_user@example.com"
}

response = requests.post(url, headers=headers, data=json.dumps(data))

if response.status_code == 200:
    user_identifier = response.text
    print(f"User created successfully. Identifier: {user_identifier}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const data = {
  username: 'new_user@example.com'
};

fetch('https://api.amove.com/api/v1/provision/create_user', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN_HERE',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
})
.then(response => {
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.text();
})
.then(userIdentifier => {
  console.log(`User created successfully. Identifier: ${userIdentifier}`);
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
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;

class Program
{
    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        {
            client.BaseAddress = new Uri("https://api.amove.com/");
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", "YOUR_TOKEN_HERE");

            var data = new { username = "new_user@example.com" };
            var content = new StringContent(JsonConvert.SerializeObject(data), Encoding.UTF8, "application/json");

            var response = await client.PostAsync("api/v1/provision/create_user", content);

            if (response.IsSuccessStatusCode)
            {
                var userIdentifier = await response.Content.ReadAsStringAsync();
                Console.WriteLine($"User created successfully. Identifier: {userIdentifier}");
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
                Console.WriteLine(await response.Content.ReadAsStringAsync());
            }
        }
    }
}
```

</details>

## Notes

- Ensure that you have the necessary permissions to create users before using this endpoint.
- The username should typically be in the form of an email address.
- Additional user details may be required depending on your system's configuration. Consult with your system administrator for any specific requirements.
- After creating a user, you may need to perform additional steps such as sending an invitation email or setting up initial permissions.

For more detailed examples and usage of other endpoints, please refer to our [Examples Directory](examples/README.md).

