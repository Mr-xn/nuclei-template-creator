# Matchers and Extractors Reference

Complete reference for Nuclei template matching and data extraction.

## Table of Contents
- [Matchers Overview](#matchers-overview)
- [Matcher Types](#matcher-types)
- [Matcher Options](#matcher-options)
- [Matcher Combination](#matcher-combination)
- [Extractors Overview](#extractors-overview)
- [Extractor Types](#extractor-types)
- [Extractor Options](#extractor-options)
- [Best Practices](#best-practices)

---

## Matchers Overview

Matchers determine if a template matches (vulnerability exists). They evaluate against response data.

---

## Matcher Types

### status
Integer status code matching:
```yaml
- type: status
  status:
    - 200
    - 302
    - 301
```

### size
Matches content length of a response part:
```yaml
# Match specific body size
- type: size
  size:
    - 9593
  part: body

# Match multiple possible sizes
- type: size
  size:
    - 0
    - 100
    - 1000
  part: body

# Match when size is NOT 0 (empty response check)
- type: size
  size:
    - 0
  part: body
  negative: true

# Match header size
- type: size
  size:
    - 0
  part: header
```

### word
Exact string matching:
```yaml
- type: word
  words:
    - "string1"
    - "string2"
  condition: and  # or "or" (default)
  part: body
  negative: false
  case-insensitive: false
  encoding: hex  # For hex-encoded matching
```

### regex
Regular expression matching:
```yaml
- type: regex
  regex:
    - "SQL syntax.{0,500}?MySQL"
    - "Warning.{0,500}?\\Wmysqli?_"
  condition: or
  part: body
  negative: false
```

### binary
Hexadecimal binary matching:
```yaml
- type: binary
  binary:
    - "504B0304"          # zip archive magic bytes
    - "526172211A070100"  # RAR archive v5.0
    - "FD377A585A0000"    # xz tar.xz archive
  condition: or
  part: body
```

### dsl
Domain Specific Language expressions:
```yaml
- type: dsl
  dsl:
    - 'status_code == 200'
    - 'contains(body, "vulnerable_pattern")'
    - 'contains_any(body, "str1", "str2")'
    - 'len(body) < 2000'
    - 'status_code_1 == 403 && status_code_2 != 403'
  condition: and
```

### xpath
XPath query matching (returns true if any results):
```yaml
- type: xpath
  part: body
  xpath:
    - "/html/head/title[contains(text(), 'Example Domain')]"
```

---

## Matcher Options

| Option | Type | Description |
|---|---|---|
| `type` | string | Matcher type: `status`, `size`, `word`, `regex`, `binary`, `dsl`, `xpath` |
| `part` | string | What to match against |
| `condition` | string | `and` or `or` (default: `or`) |
| `negative` | bool | Invert match logic |
| `encoding` | string | `hex` for hex-encoded matching (word type) |
| `name` | string | Named matcher identifier |
| `internal` | bool | Internal use only (not displayed) |

### DSL Response Parts

When using DSL matchers, these response variables are available:

| Variable | Description | Example |
|---|---|---|
| `status_code` | Response status code | `status_code == 200` |
| `content_length` | Content-Length header value | `content_length >= 1024` |
| `body` | Response body as string | `len(body) < 1024` |
| `all_headers` | All headers as single string | `len(all_headers)` |
| `header_name` | Specific header (lowercase, `-` → `_`) | `len(user_agent)` |
| `raw` | Headers + Response combined | `len(raw)` |

### Part Values

**HTTP:**
- `body` - Response body (default)
- `header` - Response headers
- `content_type` - Content-Type header
- `interactsh_protocol` - OOB interaction protocol
- `interactsh_request` - OOB interaction request
- Named extractors: `extract1`, `extract2`, etc.

**DNS:**
- `answer` - DNS answer section
- `question` - DNS question section
- `rcode` - DNS response code
- `ns` - DNS authority section
- `extra` - DNS extra section
- `request` - DNS request
- `raw` / `all` / `body` - Raw DNS message

**SSL:**
- `Subject` - Certificate subject
- `Issuer` - Certificate issuer
- `CN` - Common name
- `SAN` - Subject alternative names
- `tls_version` - TLS version
- `cipher` - Cipher suite

**Network/TCP:**
- `request` - Sent request
- `data` - Final data read
- `raw` / `body` / `all` - All received data
- Named parts from inputs

**File:**
- `raw` - Raw file content (default)
- `body` - File content

---

## Matcher Combination

### Top-level condition
```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/test"
    matchers-condition: and  # ALL matchers must match
    matchers:
      - type: status
        status:
          - 200
      - type: word
        words:
          - "indicator"
        part: body
```

### Per-matcher condition
```yaml
matchers:
  - type: word
    words:
      - "str1"
      - "str2"
    condition: or  # ANY word matches (default)
    part: body
```

### Negative Matchers
```yaml
matchers:
  - type: word
    words:
      - "404 Not Found"
    part: body
    negative: true  # Match when NOT found
```

---

## Global Matchers

Apply matchers **globally across all HTTP responses** from other templates.

```yaml
http:
  - global-matchers: true
    matchers-condition: or
    matchers:
      - type: regex
        name: asymmetric_private_key
        regex:
          - '-----BEGIN ((EC|PGP|DSA|RSA|OPENSSH) )?PRIVATE KEY( BLOCK)?-----'
        part: body
      - type: regex
        name: slack_webhook
        regex:
          - 'https://hooks.slack.com/services/T[a-zA-Z0-9_]{8,10}/B[a-zA-Z0-9_]{8,12}/[a-zA-Z0-9_]{23,24}'
        part: body
```

**Usage:**
```bash
nuclei -egm -u http://example.com -t global-matchers.yaml -t template1.yaml -t template2.yaml
```

**Key properties:**
- Only works with HTTP-protocol-based templates
- When enabled, **no requests** are sent from this template
- Extractors can also be defined
- Must be explicitly enabled via `-enable-global-matchers` / `-egm` CLI flag

---

## Extractors Overview

Extractors pull data from responses for display or use in subsequent requests.

---

## Extractor Types

### regex
Extract data using regular expressions:
```yaml
extractors:
  - type: regex
    part: body
    group: 1
    name: version
    regex:
      - 'Version: ([0-9\.]+)'
```

**Fields:**
- `type: regex`
- `part` - Response part: `header`, `body`, `all`
- `regex` - List of regex patterns
- `group` - Match group index (0=full match, 1+=capture groups)

### kval
Extract key:value/key=value data from Response Header/Cookie:
```yaml
extractors:
  - type: kval
    kval:
      - content_type
      - set_cookie
```

**Important:** Dashes (`-`) are **not accepted** in kval input. Replace with underscores (`_`):
- `content-type` → `content_type`
- `set-cookie` → `set_cookie`
- `x-powered-by` → `x_powered_by`

### json
Extract from JSON responses using JQ-like syntax:
```yaml
extractors:
  - type: json
    part: body
    name: user
    json:
      - '.[] | .id'
      - ".login"
```

**Fields:**
- `type: json`
- `part` - Response part (e.g., `body`)
- `name` - Variable name for extracted value
- `json` - List of JQ-like expressions

### xpath
Extract data from HTML responses using XPath:
```yaml
extractors:
  - type: xpath
    attribute: href  # Optional: extract specific attribute
    xpath:
      - '/html/body/div/p[2]/a'
```

**Fields:**
- `type: xpath`
- `xpath` - List of XPath expressions
- `attribute` - (optional) HTML attribute to extract from matched element

### dsl
Extract data using DSL expressions:
```yaml
extractors:
  - type: dsl
    dsl:
      - len(body)
      - cname
      - "tls_version, cipher"
      - '"Title is " + title'
```

**Fields:**
- `type: dsl`
- `dsl` - List of DSL expressions to evaluate

---

## Extractor Options

| Field | Applies To | Description |
|---|---|---|
| `type` | all | Extractor type: `regex`, `kval`, `json`, `xpath`, `dsl` |
| `part` | regex, json, dsl | Response part: `header`, `body`, `all` |
| `regex` | regex | List of regex patterns |
| `group` | regex | Match group index (0=full match, 1+=capture groups) |
| `kval` | kval | List of key names (use `_` instead of `-`) |
| `json` | json | List of JQ-like expressions |
| `xpath` | xpath | List of XPath expressions |
| `attribute` | xpath | (optional) HTML attribute to extract |
| `dsl` | dsl | List of DSL expressions |
| `name` | all (dynamic) | Variable name for storing extracted value |
| `internal` | all (dynamic) | `true` to suppress output and enable dynamic variable usage |

---

## Dynamic Extractors

Extractors can capture **dynamic values at runtime** for use in subsequent requests (e.g., CSRF tokens, session headers). Only available in RAW request format.

### Basic Dynamic Extractor
```yaml
extractors:
  - type: regex
    name: csrf_token
    part: body
    internal: true  # Required for dynamic variables
    group: 1
    regex:
      - 'name="csrf_token" value="([a-zA-Z0-9]{16})"'
```

**Key fields:**
- `name` - Variable name for use in subsequent requests
- `internal: true` - Required to suppress output and enable variable usage

### Reusable Dynamic Extractors (v3.1.4+)
Dynamic extracted values can be reused **immediately in the next extractors**:

```yaml
http:
  - raw:
      - |
        GET / HTTP/1.1
        Host: {{Hostname}}

    extractors:
      - type: regex
        name: title
        group: 1
        regex:
          - '<title>(.*)</title>'
        internal: true

      - type: dsl
        dsl:
          - '"Title is " + title'
```

---

## Best Practices

### Layered Verification Strategy
```yaml
# Layer 1: Identify the application
# Layer 2: Confirm vulnerable version
# Layer 3: Prove vulnerability exists
matchers-condition: and
matchers:
  - type: word                    # Layer 1: App identification
    words:
      - "Apache Struts Framework"
    part: body
    
  - type: regex                   # Layer 2: Version detection
    regex:
      - 'Struts 2\.[0-4]\.[0-9]+'
    part: body
    
  - type: word                    # Layer 3: Exploitation proof
    words:
      - "ognl.OgnlException"
    part: body
    condition: or
```

### Random Markers for RCE
```yaml
variables:
  marker: "{{rand_base(8)}}"
# Use {{marker}} in request, then match against it in response
```

### OOB Detection for Blind Vulns
```yaml
matchers:
  - type: word
    part: interactsh_protocol
    words:
      - "dns"  # or "http"
```

### Avoid Weak Matchers
```yaml
# BAD - Too generic
matchers:
  - type: word
    words:
      - "error"
      - "admin"

# GOOD - Specific
matchers-condition: and
matchers:
  - type: word
    words:
      - "VulnApp Management Console v2.1.0"
    part: body
  - type: word
    words:
      - "{{marker}}"  # Random exploitation proof
    part: body
```
