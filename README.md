# AMove API Documentation

## Introduction

Welcome to the AMove API documentation. This API provides a comprehensive set of endpoints for managing cloud storage, user accounts, projects, and data transfers. It allows developers to integrate AMove's powerful cloud management capabilities into their applications.

## Table of Contents

1. [Authentication](#authentication)
2. [API Endpoints](#api-endpoints)
3. [Getting Started](#getting-started)
4. [API Reference](#api-reference)
5. [SDKs and Libraries](#sdks-and-libraries)
6. [Examples](#examples)
7. [Changelog](#changelog)
8. [Support](#support)

## Authentication

The AMove API uses Bearer token authentication. Include your JWT token in the Authorization header of your requests:

```
Authorization: Bearer <your_token_here>
```

For more information on obtaining and using authentication tokens, please refer to our [Authentication Guide](authentication.md).

## API Endpoints

The AMove API is organized into the following main categories:

- [ApiToken](apitoken.md)
- [CloudAccount](cloudaccount.md)
- [Desktop](desktop.md)
- [Project](project.md)
- [Provisioning](provisioning.md)
- [SharedCloudDrive](sharedclouddrive.md)
- [SSO (Single Sign-On)](sso.md)
- [Storage](storage.md)
- [Sync](sync.md)
- [Transfer](transfer.md)
- [User](user.md)
- [UserGroup](usergroup.md)
- [UsersPermission](userspermission.md)

Each link above will take you to a detailed list of endpoints for that category.

## Getting Started

To start using the AMove API:

1. [Sign up for an AMove account](https://www.amove.com/signup)
2. [Obtain your API credentials](https://www.amove.com/dashboard/api-credentials)
3. Make your first API call (see [Examples](#examples) below)

For more detailed instructions, check out our [Getting Started Guide](getting-started.md).

## API Reference

Detailed documentation for each API endpoint is available in our API Reference. Please refer to the category links in the [API Endpoints](#api-endpoints) section above.

## SDKs and Libraries

We provide official SDKs for several popular programming languages to make integrating with the AMove API easier:

- [Python SDK](https://github.com/amove/amove-python-sdk)
- [JavaScript SDK](https://github.com/amove/amove-js-sdk)
- [C# SDK](https://github.com/amove/amove-csharp-sdk)

## Examples

Here's a simple example of how to use the AMove API to list all cloud accounts using Python:

```python
import requests

url = "https://api.amove.com/api/v1/cloudaccount/get_all"
headers = {
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}

response = requests.get(url, headers=headers)

if response.status_code == 200:
    cloud_accounts = response.json()
    for account in cloud_accounts['data']:
        print(f"Cloud Account: {account['name']}")
else:
    print(f"Error: {response.status_code}")
    print(response.text)
```

For more examples, including usage in other programming languages, see our [Examples Directory](examples/README.md).

## Changelog

For information about updates and changes to the API, please refer to our [Changelog](CHANGELOG.md).

## Support

If you encounter any issues or have questions about using the AMove API, please don't hesitate to:

- Check our [FAQ](FAQ.md)
- Visit our [Developer Forum](https://community.amove.com/c/api-developers)
- Contact our [Support Team](https://www.amove.com/support)

We're here to help you successfully integrate and use the AMove API in your applications!

