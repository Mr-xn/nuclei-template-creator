# Preprocessors Reference

Complete reference for Nuclei template preprocessors.

## Overview

Pre-processors can be specified globally anywhere in a Nuclei template. They run as soon as the template is loaded to perform tasks like generating random IDs for each template run.

---

## randstr

Generates a random ID (based on the [xid](https://github.com/rs/xid) library) for a template on each nuclei run.

### Key Properties

- **Consistent within run**: Value remains the same throughout a single template execution
- **Multiple IDs**: Can be suffixed with numbers (`randstr_1`, `randstr_2`, etc.) to create multiple distinct random IDs
- **Usable in matchers**: Can be used to match the injected value in responses

### Syntax

```yaml
# Basic usage
{{randstr}}

# Multiple distinct random IDs
{{randstr_1}}
{{randstr_2}}
{{randstr_3}}
```

### Example: RCE Detection with Random Marker

```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/level1/application/"
    headers:
      cmd: "echo '{{randstr}}'"
    matchers:
      - type: word
        words:
          - '{{randstr}}'
```

### Example: Multiple Random Markers

```yaml
http:
  - method: POST
    path:
      - "{{BaseURL}}/api/execute"
    body: |
      cmd1=echo {{randstr_1}}&cmd2=echo {{randstr_2}}
    matchers-condition: and
    matchers:
      - type: word
        words:
          - '{{randstr_1}}'
        part: body
      - type: word
        words:
          - '{{randstr_2}}'
        part: body
```

---

## Summary

| Preprocessor | Description |
|---|---|
| `{{randstr}}` | Random ID, consistent within template run |
| `{{randstr_N}}` | Multiple distinct random IDs (N = 1, 2, 3...) |
