# Workflow Templates Reference

Complete syntax reference for Nuclei Workflow templates.

## Table of Contents
- [Overview](#overview)
- [Top-Level Syntax](#top-level-syntax)
- [Generic Workflows](#generic-workflows)
- [Conditional Workflows](#conditional-workflows)
- [Matcher Name-Based Conditions](#matcher-name-based-conditions)
- [Multi-Level Nested Workflows](#multi-level-nested-workflows)
- [Shared Execution Context](#shared-execution-context)
- [Examples](#examples)

---

## Overview

Workflows enable users to orchestrate a series of actions by setting a defined execution order for various templates. Templates are activated upon predetermined conditions, establishing a streamlined method to leverage nuclei's capabilities tailored to specific requirements.

**Key Benefits:**
- Craft workflows contingent on particular technologies or targets
- Reduce scan duration by running relevant templates only
- Share extracted values across templates in the same workflow

---

## Top-Level Syntax

```yaml
workflows:
  - template: path/to/template.yaml
    subtemplates:
      - tags: technology-name
```

### Fields

| Field | Description |
|---|---|
| `template` | Path to a single template file **or a directory** |
| `tags` | Tag-based filter to select matching templates |
| `subtemplates` | Templates/tags to execute conditionally when parent matches |
| `matchers` | Matcher-based conditions; checks matcher `name` from parent template |

---

## Generic Workflows

Generic workflows define single or multiple templates to execute without conditions. Supports both files and directories.

### Example: Running specific checks

```yaml
workflows:
  - template: http/exposures/configs/git-config.yaml
  - template: http/exposures/configs/exposed-svn.yaml
  - template: http/vulnerabilities/generic/generic-env.yaml
  - template: http/exposures/backups/zip-backup-files.yaml
  - tags: xss,ssrf,cve,lfi
```

### Example: Running directories of templates

```yaml
workflows:
  - template: http/cves/
  - template: http/exposures/
  - tags: exposures
```

---

## Conditional Workflows

Conditional templates execute after matching a condition from a previous template.

### Template-Based Condition

```yaml
workflows:
  - template: http/technologies/jira-detect.yaml
    subtemplates:
      - tags: jira
      - template: exploits/jira/
```

---

## Matcher Name-Based Conditions

Execute subtemplates when a **specific matcher** from the parent template is found:

```yaml
workflows:
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: vbulletin
        subtemplates:
          - template: exploits/vbulletin-exp1.yaml
          - template: exploits/vbulletin-exp2.yaml
      - name: jboss
        subtemplates:
          - template: exploits/jboss-exp1.yaml
          - template: exploits/jboss-exp2.yaml
```

**Fields:**
- `name` - Name of the matcher from parent template results
- `subtemplates` - Templates/tags to run when that named matcher fires

---

## Multi-Level Nested Workflows

Chain template executions that run only if previous templates match:

```yaml
workflows:
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: lotus-domino
        subtemplates:
          - template: http/technologies/lotus-domino-version.yaml
            subtemplates:
              - template: http/cves/2020/xx-yy-zz.yaml
                subtemplates:
                  - template: http/cves/2020/xx-xx-xx.yaml
```

You can nest `subtemplates` arbitrarily deep, each level conditional on the parent matching.

---

## Shared Execution Context

Nuclei supports transparent **workflow cookiejar** and **key-value sharing** across templates in the same workflow.

### Key-Value Sharing Example

**Workflow file:**
```yaml
workflows:
  - template: template-with-named-extractor.yaml
    subtemplates:
      - template: template-using-named-extractor.yaml
```

**First template (extractor):**
```yaml
id: value-sharing-template1
http:
  - path:
      - "{{BaseURL}}/path1"
    extractors:
      - type: regex
        part: body
        name: extracted
        regex:
          - 'href="(.*)"'
        group: 1
```

**Second template (consumer):**
```yaml
id: value-sharing-template2
http:
  - raw:
      - |
        GET /path2 HTTP/1.1
        Host: {{Hostname}}

        {{extracted}}
```

The extractor uses `name: extracted`, making its output available as `{{extracted}}` in subsequent templates.

---

## Examples

### Generic Workflow
```yaml
workflows:
  - template: technologies/jira-detect.yaml
  - template: technologies/confluence-detect.yaml
```

### Conditional Workflow (Spring Boot)
```yaml
workflows:
  - template: security-misconfiguration/springboot-detect.yaml
    subtemplates:
      - template: cves/CVE-2018-1271.yaml
      - template: cves/CVE-2020-5410.yaml
      - template: vulnerabilities/springboot-actuators-jolokia-xxe.yaml
      - template: vulnerabilities/springboot-h2-db-rce.yaml
```

### Matcher Name-Based (WordPress)
```yaml
workflows:
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: wordpress
        subtemplates:
          - template: cves/CVE-2019-6715.yaml
          - template: cves/CVE-2019-9978.yaml
          - template: files/wordpress-db-backup.yaml
          - template: files/wordpress-debug-log.yaml
          - template: vulnerabilities/sassy-social-share.yaml
          - template: vulnerabilities/w3c-total-cache-ssrf.yaml
```

### Multiple Matchers (vBulletin + JBoss)
```yaml
workflows:
  - template: http/technologies/tech-detect.yaml
    matchers:
      - name: vbulletin
        subtemplates:
          - tags: vbulletin
      - name: jboss
        subtemplates:
          - tags: jboss
```

### Multi-Technology Workflow
```yaml
workflows:
  - template: http/technologies/nginx-detect.yaml
    subtemplates:
      - tags: nginx
  - template: http/technologies/apache-detect.yaml
    subtemplates:
      - tags: apache
  - template: http/technologies/iis-detect.yaml
    subtemplates:
      - tags: iis
```

---

## Summary of All Fields

| Field | Level | Type | Description |
|---|---|---|---|
| `workflows` | Root | List | Top-level list defining the workflow |
| `template` | Workflow item | String (file/dir) | Template(s) to execute; supports directories |
| `tags` | Workflow item | String (comma-separated) | Filter templates by tags |
| `subtemplates` | Workflow item | List | Templates/tags to execute when parent matches |
| `matchers` | Workflow item | List | Matcher-based conditional branching |
| `name` | Matcher item | String | Name of matcher from parent template |
| `subtemplates` | Matcher item | List | Templates/tags to run when named matcher fires |
