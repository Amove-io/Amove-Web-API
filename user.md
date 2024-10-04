# User Endpoints

This document provides detailed information about the User-related endpoints in the AMove API. These endpoints allow you to manage user accounts, authentication, and user-related operations.

## Endpoints

1. [Sign Up](#sign-up)
2. [Get Sign Up Info](#get-sign-up-info)
3. [Get User Info](#get-user-info)
4. [Update User](#update-user)
5. [Update Subscription](#update-subscription)
6. [Reset Password](#reset-password)
7. [Set MFA](#set-mfa)
8. [Generate MFA Token](#generate-mfa-token)
9. [Get All Users](#get-all-users)
10. [Insert User](#insert-user)
11. [Edit User](#edit-user)
12. [Delete User](#delete-user)
13. [Resend User Email](#resend-user-email)

## Sign Up

Requests a user sign-up.

- **URL**: `/api/v1/user/signup`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "firstname": "string",
  "lastname": "string",
  "password": "string",
  "token": "string"
}
```

### Response

Returns the created User object.

## Get Sign Up Info

Gets a user sign-up information. The user entity must have a non-expired sign-up token.

- **URL**: `/api/v1/user/signupinfo`
- **Method**: GET
- **Auth Required**: No

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| token | string | User's sign-up token previously sent by email |

### Response

Returns a UserStatus enum value.

## Get User Info

Requests for currently logged-in user information.

- **URL**: `/api/v1/user/userinfo`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns a UserInfo object containing details about the current user.

## Update User

Update user's information.

- **URL**: `/api/v1/user/update_user`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "accountName": "string",
  "firstName": "string",
  "lastName": "string"
}
```

### Response

Returns the updated UserInfo object.

## Update Subscription

Updates the user's subscription.

- **URL**: `/api/v1/user/update_subscription`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "subscriptionType": "integer (enum)"
}
```

### Response

A successful update returns a `200 OK` status with no body.

## Reset Password

Requests a reset password procedure.

- **URL**: `/api/v1/user/reset_password`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "password": "string",
  "currentPassword": "string"
}
```

### Response

A successful password reset returns a `200 OK` status with no body.

## Set MFA

Enables/Disables the user's multi-factor authentication (MFA) preference.

- **URL**: `/api/v1/user/set_mfa`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "enabled": "boolean",
  "userCode": "string"
}
```

### Response

A successful MFA preference update returns a `200 OK` status with no body.

## Generate MFA Token

Returns a unique generated shared secret key code for the user account.

- **URL**: `/api/v1/user/generate_mfa_token`
- **Method**: POST
- **Auth Required**: Yes

### Response

Returns a string containing the generated MFA token.

## Get All Users

Retrieves the list of users defined in the system.

- **URL**: `/api/v1/user/get_all_users`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page |
| pagesize | integer | 50 | Page size |
| sortfield | string | "CreateDate" | Field to sort by |
| descending | boolean | true | Sort direction; descending: true |
| deleted | boolean | false | When true, includes deleted records in the result |
| userStatus | integer (enum) | - | When provided, includes only records with the specified status |
| userType | integer (enum) | - | When provided, includes only records with the specified type |
| username | string | - | When provided, looks for a specified username |

### Response

Returns a collection of User objects.

## Insert User

Creates a temp user in the system, with a temporary token.

- **URL**: `/api/v1/user/insert_user`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

User object

### Response

Returns the created User object.

## Edit User

Update existing user in the system.

- **URL**: `/api/v1/user/edit_user`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

User object

### Response

Returns the updated User object.

## Delete User

Delete existing user in the system.

- **URL**: `/api/v1/user/delete_user`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | ID of the user to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Resend User Email

Re-sends the sign-up email to the specified user.

- **URL**: `/api/v1/user/resend_user_email`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

User object

### Response

A successful email resend returns a `200 OK` status with no body.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get User Info

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/user/userinfo"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}

response = requests.get(url, headers=headers)

if response.status_code == 200:
    user_info = response.json()
    print(f"Username: {user_info['username']}")
    print(f"Email: {user_info['email']}")
    print(f"User Type: {user_info['userType']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/user/userinfo', {
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
.then(userInfo => {
  console.log(`Username: ${userInfo.username}`);
  console.log(`Email: ${userInfo.email}`);
  console.log(`User Type: ${userInfo.userType}`);
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

            var response = await client.GetAsync("api/v1/user/userinfo");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var userInfo = JObject.Parse(content);
                Console.WriteLine($"Username: {userInfo["username"]}");
                Console.WriteLine($"Email: {userInfo["email"]}");
                Console.WriteLine($"User Type: {userInfo["userType"]}");
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

