# ApiToken Endpoints

This document provides detailed information about the ApiToken-related endpoints in the AMove API. These endpoints allow you to manage API tokens for authentication and authorization purposes.

## Endpoints

1. [Get All API Tokens](#get-all-api-tokens)
2. [Insert API Token](#insert-api-token)
3. [Delete API Token](#delete-api-token)

## Get All API Tokens

Retrieves all API tokens associated with the current user's account.

- **URL**: `/api/v1/apitoken/get_all`
- **Method**: GET
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | The page number for pagination |
| pagesize | integer | 50 | The number of items per page |
| sortfield | string | "CreateDate" | The field to sort the results by |
| descending | boolean | true | Whether to sort in descending order |

### Response

```json
{
  "data": [
    {
      "id": "string (uuid)",
      "userId": "string (uuid)",
      "sessionId": "string (uuid)",
      "title": "string",
      "token": "string",
      "expirationDate": "string (date-time)",
      "createDate": "string (date-time)"
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

## Insert API Token

Creates a new API token for the current user.

- **URL**: `/api/v1/apitoken/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "title": "string",
  "expirationDate": "string (date-time)"
}
```

### Response

```json
{
  "id": "string (uuid)",
  "userId": "string (uuid)",
  "sessionId": "string (uuid)",
  "title": "string",
  "token": "string",
  "expirationDate": "string (date-time)",
  "createDate": "string (date-time)"
}
```

## Delete API Token

Deletes an existing API token.

- **URL**: `/api/v1/apitoken/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The ID of the API token to delete |

### Response

A successful deletion returns a `200 OK` status with no body.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### Get All API Tokens

<details>
<summary>Python</summary>

```python
import requests

url = "https://api.amove.com/api/v1/apitoken/get_all"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
params = {
    "page": 1,
    "pagesize": 10
}

response = requests.get(url, headers=headers, params=params)

if response.status_code == 200:
    api_tokens = response.json()
    for token in api_tokens['data']:
        print(f"Token Title: {token['title']}, Expiration: {token['expirationDate']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
fetch('https://api.amove.com/api/v1/apitoken/get_all?page=1&pagesize=10', {
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
  data.data.forEach(token => {
    console.log(`Token Title: ${token.title}, Expiration: ${token.expirationDate}`);
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

            var response = await client.GetAsync("api/v1/apitoken/get_all?page=1&pagesize=10");

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var tokens = JObject.Parse(content);
                foreach (var token in tokens["data"])
                {
                    Console.WriteLine($"Token Title: {token["title"]}, Expiration: {token["expirationDate"]}");
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

