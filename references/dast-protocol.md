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
| `part` | string | Where to inject (default `query`) |
| `parts` | list | Plural form — select multiple parts to fuzz at once |
| `type` | string | Replacement type (default `replace`) |
| `mode` | string | `multiple` (default — all values at once) or `single` (one value at a time) |
| `fuzz` | list | Payload values to inject (supports payloads, DSL functions, variables) |
| `keys` | list | Exact parameter names to fuzz |
| `keys-regex` | list | Parameter-name regex to fuzz |
| `values` | list | Value regex to fuzz |

### Part Values
- `query` (default) - Query parameters
- `path` - URL path parameters
- `header` - Request headers
- `cookie` - Cookies
- `body` - Request body (JSON/XML/form/multipart are all abstracted as key-value pairs; binary/unknown bodies become a single `value` pair)
- `request` - Special: fuzz the entire request (all parts above)

```yaml
fuzzing:
  - parts: [query, body, header]   # multiple selective parts
```

### Type Values
- `replace` (default) - Replace the value with the payload
- `prefix` - Prepend payload to the value
- `postfix` - Append payload to the value
- `infix` - Insert payload inside the value
- `replace-regex` - Replace via regex

### Key-Value Abstraction
Nuclei converts every request part into key/value pairs, so ONE rule covers all body formats (JSON, XML, form, multipart): a body rule for SQLi works on every format automatically. E.g. `{"password":"12345678"}` → key `password`, value `12345678`.

## Analyzers (extra verification requests)

`time_delay` — verifies the response time is actually controllable by the payload using linear regression (ported from ZAP) with alternating delays instead of naive single-shot timing:

```yaml
analyzer:
  name: time_delay
  parameters:            # all optional, defaults are fine
    sleep_duration: 10           # default 5
    requests_limit: 6            # default 4
    time_correlation_error_range: 0.30   # default 0.15
    time_slope_error_range: 0.40         # default 0.30
```

Dynamic placeholders available in payloads with this analyzer: `[SLEEPTIME]` (sleep seconds) and `[INFERENCE]` (`%d=%d` condition). Match analyzer results with `part: analyzer`:

```yaml
matchers:
  - type: word
    part: analyzer
    words: ["true"]
```

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
