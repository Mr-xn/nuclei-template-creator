# Headless Protocol Reference

Complete syntax reference for Nuclei Headless browser templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Actions](#actions)
- [Configuration Options](#configuration-options)
- [Examples](#examples)

---

## Basic Structure

```yaml
headless:
  - steps:
      - action: navigate
        args:
          url: "{{BaseURL}}"
      - action: waitload
      - action: script
        name: extract1
        args:
          code: |
            () => { return document.title }
    matchers:
      - type: word
        part: extract1
        words:
          - "Vulnerable Page"
```

---

## Actions

### navigate
Navigate to a URL:
```yaml
- action: navigate
  args:
    url: "{{BaseURL}}"
```

### waitload
Wait for page to load:
```yaml
- action: waitload
```

### waitstable
Wait for DOM to stabilize:
```yaml
- action: waitstable
  args:
    duration: 5s
```

### screenshot
Take a screenshot:
```yaml
- action: screenshot
  args:
    fullpage: "true"
    mkdir: "true"
    to: "{{dir}}/{{filename}}"
```

### script
Execute JavaScript:
```yaml
- action: script
  name: extract1
  args:
    code: |
      () => { return document.title }
```

### click
Click an element:
```yaml
- action: click
  args:
    by: selector
    selector: "#button"
    timeout: 5
```

### text
Extract text from element:
```yaml
- action: text
  name: extracted_text
  args:
    by: selector
    selector: ".content"
```

### setheader
Set request headers:
```yaml
- action: setheader
  args:
    part: request
    key: "User-Agent"
    value: "Mozilla/5.0..."
```

### addheader
Add request headers:
```yaml
- action: addheader
  args:
    part: request
    key: "X-Custom"
    value: "value"
```

### setbody
Set request body:
```yaml
- action: setbody
  args:
    part: request
    body: "custom body"
```

### keyboard
Simulate keyboard input:
```yaml
- action: keyboard
  args:
    keys: "test input"
```

### debug
Debug action for troubleshooting:
```yaml
- action: debug
```

---

## Configuration Options

| Option | Type | Description |
|---|---|---|
| `steps` | list | Sequence of browser actions |
| `args` | map | Action-specific arguments |
| `name` | string | Variable name for extracted data (script/text actions) |

---

## Examples

### Open Redirect Detection
```yaml
headless:
  - steps:
      - action: navigate
        args:
          url: "{{BaseURL}}/redirect?url=https://evil.com"
      - action: waitload
      - action: script
        name: current_url
        args:
          code: |
            () => { return document.location.href }
    matchers:
      - type: word
        part: current_url
        words:
          - "evil.com"
```

### Prototype Pollution Check
```yaml
variables:
  key: "{{rand_base(6)}}"
  value: "{{rand_base(6)}}"

headless:
  - steps:
      - action: navigate
        args:
          url: "{{BaseURL}}"
      - action: waitload
      - action: script
        args:
          code: |
            () => {
              let url = new URL(window.location.href);
              url.searchParams.set('__proto__.{{key}}', '{{value}}');
              window.location = url.href;
            }
      - action: waitload
      - action: script
        name: extract1
        args:
          code: |
            () => { return window.vulnerableprop || "" }
    matchers:
      - type: word
        part: extract1
        words:
          - "{{value}}"
```

### Cookie Consent Detection
```yaml
headless:
  - steps:
      - action: setheader
        args:
          part: request
          key: "User-Agent"
          value: "Mozilla/5.0..."
      - action: navigate
        args:
          url: "{{BaseURL}}"
      - action: waitload
      - action: script
        name: cookie_banner
        args:
          code: |
            () => {
              const selectors = [
                '[class*="cookie"]', '[id*="cookie"]',
                '[class*="consent"]', '[id*="consent"]',
                '[class*="gdpr"]', '[id*="gdpr"]'
              ];
              for (const sel of selectors) {
                if (document.querySelector(sel)) return "found";
              }
              return "not_found";
            }
    matchers:
      - type: word
        part: cookie_banner
        words:
          - "found"
```

### XSS via DOM
```yaml
headless:
  - steps:
      - action: navigate
        args:
          url: "{{BaseURL}}/#<img src=x onerror=alert(1)>"
      - action: waitload
      - action: script
        name: xss_check
        args:
          code: |
            () => {
              let body = document.body.innerHTML;
              if (body.includes('onerror=alert(1)')) return "vulnerable";
              return "safe";
            }
    matchers:
      - type: word
        part: xss_check
        words:
          - "vulnerable"
```
