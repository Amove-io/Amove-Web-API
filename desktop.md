# Desktop Endpoints

This document provides detailed information about the Desktop-related endpoints in the AMove API. These endpoints are primarily used for desktop application integrations and operations.

## Endpoints

1. [Insert Cloud Account](#insert-cloud-account)
2. [List Buckets](#list-buckets)
3. [List Objects](#list-objects)
4. [Create Folder](#create-folder)
5. [Delete Cloud Account](#delete-cloud-account)
6. [Generate Download URL](#generate-download-url)
7. [Send Download URL](#send-download-url)
8. [Delete Object](#delete-object)
9. [Sign Up](#sign-up)
10. [Confirm Sign Up](#confirm-sign-up)
11. [Resend Confirmation Email](#resend-confirmation-email)
12. [Google Sign Up](#google-sign-up)
13. [Google Sign In](#google-sign-in)
14. [Setup Account Log Setting](#setup-account-log-setting)
15. [Get Account Log Setting](#get-account-log-setting)
16. [Update Account Log Setting](#update-account-log-setting)
17. [Delete Account Log Setting](#delete-account-log-setting)
18. [Generate Dropbox Authorization URL](#generate-dropbox-authorization-url)
19. [Dropbox Authorization Callback](#dropbox-authorization-callback)
20. [Generate OneDrive Authorization URL](#generate-onedrive-authorization-url)
21. [OneDrive Authorization Callback](#onedrive-authorization-callback)
22. [Generate Google Drive Authorization URL](#generate-google-drive-authorization-url)
23. [Google Drive Authorization Callback](#google-drive-authorization-callback)
24. [Generate Box Authorization URL](#generate-box-authorization-url)
25. [Box Authorization Callback](#box-authorization-callback)

## Insert Cloud Account

Creates a new cloud account for the current user.

- **URL**: `/api/v1/desktop/insert`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "accountId": "string (uuid)",
  "userId": "string (uuid)",
  "cloudType": "integer (enum)",
  "name": "string",
  "accessKey": "string",
  "secretKey": "string",
  "credentialsData": "string",
  "serviceUrl": "string",
  "active": "boolean",
  "shared": "boolean",
  "internalStorage": "boolean",
  "storageTier": "integer (enum)"
}
```

### Response

Returns the created cloud account object.

## List Buckets

Retrieves the list of buckets associated with a specified cloud account.

- **URL**: `/api/v1/desktop/list_buckets`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "includeRegion": "boolean"
}
```

### Response

```json
[
  {
    "id": "string",
    "name": "string",
    "region": "string",
    "size": "number",
    "creationDate": "string (date-time)"
  }
]
```

## List Objects

Retrieves the list of objects in a specified bucket.

- **URL**: `/api/v1/desktop/list_objects`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "bucketName": "string",
  "path": "string",
  "continuationToken": "string",
  "count": "integer"
}
```

### Response

```json
{
  "data": [
    {
      "id": "string",
      "name": "string",
      "path": "string",
      "size": "integer",
      "lastModifiedUtc": "string (date-time)",
      "type": "integer (enum)"
    }
  ],
  "continuationToken": "string",
  "bucketName": "string",
  "path": "string"
}
```

## Create Folder

Creates a new folder in a cloud storage.

- **URL**: `/api/v1/desktop/create_folder`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "path": "string",
  "folderName": "string"
}
```

### Response

A successful creation returns a `200 OK` status with no body.

## Delete Cloud Account

Deletes an existing cloud account.

- **URL**: `/api/v1/desktop/delete`
- **Method**: DELETE
- **Auth Required**: Yes

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string (uuid) | The ID of the cloud account to delete |

### Response

Returns a boolean indicating success or failure.

## Generate Download URL

Generates a download URL for a given object.

- **URL**: `/api/v1/desktop/generate_download_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": "integer",
  "forceToDownload": "boolean"
}
```

### Response

Returns a string containing the generated download URL.

## Send Download URL

Sends a download URL for a given object.

- **URL**: `/api/v1/desktop/send_download_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "email": "string",
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string",
  "expireHours": "integer",
  "note": "string"
}
```

### Response

A successful send returns a `200 OK` status with no body.

## Delete Object

Deletes an object from a cloud storage.

- **URL**: `/api/v1/desktop/delete_object`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudAccountId": "string (uuid)",
  "id": "string",
  "storageName": "string",
  "key": "string"
}
```

### Response

A successful deletion returns a `200 OK` status with no body.

## Sign Up

Signs up a new user and sends an activation code to the user's email.

- **URL**: `/api/v1/desktop/signup`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "requestToken": "string",
  "name": "string",
  "email": "string",
  "password": "string"
}
```

### Response

A successful signup returns a `200 OK` status with no body.

## Confirm Sign Up

Confirms user signup by validating the activation code.

- **URL**: `/api/v1/desktop/confirm_signup`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "requestToken": "string",
  "token": "string"
}
```

### Response

Returns a string (likely an authentication token).

## Resend Confirmation Email

Resends the confirmation email to the user's email address.

- **URL**: `/api/v1/desktop/resend_confirmation_email`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "requestToken": "string",
  "email": "string"
}
```

### Response

A successful resend returns a `200 OK` status with no body.

## Google Sign Up

Handles Google sign up.

- **URL**: `/api/v1/desktop/google_signup`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "requestToken": "string",
  "idToken": "string"
}
```

### Response

A successful Google sign up returns a `200 OK` status with no body.

## Google Sign In

Handles Google sign in.

- **URL**: `/api/v1/desktop/google_signin`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "requestToken": "string",
  "idToken": "string"
}
```

### Response

Returns a string (likely an authentication token).

## Setup Account Log Setting

Sets up the account log settings.

- **URL**: `/api/v1/desktop/setup_accountlogsetting`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "accountId": "string (uuid)",
  "token": "string",
  "serverAddress": "string",
  "servicePort": "string",
  "active": "boolean",
  "provider": "integer (enum)"
}
```

### Response

Returns the updated account log setting object.

## Get Account Log Setting

Retrieves the account log settings.

- **URL**: `/api/v1/desktop/get_accountlogsetting`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns the account log setting object.

## Update Account Log Setting

Updates the account log settings.

- **URL**: `/api/v1/desktop/update_accountlogsetting`
- **Method**: PUT
- **Auth Required**: Yes

### Request Body

```json
{
  "id": "string (uuid)",
  "accountId": "string (uuid)",
  "token": "string",
  "serverAddress": "string",
  "servicePort": "string",
  "active": "boolean",
  "provider": "integer (enum)"
}
```

### Response

Returns the updated account log setting object.

## Delete Account Log Setting

Deletes the account log settings.

- **URL**: `/api/v1/desktop/delete_accountlogsetting`
- **Method**: DELETE
- **Auth Required**: Yes

### Response

A successful deletion returns a `200 OK` status with no body.

## Generate Dropbox Authorization URL

Generates an authorization URL for Dropbox.

- **URL**: `/api/v1/desktop/generate_dropbox_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "redirectURL": "string",
  "cloudName": "string"
}
```

### Response

Returns a string containing the generated authorization URL.

## Dropbox Authorization Callback

Handles the Dropbox authorization callback.

- **URL**: `/api/v1/desktop/dropbox_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "redirectURL": "string"
}
```

### Response

Returns the created cloud account object.

## Generate OneDrive Authorization URL

Generates an authorization URL for OneDrive.

- **URL**: `/api/v1/desktop/generate_onedrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudName": "string",
  "redirectURL": "string"
}
```

### Response

Returns a string containing the generated authorization URL.

## OneDrive Authorization Callback

Handles the OneDrive authorization callback.

- **URL**: `/api/v1/desktop/onedrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "error": "string",
  "errorDescription": "string",
  "redirectURL": "string"
}
```

### Response

Returns the created cloud account object.

## Generate Google Drive Authorization URL

Generates an authorization URL for Google Drive.

- **URL**: `/api/v1/desktop/generate_googledrive_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "cloudName": "string",
  "redirectURL": "string"
}
```

### Response

Returns a string containing the generated authorization URL.

## Google Drive Authorization Callback

Handles the Google Drive authorization callback.

- **URL**: `/api/v1/desktop/googledrive_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "error": "string",
  "errorDescription": "string",
  "redirectURL": "string"
}
```

### Response

Returns the created cloud account object.

## Generate Box Authorization URL

Generates an authorization URL for Box.

- **URL**: `/api/v1/desktop/generate_box_authorization_url`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "redirectURL": "string",
  "cloudName": "string"
}
```

### Response

Returns a string containing the generated authorization URL.

## Box Authorization Callback

Handles the Box authorization callback.

- **URL**: `/api/v1/desktop/box_authorization_callback`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "code": "string",
  "state": "string",
  "redirectURL": "string"
}
```

### Response

Returns the created cloud account object.

## Error Responses

All endpoints may return the following error responses:

- `400 Bad Request`: The request was invalid or cannot be served.
- `401 Unauthorized`: The request requires authentication.
- `403 Forbidden`: The server understood the request but refuses to authorize it.
- `404 Not Found`: The requested resource could not be found.
- `500 Internal Server Error`: The server encountered an unexpected condition that prevented it from fulfilling the request.

## Sample Code

### List Buckets

<details>
<summary>Python</summary>

```python
import requests
import json

url = "https://api.amove.com/api/v1/desktop/list_buckets"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE",
    "Content-Type": "application/json"
}
data = {
    "cloudAccountId": "YOUR_CLOUD_ACCOUNT_ID",
    "includeRegion": True
}

response = requests.post(url, headers=headers, data=json.dumps(data))

if response.status_code == 200:
    buckets = response.json()
    for bucket in buckets:
        print(f"Bucket: {bucket['name']}, Region: {bucket['region']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

</details>

<details>
<summary>JavaScript</summary>

```javascript
const data = {
  cloudAccountId: 'YOUR_CLOUD_ACCOUNT_ID',
  includeRegion: true
};

fetch('https://api.amove.com/api/v1/desktop/list_buckets', {
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
  return response.json();
})
.then(buckets => {
  buckets.forEach(bucket => {
    console.log(`Bucket: ${bucket.name}, Region: ${bucket.region}`);
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
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

class Program
{
    static async Task Main(string[] args)
    {
        using (var client = new HttpClient())
        