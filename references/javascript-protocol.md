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
  - init: |              # optional: runs ONCE after compile, before any target
      let m = require('nuclei/fs');
      updatePayload('keys', m.ReadFilesFromDir(keysDir));
    pre-condition: |     # optional: runs per target; code runs only if it returns true
      isPortOpen(Host, Port);
    code: |              # main code; value of last expression is the output
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

On error, an `error` variable is exposed to matchers/extractors with the error message.

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

Nuclei v3 ships 15+ libraries tailored for exploits (`ssh`, `ftp`, `RDP`, `Kerberos`, `Redis`, ...), all imported with `require("nuclei/<lib>")`. Documented modules:

### nuclei/ssh
```javascript
var m = require("nuclei/ssh");
var c = m.SSHClient();
var response = c.ConnectSSHInfoMode(Host, Port);  // banner / auth methods, no credentials
to_json(response);
```

### nuclei/bytes
Byte buffer construction for raw protocol packets (e.g. CVE-2020-0796 style handcrafted packets):
```javascript
var bytes = require("nuclei/bytes");
var b = new bytes.Buffer();   // WriteString / WriteByte / Bytes() ...
```

### nuclei/fs
Local file access (use in `init` to preload data once, not per target):
```javascript
var fs = require("nuclei/fs");
fs.ListDir(path, 'file'|'dir'|'');        // string[] | null
fs.ReadFile(path); fs.ReadFileAsString(path);
fs.ReadFilesFromDir(dir);                 // all file contents in a dir
```

### nuclei/vnc
```javascript
var vnc = require("nuclei/vnc");
var resp = vnc.IsVNC(Host, Port);   // { IsVNC: bool, Banner: string } | null
```

### nuclei/net
Network operations:
```javascript
var m = require("nuclei/net");
var c = m.NewTCPClient(Host, Port);   // also NewTLSClient
c.Send("data");
var resp = c.RecvString(1024);
```

### nuclei/mssql / mysql / postgres (and more)
```javascript
var c = require("nuclei/mssql").MSSQLClient();
c.IsMssql(Host, Port);   // same pattern: .IsMySQL / .IsPostgres
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
