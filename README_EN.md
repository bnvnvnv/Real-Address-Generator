# Real Address Generator

English / [中文](README.md)

This is a real address generator based on Cloudflare Workers. The project currently uses a **single-file Worker architecture** with two separate access modes:

- **Public page `/`**: for regular visitors, no authorization required. The Worker generates the data server-side and renders the HTML directly.
- **Protected API `/api/v1/address`**: for developers, backend services, or other controlled clients. Access requires a Bearer Token.

> Note: the public page and the protected API are two separate access patterns. Do not embed `API_TOKEN` in the HTML template. If you ever need browser-side access to the protected endpoint, add a dedicated login/session mechanism instead of reusing a static API key.

## Demo

![](https://github.com/Adonis142857/Real-Address-Generator/blob/main/example.png)

Use online: [https://realaddress.fuyiran.com/](https://realaddress.fuyiran.com/)

## Features

- Generate random real addresses that exist on the map
- Support multiple country options
- Display address locations using Google Maps
- Generate random phone numbers, names, and genders, with click-to-copy support
- Save generated information with notes
- Provide both a public page and a protected developer API

## Access Modes

### 1. Public page `/`

- **Path**: `/`
- **Authorization**: not required
- **Use case**: regular browser visitors
- **Rendering**: the Worker generates the address data server-side and returns HTML
- **Optional query parameter**: `country`

Example:

```text
GET /?country=US
```

### 2. Protected API `/api/v1/address`

- **Path**: `/api/v1/address`
- **Authorization**: Bearer Token required
- **Intended callers**: backend services, automation, controlled external clients
- **Not recommended**: exposing a static token in a public web frontend

#### Request header format

```http
Authorization: Bearer <API_TOKEN>
```

#### Example request

```bash
curl "https://your-worker.example.workers.dev/api/v1/address?country=US" \
  -H "Authorization: Bearer your-api-token"
```

#### Success response

```json
{
  "success": true,
  "data": {
    "country": "US",
    "name": "John Smith",
    "gender": "Male",
    "phone": "+1 (415) 555-1234",
    "address": "123 Main St, San Francisco, 94105, US"
  }
}
```

#### Failure responses

Missing or invalid token:

```json
{
  "success": false,
  "error": {
    "code": "unauthorized",
    "message": "Missing or invalid Bearer token"
  }
}
```

Invalid `country` parameter:

```json
{
  "success": false,
  "error": {
    "code": "invalid_country",
    "message": "country must be one of the supported ISO-like country codes",
    "supportedCountries": ["US", "UK", "FR"]
  }
}
```

#### `country` parameter

`country` is optional on the public page and required on the protected API. Supported values are:

`US`, `UK`, `FR`, `DE`, `CN`, `TW`, `HK`, `JP`, `IN`, `AU`, `BR`, `CA`, `RU`, `ZA`, `MX`, `KR`, `IT`, `ES`, `TR`, `SA`, `AR`, `EG`, `NG`, `ID`

Notes:

- If `country` is omitted on the public page, the Worker randomly selects one.
- If `country` is missing or unsupported on the protected API, the API returns an error.
- For predictable integrations, callers should always pass `country` explicitly.

## Deployment

To deploy this website, use Cloudflare Workers to deploy the provided JavaScript code.

1. Register or log in to your [Cloudflare](https://www.cloudflare.com/) account.
2. Create a new Cloudflare Worker.
3. Copy `worker.js` into the Cloudflare Worker script editor.
4. Configure a Worker Secret / Variable named `API_TOKEN` for the protected API.
5. Save and deploy your Worker.

### Deployment guidance

- Keep `/` as a public page rendered server-side.
- Treat `/api/v1/address` as a developer-facing API and allow only controlled callers with a Bearer Token.
- Never expose `API_TOKEN` in HTML templates, frontend JavaScript, or static assets.
- If browser-side access to the protected API is ever needed, add a separate login/session/signature scheme instead of reusing a static API key.

## Feedback

If you encounter any issues during use, please submit them as [issues](https://github.com/Adonis142857/Real-Address-Generator/issues).

## License

This project is licensed under the MIT License.

## Copyright

The original version is from chatgpt.org.uk, modified by Adonis142857.
