---
name: nuclei-template-creator
description: >
  Create, validate, and optimize Nuclei security scanning templates.
  Use this skill whenever the user mentions nuclei templates, vulnerability
  detection templates, security scanning rules, CVE detection, DAST testing,
  or wants to create any kind of security scanning template for ProjectDiscovery
  Nuclei. Covers all protocol types: HTTP, DNS, SSL, TCP/Network, File,
  Headless, JavaScript, Code, DAST, and Cloud templates.
---

# Nuclei Template Creator

A comprehensive skill for creating high-quality Nuclei security scanning templates across all supported protocols and vulnerability types.

## When to Use

Use this skill when the user wants to:
- Create a new Nuclei template for any vulnerability type
- Convert a vulnerability advisory into a scanning template
- Improve or optimize an existing Nuclei template
- Understand Nuclei template syntax and best practices
- Validate a template's matchers for false positive reduction
- Create multi-protocol detection templates
- Build DAST fuzzing templates
- Create workflow templates that orchestrate multiple checks

## Quick Start Decision Tree

Before writing a template, determine the type:

```
What are you detecting?
├── Web vulnerability (HTTP) → references/http-protocol.md
├── DNS misconfiguration → references/dns-protocol.md
├── SSL/TLS issue → references/ssl-protocol.md
├── Network service → references/network-protocol.md
├── Secret/key in files → references/file-protocol.md
├── Browser-based vuln → references/headless-protocol.md
├── Custom protocol logic → references/javascript-protocol.md
├── Shell/Python check → references/code-protocol.md
├── Automated fuzzing → references/dast-protocol.md
└── Cloud misconfiguration → references/cloud-protocol.md
```

## Template Creation Workflow

### Step 1: Identify the Vulnerability

Before writing any template:
- Read the vulnerability advisory/disclosure completely
- Identify unique indicators that prove the vulnerability exists
- Determine affected versions and components
- Find the root cause and exploitation method
- Note any OOB (out-of-band) interaction needed

### Step 2: Choose Template Type and ID

```yaml
# CVE templates - use CVE ID directly
id: CVE-2024-1234

# Non-CVE templates - descriptive kebab-case
id: apache-struts-ognl-injection
id: airflow-config-exposure
id: azure-subdomain-takeover
```

**ID naming conventions:**
- Use lowercase kebab-case
- Be specific: `apache-struts-rce` not `rce-vuln`
- CVE templates use the CVE ID as the ID
- Include product name when possible

### Step 3: Write the Info Block

The `info` block is required and must include:

```yaml
info:
  name: Vendor Product - Vulnerability Type    # Required
  author: your-github-username                 # Required
  severity: critical                           # Required: critical|high|medium|low|info|unknown
  
  description: |                               # Strongly recommended
    Clear explanation of what this template detects.
    Include affected versions and root cause.
  
  impact: |                                    # Optional but helpful
    What an attacker can achieve by exploiting this.
  
  remediation: |                               # Optional but helpful
    How to fix or mitigate the vulnerability.
  
  reference:                                   # Strongly recommended
    - https://advisory-url.com
    - https://nvd.nist.gov/vuln/detail/CVE-XXXX
    - https://github.com/advisory
  
  classification:                              # Recommended for CVEs
    cvss-metrics: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
    cvss-score: 9.8
    cve-id: CVE-2024-1234
    cwe-id: CWE-89
    epss-score: 0.95
    epss-percentile: 0.99
    cpe: cpe:2.3:a:vendor:product:1.0:*:*:*:*:*:*:*
  
  metadata:
    verified: true                             # Only if you tested it
    max-request: 3                             # Number of requests (auto-calculated)
    vendor: vendor-name
    product: product-name
    shodan-query: 'http.title:"Product"'
    fofa-query: 'body="product" && title="Product"'
    google-query: 'inurl:/vulnerable/endpoint'
  
  tags: cve,cve2024,rce,product,technology    # Comma-separated, be comprehensive
```

**Severity guidelines:**
- `critical`: RCE, auth bypass, full system compromise
- `high`: SQLi, SSRF with impact, significant data exposure
- `medium`: XSS, CSRF, limited info disclosure, misconfigurations
- `low`: Minor info disclosure, version disclosure, best practice violations
- `info`: Technology detection, information gathering, fingerprinting

**Tag conventions:**
- Protocol: `cve`, `dns`, `ssl`, `tcp`, `network`, `js`, `headless`, `dast`, `file`, `cloud`, `code`
- Vuln class: `rce`, `xss`, `sqli`, `ssrf`, `lfi`, `ssti`, `takeover`, `misconfig`, `exposure`, `unauth`, `redirect`, `traversal`, `disclosure`
- Technology: `wordpress`, `wp-plugin`, `apache`, `tomcat`, `spring`, `django`, `laravel`
- Discovery: `discovery`, `detect`, `tech`, `waf`, `honeypot`, `osint`
- KEV: `kev` (if in CISA KEV catalog)

### Step 4: Write the Protocol Section

