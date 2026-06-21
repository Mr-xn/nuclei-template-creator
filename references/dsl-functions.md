# DSL Functions and Variables Reference

Complete reference for Nuclei's Domain Specific Language (DSL) used in matchers and extractors. Based on official ProjectDiscovery documentation.

## Table of Contents
- [Comparison Operators](#comparison-operators)
- [Logical Operators](#logical-operators)
- [String Functions](#string-functions)
- [Encoding/Decoding Functions](#encodingdecoding-functions)
- [Hash Functions](#hash-functions)
- [Compression Functions](#compression-functions)
- [Random Generation Functions](#random-generation-functions)
- [Number/Conversion Functions](#numberconversion-functions)
- [Date/Time Functions](#datetime-functions)
- [Crypto Functions](#crypto-functions)
- [Deserialization Functions](#deserialization-functions)
- [Network Functions](#network-functions)
- [Utility Functions](#utility-functions)
- [Response Variables](#response-variables)
- [SSL Variables](#ssl-variables)

---

## Comparison Operators

| Operator | Description | Example |
|---|---|---|
| `==` | Equal | `status_code == 200` |
| `!=` | Not equal | `status_code != 404` |
| `<` | Less than | `len(body) < 1000` |
| `>` | Greater than | `len(body) > 100` |
| `<=` | Less or equal | `status_code <= 299` |
| `>=` | Greater or equal | `status_code >= 200` |

---

## Logical Operators

| Operator | Description | Example |
|---|---|---|
| `&&` | AND | `status_code == 200 && contains(body, "ok")` |
| `\|\|` | OR | `status_code == 301 \|\| status_code == 302` |
| `!` | NOT | `!contains(body, "error")` |

---

## String Functions

### contains
Verifies if a string contains a substring.
```yaml
- 'contains(body, "vulnerable")'
# Signature: contains(input, substring interface{}) bool
```

### contains_all
Verifies if input contains ALL of the substrings.
```yaml
- 'contains_all(body, "admin", "dashboard")'
# Signature: contains_all(input interface{}, substrings ...string) bool
```

### contains_any
Verifies if input contains ANY of the substrings.
```yaml
- 'contains_any(body, "error", "exception", "failed")'
# Signature: contains_any(input interface{}, substrings ...string) bool
```

### starts_with
Checks if the string starts with any of the provided prefixes.
```yaml
- 'starts_with(header, "HTTP/1.1")'
# Signature: starts_with(str string, prefix ...string) bool
```

### ends_with
Checks if the string ends with any of the provided suffixes.
```yaml
- 'ends_with(body, "</html>")'
# Signature: ends_with(str string, suffix ...string) bool
```

### line_starts_with
Checks if any line of the string starts with any of the provided prefixes.
```yaml
- 'line_starts_with(body, "admin:")'
# Signature: line_starts_with(str string, prefix ...string) bool
```

### line_ends_with
Checks if any line of the string ends with any of the provided suffixes.
```yaml
- 'line_ends_with(body, ".conf")'
# Signature: line_ends_with(str string, suffix ...string) bool
```

### join
Joins the given elements using the specified separator.
```yaml
- 'join("_", 123, "hello", "world")'  # "123_hello_world"
# Signature: join(separator string, elements ...interface{}) string
```

### concat
Concatenates the given arguments to form a string.
```yaml
- 'concat("Hello", 123, "world")'  # "Hello123world"
# Signature: concat(arguments ...interface{}) string
```

### replace
Replaces a substring in the input.
```yaml
- 'replace(body, "old", "new")'
# Signature: replace(str, old, new string) string
```

### replace_regex
Replaces substrings matching the given regex.
```yaml
- 'replace_regex(body, "(\\d+)", "NUM")'
# Signature: replace_regex(source, regex, replacement string) string
```

### trim
Trims characters from start and end.
```yaml
- 'trim(body, " \t\n\r")'
# Signature: trim(input, cutset string) string
```

### trim_left
Trims characters from the start only.
```yaml
- 'trim_left(body, "0")'
# Signature: trim_left(input, cutset string) string
```

### trim_right
Trims characters from the end only.
```yaml
- 'trim_right(body, "\n")'
# Signature: trim_right(input, cutset string) string
```

### trim_prefix
Removes the leading prefix string.
```yaml
- 'trim_prefix(body, "Bearer ")'
# Signature: trim_prefix(input, prefix string) string
```

### trim_suffix
Removes the trailing suffix string.
```yaml
- 'trim_suffix(body, "\r\n")'
# Signature: trim_suffix(input, suffix string) string
```

### trim_space
Removes all leading and trailing whitespace.
```yaml
- 'trim_space(body)'
# Signature: trim_space(input string) string
```

### len
Returns the length of the input.
```yaml
- 'len(body) > 1000'
# Signature: len(arg interface{}) int
```

### repeat
Repeats the input string N times.
```yaml
- 'repeat("../", 5)'  # "../../../../../"
# Signature: repeat(str string, count uint) string
```

### reverse
Reverses the input string.
```yaml
- 'reverse("abc")'  # "cba"
# Signature: reverse(input string) string
```

### to_lower
Converts to lowercase.
```yaml
- 'to_lower(header) == "content-type: text/html"'
# Signature: to_lower(input string) string
```

### to_upper
Converts to uppercase.
```yaml
- 'to_upper(body)'
# Signature: to_upper(input string) string
```

### remove_bad_chars
Removes specified characters from the input.
```yaml
- 'remove_bad_chars(body, "abc")'
# Signature: remove_bad_chars(input, cutset interface{}) string
```

### regex
Tests a regular expression against the input string.
```yaml
- 'regex("H([a-z]+)o", "Hello")'  # true
# Signature: regex(pattern, input string) bool
```

### regex_any
Verifies if any of the inputs match the regex.
```yaml
- 'regex_any("H([a-z]+)o", "World", "Hello")'  # true
# Signature: regex_any(pattern string, inputs ...string) bool
```

### regex_all
Verifies if all of the inputs match the regex.
```yaml
- 'regex_all("H([a-z]+)o", "Hallo", "Hello")'  # true
# Signature: regex_all(pattern string, inputs ...string) bool
```

### equals_any
Verifies if input equals any of the given items.
```yaml
- 'equals_any(status_code, 200, 201, 204)'  # true if status is 200, 201, or 204
# Signature: equals_any(s interface{}, subs ...interface{}) bool
```

### print_debug
Prints the value for debugging (use with `-debug` flag).
```yaml
- 'print_debug(status_code, body)'
# Signature: print_debug(args ...interface{})
```

---

## Encoding/Decoding Functions

### base64
Base64 encodes a string.
```yaml
- 'base64("Hello")'  # "SGVsbG8="
# Signature: base64(src interface{}) string
```

### base64_decode
Base64 decodes a string.
```yaml
- 'base64_decode("SGVsbG8=")'  # "Hello"
# Signature: base64_decode(src interface{}) []byte
```

### base64_py
Encodes to base64 like Python (with newlines).
```yaml
- 'base64_py("Hello")'  # "SGVsbG8=\n"
# Signature: base64_py(src interface{}) string
```

### url_encode
URL encodes the input string.
```yaml
- 'url_encode("https://example.com/test?a=1")'
# Signature: url_encode(input string) string
```

### url_decode
URL decodes the input string.
```yaml
- 'url_decode("https:%2F%2Fexample.com")'
# Signature: url_decode(input string) string
```

### hex_encode
Hex encodes the input.
```yaml
- 'hex_encode("aa")'  # "6161"
# Signature: hex_encode(input interface{}) string
```

### hex_decode
Hex decodes the input.
```yaml
- 'hex_decode("6161")'  # "aa"
# Signature: hex_decode(input interface{}) []byte
```

### html_escape
HTML escapes the input.
```yaml
- 'html_escape("<body>test</body>")'  # "&lt;body&gt;test&lt;/body&gt;"
# Signature: html_escape(input interface{}) string
```

### html_unescape
HTML un-escapes the input.
```yaml
- 'html_unescape("&lt;body&gt;test&lt;/body&gt;")'  # "<body>test</body>"
# Signature: html_unescape(input interface{}) string
```

### json_minify
Minifies a JSON string by removing whitespace.
```yaml
- 'json_minify("{ \"name\": \"John\" }")'  # '{"name":"John"}'
# Signature: json_minify(json) string
```

### json_prettify
Prettifies a JSON string with indentation.
```yaml
- 'json_prettify("{\"name\":\"John\"}")'
# Signature: json_prettify(json) string
```

---

## Hash Functions

### md5
Calculates MD5 hash.
```yaml
- 'md5("Hello")'  # "8b1a9953c4611296a827abf8c47804d7"
# Signature: md5(input interface{}) string
```

### sha1
Calculates SHA1 hash.
```yaml
- 'sha1("Hello")'  # "f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0"
# Signature: sha1(input interface{}) string
```

### sha256
Calculates SHA256 hash.
```yaml
- 'sha256("Hello")'  # "185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969"
# Signature: sha256(input interface{}) string
```

### mmh3
Calculates MurmurHash3 hash.
```yaml
- 'mmh3("Hello")'  # "316307400"
# Signature: mmh3(input interface{}) string
```

### hmac
HMAC computation with specified algorithm.
```yaml
- 'hmac("sha1", "test", "scrt")'  # "8856b111056d946d5c6c92a21b43c233596623c6"
- 'hmac("sha256", "data", "key")'
# Signature: hmac(algorithm, data, secret) string
# Supported algorithms: sha1, sha256, sha512, md5
```

---

## Compression Functions

### gzip
Compresses input using GZip.
```yaml
- 'gzip("Hello")'
# Signature: gzip(input string) string
```

### gzip_decode
Decompresses GZip compressed input.
```yaml
- 'gzip_decode(compressed_data)'
# Signature: gzip_decode(input string) string
```

### zlib
Compresses input using Zlib.
```yaml
- 'zlib("Hello")'
# Signature: zlib(input string) string
```

### zlib_decode
Decompresses Zlib compressed input.
```yaml
- 'zlib_decode(compressed_data)'
# Signature: zlib_decode(input string) string
```

---

## Random Generation Functions

### rand_base
Generates a random string of given length from optional charset.
```yaml
- 'rand_base(8)'  # "aB3dE5fG"
- 'rand_base(5, "abc")'  # "caccb"
# Signature: rand_base(length uint, optionalCharSet string) string
```

### rand_char
Generates a single random character.
```yaml
- 'rand_char("abc")'  # "a"
# Signature: rand_char(optionalCharSet string) string
```

### rand_int
Generates a random integer in range.
```yaml
- 'rand_int(1, 10)'  # 6
- 'rand_int()'  # Random 0 to MaxInt32
# Signature: rand_int(optionalMin, optionalMax uint) int
```

### rand_text_alpha
Generates random alphabetic string, excluding optional bad chars.
```yaml
- 'rand_text_alpha(10)'  # "WKozhjJWlJ"
- 'rand_text_alpha(10, "abc")'  # Excludes a, b, c
# Signature: rand_text_alpha(length uint, optionalBadChars string) string
```

### rand_text_alphanumeric
Generates random alphanumeric string, excluding optional bad chars.
```yaml
- 'rand_text_alphanumeric(10)'  # "NthI0IiY8r"
- 'rand_text_alphanumeric(10, "ab12")'  # Excludes a, b, 1, 2
# Signature: rand_text_alphanumeric(length uint, optionalBadChars string) string
```

### rand_text_numeric
Generates random numeric string, excluding optional bad numbers.
```yaml
- 'rand_text_numeric(10)'  # "0654087985"
- 'rand_text_numeric(10, 123)'  # Excludes 1, 2, 3
# Signature: rand_text_numeric(length uint, optionalBadNumbers string) string
```

### rand_ip
Generates a random IP address within optional CIDR range.
```yaml
- 'rand_ip("192.168.0.0/24")'  # "192.168.0.171"
# Signature: rand_ip(cidr ...string) string
```

---

## Number/Conversion Functions

### bin_to_dec
Converts binary to decimal.
```yaml
- 'bin_to_dec("0b1010")'  # 10
- 'bin_to_dec(1010)'  # 10
# Signature: bin_to_dec(binaryNumber number | string) float64
```

### hex_to_dec
Converts hexadecimal to decimal.
```yaml
- 'hex_to_dec("ff")'  # 255
- 'hex_to_dec("0xff")'  # 255
# Signature: hex_to_dec(hexNumber number | string) float64
```

### oct_to_dec
Converts octal to decimal.
```yaml
- 'oct_to_dec("0o1234567")'  # 342391
- 'oct_to_dec(1234567)'  # 342391
# Signature: oct_to_dec(octalNumber number | string) float64
```

### dec_to_hex
Converts decimal to hexadecimal.
```yaml
- 'dec_to_hex(7001)'  # "1b59"
# Signature: dec_to_hex(number number | string) string
```

---

## Date/Time Functions

### unix_time
Returns current Unix timestamp with optional seconds added.
```yaml
- 'unix_time()'  # Current timestamp
- 'unix_time(10)'  # Current + 10 seconds
# Signature: unix_time(optionalSeconds uint) float64
```

### to_unix_time
Parses a date string and returns Unix timestamp.
```yaml
- 'to_unix_time("2022-01-13T16:30:10+00:00")'  # 1642091410
- 'to_unix_time("2022-01-13", "2006-01-02")'
# Signature: to_unix_time(input string, layout string) int
```

### date_time
Returns formatted date time using simplified or Go style layout.
```yaml
- 'date_time("%Y-%M-%D %H:%m")'  # "2022-06-10 14:18"
- 'date_time("%Y-%M-%D %H:%m", 1654870680)'  # With unix time
- 'date_time("2006-01-02 15:04", unix_time())'  # Go layout
# Signature: date_time(dateTimeFormat string, optionalUnixTime interface{}) string
```

**Simplified format tokens:**
- `%Y` - Year (4 digits)
- `%M` - Month (2 digits)
- `%D` - Day (2 digits)
- `%H` - Hour (24h)
- `%m` - Minute
- `%S` - Second

---

## Crypto Functions

### aes_gcm
AES GCM encrypts a string with key.
```yaml
- 'aes_gcm("AES256Key-32Characters1234567890", "exampleplaintext")'
- 'hex_encode(aes_gcm(key, plaintext))'
# Signature: aes_gcm(key, plaintext interface{}) []byte
# Key must be 16, 24, or 32 bytes for AES-128/192/256
```

### generate_jwt
Generates a JSON Web Token (JWT).
```yaml
- 'generate_jwt("{\"name\":\"John\",\"foo\":\"bar\"}", "HS256", "secret")'
- 'generate_jwt(claims_json, algorithm, signature, unixMaxAge)'
# Signature: generate_jwt(json, algorithm, signature, unixMaxAge) []byte
# Supported algorithms: HS256, HS384, HS512
```

---

## Deserialization Functions

### generate_java_gadget
Generates a Java deserialization gadget payload.
```yaml
# DNS callback gadget (for vulnerability detection)
- 'generate_java_gadget("dns", "http://{{interactsh-url}}", "base64")'
# Used in: CVE-2016-4437 (Apache Shiro)

# Specific library gadget (for command execution)
- 'generate_java_gadget("commons-collections3.1", "wget http://{{interactsh-url}}", "base64")'
# Used in: CVE-2021-41419 (QVIS DVR)

# Signature: generate_java_gadget(gadget, cmd, encoding interface{}) string
# Common gadgets: dns, commons-collections3.1, commons-collections4.0, groovy1
```

### generate_dotnet_gadget
Generates a .NET deserialization gadget payload.
```yaml
- 'generate_dotnet_gadget("type-confuse-delegate", "calc", "binary", "base64-raw")'
# Signature: generate_dotnet_gadget(gadget, cmd, formatter, encoding string) string
```

---

## Network Functions

### resolve
Resolves a hostname using specified DNS type.
```yaml
- 'resolve("localhost", 4)'  # "127.0.0.1" (type 4 = A)
# Signature: resolve(host string, format string) string
```

### ip_format
Formats an IP address (e.g., convert IPv4 to IPv6 format).
```yaml
- 'ip_format("169.254.169.254", 4)'  # Format as IPv4
# Signature: ip_format(ip string, format int) string
```

### wait_for
Pauses execution for given seconds.
```yaml
- 'wait_for(10)'
# Signature: wait_for(seconds uint)
```

---

## Utility Functions

### compare_versions
Compares version against constraints.
```yaml
- 'compare_versions("v1.0.0", ">v0.0.1", "<v1.0.1")'  # true
- 'compare_versions(version, ">=2.0.0", "<3.0.0")'
# Signature: compare_versions(versionToCheck string, constraints ...string) bool
```

### iterate
Iterates over array (used in flow control).
```yaml
- 'for(let item of iterate(template.items))'
```

---

## Complete Function Quick Reference

| Function | Category | Description |
|---|---|---|
| `aes_gcm` | Crypto | AES GCM encryption |
| `base64` | Encoding | Base64 encode |
| `base64_decode` | Encoding | Base64 decode |
| `base64_py` | Encoding | Base64 encode (Python style) |
| `bin_to_dec` | Number | Binary to decimal |
| `compare_versions` | Utility | Version comparison |
| `concat` | String | Concatenate strings |
| `contains` | String | Check substring exists |
| `contains_all` | String | Check all substrings exist |
| `contains_any` | String | Check any substring exists |
| `date_time` | DateTime | Format date/time |
| `dec_to_hex` | Number | Decimal to hex |
| `ends_with` | String | Check string suffix |
| `equals_any` | String | Check equals any value |
| `generate_dotnet_gadget` | Deserialization | .NET gadget payload |
| `generate_java_gadget` | Deserialization | Java gadget payload |
| `generate_jwt` | Crypto | Generate JWT token |
| `gzip` | Compression | GZip compress |
| `gzip_decode` | Compression | GZip decompress |
| `hex_decode` | Encoding | Hex decode |
| `hex_encode` | Encoding | Hex encode |
| `hex_to_dec` | Number | Hex to decimal |
| `hmac` | Hash | HMAC hash |
| `html_escape` | Encoding | HTML escape |
| `html_unescape` | Encoding | HTML unescape |
| `ip_format` | Network | Format IP address |
| `join` | String | Join elements |
| `json_minify` | Encoding | Minify JSON |
| `json_prettify` | Encoding | Prettify JSON |
| `len` | String | String/object length |
| `line_ends_with` | String | Line suffix check |
| `line_starts_with` | String | Line prefix check |
| `md5` | Hash | MD5 hash |
| `mmh3` | Hash | MurmurHash3 |
| `oct_to_dec` | Number | Octal to decimal |
| `print_debug` | Utility | Debug output |
| `rand_base` | Random | Random string |
| `rand_char` | Random | Random character |
| `rand_int` | Random | Random integer |
| `rand_ip` | Random | Random IP address |
| `rand_text_alpha` | Random | Random alpha string |
| `rand_text_alphanumeric` | Random | Random alphanumeric |
| `rand_text_numeric` | Random | Random numeric string |
| `regex` | String | Regex test |
| `regex_all` | String | Regex match all |
| `regex_any` | String | Regex match any |
| `remove_bad_chars` | String | Remove characters |
| `repeat` | String | Repeat string |
| `replace` | String | Replace substring |
| `replace_regex` | String | Regex replace |
| `resolve` | Network | DNS resolution |
| `reverse` | String | Reverse string |
| `sha1` | Hash | SHA1 hash |
| `sha256` | Hash | SHA256 hash |
| `starts_with` | String | Check string prefix |
| `to_lower` | String | Convert to lowercase |
| `to_unix_time` | DateTime | Date to unix timestamp |
| `to_upper` | String | Convert to uppercase |
| `trim` | String | Trim characters |
| `trim_left` | String | Trim left characters |
| `trim_prefix` | String | Remove prefix |
| `trim_right` | String | Trim right characters |
| `trim_space` | String | Trim whitespace |
| `trim_suffix` | String | Remove suffix |
| `unix_time` | DateTime | Current unix timestamp |
| `url_decode` | Encoding | URL decode |
| `url_encode` | Encoding | URL encode |
| `wait_for` | Utility | Pause execution |
| `zlib` | Compression | Zlib compress |
| `zlib_decode` | Compression | Zlib decompress |

---

## Response Variables

### HTTP Variables
| Variable | Description |
|---|---|
| `status_code` | HTTP status code |
| `body` | Response body |
| `header` | All response headers |
| `content_type` | Content-Type header value |
| `header["name"]` | Specific header value (e.g., `header["server"]`) |
| `status_code_N` | Status code from request N (multi-request) |
| `body_N` | Body from request N (multi-request) |
| `header_N` | Headers from request N (multi-request) |
| `content_type_N` | Content type from request N |

### DNS Variables
| Variable | Description |
|---|---|
| `answer` | DNS answer section |
| `question` | DNS question section |
| `rcode` | DNS response code |
| `cname` | CNAME record value |
| `A` | A record value |
| `AAAA` | AAAA record value |
| `MX` | MX record value |
| `NS` | NS record value |
| `TXT` | TXT record value |

### Network Variables
| Variable | Description |
|---|---|
| `response` | Response data |
| `data` | Received data |

### General Variables
| Variable | Description |
|---|---|
| `response` | Generic response |
| `request` | Sent request |

---

## SSL Variables

| Variable | Description |
|---|---|
| `expired` | Boolean: certificate expired |
| `self_signed` | Boolean: self-signed certificate |
| `not_after` | Certificate expiry date |
| `not_before` | Certificate start date |
| `tls_version` | Negotiated TLS version |
| `cipher` | Negotiated cipher suite |
| `Subject` | Certificate subject |
| `Issuer` | Certificate issuer |
| `CN` | Common name |
| `SAN` | Subject alternative names |
| `Serial` | Certificate serial number |
| `fingerprint_hash` | Certificate fingerprint |
