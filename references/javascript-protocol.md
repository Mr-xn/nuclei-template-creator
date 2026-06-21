# JavaScript Protocol Reference

Complete syntax reference for Nuclei JavaScript templates.

## Table of Contents
- [Basic Structure](#basic-structure)
- [Configuration Options](#configuration-options)
- [JavaScript Runtime Functions](#javascript-runtime-functions)
- [Template Flow Functions](#template-flow-functions)
- [Code Protocol OS Detection](#code-protocol-os-detection)
- [Code Protocol Architecture Detection](#code-protocol-architecture-detection)
- [Available Libraries](#available-libraries)
- [Examples](#examples)

---

## Basic Structure

```yaml
javascript:
  - pre-condition: |
      isPortOpen(Host, Port);
    code: |
      let packet = bytes.NewBuffer();
      // ... JavaScript code ...
      Export("result");
    args:
      Host: "{{Host}}"
      Port: 8080
    matchers:
      - type: dsl
        dsl:
          - response
```

---

## Configuration Options

| Option | Type | Description |
|---|---|---|
| `pre-condition` | string | JavaScript code to run before main code (port checks) |
| `code` | string | Main JavaScript code to execute |
| `args` | map | Arguments passed to the script |

---

## JavaScript Runtime Functions

| Function | Signature | Description |
|---|---|---|
| `atob` | `atob(string) string` | Base64 decodes a given string |
| `btoa` | `btoa(string) string` | Base64 encodes a given string |
| `to_json` | `to_json(any) object` | Converts a given object to JSON |
| `dump_json` | `dump_json(any)` | Prints a given object as JSON in console |
| `to_array` | `to_array(any) array` | Sets/Updates objects prototype to array |
| `hex_to_ascii` | `hex_to_ascii(string) string` | Converts hex string to ascii |
| `Rand` | `Rand(n int) []byte` | Returns random byte slice of length n |
| `RandInt` | `RandInt() int` | Returns a random int |
| `log` | `log(msg string)` | Prints input to stdout with `[JS]` prefix for debugging |
| `getNetworkPort` | `getNetworkPort(port, defaultPort) string` | Registers defaultPort and returns it |
| `isPortOpen` | `isPortOpen(host, port, [timeout]) bool` | Checks if TCP port is open (timeout default: 5s) |
| `isUDPPortOpen` | `isUDPPortOpen(host, port, [timeout]) bool` | Checks if UDP port is open (timeout default: 5s) |
| `ToBytes` | `ToBytes(...interface{}) []byte` | Converts input to byte slice |
| `ToString` | `ToString(...interface{}) string` | Converts input to string |
| `Export` | `Export(value any)` | Appends value to script output |
| `ExportAs` | `ExportAs(key, value)` | Exports value with key for DSL/response |

---

## Template Flow Functions

| Function | Signature | Description |
|---|---|---|
| `log` | `log(obj any) any` | Logs object/message to stdout (debugging) |
| `iterate` | `iterate(...any) []any` | Normalizes and iterates over arguments, returns array |
| `Dedupe` | `new Dedupe()` | De-duplicates values, returns unique array |

---

## Code Protocol OS Detection

| Function | Signature | Description |
|---|---|---|
| `OS` | `OS() string` | Returns current OS |
| `IsLinux` | `IsLinux() bool` | Checks if OS is Linux |
| `IsWindows` | `IsWindows() bool` | Checks if OS is Windows |
| `IsOSX` | `IsOSX() bool` | Checks if OS is OSX |
| `IsAndroid` | `IsAndroid() bool` | Checks if OS is Android |
| `IsIOS` | `IsIOS() bool` | Checks if OS is IOS |
| `IsJS` | `IsJS() bool` | Checks if OS is JS |
| `IsFreeBSD` | `IsFreeBSD() bool` | Checks if OS is FreeBSD |
| `IsOpenBSD` | `IsOpenBSD() bool` | Checks if OS is OpenBSD |
| `IsSolaris` | `IsSolaris() bool` | Checks if OS is Solaris |

---

## Code Protocol Architecture Detection

| Function | Signature | Description |
|---|---|---|
| `Arch` | `Arch() string` | Returns current architecture |
| `Is386` | `Is386() bool` | Checks if architecture is 386 |
| `IsAmd64` | `IsAmd64() bool` | Checks if architecture is Amd64 |
| `IsARM` | `IsARM() bool` | Checks if architecture is ARM |
| `IsARM64` | `IsARM64() bool` | Checks if architecture is ARM64 |
| `IsWasm` | `IsWasm() bool` | Checks if architecture is Wasm |

---

## JavaScript Protocol Functions

| Function | Signature | Description |
|---|---|---|
| `set` | `set(string, interface{})` | Set variable from init code block only |
| `updatePayload` | `updatePayload(string, interface{})` | Update/override payload from init code block only |

---

## Available Libraries

### nuclei/net
Network operations:
```javascript
var m = require("nuclei/net");
var c = m.NewTCPClient(Host, Port);
c.Send("data");
var resp = c.RecvString(1024);
```

### nuclei/mssql
MSSQL operations:
```javascript
var m = require("nuclei/mssql");
var c = m.MSSQLClient();
c.IsMssql(Host, Port);
```

### nuclei/mysql
MySQL operations:
```javascript
var m = require("nuclei/mysql");
var c = m.MySQLClient();
c.IsMySQL(Host, Port);
```

### nuclei/postgres
PostgreSQL operations:
```javascript
var p = require("nuclei/postgres");
var c = p.PostgresClient();
c.IsPostgres(Host, Port);
```

---

## Examples

### Port Detection with Pre-condition
```yaml
javascript:
  - pre-condition: |
      isPortOpen(Host, Port);
    code: |
      let packet = bytes.NewBuffer();
      packet.WriteString("VERSION\r\n");
      let c = require("nuclei/net");
      let client = c.NewTCPClient(Host, Port);
      client.Send(packet.Bytes());
      let resp = client.RecvString(1024);
      Export("Detected: " + resp);
    args:
      Host: "{{Host}}"
      Port: 61616
    matchers:
      - type: dsl
        dsl:
          - response != ""
```

### MSSQL Detection
```yaml
javascript:
  - code: |
      var m = require("nuclei/mssql");
      var c = m.MSSQLClient();
      c.IsMssql(Host, Port);
    args:
      Host: "{{Host}}"
      Port: "1433"
    matchers:
      - type: dsl
        dsl:
          - "response == true"
          - "success == true"
        condition: and
```

### Custom Protocol Detection
```yaml
javascript:
  - pre-condition: |
      isPortOpen(Host, Port);
    code: |
      var c = require("nuclei/net");
      var client = c.NewTLSClient(Host, Port);
      client.Send("HELLO\r\n");
      var response = client.RecvString(1024);
      Export(response);
    args:
      Host: "{{Host}}"
      Port: 993
    extractors:
      - type: dsl
        dsl:
          - response
```
