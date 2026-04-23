# Amove API Documentation

## Introduction

Welcome to the Amove API documentation. This API provides a comprehensive set of endpoints for managing cloud storage, user accounts, projects, and data transfers. It allows developers to integrate Amove's cloud management capabilities into their applications.

## Table of Contents

1. [Base URLs](#base-urls)
2. [Authentication](#authentication)
3. [API Endpoints](#api-endpoints)
4. [Error Model](#error-model)
5. [Pagination](#pagination)
6. [Routes and Versioning](#routes-and-versioning)
7. [Getting Started](#getting-started)
8. [Support](#support)

## Base URLs

The Amove platform exposes two public services:

| Service | Base URL | Purpose |
|---|---|---|
| Authentication API | `https://auth.amove.io` | Obtain and manage JWT tokens |
| Main API | `https://api.amove.io` | All other endpoints (users, cloud accounts, transfers, etc.) |

You first obtain a JWT from the Authentication API, then use it as a Bearer token on every call to the Main API.

## Authentication

Amove uses a two-step request-token handshake followed by a standard JWT bearer-token pattern:

1. `POST https://auth.amove.io/api/authentication/request_token` — returns a short-lived request token (60-second TTL).
2. `POST https://auth.amove.io/api/authentication/login` with the request token and credentials — returns a JWT.
3. Include the JWT on every subsequent call to the Main API:

```
Authorization: Bearer <your_jwt_here>
```

If your account has MFA enabled, `login` returns a TOTP challenge and encrypted session instead of a JWT; complete the challenge via `mfa_respond` to obtain the JWT.

See [authentication.md](authentication.md) for the full flow, MFA handling, and code samples.

## API Endpoints

The Main API is organized into these categories:

- [ApiToken](apitoken.md) — create and manage long-lived API tokens
- [CloudAccount](cloudaccount.md) — connect external cloud storage providers
- [Desktop](desktop.md) — desktop-client-specific endpoints (signup, presigned URLs, OAuth flows)
- [Fastr](fastr.md) — register and manage Fastr P2P servers
- [FastrLicense](fastr_license.md) — Fastr license generation and activation
- [FastrSettings](fastr_settings.md) — Fastr server settings per user and cloud account
- [Project](project.md) — project management
- [SharedCloudDrive](sharedclouddrive.md) — shared cloud drive management
- [SSO](sso.md) — Okta, SAML, and Entra ID single sign-on configuration
- [Storage](storage.md) — internal storage key and bucket management
- [Sync](sync.md) — cloud-to-cloud sync configuration
- [Transfer](transfer.md) — cloud-to-cloud file transfers
- [TransferHistory](transferhistory.md) — historical transfer records and trends
- [User](user.md) — user account management
- [UserGroup](usergroup.md) — user group management
- [UsersPermission](userspermission.md) — project and shared drive permission assignment

Cross-cutting documents:

- [Authentication](authentication.md) — login, logout, MFA, password reset
- [Error Model](errors.md) — HTTP status codes and application error codes

## Error Model

Amove uses a non-standard status code for application-level validation errors: **HTTP 499**. Successful responses use `200 OK`. Authentication failures return `401 Unauthorized`. Unexpected server errors return `500 Internal Server Error` with no body.

Every `499` response carries a `ValidationProblemDetails` JSON body with an `errors` dictionary keyed by an application error code (e.g., `AUTH`, `DUPLICATE`, `NOT_FOUND`), plus an `error-code` response header. See [errors.md](errors.md) for the complete reference.

## Pagination

List endpoints follow a consistent pagination convention. They accept:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Starting page (1-indexed) |
| pagesize | integer | 50 | Records per page |
| sortfield | string | varies | Field to sort by (e.g., `CreateDate`, `Name`) |
| descending | boolean | varies | Sort direction |

Paginated responses wrap the data collection:

```json
{
  "data": [ ],
  "total": 123,
  "options": {
    "pageSize": 50,
    "page": 1,
    "sort": [
      { "field": "CreateDate", "descending": true }
    ]
  }
}
```

## Routes and Versioning

All Main API routes are exposed under `/api/v1/<resource>/<action>`. Use this pattern in all integrations. A legacy un-versioned alias (`/api/<resource>/<action>`) exists for backward compatibility but is deprecated; do not use it in new integrations.

## Getting Started

1. Contact Amove to set up your account (https://www.amove.io).
2. Follow the [Authentication](authentication.md) flow to obtain a JWT.
3. Make your first call — for example, retrieve the current user profile:

```python
import requests

response = requests.get(
    "https://api.amove.io/api/v1/user/userinfo",
    headers={"Authorization": "Bearer YOUR_JWT"},
)
print(response.json())
```

## Support

For help integrating the Amove API, visit our [Support](https://www.amove.io/support) page.
