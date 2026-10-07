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
| `engine` | list | Script interpreters to use, searched in order until one is found |
| `source` | string | Script source code, or a file path (e.g. `helpers/code/pyfile.py`) |
| `args` | list | Arguments passed to the engine (e.g. `-ExecutionPolicy, Bypass, -File` for pwsh) |
| `pattern` | string | Glob for the temp file name/extension of the snippet (e.g. `"*.ps1"`) |
| `matchers` | list | Matching rules |
| `extractors` | list | Data extraction rules |

**Important:**
- Code templates are NOT executed by default — run nuclei with the `-code` flag to enable the code protocol.
- Templates typically use `self-contained: true` since they don't need a target URL; with a target, the target is passed to the script via **stdin**.
- Matcher/extractor parts: `response` (stdout, trailing whitespace filtered) and `stderr`.
- In multi-protocol templates, previous code outputs are available as `{{code_1_response}}`, `{{code_2_response}}`, ...

---

## Engines

| Engine | Description |
|---|---|
| `sh` / `bash` | Shell script |
| `py` / `python` / `python3` | Python script (preinstalled on macOS & most Linux distros) |
| `go` | Go program |
| `ps` / `pwsh` / `powershell` / `powershell.exe` | PowerShell (may need separate install) |
| `ruby` / `perl` / `node` / `lua` / `java` (JShell) | Other interpreters |

Engines are tried in the order listed; the first one found on the system runs the snippet.

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
