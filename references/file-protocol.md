# File Protocol Reference

Complete syntax reference for Nuclei File scanning templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Configuration Options](#configuration-options)
- [Matcher Parts](#matcher-parts)
- [Examples](#examples)

---

## Basic Structure

```yaml
file:
  - extensions:
      - all
    matchers:
      - type: word
        part: raw
        words:
          - "secret_pattern"
```

---

## Configuration Options

| Option | Type | Description |
|---|---|---|
| `extensions` | list | File extensions to scan. Use `all` for all types |
| `max-size` | int | Maximum file size to scan |

Common extension values: `yaml`, `json`, `xml`, `env`, `config`, `ini`, `conf`, `txt`, `log`, `key`, `pem`, `p12`, `jks`, `all`

---

## Matcher Parts

| Part | Description |
|---|---|
| `raw` | Raw file content (most common) |
| `body` | File body content |
| `all` | All file content |

---

## Examples

### API Key Detection
```yaml
file:
  - extensions:
      - all
    matchers:
      - type: regex
        regex:
          - '(?i)(?:airtable)[a-z0-9_]*[ \t]*[=:]+[ \t]*["\']?([a-zA-Z0-9]{17})'
        part: raw
    extractors:
      - type: regex
        part: body
        regex:
          - '(?i)(?:airtable)[a-z0-9_]*[ \t]*[=:]+[ \t]*["\']?([a-zA-Z0-9]{17})'
        group: 1
```

### Private Key Detection
```yaml
file:
  - extensions:
      - pem
      - key
      - p12
    matchers:
      - type: word
        part: raw
        words:
          - "-----BEGIN RSA PRIVATE KEY-----"
          - "-----BEGIN EC PRIVATE KEY-----"
          - "-----BEGIN PRIVATE KEY-----"
        condition: or
```

### Malware Detection
```yaml
file:
  - extensions:
      - all
    matchers:
      - type: word
        part: raw
        words:
          - "malware_signature"
          - "suspicious_pattern"
        condition: or
```

### Configuration File Exposure
```yaml
file:
  - extensions:
      - yaml
      - yml
      - json
      - env
      - config
    matchers:
      - type: regex
        regex:
          - '(?i)(password|secret|api_key|token)[ \t]*[=:]+[ \t]*["\']?[^\s"\']{8,}'
        part: raw
```
