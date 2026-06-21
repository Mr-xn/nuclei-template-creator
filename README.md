# Nuclei Template Creator

**English** | [中文](README_CN.md)

A comprehensive skill for creating high-quality [Nuclei](https://github.com/projectdiscovery/nuclei) security scanning templates across all supported protocols and vulnerability types.

## Overview

This skill provides complete documentation and reference materials for writing Nuclei templates, covering everything from basic HTTP vulnerability detection to advanced cloud security scanning and workflow orchestration.

### What's Included

| Category | Description |
|---|---|
| **10 Protocol Types** | HTTP, DNS, SSL/TLS, Network/TCP, File, Headless, JavaScript, Code, DAST, Cloud |
| **7 Matcher Types** | status, size, word, regex, binary, dsl, xpath |
| **5 Extractor Types** | regex, kval, json, xpath, dsl |
| **60+ DSL Functions** | String, encoding, hash, compression, random, date/time, crypto functions |
| **35 JS Functions** | Runtime, flow control, OS/architecture detection |
| **9 Example Templates** | Real-world patterns for CVE, misconfiguration, takeover, default login, etc. |

## Directory Structure

```
nuclei-template-creator/
├── SKILL.md                          # Main skill file (entry point)
└── references/
    ├── http-protocol.md              # HTTP template syntax
    ├── dns-protocol.md               # DNS template syntax
    ├── ssl-protocol.md               # SSL/TLS template syntax
    ├── network-protocol.md           # TCP/Network template syntax
    ├── file-protocol.md              # File scanning syntax
    ├── headless-protocol.md          # Headless browser syntax
    ├── javascript-protocol.md        # JavaScript protocol syntax
    ├── code-protocol.md              # Code execution syntax
    ├── dast-protocol.md              # DAST fuzzing syntax
    ├── cloud-protocol.md             # Cloud security syntax
    ├── matchers-extractors.md        # Matchers & extractors reference
    ├── dsl-functions.md              # DSL functions reference
    ├── variables.md                  # Built-in variables reference
    ├── preprocessors.md              # Preprocessors reference
    ├── oob-testing.md                # OOB testing reference
    ├── workflows.md                  # Workflow template syntax
    ├── flow.md                       # Flow control syntax
    └── examples/                     # Real-world template examples
        ├── cve-http.yaml
        ├── misconfiguration.yaml
        ├── takeover.yaml
        ├── default-login.yaml
        ├── token-spray.yaml
        ├── dns-detect.yaml
        ├── ssl-check.yaml
        ├── network-service.yaml
        └── workflow.yaml
```

## Quick Start

### Creating a CVE Template

```yaml
id: CVE-2024-1234

info:
  name: Example App RCE
  author: your-username
  severity: critical
  description: |
    Remote code execution in Example App via unsafe deserialization.
  reference:
    - https://nvd.nist.gov/vuln/detail/CVE-2024-1234
  classification:
    cvss-metrics: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
    cvss-score: 9.8
    cve-id: CVE-2024-1234
    cwe-id: CWE-502
  tags: cve,cve2024,rce,example

variables:
  marker: "{{rand_base(8)}}"

http:
  - method: POST
    path:
      - "{{BaseURL}}/api/deserialize"
    body: '{"data":"{{base64(marker)}}"}'
    matchers:
      - type: word
        words:
          - "{{marker}}"
        part: body
```

### Detecting Misconfigurations

```yaml
id: app-config-exposure

info:
  name: App Configuration Exposure
  author: your-username
  severity: high
  tags: misconfig,exposure

http:
  - method: GET
    path:
      - "{{BaseURL}}/config.yml"
      - "{{BaseURL}}/config.json"
    stop-at-first-match: true
    matchers-condition: and
    matchers:
      - type: word
        words:
          - "database:"
          - "secret_key:"
        condition: or
      - type: status
        status:
          - 200
```

### SSRF Detection with OOB

```yaml
id: ssrf-oob-detect

info:
  name: SSRF via OOB Detection
  author: your-username
  severity: high
  tags: ssrf,oast

http:
  - method: GET
    path:
      - "{{BaseURL}}/fetch?url={{interactsh-url}}"
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "http"
```

## Documentation Coverage

### Protocols

| Protocol | Documentation | Example Templates |
|---|---|---|
| HTTP | [http-protocol.md](references/http-protocol.md) | CVE detection, misconfig, takeovers |
| DNS | [dns-protocol.md](references/dns-protocol.md) | CNAME detection, SPF/DMARC checks |
| SSL/TLS | [ssl-protocol.md](references/ssl-protocol.md) | Expired certs, weak ciphers |
| Network | [network-protocol.md](references/network-protocol.md) | Service detection, protocol probing |
| File | [file-protocol.md](references/file-protocol.md) | Secret scanning, malware detection |
| Headless | [headless-protocol.md](references/headless-protocol.md) | DOM XSS, browser-based detection |
| JavaScript | [javascript-protocol.md](references/javascript-protocol.md) | Custom protocol logic |
| Code | [code-protocol.md](references/code-protocol.md) | Shell/Python execution |
| DAST | [dast-protocol.md](references/dast-protocol.md) | SQLi, XSS, SSTI fuzzing |
| Cloud | [cloud-protocol.md](references/cloud-protocol.md) | AWS, GCP misconfigurations |

### Core Concepts

| Topic | Documentation |
|---|---|
| Matchers & Extractors | [matchers-extractors.md](references/matchers-extractors.md) |
| DSL Functions | [dsl-functions.md](references/dsl-functions.md) |
| Variables | [variables.md](references/variables.md) |
| Preprocessors | [preprocessors.md](references/preprocessors.md) |
| OOB Testing | [oob-testing.md](references/oob-testing.md) |
| Flow Control | [flow.md](references/flow.md) |
| Workflows | [workflows.md](references/workflows.md) |

## Features Covered

### HTTP Options

```yaml
http:
  - method: GET/POST/PUT/DELETE/PATCH
    path:
      - "{{BaseURL}}/endpoint"
    headers:
      Key: Value
    body: "request body"
    raw:
      - |
        GET / HTTP/1.1
        Host: {{Hostname}}
    redirects: true
    max-redirects: 5
    host-redirects: true          # Follow cross-host redirects
    cookie-reuse: true
    disable-cookie: false
    unsafe: true                  # Send raw without encoding
    skip-variables-check: true    # Skip unresolved variable validation
    stop-at-first-match: true
    matchers-condition: and/or
    payloads:
      name:
        - value1
        - value2
    attack: clusterbomb/pitchfork
    fuzzing:
      - part: query/body/path
        type: postfix/replace
```

### Matcher Types

```yaml
matchers:
  # Status code matching
  - type: status
    status:
      - 200
      - 302

  # Content length matching
  - type: size
    size:
      - 0
    part: body

  # String matching
  - type: word
    words:
      - "string1"
      - "string2"
    condition: and
    negative: true
    encoding: hex

  # Regex matching
  - type: regex
    regex:
      - "pattern"

  # Binary/hex matching
  - type: binary
    binary:
      - "504B0304"

  # DSL expression matching
  - type: dsl
    dsl:
      - 'status_code == 200'
      - 'contains(body, "text")'

  # XPath matching
  - type: xpath
    xpath:
      - "/html/head/title"
```

### Extractor Types

```yaml
extractors:
  # Regex extraction
  - type: regex
    part: body
    group: 1
    name: version
    internal: true
    regex:
      - "Version: ([0-9.]+)"

  # Key-value extraction (headers/cookies)
  - type: kval
    kval:
      - content_type
      - set_cookie

  # JSON extraction
  - type: json
    json:
      - ".data.id"
      - ".items[].name"

  # XPath extraction
  - type: xpath
    attribute: href
    xpath:
      - "/html/body//a"

  # DSL extraction
  - type: dsl
    dsl:
      - 'len(body)'
      - 'status_code'
```

### DSL Functions (60+)

| Category | Functions |
|---|---|
| **String** | `contains`, `contains_all`, `contains_any`, `starts_with`, `ends_with`, `join`, `concat`, `replace`, `replace_regex`, `trim`, `len`, `reverse`, `to_lower`, `to_upper`, `regex` |
| **Encoding** | `base64`, `base64_decode`, `url_encode`, `url_decode`, `hex_encode`, `hex_decode`, `html_escape`, `html_unescape`, `json_minify`, `json_prettify` |
| **Hash** | `md5`, `sha1`, `sha256`, `mmh3`, `hmac` |
| **Compression** | `gzip`, `gzip_decode`, `zlib`, `zlib_decode` |
| **Random** | `rand_base`, `rand_char`, `rand_int`, `rand_text_alpha`, `rand_text_alphanumeric`, `rand_text_numeric`, `rand_ip` |
| **DateTime** | `unix_time`, `to_unix_time`, `date_time` |
| **Crypto** | `aes_gcm`, `generate_jwt` |
| **Conversion** | `bin_to_dec`, `hex_to_dec`, `oct_to_dec`, `dec_to_hex` |
| **Utility** | `compare_versions`, `wait_for`, `ip_format` |

### Flow Control

```yaml
# Sequential execution
flow: http(1) && http(2)

# Cross-protocol
flow: tcp(1) && javascript(1)

# Iteration over extracted arrays
flow: |
  code(1)
  for(let item of iterate(template.items)){
    set("item_name", item)
    code(2)
  }
```

### Workflows

```yaml
workflows:
  # Simple template execution
  - template: http/technologies/tech-detect.yaml

  # Conditional with tags
  - template: http/technologies/jira-detect.yaml
    subtemplates:
      - tags: jira

  # Matcher name-based
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: wordpress
        subtemplates:
          - template: cves/CVE-2019-6715.yaml
```

## Validation Checklist

Before submitting a template:

- [ ] **ID** is unique, descriptive, lowercase kebab-case
- [ ] **Info block** has name, author, severity
- [ ] **Description** clearly explains what is detected
- [ ] **References** include advisory, NVD, and relevant links
- [ ] **Tags** are comprehensive and follow conventions
- [ ] **Matchers** use layered verification (identify → confirm → prove)
- [ ] **Template** detects vulnerability on vulnerable systems
- [ ] **Template** does NOT match on patched/non-vulnerable systems
- [ ] **YAML syntax** is valid (`nuclei -validate -t template.yaml`)

## Official Documentation

- [Nuclei Templates Introduction](https://docs.projectdiscovery.io/templates/introduction)
- [Template Creation Guide](https://docs.projectdiscovery.io/templates/)
- [Helper Functions](https://docs.projectdiscovery.io/templates/reference/helper-functions)
- [Matchers Reference](https://docs.projectdiscovery.io/templates/reference/matchers)
- [Extractors Reference](https://docs.projectdiscovery.io/templates/reference/extractors)

## Statistics

| Metric | Count |
|---|---|
| Total Files | 27 |
| Total Lines | ~5,100 |
| Protocol Docs | 10 |
| Matcher Types | 7 |
| Extractor Types | 5 |
| DSL Functions | 60+ |
| JS Functions | 35 |
| Example Templates | 9 |

## License

This skill is provided as-is for educational and security research purposes.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Validate with `nuclei -validate -t your-template.yaml`
5. Submit a pull request

---

**Built with ❤️ for the security community**
