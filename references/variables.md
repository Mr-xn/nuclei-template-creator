# Built-in Variables Reference

Complete reference for Nuclei template built-in variables.

## Table of Contents
- [Variable Basics](#variable-basics)
- [URL Variables](#url-variables)
- [Target Variables](#target-variables)
- [Interaction Variables](#interaction-variables)
- [Random Variables](#random-variables)
- [CLI Variables](#cli-variables)
- [Template Variables](#template-variables)

---

## Variable Basics

### Declaration
Variables are declared at template level under `variables:` key:
```yaml
variables:
  a1: "test"                          # Simple string
  a2: "{{to_lower(rand_base(5))}}"    # DSL function
```

### Supported Protocols
Variables are supported in: `dns`, `http`, `headless`, `network`

### Key Properties
- **Immutable**: Value is calculated once and does not change
- **DSL syntax**: Helper functions enclosed in `{{...}}`
- **Usable in**: Raw requests, host fields, inputs, matchers, extractors

---

## URL Variables

For target `https://example.com:443/foo/bar.php`:

| Variable | Value | Description |
|---|---|---|
| `{{BaseURL}}` | `https://example.com:443/foo/bar.php` | Full input URL |
| `{{RootURL}}` | `https://example.com:443` | Root URL (scheme + host + port) |
| `{{Hostname}}` | `example.com:443` | Hostname including port |
| `{{Host}}` | `example.com` | Host without port |
| `{{Port}}` | `443` | Port number |
| `{{Path}}` | `/foo` | URL path |
| `{{File}}` | `bar.php` | Filename |
| `{{Scheme}}` | `https` | Protocol scheme |

---

## Target Variables

| Variable | Description |
|---|---|
| `{{FQDN}}` | Fully qualified domain name (e.g., `httpbin.org`) |
| `{{IP}}` | Target IP address |

---

## Interaction Variables

For out-of-band (OOB) testing:

| Variable | Description |
|---|---|
| `{{interactsh-url}}` | Generate unique OOB interaction URL |
| `{{interactsh-domain}}` | OOB interaction domain |

Used in matchers:
```yaml
matchers:
  - type: word
    part: interactsh_protocol
    words:
      - "dns"  # or "http"
```

---

## Random Variables

| Variable | Description |
|---|---|
| `{{randstr}}` | Random string |
| `{{rand_base(N)}}` | Random string of length N |
| `{{rand_int(min, max)}}` | Random integer in range |
| `{{rand_text_alpha(N)}}` | Random alphabetic string |
| `{{rand_text_alphanumeric(N)}}` | Random alphanumeric string |
| `{{rand_text_numeric(N)}}` | Random numeric string |

---

## CLI Variables

Variables provided via command line with `-var` flag:

```bash
nuclei -t template.yaml -var username=admin -var password=test
```

In template:
```yaml
variables:
  username: "{{username}}"
  password: "{{password}}"
```

---

## Template Variables

Variables defined in the template's `variables:` block:

```yaml
variables:
  username: "admin"
  payload: "{{rand_base(8)}}"
  region: "us-east-1"
  OAST: "{{interactsh-url}}"
  first: "{{rand_int(10000, 99999)}}"
  filename: '{{replace(BaseURL,"/","_")}}'
```

Template variables can reference built-in variables and other template variables.

---

## Variable Precedence

1. CLI variables (`-var` flag) - highest priority
2. Template variables (`variables:` block)
3. Built-in variables - lowest priority

CLI variables override template variables of the same name.
