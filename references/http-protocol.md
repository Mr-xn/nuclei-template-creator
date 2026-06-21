# HTTP Protocol Reference

Complete syntax reference for Nuclei HTTP templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Request Methods](#request-methods)
- [Path and Variables](#path-and-variables)
- [Headers](#headers)
- [Body](#body)
- [Raw Requests](#raw-requests)
- [Redirects](#redirects)
- [Cookie Management](#cookie-management)
- [Multi-Request Templates](#multi-request-templates)
- [Payloads](#payloads)
- [Request Conditions](#request-conditions)
- [Self-Contained Templates](#self-contained-templates)

---

## Basic Structure

HTTP templates use the `http:` key and support two formats:

### Method + Path Format
```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/endpoint"
    matchers:
      - type: word
        words:
          - "indicator"
        part: body
```

### Raw HTTP Request Format
```yaml
http:
  - raw:
      - |
        GET /endpoint HTTP/1.1
        Host: {{Hostname}}
        Custom-Header: value
    matchers:
      - type: word
        words:
          - "indicator"
        part: body
```

---

## Request Methods

Supported: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS`

```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/api/login"
    body: '{"username":"admin","password":"test"}'
    headers:
      Content-Type: application/json
```

---

## Path and Variables

Multiple paths can be specified - all will be requested:

```yaml
path:
  - "{{BaseURL}}/path1"
  - "{{BaseURL}}/path2"
```

### Built-in URL Variables

For target `https://example.com:443/foo/bar.php`:

| Variable | Value |
|---|---|
| `{{BaseURL}}` | `https://example.com:443/foo/bar.php` |
| `{{RootURL}}` | `https://example.com:443` |
| `{{Hostname}}` | `example.com:443` |
| `{{Host}}` | `example.com` |
| `{{Port}}` | `443` |
| `{{Path}}` | `/foo` |
| `{{File}}` | `bar.php` |
| `{{Scheme}}` | `https` |

---

## Headers

Key-value pairs:

```yaml
headers:
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
  Content-Type: application/json
  Origin: https://example.com
  X-Forwarded-For: 127.0.0.1
  Authorization: Bearer {{token}}
```

---

## Body

```yaml
# Form data
body: "username=admin&password=test"

# JSON
body: '{"key": "value"}'

# Multipart (use raw request for complex multipart)
body: |
  --boundary
  Content-Disposition: form-data; name="file"
  
  file content
  --boundary--
```

---

## Raw Requests

For full control over the HTTP request:

```yaml
http:
  - raw:
      - |
        POST /api/login HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/json
        X-Forwarded-For: {{interactsh-url}}
        
        {"username":"admin","password":"test"}
```

Multiple raw requests can be chained:

```yaml
http:
  - raw:
      - |
        GET /login HTTP/1.1
        Host: {{Hostname}}
      - |
        POST /login HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/x-www-form-urlencoded
        
        csrf={{csrf_token}}&user=admin&pass=test
    cookie-reuse: true
```

---

## Redirects

Not followed by default. Enable per template:

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/redirect"
    redirects: true
    max-redirects: 5
    matchers:
      - type: word
        words:
          - "final destination"
        part: body
```

---

## Cookie Management

Cookies are reused by default across requests in a template. Disable with:

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/endpoint"
    disable-cookie: true
```

Or enable explicitly:

```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/login"
    cookie-reuse: true
```

---

## Multi-Request Templates

Templates can have multiple request blocks for exploit chains:

```yaml
http:
  # Request 1: Get CSRF token
  - method: GET
    path:
      - "{{BaseURL}}/login"
    extractors:
      - type: regex
        name: csrf_token
        internal: true
        regex:
          - 'name="csrf" value="([^"]+)"'
        group: 1

  # Request 2: Use token to authenticate
  - method: POST
    path:
      - "{{BaseURL}}/login"
    body: "csrf={{csrf_token}}&user=admin&pass=test"
    matchers:
      - type: word
        words:
          - "Dashboard"
        part: body
```

**Request indexing:** In multi-request templates, reference previous responses:
- `body_1`, `body_2` - Response body from request 1, 2
- `status_code_1`, `status_code_2` - Status codes from request 1, 2
- `header_1`, `header_2` - Headers from request 1, 2
- `content_type_1`, `content_type_2` - Content types

---

## Payloads

Define payloads for brute-force or fuzzing:

```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/login"
    body: "user={{username}}&pass={{password}}"
    payloads:
      username:
        - admin
        - root
        - administrator
      password:
        - admin
        - password
        - 123456
    attack: clusterbomb  # or pitchfork
    matchers:
      - type: status
        status:
          - 200
        negative: true
```

**Attack modes:**
- `clusterbomb` - All combinations (default)
- `pitchfork` - Pair values sequentially

### External Wordlists

```yaml
payloads:
  username:
    - "path/to/usernames.txt"
  password:
    - "path/to/passwords.txt"
```

---

## Request Conditions

Check conditions between multiple requests using suffixed variables:

```yaml
http:
  # Request 1
  - method: GET
    path:
      - "{{BaseURL}}/admin"

  # Request 2
  - method: GET
    path:
      - "{{BaseURL}}/admin"
    headers:
      X-Forwarded-For: 127.0.0.1
    matchers:
      - type: dsl
        dsl:
          - "status_code_1 == 403 && status_code_2 == 200"
```

---

## Self-Contained Templates

Templates that don't need a target URL:

```yaml
self-contained: true
http:
  - method: GET
    path:
      - "https://api.example.com/validate?token={{token}}"
    matchers:
      - type: word
        words:
          - "valid"
        part: body
```

---

## Host Redirects

Follow redirects to different hosts (useful for favicon detection, CDN-following):

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/favicon.ico"
    host-redirects: true
    max-redirects: 2
    matchers:
      - type: dsl
        dsl:
          - "status_code==200"
```

---

## Unsafe Raw Requests

Send raw requests without URL encoding (useful for SSRF testing, protocol smuggling):

```yaml
http:
  - raw:
      - |
        {{verb}} http://127.0.0.1:22 HTTP/1.1
        Host: {{Hostname}}
    unsafe: true
    matchers:
      - type: word
        words:
          - "SSH"
```

---

## Skip Variables Check

Skip validation of unresolved variables (useful when variables are optional):

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/api?token={{optional_token}}"
    skip-variables-check: true
    matchers:
      - type: word
        words:
          - "success"
```

---

## Common Options Summary

| Option | Type | Description |
|---|---|---|
| `method` | string | HTTP method |
| `path` | list | Request paths |
| `headers` | map | Custom headers |
| `body` | string | Request body |
| `raw` | list | Raw HTTP requests |
| `redirects` | bool | Follow redirects |
| `max-redirects` | int | Max redirect count |
| `host-redirects` | bool | Follow redirects to different hosts |
| `disable-cookie` | bool | Disable cookie reuse |
| `cookie-reuse` | bool | Enable cookie reuse |
| `unsafe` | bool | Send raw request without encoding |
| `skip-variables-check` | bool | Skip validation of unresolved variables |
| `matchers` | list | Matching rules |
| `extractors` | list | Data extraction rules |
| `payloads` | map | Payload definitions |
| `attack` | string | Attack mode |
| `stop-at-first-match` | bool | Stop after first match |
| `matchers-condition` | string | `and` or `or` |
