---
name: manage-secrets
description: Manage encrypted secrets with Ansible Vault and OpenSSL. View, encrypt, decrypt files; encrypt strings; generate RSA keys; file digests. Use when managing vault files or rotating secrets.
---

# Manage Secrets

Manage encrypted secrets using Ansible Vault and OpenSSL.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | view, encrypt, decrypt, rotate, generate |
| `file_path` | string | "" | Path to secrets file |
| `variable_name` | string | "" | Variable name for string encryption |

## Persona

Load **infra** persona (ansible vault, openssl, ssh tools).

## Workflow

### 1. Bootstrap
- `persona_load("infra")`
- `check_known_issues("ansible_vault", "")` — known vault issues

### 2. Vault Operations
- If action=view and file_path: `ansible_vault_view(file=file_path)`
- If action=encrypt and file_path: `ansible_vault_encrypt(file=file_path)`
- If action=decrypt and file_path: `ansible_vault_decrypt(file=file_path)`
- If action=encrypt and variable_name: `ansible_vault_encrypt_string(name=variable_name)` — prompts for value

### 3. OpenSSL Operations
- If action=generate: `openssl_genrsa(bits=4096)`
- If file_path: `openssl_dgst(file=file_path, algorithm="sha256")` — file digest

### 4. SSH Fingerprint
- If file_path (SSH key): `ssh_fingerprint(key_file=file_path)`

### 5. Error Handling
- On "vault password" or "decryption failed": `learn_tool_fix("ansible_vault", "vault password", "Password not provided or incorrect", "Set ANSIBLE_VAULT_PASSWORD_FILE or --vault-password-file")`
- On "no such file": verify file path

### 6. Session Log
- `memory_session_log("Manage secrets: {action}", "file={file_path}")`

## Related Skills

- `ansible_configure_vm` — use vault files in playbooks
