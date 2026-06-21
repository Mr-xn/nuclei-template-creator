# DNS Protocol Reference

Complete syntax reference for Nuclei DNS templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Configuration Options](#configuration-options)
- [Record Types](#record-types)
- [Matcher Parts](#matcher-parts)
- [Examples](#examples)

---

## Basic Structure

```yaml
dns:
  - name: "{{FQDN}}"
    type: A
    class: inet
    recursion: true
    retries: 3
    matchers:
      - type: word
        part: answer
        words:
          - "IN\tA"
```

---

## Configuration Options

| Option | Type | Description |
|---|---|---|
| `name` | string | DNS name to resolve. Use `{{FQDN}}` for target's FQDN |
| `type` | string | DNS record type |
| `class` | string | DNS class (usually `inet`) |
| `recursion` | bool | Enable recursion (recommended: true) |
| `retries` | int | Number of retry attempts |

---

## Record Types

Supported DNS record types:
- `A` - IPv4 address
- `AAAA` - IPv6 address
- `CNAME` - Canonical name
- `MX` - Mail exchange
- `NS` - Nameserver
- `TXT` - Text record
- `SOA` - Start of authority
- `PTR` - Pointer record
- `SRV` - Service record
- `CAA` - Certificate authority authorization

---

## Matcher Parts

| Part | Description |
|---|---|
| `request` | DNS request |
| `rcode` | DNS response code |
| `question` | DNS question section |
| `extra` | DNS extra section |
| `answer` | DNS answer section (most common) |
| `ns` | DNS authority section |
| `raw` / `all` / `body` | Raw DNS message |

---

## Examples

### CNAME Detection
```yaml
dns:
  - name: "{{FQDN}}"
    type: CNAME
    matchers:
      - type: word
        part: answer
        words:
          - ".azure-api.net"
          - ".cloudfront.net"
        condition: or
```

### MX Record Check
```yaml
dns:
  - name: "{{FQDN}}"
    type: MX
    matchers:
      - type: word
        part: answer
        words:
          - "google.com"
          - "outlook.com"
        condition: or
```

### TXT Record (SPF/DMARC)
```yaml
dns:
  - name: "{{FQDN}}"
    type: TXT
    matchers:
      - type: word
        part: answer
        words:
          - "v=spf1"
    extractors:
      - type: regex
        part: answer
        regex:
          - '"(v=spf1[^"]+)"'
        group: 1
```

### DKIM Detection with Payloads
```yaml
dns:
  - name: "{{selector}}._domainkey.{{FQDN}}"
    type: TXT
    payloads:
      selector:
        - default
        - google
        - selector1
        - selector2
        - k1
        - k2
        - mail
        - dkim
    matchers:
      - type: word
        part: answer
        words:
          - "v=DKIM1"
```

### DNS Rebinding Detection
```yaml
dns:
  - name: "{{FQDN}}"
    type: A
    matchers:
      - type: dsl
        dsl:
          - "cname == ''"
          - "A == '127.0.0.1'"
        condition: and
```

### WAF Detection via DNS
```yaml
dns:
  - name: "{{FQDN}}"
    type: CNAME
    matchers:
      - type: word
        name: cloudflare
        part: answer
        words:
          - ".cloudflare.com"
      - type: word
        name: akamai
        part: answer
        words:
          - ".akamaiedge.net"
          - ".akamai.net"
        condition: or
```
