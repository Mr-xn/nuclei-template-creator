# Multi-Protocol Reference

Execute multiple protocols in ONE template (Nuclei v3.0+) — e.g. subdomain takeover: check DNS CNAME, then verify the HTTP response string. Unlike workflows (which orchestrate separate template FILES), multi-protocol keeps all logic in a single template.

## Basic Structure

```yaml
id: dns-http-template
info:
  name: dns + http takeover template
  author: pdteam
  severity: info

dns:
  - name: "{{FQDN}}"
    type: cname

http:
  - method: GET
    path:
      - "{{BaseURL}}"

matchers:
  - type: dsl
    dsl:
      - "contains(http_body, 'Domain not found')"   # from http response
      - "contains(dns_cname, 'github.io')"          # from dns response
    condition: and
```

## How It Works

- Protocols execute SERIALLY, in the order defined in the template.
- Response fields are exported to the Template Context as soon as each protocol runs; variables are re-evaluated after each protocol.
- Exit-on-error: if a protocol fails, remaining protocols are skipped.
- No duplicate protocols (e.g. `dns -> http -> ssl -> http` is not supported).
- v3 only; any implemented protocol is allowed, no count limit.

## Protocol-Scoped Variables (no extractor needed)

All response fields of every protocol are exported with a protocol prefix:

| Protocol | Field | Exported variable |
|---|---|---|
| ssl | subject_cn | `ssl_subject_cn` |
| dns | cname | `dns_cname` |
| http | header | `http_header` |
| code | response | `code_response` |

List ALL exported fields for a target: `nuclei -t tpl.yaml -u host -debug -svd`

## Data Export via Dynamic Extractors

Named dynamic extractors are stored in the template context and usable across all protocols:

```yaml
dns:
  - name: "{{FQDN}}"
    type: cname
extractors:
  - type: dsl
    name: exported_cname
    dsl: [cname]
    internal: true
http:
  - method: GET
    path: ["{{BaseURL}}"]
matchers:
  - type: dsl
    dsl:
      - "contains(body, 'Domain not found')"
      - "contains(exported_cname, 'github.io')"
    condition: and
```

## Multi-Protocol vs Workflows

| | Multi-protocol template | Workflow |
|---|---|---|
| Scope | one template, several protocols | several template FILES orchestrated by a workflow file |
| Logic | contains the vulnerability logic itself | only chains template execution (conditions/subtemplates) |
| Variables | full template context + dynamic extractors | limited support |
