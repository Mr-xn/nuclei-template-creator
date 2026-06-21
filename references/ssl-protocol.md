# SSL/TLS Protocol Reference

Complete syntax reference for Nuclei SSL/TLS templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Configuration Options](#configuration-options)
- [Version Control](#version-control)
- [DSL Variables](#dsl-variables)
- [Matcher Parts](#matcher-parts)
- [Examples](#examples)

---

## Basic Structure

```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    matchers:
      - type: dsl
        dsl:
          - "expired == true"
```

---

## Configuration Options

| Option | Type | Description |
|---|---|---|
| `address` | string | Target address, typically `{{Host}}:{{Port}}` |
| `min_version` | string | Minimum TLS version to accept |
| `max_version` | string | Maximum TLS version to accept |

---

## Version Control

Supported TLS versions:
- `ssl30` - SSL 3.0
- `tls10` - TLS 1.0
- `tls11` - TLS 1.1
- `tls12` - TLS 1.2
- `tls13` - TLS 1.3

```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    min_version: tls10
    max_version: tls12
```

---

## DSL Variables

SSL templates expose special DSL variables:

| Variable | Type | Description |
|---|---|---|
| `expired` | bool | Certificate is expired |
| `self_signed` | bool | Certificate is self-signed |
| `not_after` | string | Certificate expiry date |
| `not_before` | string | Certificate start date |
| `tls_version` | string | Negotiated TLS version |
| `cipher` | string | Negotiated cipher suite |
| `host` | string | Target host |

---

## Matcher Parts

| Part | Description |
|---|---|
| `Subject` | Certificate subject |
| `Issuer` | Certificate issuer |
| `CN` | Common name |
| `SAN` | Subject alternative names |
| `Serial` | Certificate serial number |
| `fingerprint_hash` | Certificate fingerprint |
| `tls_version` | TLS version string |
| `cipher` | Cipher suite string |
| `not_after` | Expiry date |
| `not_before` | Start date |
| `raw` / `all` / `body` | Full certificate data |

---

## Examples

### Expired Certificate Detection
```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    matchers:
      - type: dsl
        dsl:
          - "expired == true"
    extractors:
      - type: dsl
        dsl:
          - "not_after"
```

### Self-Signed Certificate
```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    matchers:
      - type: dsl
        dsl:
          - "self_signed == true"
```

### Deprecated TLS Version
```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    min_version: tls10
    max_version: tls11
    matchers:
      - type: dsl
        dsl:
          - "tls_version == 'tls10' || tls_version == 'tls11' || tls_version == 'ssl30'"
```

### Weak Cipher Suite
```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    matchers:
      - type: word
        words:
          - "TLS_RSA_WITH_3DES_EDE_CBC_SHA"
          - "TLS_RSA_WITH_RC4_128_SHA"
          - "TLS_RSA_WITH_RC4_128_MD5"
        part: cipher
```

### Certificate Issuer Detection
```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    matchers:
      - type: word
        part: Issuer
        words:
          - "Let's Encrypt"
          - "DigiCert"
        condition: or
```

### Specific Cipher Suite Detection
```yaml
ssl:
  - address: "{{Host}}:{{Port}}"
    extractors:
      - type: dsl
        dsl:
          - "tls_version"
          - "cipher"
    matchers:
      - type: dsl
        dsl:
          - "tls_version == 'tls12'"
          - "cipher != ''"
        condition: and
```
