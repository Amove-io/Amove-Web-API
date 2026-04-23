# Fastr License Endpoints

This document provides detailed information about Fastr license endpoints in the Amove API. A license binds a Fastr server installation to a specific machine and Amove account; it must be generated for a user, activated on a machine, and periodically validated while the server runs.

Two endpoints — [Activate License](#activate-license) and [Validate License](#validate-license) — are `[AllowAnonymous]`. They are designed to be called by the Fastr server daemon itself on the machine where it is installed, during activation and on subsequent heartbeats. End-user integrations typically only call the other four endpoints on this page.

## Endpoints

1. [Generate License](#generate-license)
2. [Activate License](#activate-license)
3. [Validate License](#validate-license)
4. [Get License Info](#get-license-info)
5. [Revoke License](#revoke-license)
6. [Get My Licenses](#get-my-licenses)


## Generate License

Creates a new Fastr license for the current user. If a `Professional` license is requested and the account is not on an active Stripe subscription, a pending Fastr subscription is created internally against the account owner's product package before the license itself is generated.

- **URL**: `/api/v1/fastrLicense/generate`
- **Method**: POST
- **Auth Required**: Yes

### Request Body

```json
{
  "licenseType": "integer (LicenseType)",
  "validityDays": "integer",
  "maxConnections": "integer"
}
```

- `licenseType` — see [License Type Values](#license-type-values). Defaults to `Standard` (1).
- `validityDays` — number of days before the license expires. Defaults to `365`.
- `maxConnections` — optional cap on concurrent connections the Fastr server will accept while running under this license.

### Response

Returns a `FastrLicenseResponse`. The license is in the `Generated` state and has not yet been bound to a machine.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "licenseKey": "XXXX-XXXX-XXXX-XXXX",
  "status": 0,
  "licenseType": 1,
  "createdAt": "2026-04-22T10:00:00Z",
  "expiredAt": "2027-04-22T10:00:00Z",
  "activatedAt": null,
  "machineName": null,
  "maxConnections": 10,
  "isActivated": false
}
```


## Activate License

Binds a previously-generated license to a specific machine. This endpoint is called by the Fastr server daemon itself during its first-run setup; it reads the license key from local configuration and posts the machine fingerprint to this URL.

After a successful activation, a `FastrServer` `CloudAccount` is automatically created (or linked, if one already exists) under the license's account so that the newly-activated server becomes a usable backing store in the Amove platform.

- **URL**: `/api/v1/fastrLicense/activate`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "licenseKey": "string",
  "machineId": "string",
  "machineName": "string"
}
```

- `licenseKey` — the license key returned by [Generate License](#generate-license).
- `machineId` — a stable identifier for the machine; it is hashed server-side before storage and is what the license becomes bound to.
- `machineName` — a human-readable machine name shown in `my-licenses`. Optional; defaults to `Unknown Machine`.

### Response

```json
{
  "success": true,
  "error": null,
  "licenseKey": "XXXX-XXXX-XXXX-XXXX",
  "activatedAt": "2026-04-22T10:05:00Z",
  "expiresAt": "2027-04-22T10:00:00Z",
  "licenseType": 1,
  "maxConnections": 10,
  "cloudAccountId": "33333333-3333-3333-3333-333333333333",
  "accountId": "11111111-1111-1111-1111-111111111111",
  "userId": "22222222-2222-2222-2222-222222222222"
}
```

Activation fails (HTTP 400 with `success: false` and an `error` message) when:

- The license key or machine ID is missing.
- The license does not exist or has already expired.
- The license is not in the `Generated` state (e.g., it has already been activated, revoked, or suspended).
- The license is already bound to a different machine. To move a license to a new machine, first [Revoke](#revoke-license) it and then re-activate.


## Validate License

Verifies that a license is still valid for a given machine. This is the heartbeat endpoint the Fastr server daemon calls periodically to check that it is still authorized to run.

- **URL**: `/api/v1/fastrLicense/validate`
- **Method**: POST
- **Auth Required**: No

### Request Body

```json
{
  "licenseKey": "string",
  "machineId": "string"
}
```

- `machineId` must be the same machine fingerprint that was used during activation; the server compares it to the stored hash.

### Response

```json
{
  "isValid": true,
  "error": null,
  "expiresAt": "2027-04-22T10:00:00Z",
  "licenseType": 1,
  "maxConnections": 10
}
```

`isValid` is `false` when the license is missing, expired, revoked, suspended, not yet activated, or bound to a different machine; the `error` field carries a short human-readable reason in each case.


## Get License Info

Returns the full record for a single license by its license key. Only the license owner may call this endpoint; `ProviderAdmin` users may additionally inspect any license in the system. All other callers receive `403 Forbidden`.

- **URL**: `/api/v1/fastrLicense/info/{licenseKey}`
- **Method**: GET
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| licenseKey | string | The license key, e.g., `XXXX-XXXX-XXXX-XXXX` |

### Response

Returns the same `FastrLicenseResponse` schema as [Generate License](#generate-license). If the license key is unknown, the endpoint returns `404 Not Found`.


## Revoke License

Revokes a license and resets it so it can be activated on a new machine. The license moves back to the `Generated` state; its machine binding is cleared; its activation timestamp is erased. The license is not deleted and remains usable.

Use this endpoint when a Fastr server is decommissioned or the license needs to be transferred to a different host.

- **URL**: `/api/v1/fastrLicense/revoke/{licenseId}`
- **Method**: POST
- **Auth Required**: Yes

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| licenseId | string (uuid) | ID of the license (the `id` field, not the `licenseKey`) |

### Response

```json
{
  "success": true,
  "error": null,
  "message": "License has been revoked and reset. It can now be activated on another machine."
}
```


## Get My Licenses

Returns every Fastr license owned by the current user.

- **URL**: `/api/v1/fastrLicense/my-licenses`
- **Method**: GET
- **Auth Required**: Yes

### Response

Returns an array of `FastrLicenseResponse` objects. Each entry carries the current activation state (`status`, `activatedAt`, `machineName`, `isActivated`) so callers can distinguish licenses that are ready to be activated from licenses already bound to a running Fastr server.


## License Status Values

The `status` field on a `FastrLicenseResponse` uses the following integer values:

| Value | Name | Description |
|---|---|---|
| 0 | Generated | License has been created but is not yet bound to a machine |
| 1 | Activated | License is bound to a machine and is in use |
| 2 | Expired | License is past its `expiredAt` date |
| 3 | Revoked | License has been revoked (and may have been reset for reuse) |
| 4 | Suspended | License has been suspended administratively |


## License Type Values

The `licenseType` field uses the following integer values:

| Value | Name |
|---|---|
| 0 | Trial |
| 1 | Standard |
| 2 | Professional |
| 3 | Enterprise |


## Sample Code

### Generate a license and list your licenses

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
HEADERS = {"Authorization": "Bearer YOUR_JWT"}

license_ = requests.post(
    f"{BASE}/api/v1/fastrLicense/generate",
    headers=HEADERS,
    json={"licenseType": 1, "validityDays": 365, "maxConnections": 10},
).json()
print("New license key:", license_["licenseKey"])

my_licenses = requests.get(
    f"{BASE}/api/v1/fastrLicense/my-licenses",
    headers=HEADERS,
).json()
for lic in my_licenses:
    print(lic["licenseKey"], "status:", lic["status"], "activated:", lic["isActivated"])
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

const lic = await fetch(`${BASE}/api/v1/fastrLicense/generate`, {
  method: "POST",
  headers,
  body: JSON.stringify({ licenseType: 1, validityDays: 365, maxConnections: 10 })
}).then(r => r.json());
console.log("New license key:", lic.licenseKey);

const myLicenses = await fetch(`${BASE}/api/v1/fastrLicense/my-licenses`, { headers })
  .then(r => r.json());
console.log(myLicenses);
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

var generated = await client.PostAsJsonAsync("api/v1/fastrLicense/generate", new {
    licenseType = 1,
    validityDays = 365,
    maxConnections = 10
});
var lic = await generated.Content.ReadFromJsonAsync<JsonElement>();
Console.WriteLine("New license key: " + lic.GetProperty("licenseKey").GetString());

string myLicenses = await client.GetStringAsync("api/v1/fastrLicense/my-licenses");
Console.WriteLine(myLicenses);
```

</details>

### Activate and validate a license (from the Fastr server daemon)

These two endpoints are anonymous and intended to be called by the Fastr server itself. No JWT is required.

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"

# First-run activation
activation = requests.post(
    f"{BASE}/api/v1/fastrLicense/activate",
    json={
        "licenseKey": "XXXX-XXXX-XXXX-XXXX",
        "machineId": "machine-fingerprint-123",
        "machineName": "prod-fastr-01",
    },
).json()

if activation["success"]:
    print("Activated until:", activation["expiresAt"])
else:
    print("Activation failed:", activation["error"])

# Periodic heartbeat
validation = requests.post(
    f"{BASE}/api/v1/fastrLicense/validate",
    json={
        "licenseKey": "XXXX-XXXX-XXXX-XXXX",
        "machineId": "machine-fingerprint-123",
    },
).json()

if not validation["isValid"]:
    raise SystemExit(f"License no longer valid: {validation['error']}")
```

</details>

### Revoke a license to move it to a new machine

<details>
<summary>Python</summary>

```python
import requests

BASE = "https://api.amove.io"
HEADERS = {"Authorization": "Bearer YOUR_JWT"}

license_id = "00000000-0000-0000-0000-000000000000"
resp = requests.post(
    f"{BASE}/api/v1/fastrLicense/revoke/{license_id}",
    headers=HEADERS,
).json()
print(resp["message"])
# The same license key can now be re-activated on a different machine.
```

</details>

For error handling, see [Error Model](errors.md).