Choose the appropriate protocol and read its reference:
- **HTTP**: Most common. Read `references/http-protocol.md` for full syntax.
- **DNS**: For DNS record checks. Read `references/dns-protocol.md`.
- **SSL/TLS**: For certificate/TLS checks. Read `references/ssl-protocol.md`.
- **Network/TCP**: For raw TCP services. Read `references/network-protocol.md`.
- **File**: For scanning local files. Read `references/file-protocol.md`.
- **Headless**: For browser-based detection. Read `references/headless-protocol.md`.
- **JavaScript**: For custom protocol logic. Read `references/javascript-protocol.md`.
- **Code**: For shell/Python execution. Read `references/code-protocol.md`.
- **DAST**: For automated fuzzing. Read `references/dast-protocol.md`.
- **Cloud**: For cloud misconfigs. Read `references/cloud-protocol.md`.

### Step 5: Write Matchers (Critical for Quality)

Matchers determine if a vulnerability exists. Poor matchers cause false positives.

**Golden rules:**
1. Use multiple matchers with `matchers-condition: and` for specificity
2. Layer your matchers: identify app → confirm version → prove vulnerability
3. Avoid generic words like "error", "admin", "login"
4. Use unique exploitation markers (random strings, specific error patterns)
5. Combine word, regex, status, and DSL matchers

**Read `references/matchers-extractors.md` for complete matcher syntax and patterns.**

### Step 6: Test Your Template

```bash
# Validate syntax
nuclei -validate -t your-template.yaml

# Test against vulnerable target
nuclei -t your-template.yaml -target http://vulnerable-app.local -debug

# Test against patched/non-vulnerable target (should NOT match)
nuclei -t your-template.yaml -target http://patched-app.local -debug

# Test against similar but different apps (should NOT match)
nuclei -t your-template.yaml -target http://different-app.local -debug
```

## Common Vulnerability Patterns

### SQL Injection
```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/search"
    body: "q={{payload}}"
    payloads:
      payload:
        - "' OR '1'='1"
        - "' UNION SELECT version()--"
    matchers:
      - type: word
        words:
          - "You have an error in your SQL syntax"
          - "Microsoft OLE DB Provider for ODBC"
        part: body
```

### Remote Code Execution
```yaml
variables:
  marker: "{{rand_base(8)}}"
http:
  - method: POST
    path:
      - "{{BaseURL}}/execute"
    body: "command=echo {{marker}}"
    matchers:
      - type: word
        words:
          - "{{marker}}"
        part: body
```

### Local File Inclusion
```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/view?file=../../../etc/passwd"
      - "{{BaseURL}}/view?file=..\\..\\..\\windows\\win.ini"
    matchers:
      - type: regex
        regex:
          - 'root:.*?:[0-9]*:[0-9]*:'
          - '\[fonts\]'
        part: body
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
```

### Subdomain Takeover
```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/"
    matchers-condition: and
    matchers:
      - type: word
        words:
          - "There is no app configured at that hostname"
          - "NoSuchBucket"
        part: body
        condition: or
      - type: status
        status:
          - 404
          - 403
```

### XSS Detection
```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/search?q={{payload}}"
    payloads:
      payload:
        - '<script>alert(document.domain)</script>'
    matchers-condition: and
    matchers:
      - type: word
        words:
          - '<script>alert(document.domain)</script>'
        part: body
      - type: word
        words:
          - "text/html"
        part: header
```

## Validation Checklist

Before finalizing a template:

- [ ] **ID** is unique, descriptive, lowercase kebab-case
- [ ] **Info block** has all required fields (name, author, severity)
- [ ] **Description** clearly explains what is detected
- [ ] **References** include advisory, NVD, and relevant links
- [ ] **Tags** are comprehensive and follow conventions
- [ ] **Classification** includes CVSS, CVE, CWE where applicable
- [ ] **Matchers** are specific enough to avoid false positives
- [ ] **Matchers** use layered verification (identify → confirm → prove)
- [ ] **Template** detects vulnerability on vulnerable systems
- [ ] **Template** does NOT match on patched/non-vulnerable systems
- [ ] **Template** does NOT match on similar but different applications
- [ ] **YAML syntax** is valid (`nuclei -validate -t template.yaml`)
- [ ] **Random markers** used for RCE/injection proof
- [ ] **OOB interaction** used where blind exploitation is possible

> **Note:** Every template ends with a `# digest:` comment which is auto-generated by Nuclei for integrity verification. Do not manually add or modify this field.

## Reference Files

Read these files as needed for detailed protocol syntax:

- `references/http-protocol.md` - HTTP template complete syntax
- `references/dns-protocol.md` - DNS template syntax
- `references/ssl-protocol.md` - SSL/TLS template syntax
- `references/network-protocol.md` - TCP/Network template syntax
- `references/file-protocol.md` - File scanning template syntax
- `references/headless-protocol.md` - Headless browser template syntax
- `references/javascript-protocol.md` - JavaScript template syntax
- `references/code-protocol.md` - Code execution template syntax
- `references/dast-protocol.md` - DAST fuzzing template syntax
- `references/cloud-protocol.md` - Cloud template syntax
- `references/matchers-extractors.md` - Matchers and extractors reference
- `references/dsl-functions.md` - DSL functions and variables reference
- `references/variables.md` - Built-in variables reference
- `references/preprocessors.md` - Preprocessors reference (randstr)
- `references/oob-testing.md` - OOB testing reference (interactsh)
- `references/workflows.md` - Workflow template syntax
- `references/flow.md` - Flow control syntax (sequential, iteration, cross-protocol)
- `references/examples/` - Real-world template examples by category
