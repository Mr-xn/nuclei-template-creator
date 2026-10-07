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
Navigate to a URL (supports `{{BaseURL}}`, `{{Hostname}}`, etc.):
```yaml
- action: navigate
  args:
    url: "{{BaseURL}}"
```

### script
Run JavaScript on the current page. `code` STRICTLY requires a function reference — a direct expression will NOT work:
```yaml
- action: script
  name: extract1              # optional: store return value, matchable via part: extract1
  args:
    code: |
      () => { return document.title }   # ✅ function reference
      # alert(document.domain)          # ❌ NOT a function reference
```
Run JS before any page loads with `hook: true` (e.g. neuter window.alert so dialogs don't block the flow):
```yaml
- action: script
  args:
    code: () => (function() { window.alert=function(){} })()
    hook: true
```

### click / rightclick
Click (left / right mouse button) an element located by a selector:
```yaml
- action: click
  args:
    by: xpath
    xpath: /html/body/div[1]/div[3]/form/div[2]/div[1]/div[1]/div/div[2]/input
- action: rightclick
  args:
    by: selector
    selector: "#button"
```

### text
TYPE text into an input element with the keyboard (NOT text extraction — use `extract` for that):
```yaml
- action: text
  args:
    by: xpath
    xpath: /html/body/div/div[2]/form/fieldset/input
    value: admin
```

### keyboard
Simulate a single key press (`keys` accepts key-codes):
```yaml
- action: keyboard
  args:
    keys: '\r'    # Enter
```

### time / select / files
Fill time inputs (RFC3339), select options, handle file uploads:
```yaml
- action: time
  args:
    by: xpath
    xpath: //input[@type="time"]
    value: 2006-01-02T15:04:05Z07:00
- action: select
  args:
    by: xpath
    xpath: //select
    selected: true
    value: option[value=two]
- action: files
  args:
    by: xpath
    xpath: //input[@type="file"]
    value: /root/test/payload.txt
```

### screenshot
Take a screenshot (add `fullpage: true` for full-page):
```yaml
- action: screenshot
  args:
    to: "{{dir}}/{{filename}}"
    fullpage: "true"
    mkdir: "true"
```

### wait actions
| Action | Waits for |
|---|---|
| `waitfcp` | First Contentful Paint |
| `waitfmp` | First Meaningful Paint |
| `waitdom` | DOMContentLoaded (HTML parsed, no subresources) |
| `waitload` | Full page load (stylesheets, images) |
| `waitidle` | Network idle (no more requests) |
| `waitstable` | Page stable for N duration (default 1s): `args: {duration: 5s}` |

### waitdialog
Wait for a JS dialog (`alert`/`confirm`/`prompt`/`onbeforeunload`) and auto-accept it — accurate XSS detection with low FP. `name` is REQUIRED to expose output variables:
```yaml
- action: waitdialog
  name: alert
  args:
    max-duration: 5s   # default 10s
```
Output variables: `NAME` (bool, dialog triggered), `NAME_type` (dialog type), `NAME_message` (displayed message).

### waitevent
Wait for a page event (full list: go-rod proto definitions):
```yaml
- action: waitevent
  args:
    event: 'Page.loadEventFired'
```

### extract
Extract the text of an element (or one of its attributes) into a named variable:
```yaml
- action: extract
  name: extracted-value
  args:
    by: xpath
    xpath: /html/body/div/p[2]/a
    # target: attribute      # to extract an attribute instead of text:
    # attribute: href
```

### getresource
Return the `src` attribute of an element:
```yaml
- action: getresource
  name: extracted-value-src
  args:
    by: xpath
    xpath: //img[1]
```

### Header / body / method manipulation
```yaml
- action: setmethod          # override request method
  args: {part: request, method: DELETE}
- action: addheader          # add (does NOT overwrite existing)
  args: {part: response, key: X-Custom, value: "v"}   # part: request|response
- action: setheader          # set (overwrites)
  args: {part: request, key: User-Agent, value: "Mozilla/5.0..."}
- action: deleteheader
  args: {part: response, key: Content-Security-Policy}
- action: setbody
  args: {part: response, body: '{"success":"ok"}'}
```

### sleep / debug
```yaml
- action: sleep
  args: {duration: 5}
- action: debug   # 5s delay between actions + trace of all headless events (debugging only)
```

---

## Selectors

| Selector (`by:`) | Description |
|---|---|
| `selector` (default) | CSS selector |
| `x` / `xpath` | XPath selector |
| `r` / `regex` | CSS selector whose text matches regex |
| `js` | Return elements from a JS function |
| `search` | Search query (text, XPATH, or CSS) |

---

## Matchers / Extractor Parts

| Part | Description |
|---|---|
| `request` | Headless request |
| `<out_names>` | Action names with stored values (from `name:`) |
| `raw` / `body` / `data` | Final DOM response from the browser |

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
