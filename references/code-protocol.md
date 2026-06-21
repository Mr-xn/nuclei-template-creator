# Code Protocol Reference

Complete syntax reference for Nuclei Code execution templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Configuration Options](#configuration-options)
- [Engines](#engines)
- [Examples](#examples)

---

## Basic Structure

```yaml
self-contained: true
code:
  - engine:
      - sh
      - bash
    source: |
      echo "hello world"
    matchers:
      - type: word
        words:
          - "hello"
```

---

## Configuration Options

| Option | Type | Description |
|---|---|---|
| `engine` | list | Script interpreters to use |
| `source` | string | Script source code |
| `matchers` | list | Matching rules |
| `extractors` | list | Data extraction rules |

**Important:** Code templates typically use `self-contained: true` since they don't need a target URL.

---

## Engines

| Engine | Description |
|---|---|
| `sh` | Shell script |
| `bash` | Bash script |
| `python` / `python3` | Python script |
| `ruby` | Ruby script |
| `perl` | Perl script |
| `node` | Node.js script |
| `lua` | Lua script |
| `java` | Java (via JShell) |

Variables from the `variables:` block are available as environment variables in shell scripts, or via `os.getenv()` in Python.

---

## Examples

### AWS Credential Check
```yaml
self-contained: true
variables:
  region: "us-east-1"
code:
  - engine:
      - sh
      - bash
    source: |
      aws sts get-caller-identity --region {{region}} 2>&1
    matchers:
      - type: word
        words:
          - "Account"
          - "Arn"
        condition: and
    extractors:
      - type: json
        json:
          - ".Account"
          - ".Arn"
```

### Python Environment Check
```yaml
self-contained: true
code:
  - engine:
      - python
      - python3
    source: |
      import os
      import json
      result = {
        "user": os.getenv("USER", "unknown"),
        "home": os.getenv("HOME", "unknown"),
        "path": os.getenv("PATH", "unknown")
      }
      print(json.dumps(result))
    matchers:
      - type: word
        words:
          - "user"
```

### System Info Gathering
```yaml
self-contained: true
code:
  - engine:
      - sh
      - bash
    source: |
      uname -a
      whoami
      id
    matchers:
      - type: word
        words:
          - "root"
          - "uid=0"
        condition: or
```

### CVE Check via Script
```yaml
self-contained: true
variables:
  version_file: "/etc/app/version"
code:
  - engine:
      - sh
      - bash
    source: |
      if [ -f "{{version_file}}" ]; then
        cat "{{version_file}}"
      else
        echo "file not found"
      fi
    matchers:
      - type: regex
        regex:
          - "v[0-9]+\\.[0-9]+\\.[0-9]+"
```
