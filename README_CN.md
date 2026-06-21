# Nuclei 模板创建技能

[English](README.md) | **中文**

一个用于创建高质量 [Nuclei](https://github.com/projectdiscovery/nuclei) 安全扫描模板的综合技能，覆盖所有支持的协议类型和漏洞类型。

## 概述

本技能提供了编写 Nuclei 模板的完整文档和参考资料，涵盖从基础的 HTTP 漏洞检测到高级的云安全扫描和工作流编排。

### 包含内容

| 类别 | 描述 |
|---|---|
| **10 种协议类型** | HTTP、DNS、SSL/TLS、Network/TCP、File、Headless、JavaScript、Code、DAST、Cloud |
| **7 种匹配器类型** | status、size、word、regex、binary、dsl、xpath |
| **5 种提取器类型** | regex、kval、json、xpath、dsl |
| **60+ DSL 函数** | 字符串、编码、哈希、压缩、随机、日期时间、加密函数 |
| **35 个 JS 函数** | 运行时、流程控制、操作系统/架构检测 |
| **9 个示例模板** | CVE、错误配置、子域名接管、默认登录等真实模式 |

## 目录结构

```
nuclei-template-creator/
├── SKILL.md                          # 主技能文件（入口点）
└── references/
    ├── http-protocol.md              # HTTP 模板语法
    ├── dns-protocol.md               # DNS 模板语法
    ├── ssl-protocol.md               # SSL/TLS 模板语法
    ├── network-protocol.md           # TCP/网络模板语法
    ├── file-protocol.md              # 文件扫描语法
    ├── headless-protocol.md          # 无头浏览器语法
    ├── javascript-protocol.md        # JavaScript 协议语法
    ├── code-protocol.md              # 代码执行语法
    ├── dast-protocol.md              # DAST 模糊测试语法
    ├── cloud-protocol.md             # 云安全语法
    ├── matchers-extractors.md        # 匹配器和提取器参考
    ├── dsl-functions.md              # DSL 函数参考
    ├── variables.md                  # 内置变量参考
    ├── preprocessors.md              # 预处理器参考
    ├── oob-testing.md                # OOB 测试参考
    ├── workflows.md                  # 工作流模板语法
    ├── flow.md                       # 流程控制语法
    └── examples/                     # 真实模板示例
        ├── cve-http.yaml
        ├── misconfiguration.yaml
        ├── takeover.yaml
        ├── default-login.yaml
        ├── token-spray.yaml
        ├── dns-detect.yaml
        ├── ssl-check.yaml
        ├── network-service.yaml
        └── workflow.yaml
```

## 快速开始

### 创建 CVE 模板

```yaml
id: CVE-2024-1234

info:
  name: Example App RCE
  author: your-username
  severity: critical
  description: |
    Example App 中存在远程代码执行漏洞，通过不安全的反序列化实现。
  reference:
    - https://nvd.nist.gov/vuln/detail/CVE-2024-1234
  classification:
    cvss-metrics: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
    cvss-score: 9.8
    cve-id: CVE-2024-1234
    cwe-id: CWE-502
  tags: cve,cve2024,rce,example

variables:
  marker: "{{rand_base(8)}}"

http:
  - method: POST
    path:
      - "{{BaseURL}}/api/deserialize"
    body: '{"data":"{{base64(marker)}}"}'
    matchers:
      - type: word
        words:
          - "{{marker}}"
        part: body
```

### 检测错误配置

```yaml
id: app-config-exposure

info:
  name: 应用配置文件暴露
  author: your-username
  severity: high
  tags: misconfig,exposure

http:
  - method: GET
    path:
      - "{{BaseURL}}/config.yml"
      - "{{BaseURL}}/config.json"
    stop-at-first-match: true
    matchers-condition: and
    matchers:
      - type: word
        words:
          - "database:"
          - "secret_key:"
        condition: or
      - type: status
        status:
          - 200
```

### SSRF 检测（带 OOB）

```yaml
id: ssrf-oob-detect

info:
  name: 通过 OOB 检测 SSRF
  author: your-username
  severity: high
  tags: ssrf,oast

http:
  - method: GET
    path:
      - "{{BaseURL}}/fetch?url={{interactsh-url}}"
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "http"
```

## 文档覆盖

### 协议类型

| 协议 | 文档 | 示例模板 |
|---|---|---|
| HTTP | [http-protocol.md](references/http-protocol.md) | CVE 检测、错误配置、子域名接管 |
| DNS | [dns-protocol.md](references/dns-protocol.md) | CNAME 检测、SPF/DMARC 检查 |
| SSL/TLS | [ssl-protocol.md](references/ssl-protocol.md) | 过期证书、弱加密套件 |
| Network | [network-protocol.md](references/network-protocol.md) | 服务检测、协议探测 |
| File | [file-protocol.md](references/file-protocol.md) | 密钥扫描、恶意软件检测 |
| Headless | [headless-protocol.md](references/headless-protocol.md) | DOM XSS、基于浏览器的检测 |
| JavaScript | [javascript-protocol.md](references/javascript-protocol.md) | 自定义协议逻辑 |
| Code | [code-protocol.md](references/code-protocol.md) | Shell/Python 执行 |
| DAST | [dast-protocol.md](references/dast-protocol.md) | SQLi、XSS、SSTI 模糊测试 |
| Cloud | [cloud-protocol.md](references/cloud-protocol.md) | AWS、GCP 错误配置 |

### 核心概念

| 主题 | 文档 |
|---|---|
| 匹配器和提取器 | [matchers-extractors.md](references/matchers-extractors.md) |
| DSL 函数 | [dsl-functions.md](references/dsl-functions.md) |
| 变量 | [variables.md](references/variables.md) |
| 预处理器 | [preprocessors.md](references/preprocessors.md) |
| OOB 测试 | [oob-testing.md](references/oob-testing.md) |
| 流程控制 | [flow.md](references/flow.md) |
| 工作流 | [workflows.md](references/workflows.md) |

## 功能覆盖

### HTTP 选项

```yaml
http:
  - method: GET/POST/PUT/DELETE/PATCH
    path:
      - "{{BaseURL}}/endpoint"
    headers:
      Key: Value
    body: "请求体"
    raw:
      - |
        GET / HTTP/1.1
        Host: {{Hostname}}
    redirects: true
    max-redirects: 5
    host-redirects: true          # 跟随跨主机重定向
    cookie-reuse: true
    disable-cookie: false
    unsafe: true                  # 发送原始请求，不进行编码
    skip-variables-check: true    # 跳过未解析变量验证
    stop-at-first-match: true
    matchers-condition: and/or
    payloads:
      name:
        - value1
        - value2
    attack: clusterbomb/pitchfork
    fuzzing:
      - part: query/body/path
        type: postfix/replace
```

### 匹配器类型

```yaml
matchers:
  # 状态码匹配
  - type: status
    status:
      - 200
      - 302

  # 内容长度匹配
  - type: size
    size:
      - 0
    part: body

  # 字符串匹配
  - type: word
    words:
      - "string1"
      - "string2"
    condition: and
    negative: true
    encoding: hex

  # 正则匹配
  - type: regex
    regex:
      - "pattern"

  # 二进制/十六进制匹配
  - type: binary
    binary:
      - "504B0304"

  # DSL 表达式匹配
  - type: dsl
    dsl:
      - 'status_code == 200'
      - 'contains(body, "text")'

  # XPath 匹配
  - type: xpath
    xpath:
      - "/html/head/title"
```

### 提取器类型

```yaml
extractors:
  # 正则提取
  - type: regex
    part: body
    group: 1
    name: version
    internal: true
    regex:
      - "Version: ([0-9.]+)"

  # 键值提取（头/Cookie）
  - type: kval
    kval:
      - content_type
      - set_cookie

  # JSON 提取
  - type: json
    json:
      - ".data.id"
      - ".items[].name"

  # XPath 提取
  - type: xpath
    attribute: href
    xpath:
      - "/html/body//a"

  # DSL 提取
  - type: dsl
    dsl:
      - 'len(body)'
      - 'status_code'
```

### DSL 函数（60+）

| 类别 | 函数 |
|---|---|
| **字符串** | `contains`、`contains_all`、`contains_any`、`starts_with`、`ends_with`、`join`、`concat`、`replace`、`replace_regex`、`trim`、`len`、`reverse`、`to_lower`、`to_upper`、`regex` |
| **编码** | `base64`、`base64_decode`、`url_encode`、`url_decode`、`hex_encode`、`hex_decode`、`html_escape`、`html_unescape`、`json_minify`、`json_prettify` |
| **哈希** | `md5`、`sha1`、`sha256`、`mmh3`、`hmac` |
| **压缩** | `gzip`、`gzip_decode`、`zlib`、`zlib_decode` |
| **随机** | `rand_base`、`rand_char`、`rand_int`、`rand_text_alpha`、`rand_text_alphanumeric`、`rand_text_numeric`、`rand_ip` |
| **日期时间** | `unix_time`、`to_unix_time`、`date_time` |
| **加密** | `aes_gcm`、`generate_jwt` |
| **转换** | `bin_to_dec`、`hex_to_dec`、`oct_to_dec`、`dec_to_hex` |
| **工具** | `compare_versions`、`wait_for`、`ip_format` |

### 流程控制

```yaml
# 顺序执行
flow: http(1) && http(2)

# 跨协议
flow: tcp(1) && javascript(1)

# 遍历提取的数组
flow: |
  code(1)
  for(let item of iterate(template.items)){
    set("item_name", item)
    code(2)
  }
```

### 工作流

```yaml
workflows:
  # 简单模板执行
  - template: http/technologies/tech-detect.yaml

  # 带标签的条件执行
  - template: http/technologies/jira-detect.yaml
    subtemplates:
      - tags: jira

  # 基于匹配器名称
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: wordpress
        subtemplates:
          - template: cves/CVE-2019-6715.yaml
```

## 验证清单

提交模板前：

- [ ] **ID** 唯一、描述性、小写连字符格式
- [ ] **Info 块** 包含 name、author、severity
- [ ] **描述** 清晰说明检测内容
- [ ] **引用** 包含公告、NVD 和相关链接
- [ ] **标签** 全面且遵循命名规范
- [ ] **匹配器** 使用分层验证（识别 → 确认 → 证明）
- [ ] **模板** 在易受攻击的系统上能检测到漏洞
- [ ] **模板** 在已修补/不易受攻击的系统上不会匹配
- [ ] **YAML 语法** 有效（`nuclei -validate -t template.yaml`）

## 官方文档

- [Nuclei 模板介绍](https://docs.projectdiscovery.io/templates/introduction)
- [模板创建指南](https://docs.projectdiscovery.io/templates/)
- [辅助函数](https://docs.projectdiscovery.io/templates/reference/helper-functions)
- [匹配器参考](https://docs.projectdiscovery.io/templates/reference/matchers)
- [提取器参考](https://docs.projectdiscovery.io/templates/reference/extractors)

## 统计数据

| 指标 | 数量 |
|---|---|
| 总文件数 | 27 |
| 总行数 | ~5,100 |
| 协议文档 | 10 |
| 匹配器类型 | 7 |
| 提取器类型 | 5 |
| DSL 函数 | 60+ |
| JS 函数 | 35 |
| 示例模板 | 9 |

## 许可证

本技能按原样提供，用于教育和安全研究目的。

## 贡献

1. Fork 仓库
2. 创建功能分支
3. 进行更改
4. 使用 `nuclei -validate -t your-template.yaml` 验证
5. 提交 Pull Request

---

**为安全社区用心构建 ❤️**
