# Cloud Protocol Reference

Complete syntax reference for Nuclei Cloud templates (AWS, Azure, GCP, etc.).

## Table of Contents
- [Basic Structure](#basic-structure)
- [AWS Templates](#aws-templates)
- [Flow Control](#flow-control)
- [Examples](#examples)

---

## Basic Structure

Cloud templates use `code:` protocol with `self-contained: true`:

```yaml
self-contained: true
variables:
  region: "us-east-1"

code:
  - engine:
      - sh
      - bash
    source: |
      aws ec2 describe-instances --region {{region}} --output json
    matchers:
      - type: word
        words:
          - "Instances"
```

---

## AWS Templates

AWS templates use the AWS CLI and require proper credentials configured.

### Common AWS Services

| Service | CLI Command |
|---|---|
| EC2 | `aws ec2 describe-instances` |
| S3 | `aws s3api list-buckets` |
| IAM | `aws iam list-users` |
| RDS | `aws rds describe-db-instances` |
| Lambda | `aws lambda list-functions` |
| CloudFormation | `aws cloudformation describe-stacks` |
| EKS | `aws eks list-clusters` |
| ECS | `aws ecs list-clusters` |

---

## Flow Control

Cloud templates use `flow:` with JavaScript for iteration:

```yaml
flow: |
  code(1)
  for(let instance of iterate(template.instances)){
    set("instance_id", instance)
    code(2)
  }
```

### Flow Syntax
- `code(N)` - Execute code block N (1-based)
- `for...of iterate(var)` - Loop over extracted array
- `set("var", value)` - Set variable for next block
- `template.var` - Access extracted variables from previous blocks

---

## Examples

### EC2 Instance Enumeration
```yaml
self-contained: true
variables:
  region: "us-east-1"

flow: |
  code(1)
  for(let instance of iterate(template.instances)){
    set("instance_id", instance)
    code(2)
  }

code:
  # Block 1: List instances
  - engine:
      - sh
      - bash
    source: |
      aws ec2 describe-instances --region {{region}} --output json \
        --query 'Reservations[*].Instances[*].InstanceId' --output text
    extractors:
      - type: regex
        name: instances
        internal: true
        regex:
          - "(i-[a-z0-9]+)"

  # Block 2: Check each instance
  - engine:
      - sh
      - bash
    source: |
      aws ec2 describe-instance-attribute --region {{region}} \
        --instance-id {{instance_id}} --attribute metadataOptions \
        --query 'HttpTokens' --output text
    matchers:
      - type: word
        words:
          - "optional"
```

### S3 Bucket Public Access
```yaml
self-contained: true
variables:
  region: "us-east-1"

flow: |
  code(1)
  for(let bucket of iterate(template.buckets)){
    set("bucket_name", bucket)
    code(2)
  }

code:
  - engine:
      - sh
      - bash
    source: |
      aws s3api list-buckets --query 'Buckets[*].Name' --output text
    extractors:
      - type: regex
        name: buckets
        internal: true
        regex:
          - "([a-z0-9][a-z0-9.-]+[a-z0-9])"

  - engine:
      - sh
      - bash
    source: |
      aws s3api get-bucket-acl --bucket {{bucket_name}} --output json
    matchers:
      - type: word
        words:
          - '"URI": "http://acs.amazonaws.com/groups/global/AllUsers"'
```

### CloudFormation Stack Notification Check
```yaml
self-contained: true
variables:
  region: "us-east-1"

flow: |
  code(1)
  for(let stack of iterate(template.stacks)){
    set("stack_name", stack)
    code(2)
  }

code:
  - engine:
      - sh
      - bash
    source: |
      aws cloudformation describe-stacks --region {{region}} \
        --query 'Stacks[*].StackName' --output text
    extractors:
      - type: regex
        name: stacks
        internal: true
        regex:
          - "([a-zA-Z0-9-]+)"

  - engine:
      - sh
      - bash
    source: |
      aws cloudformation describe-stacks --region {{region}} \
        --stack-name {{stack_name}} \
        --query 'Stacks[0].NotificationARNs' --output json
    matchers:
      - type: word
        words:
          - "[]"
    extractors:
      - type: dsl
        dsl:
          - '"CloudFormation Stack " + stack_name + " has no notifications configured"'
```

### GCP Project Check
```yaml
self-contained: true
code:
  - engine:
      - sh
      - bash
    source: |
      gcloud projects list --format=json 2>&1
    matchers:
      - type: word
        words:
          - "projectId"
    extractors:
      - type: json
        json:
          - ".[].projectId"
```
