# Template Signing Reference

ECDSA private/public key signing for template integrity & authenticity (Nuclei v3).

## Key Facts

- All official nuclei-templates are signed and verified with ProjectDiscovery's public key (shipped with the binary).
- Signing is OPTIONAL for all protocols **except `code`** — unsigned code templates are DISABLED and cannot be executed. Only templates signed by yourself or ProjectDiscovery run.
- A signed template carries a trailing `# digest: <signature>:<fragment>` comment (auto-generated — never hand-write or modify it). The fragment is MD5(public key) metadata that blocks re-signing of code templates written by others.
- Any edit after signing invalidates the digest → re-sign code templates after every change.
- Code file references (e.g. `source: protocols/code/pyfile.py`) ARE included in the digest; payload file references (e.g. `payloads: params.txt`) are NOT.
- Digest is deterministic — a template signed with `-lfa` (local file access) will FAIL verification when run without `-lfa`.

## Signing / Key Management

```bash
# generates a key pair on first use (prompts for user/org name + optional passphrase)
nuclei -t template.yaml -sign

# keys are stored in $CONFIG/nuclei/keys/
#   nuclei-user-private-key.pem   (encrypted, PEMCipherAES256 if passphrase set)
#   nuclei-user.crt               (self-signed cert with public key + identifier)
```

Distribute keys via env vars instead of copying files:

```bash
export NUCLEI_USER_CERTIFICATE=$(cat path/to/nuclei-user.crt)
export NUCLEI_USER_PRIVATE_KEY=$(cat path/to/nuclei-user-private-key.pem)
```

`HIDE_TEMPLATE_SIG_WARNING=true` silences the unsigned-template warning (not recommended).

## Common Failures

| Message | Meaning / Fix |
|---|---|
| `Found X unsigned or tampered code template` | code template unsigned or modified after signing; examine, then `-sign` (writer) or audit carefully (consumer) |
| `re-signing code templates are not allowed for security reasons` | template was signed by someone else; to adopt it: review the code, remove the existing `# digest:` line manually, then `-sign` |
