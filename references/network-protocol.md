# Network/TCP Protocol Reference

Complete syntax reference for Nuclei Network/TCP templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Inputs](#inputs)
- [Host and Port](#host-and-port)
- [TLS Connections](#tls-connections)
- [Matcher Parts](#matcher-parts)
- [Examples](#examples)

---

## Basic Structure

```yaml
tcp:
  - inputs:
      - data: "TEST\r\n"
    host:
      - "{{Hostname}}"
    port: 8080
    read-size: 2048
    matchers:
      - type: word
        words:
          - "indicator"
        part: data
```

---

## Inputs

### Simple String
```yaml
inputs:
  - data: "VERSION\r\n"
```

### Hex Encoded
```yaml
inputs:
  - data: "50494e47"
    type: hex
  - data: "\r\n"
```

### Helper Functions
```yaml
inputs:
  - data: 'hex_decode("50494e47")\r\n'
```

### Reading from Socket
```yaml
inputs:
  - read-size: 8
    name: prefix
matchers:
  - type: word
    part: prefix
    words:
      - "CAFEBABE"
```

### Multi-Step Input/Read
```yaml
inputs:
  - data: "USER admin\r\n"
  - read-size: 1024
    name: response1
  - data: "PASS test\r\n"
  - read-size: 1024
    name: response2
```

---

## Host and Port

```yaml
host:
  - "{{Hostname}}"
port: 27017
```

**Port options:**
- Single port: `port: 8080`
- Multiple ports: `port: 5432,5433`
- Exclude default ports: `exclude-ports: 80,443`

Default reserved ports (excluded unless specified): `80`, `443`, `8080`, `8443`, `8081`, `53`

---

## TLS Connections

```yaml
host:
  - "tls://{{Hostname}}"
port: 443
```

---

## Matcher Parts

| Part | Description |
|---|---|
| `request` | The request sent |
| `data` | Final data read from socket |
| `raw` / `body` / `all` | All data received from socket |
| Named parts | Named via `name:` in inputs (e.g., `prefix`, `response1`) |

---

## Examples

### MongoDB Detection
```yaml
tcp:
  - inputs:
      - data: "{{hex_decode('3a000000...')}}"
    host:
      - "{{Hostname}}"
    port: 27017
    read-size: 2048
    matchers:
      - type: word
        words:
          - "logicalSessionTimeout"
          - "localTime"
```

### Redis Detection
```yaml
tcp:
  - inputs:
      - data: "INFO\r\n"
    host:
      - "{{Hostname}}"
    port: 6379
    read-size: 4096
    matchers:
      - type: word
        words:
          - "redis_version"
        part: data
```

### SSH Version Detection
```yaml
tcp:
  - inputs:
      - data: "\r\n"
    host:
      - "{{Hostname}}"
    port: 22
    read-size: 1024
    matchers:
      - type: regex
        regex:
          - "SSH-2.0-OpenSSH"
        part: data
    extractors:
      - type: regex
        regex:
          - "(SSH-[0-9.]+-[^\r\n]+)"
        group: 1
```

### MySQL Detection
```yaml
tcp:
  - inputs:
      - data: "\r\n"
    host:
      - "{{Hostname}}"
    port: 3306
    read-size: 1024
    matchers:
      - type: word
        words:
          - "mysql"
          - "mariadb"
        part: data
        case-insensitive: true
```

### PostgreSQL Detection
```yaml
tcp:
  - inputs:
      - data: "{{hex_decode('0000000804d2162f')}}"
    host:
      - "{{Hostname}}"
    port: 5432
    read-size: 1024
    matchers:
      - type: word
        words:
          - "PostgreSQL"
        part: data
```

### BGP Detection
```yaml
tcp:
  - inputs:
      - data: "FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF001D0104..."
        type: hex
    host:
      - "{{Hostname}}"
    port: 179
    read-size: 16
    matchers:
      - type: word
        encoding: hex
        words:
          - "ffffffffffffffffffffffffffffffff"
```
