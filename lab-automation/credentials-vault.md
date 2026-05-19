# Credentials Vault SOP

## Vault User Guide

> [!IMPORTANT]
> This SOP exists to eliminate hard-coded credentials (IP/password/API keys) from GitHub repositories.
>
> - Never commit secrets to Git.
> - Treat Vault tokens like passwords.
> - Prefer short-lived access (TTL) and least-privilege policies.

- [Credentials Vault SOP](#credentials-vault-sop)
  - [Vault User Guide](#vault-user-guide)
    - [Document control](#document-control)
    - [Purpose and scope](#purpose-and-scope)
    - [Definitions](#definitions)
    - [Governance requirements](#governance-requirements)
    - [Quick start](#quick-start)
    - [1) Admin (Vault server operator)](#1-admin-vault-server-operator)
      - [1.1 Access lifecycle (onboarding, changes, offboarding)](#11-access-lifecycle-onboarding-changes-offboarding)
      - [1.2 Break-glass (root token / unseal keys)](#12-break-glass-root-token--unseal-keys)
    - [2) Members (Vault service user)](#2-members-vault-service-user)
      - [2.1 Prerequisites](#21-prerequisites)
      - [2.2 Login](#22-login)
      - [2.3 Repo project usage (recommended pattern)](#23-repo-project-usage-recommended-pattern)
        - [2.3.1 Golden rules (mandatory)](#231-golden-rules-mandatory)
        - [2.3.2 Where the Vault token should live (safe options)](#232-where-the-vault-token-should-live-safe-options)
        - [2.3.3 Repo layout (recommended)](#233-repo-layout-recommended)
        - [2.3.4 Generate `.env` locally (developer workflow)](#234-generate-env-locally-developer-workflow)
        - [2.3.5 How to secure Vault auth in the repository](#235-how-to-secure-vault-auth-in-the-repository)
        - [2.3.6 CI/CD (GitHub Actions) safe pattern](#236-cicd-github-actions-safe-pattern)
        - [2.3.7 Long-running services on a server (best practice)](#237-long-running-services-on-a-server-best-practice)
      - [2.4 SSH access (recommended)](#24-ssh-access-recommended)
      - [2.5 Common member commands](#25-common-member-commands)
      - [2.6 If you accidentally committed a secret](#26-if-you-accidentally-committed-a-secret)
  - [Vault Installation Guide](#vault-installation-guide)
    - [1) Architecture](#1-architecture)
    - [2) Install Vault on Ubuntu](#2-install-vault-on-ubuntu)
    - [3) Configure Vault server (production-ready baseline)](#3-configure-vault-server-production-ready-baseline)
      - [3.1 Create directories](#31-create-directories)
      - [3.2 TLS certificates](#32-tls-certificates)
      - [3.3 Configuration file (Raft + TLS + UI)](#33-configuration-file-raft--tls--ui)
      - [3.4 Start Vault (systemd)](#34-start-vault-systemd)
    - [4) Initialize and unseal (admin-only)](#4-initialize-and-unseal-admin-only)
      - [4.1 Initialize](#41-initialize)
      - [4.2 Unseal](#42-unseal)
      - [4.3 Login as root (only for bootstrap)](#43-login-as-root-only-for-bootstrap)
    - [5) Mandatory hardening steps](#5-mandatory-hardening-steps)
      - [5.1 Enable audit logging](#51-enable-audit-logging)
      - [5.2 Create admin policy + admin auth (do not use root daily)](#52-create-admin-policy--admin-auth-do-not-use-root-daily)
    - [6) Secrets engine for repo credentials (KV v2)](#6-secrets-engine-for-repo-credentials-kv-v2)
    - [7) Access control (ACL) for projects](#7-access-control-acl-for-projects)
      - [7.1 Read-only developer policy (example)](#71-read-only-developer-policy-example)
      - [7.2 AppRole for CI / servers (recommended)](#72-approle-for-ci--servers-recommended)
    - [8) Vault-backed SSH access (server setup)](#8-vault-backed-ssh-access-server-setup)
      - [8.1 Enable SSH secrets engine](#81-enable-ssh-secrets-engine)
      - [8.2 Configure as CA](#82-configure-as-ca)
      - [8.3 Configure target Ubuntu servers to trust the Vault SSH CA](#83-configure-target-ubuntu-servers-to-trust-the-vault-ssh-ca)
      - [8.4 Create SSH signing roles and policies](#84-create-ssh-signing-roles-and-policies)
    - [9) Operating procedures (day-2 ops)](#9-operating-procedures-day-2-ops)
      - [9.1 Service management](#91-service-management)
      - [9.2 Seal / unseal](#92-seal--unseal)
      - [9.3 Backups (Raft snapshot)](#93-backups-raft-snapshot)
      - [9.4 Rotation and revocation](#94-rotation-and-revocation)
      - [9.5 Audit review](#95-audit-review)
    - [10) Repo project rules (mandatory)](#10-repo-project-rules-mandatory)
    - [11) Minimal quick start checklist](#11-minimal-quick-start-checklist)
      - [Admin (quick start)](#admin-quick-start)
      - [Lab member (quick start)](#lab-member-quick-start)


### Document control

| Field | Value |
| --- | --- |
| Document name | Credentials Vault SOP |
| System | HashiCorp Vault (Community Edition) |
| Owner | Lab Admin Team |
| Approver | Lab Admin Lead (or delegated maintainer) |
| Applies to | All lab GitHub repositories and lab-managed servers |
| Versioning | Tracked by Git commit history |
| Review cadence | At least once per semester, and after any credential leak or major infrastructure change |

### Purpose and scope

This SOP defines how the lab stores, accesses, and rotates credentials using **HashiCorp Vault (Community Edition)**.

In scope:

- Shared credentials used by code (database passwords, API keys, service accounts)
- Server access credentials (SSH via short-lived certificates)
- CI/CD secret access (GitHub Actions via AppRole)

Out of scope:

- Personal password managers for individual accounts
- OS user management (Linux users/groups are managed on servers)

### Definitions

- **Vault token:** A bearer credential used to access Vault APIs, scoped by policies and TTL.
- **Policy:** A set of permissions (ACL) defining which Vault paths an identity can access.
- **KV v2:** Vault key-value secrets engine with versioning.
- **OIDC:** Single sign-on for humans.
- **AppRole:** Machine identity auth method for CI and services.
- **SSH Client Signer:** Vault acting as an SSH Certificate Authority (CA) to issue short-lived user certificates.

### Governance requirements

- No secrets (values) are permitted in Git, including tests, examples, or documentation.
- Root token is for bootstrap only and must be stored offline under admin control.
- All non-human access must use AppRole (or equivalent) with least privilege.
- Audit logging must be enabled and reviewed periodically.
- Exceptions must be explicitly approved by the lab admin lead and documented with scope and expiry.
- Non-compliance may result in credential rotation and access revocation.

### Quick start

**Admins:** follow the Vault Installation Guide, then create policies + AppRole for each project.

**Members:** login, then generate runtime environment variables (Vault-backed `.env` workflow below) and/or request SSH certificates.

### 1) Admin (Vault server operator)

**Goal:** Operate the Vault service VM and enforce access rights.

Responsibilities:

- Operate Vault on the dedicated VM (install, start/stop, backup).
- Create/maintain policies per project and environment.
- Onboard/offboard users and CI identities.
- Rotate and revoke secrets/tokens when needed.

#### 1.1 Access lifecycle (onboarding, changes, offboarding)

- **Onboarding:** assign users to the correct project/environment policies (prefer OIDC group mapping). Verify least privilege before granting access.
- **Access changes:** treat any policy change as a controlled change; record what changed, who approved, and why.
- **Offboarding:** remove user access (OIDC group removal or userpass disable), revoke active tokens if needed, and rotate any shared credentials the user could access.

#### 1.2 Break-glass (root token / unseal keys)

- Root token is for initial bootstrap and emergency recovery only.
- Unseal keys and root token must be stored offline under admin control.
- Break-glass usage must be documented (who, when, why, what action was taken).

> [!IMPORTANT]
> Do not use the root token for daily operations. Create an admin identity and policy, then lock away the root token offline.

Common admin operations:

```bash
# Check service status from an admin workstation
export VAULT_ADDR='https://vault.lab.example:8200'
vault status

# Seal/unseal (admin-only)
vault operator seal
vault operator unseal

# Create/update a policy
vault policy write <policy-name> <policy-file.hcl>

# Create a user (if using userpass)
vault write auth/userpass/users/<user-id> password='<strong-password>' policies='<policy1,policy2>'

# Revoke a leaked token
vault token revoke <token>
```

### 2) Members (Vault service user)

**Goal:** Use Vault safely to access lab resources without sharing passwords or committing secrets.

Members must not store or share long-lived secrets. Use Vault authentication to retrieve short-lived access at runtime.

#### 2.1 Prerequisites

- You have a Vault account and the correct policies from the admin.
- You can reach the Vault server (VPN if required).
- You have the Vault CLI installed:
  - macOS: `brew install vault`
  - Ubuntu/Debian: install `vault` from the HashiCorp apt repo (admin can provide a one-liner).

#### 2.2 Login

```bash
export VAULT_ADDR='https://vault.lab.example:8200'

# OIDC (preferred, if enabled)
vault login -method=oidc

# OR: userpass (if enabled)
vault login -method=userpass username="<your-id>"
```

> [!CAUTION]
> Treat your Vault token like a password. Never paste tokens into chat, issues, or emails.

#### 2.3 Repo project usage (recommended pattern)

Objective: allow repository code to load credentials at runtime (server host/user/password/keys) without committing secret values.

Inputs:

- `VAULT_ADDR`
- A valid Vault login method (OIDC/userpass for humans; AppRole for machines)
- Project-approved Vault paths and policies

Outputs:

- Runtime environment variables for the app process
- Optional local `.env` file (generated, not committed)

Member procedure:

1. Authenticate to Vault (see Login section) and ensure `VAULT_ADDR` is set.
2. Generate env vars at runtime using the repo's non-secret mapping file:

    - Preferred: `env.yaml` + a project-provided loader (recommended for larger projects).
    - Alternative: `vault.envmap` + the bash loop in section 2.3.4 (recommended for minimal dependencies).

3. Run the application with the generated env vars.
4. Remove local `.env` files when not required.

Token handling requirement:

- Vault requires an auth credential (token) to read secrets.
- Tokens must be obtained and stored outside the repository.
- Git ignore rules are a backstop. The primary rule is: the repo never contains tokens.

##### 2.3.1 Golden rules (mandatory)

- Never store secrets or Vault tokens in Git (even in private repos).
- Do not commit `.env` with real values.
- For CI/services (non-human), do **not** use human login tokens. Use **AppRole**.
- Use short-lived tokens (TTL) and least-privilege policies.
- Keep secret values in Vault only. Repos store names, paths, and mappings only.

##### 2.3.2 Where the Vault token should live (safe options)

Choose based on where your code runs:

- **Developer laptop (human user):** run `vault login`. The CLI stores the token in `~/.vault-token` (outside the repo).
- **Developer laptop (per-project, if you really need it):** keep a gitignored token file like `.vault-token` in the repo root with `chmod 600 .vault-token`. Prefer the default `~/.vault-token` when possible.
- **CI/CD:** store `VAULT_ROLE_ID` and `VAULT_SECRET_ID` in the CI secret store, then exchange them for a short-lived token at runtime.
- **Long-running server services:** prefer Vault Agent with AppRole; store AppRole material in root-owned paths (e.g., `/etc/<project>/vault/`), not in the git checkout.

##### 2.3.3 Repo layout (recommended)

In each project repo:

- `.env.example` (committed)
- `.env` (NOT committed; generated locally or on server)
- Recommended: `env.yaml` (committed, non-secret) describing how env vars map to Vault fields
- Optional: `vault.envmap` (committed, non-secret) for a minimal, no-dependency workflow

Member runtime usage examples:

- If the repo provides a loader for `env.yaml`, the workflow should be equivalent to:

  ```bash
  export VAULT_ADDR='https://vault.lab.example:8200'
  vault login -method=oidc

  # Example only: the actual command depends on the project.
  # It should read env.yaml, fetch required fields from Vault, then run the app.
  ./scripts/run_with_vault_env.sh --env dev
  ```

- If the repo uses `vault.envmap`, use the bash loop in section 2.3.4 to generate `.env` and run the app.

Recommended conventions:

- One Vault path per component or server.
- For multiple servers, use a stable prefix per server (e.g., `SERVER_A_*`, `SERVER_B_*`).
- Do not place environment names in code. Use `dev/staging/prod` in the Vault path.

Example `env.yaml` (mapping only; values live in Vault):

```yaml
version: 1

# Each key under `env:` becomes an environment variable.
# For secrets, reference a Vault KV v2 path + field.
env:
  HOST:
    vault:
      path: secret/lab/myproj/dev/server
      field: host
  PASS:
    vault:
      path: secret/lab/myproj/dev/server
      field: password
```

Example `env.yaml` for multiple servers (recommended pattern: one Vault path per server, env vars are prefixed):

```yaml
version: 1

env:
  # Server A
  SERVER_A_HOST:
    vault:
      path: secret/lab/myproj/dev/servers/server-a
      field: host
  SERVER_A_USER:
    vault:
      path: secret/lab/myproj/dev/servers/server-a
      field: username
  SERVER_A_PASS:
    vault:
      path: secret/lab/myproj/dev/servers/server-a
      field: password

  # Server B
  SERVER_B_HOST:
    vault:
      path: secret/lab/myproj/dev/servers/server-b
      field: host
  SERVER_B_USER:
    vault:
      path: secret/lab/myproj/dev/servers/server-b
      field: username
  SERVER_B_PASS:
    vault:
      path: secret/lab/myproj/dev/servers/server-b
      field: password
```

Example values stored in HashiCorp Vault (KV v2):

```bash
# Admin (or CI/service) writes secrets to Vault
vault kv put secret/lab/myproj/dev/server \
  host="10.42.0.18" \
  password="CorrectHorseBatteryStaple!"

# Multiple servers (one path per server)
vault kv put secret/lab/myproj/dev/servers/server-a \
  host="10.42.0.21" \
  username="ubuntu" \
  password="A-Strong-Pass!"

vault kv put secret/lab/myproj/dev/servers/server-b \
  host="10.42.0.22" \
  username="ubuntu" \
  password="B-Strong-Pass!"

# Member/app reads specific fields at runtime
vault kv get -field=host secret/lab/myproj/dev/server
# -> 10.42.0.18

vault kv get -field=password secret/lab/myproj/dev/server
# -> CorrectHorseBatteryStaple!
```

```text
# ENV_VAR=secret/path field
DB_PASSWORD=secret/lab/<project>/<env>/db password
DB_HOST=secret/lab/<project>/<env>/db host
```

##### 2.3.4 Generate `.env` locally (developer workflow)

After `vault login`, generate a local `.env` file. This is a convenience for local development only.

Requirement: `.env` must never be committed.

```bash
export VAULT_ADDR='https://vault.lab.example:8200'

rm -f .env
while read -r line; do
  [[ -z "$line" || "$line" =~ ^# ]] && continue
  var="${line%%=*}"
  rest="${line#*=}"
  path="${rest% *}"
  field="${rest##* }"
  value="$(vault kv get -field="$field" "$path")"
  printf '%s=%q\n' "$var" "$value" >> .env
done < vault.envmap

# run your app with the generated env
set -a && source .env && set +a
python my_app.py
```

##### 2.3.5 How to secure Vault auth in the repository

**Should we ignore the token?** Yes — but also **don’t store tokens in the repo at all**.

Member requirements:

- Do not copy `~/.vault-token` into the repo.
- If a per-project `.vault-token` is used, it must be gitignored and protected with file permissions.
- Do not pass tokens in command-line arguments.

Recommended `.gitignore` entries for projects:

```gitignore
# Local environment files
.env
.env.*

# Accidentally copied Vault token files
.vault-token
vault-token*

# SSH private keys / certs (should never be in repos)
*.pem
.ssh/id_*
.ssh/id_*-cert.pub
```

##### 2.3.6 CI/CD (GitHub Actions) safe pattern

Use **AppRole** and store `ROLE_ID` + `SECRET_ID` in GitHub Actions Secrets.

- GitHub repo settings → Secrets and variables → Actions → add:
  - `VAULT_ADDR`
  - `VAULT_ROLE_ID`
  - `VAULT_SECRET_ID`

At runtime, exchange AppRole for a short-lived token:

```bash
export VAULT_ADDR="$VAULT_ADDR"

export VAULT_TOKEN="$(vault write -field=token auth/approle/login \
  role_id="$VAULT_ROLE_ID" \
  secret_id="$VAULT_SECRET_ID")"

export DB_PASSWORD="$(vault kv get -field=password secret/lab/<project>/<env>/db)"
python my_app.py
```

Security notes:

- The AppRole policy should only allow the minimum read paths required.
- Use TTLs (e.g., 1h) and rotate `SECRET_ID` periodically.

##### 2.3.7 Long-running services on a server (best practice)

For services running on lab servers (systemd), avoid baking tokens into repo config.

Recommended approaches:

- **Vault Agent (recommended):** runs on the server, authenticates via AppRole, and writes secrets to a root-owned file or renders templates; the app reads env vars or files.
- **AppRole login at startup:** systemd unit runs a short script that logs in via AppRole and exports env vars, without writing tokens to the repo.

Minimum requirement:

- Any token or secret material must live outside the git checkout, with OS permissions (e.g., `/etc/<project>/` root-only).
- Never pass tokens via command-line arguments in process lists; prefer environment variables or agent-managed files.

#### 2.4 SSH access (recommended)

We do not share SSH passwords. Instead, Vault issues short-lived SSH certificates.

```bash
# One-time: create an SSH keypair (skip if you already have one)
ssh-keygen -t ed25519 -C "<your-email>" -f ~/.ssh/id_ed25519

# Request a short-lived SSH certificate
vault write -field=signed_key ssh-client-signer/sign/lab \
  public_key=@~/.ssh/id_ed25519.pub \
  valid_principals="<linux-username>" \
  ttl="8h" \
  > ~/.ssh/id_ed25519-cert.pub

# SSH normally (OpenSSH will use the cert automatically)
ssh <linux-username>@<server-ip>
```

Renewing access: re-run the `vault write ... sign/lab` command when the cert expires.

#### 2.5 Common member commands

```bash
vault token lookup
vault kv get secret/lab/<project>/<env>/<name>
vault kv get -field=password secret/lab/<project>/<env>/<name>
vault kv list secret/lab/<project>/<env>
```

#### 2.6 If you accidentally committed a secret

Treat this as a security incident.

1. **Contain:** stop using the credential immediately; do not share it further.
2. **Notify:** inform the lab admin team with the repo name, file path, and approximate time/commit.
3. **Revoke/rotate:** admins must revoke the relevant Vault token(s) and rotate the underlying secret value.
4. **Remediate Git history:** remove the secret from Git history (do not rely on deleting a line in a new commit).
5. **Verify:** confirm the credential is no longer valid and services are updated.
6. **Prevent recurrence:** enable/verify secret scanning (pre-commit and CI) and add/confirm `.gitignore` rules.

---

## Vault Installation Guide

> [!WARNING]
> This is for **admins only**. A production Vault deployment must use:
>
> - TLS (HTTPS)
> - persistent storage (e.g., integrated Raft)
> - audit logging
> - strong access control policies
> - a safe unseal process (multiple key holders)

### 1) Architecture

- **Infrastructure:** 1 Dedicated Virtual Machine (VM).
- **OS:** Ubuntu 22.04 LTS or 24.04 LTS.
- **Storage:** Integrated Raft Storage (Local disk on VM).
- **Access:**
  - Humans: OIDC (preferred) or userpass
  - CI / services: AppRole
- **Secrets Engines:**
  - KV v2 for app credentials
  - SSH Client Signer for SSH certificates
- **Network controls:**
  - Only allow port `8200/tcp` from VPN/internal subnets

### 2) Install Vault on Ubuntu

1. **Add the HashiCorp GPG Key and Repository**

    ```bash
    sudo apt update && sudo apt install gpg
    wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    ```

2. **Install Vault**

    ```bash
    sudo apt update && sudo apt install vault
    ```

3. **Verify Installation**

    ```bash
    vault version
    ```

### 3) Configure Vault server (production-ready baseline)

#### 3.1 Create directories

```bash
sudo mkdir -p /etc/vault.d /etc/vault.d/tls /opt/vault/data /var/log/vault
sudo chown -R vault:vault /opt/vault /var/log/vault /etc/vault.d
sudo chmod 0750 /etc/vault.d
sudo chmod 0750 /etc/vault.d/tls
```

#### 3.2 TLS certificates

Use a certificate trusted by lab machines (internal CA or public CA).

- Certificate: `/etc/vault.d/tls/vault.crt`
- Key: `/etc/vault.d/tls/vault.key`

> [!IMPORTANT]
> Production Vault must use HTTPS. Do not expose Vault over plain HTTP.

#### 3.3 Configuration file (Raft + TLS + UI)

Create `/etc/vault.d/vault.hcl`:

```hcl
ui = true

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/etc/vault.d/tls/vault.crt"
  tls_key_file  = "/etc/vault.d/tls/vault.key"
}

storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-1"
}

api_addr     = "https://vault.lab.example:8200"
cluster_addr = "https://vault.lab.example:8201"

disable_mlock = true
```

> [!NOTE]
> `disable_mlock = true` is often required in VMs/containers. If you can support `mlock`, enable it for better protection.

#### 3.4 Start Vault (systemd)

The HashiCorp package includes a systemd unit. Enable and start it:

```bash
sudo systemctl enable vault
sudo systemctl start vault
sudo systemctl status vault --no-pager
```

### 4) Initialize and unseal (admin-only)

On the Vault server (or admin workstation with network access):

```bash
export VAULT_ADDR='https://vault.lab.example:8200'
vault status
```

#### 4.1 Initialize

Use multiple unseal keys (Shamir) so no single person can unseal alone.

```bash
vault operator init -key-shares=5 -key-threshold=3
```

#### 4.2 Unseal

Run 3 times with 3 different unseal keys:

```bash
vault operator unseal
```

#### 4.3 Login as root (only for bootstrap)

```bash
vault login
```

### 5) Mandatory hardening steps

#### 5.1 Enable audit logging

```bash
sudo touch /var/log/vault/audit.log
sudo chown vault:vault /var/log/vault/audit.log
sudo chmod 0640 /var/log/vault/audit.log

vault audit enable file file_path=/var/log/vault/audit.log
```

#### 5.2 Create admin policy + admin auth (do not use root daily)

Create `admin.hcl` (example):

```hcl
path "*" {
  capabilities = ["create", "read", "update", "delete", "list", "sudo"]
}
```

Apply it:

```bash
vault policy write lab-admin admin.hcl
```

Then create human authentication:

- **OIDC** (preferred): integrate with your identity provider.
- **userpass** (simple): local users in Vault.

Example: enable userpass:

```bash
vault auth enable userpass
```

Create a named admin user:

```bash
vault write auth/userpass/users/<admin-id> \
  password='<set-strong-password>' \
  policies='lab-admin'
```

Then stop using root token.

### 6) Secrets engine for repo credentials (KV v2)

Enable KV v2 at `secret/`:

```bash
vault secrets enable -path=secret kv-v2
```

**Path convention (recommended):**

`secret/lab/<project>/<env>/<component>`

Examples:

```bash
vault kv put secret/lab/my-app/dev/db \
  host="192.168.1.50" \
  port="5432" \
  username="app" \
  password="<redacted>"
```

### 7) Access control (ACL) for projects

Vault supports access rights via **policies** (HCL).

#### 7.1 Read-only developer policy (example)

Create `my-app-dev-read.hcl`:

```hcl
path "secret/data/lab/my-app/dev/*" {

  capabilities = ["read"]
}

path "secret/metadata/lab/my-app/dev/*" {

  capabilities = ["list"]
}
```

Apply:

```bash
vault policy write my-app-dev-read my-app-dev-read.hcl
```

Assign to a user:

```bash
vault write auth/userpass/users/<student-id> \
  password='<set-strong-password>' \
  policies='my-app-dev-read'
```

> [!NOTE]
> Prefer mapping policies through OIDC groups if using SSO.

#### 7.2 AppRole for CI / servers (recommended)

Enable AppRole:

```bash
vault auth enable approle
```

Create role:

```bash
vault write auth/approle/role/my-app-ci \
  token_policies="my-app-dev-read" \
  token_ttl="1h" \
  token_max_ttl="4h"
```

Generate RoleID / SecretID and store them securely in the CI secret store (NOT in Git):

```bash
vault read auth/approle/role/my-app-ci/role-id
vault write -force auth/approle/role/my-app-ci/secret-id
```

### 8) Vault-backed SSH access (server setup)

We use the **SSH Client Signer** engine (Vault acts as an SSH CA).

#### 8.1 Enable SSH secrets engine

```bash
vault secrets enable -path=ssh-client-signer ssh
```

#### 8.2 Configure as CA

```bash
vault write ssh-client-signer/config/ca generate_signing_key=true
```

Fetch the CA public key:

```bash
vault read -field=public_key ssh-client-signer/config/ca
```

#### 8.3 Configure target Ubuntu servers to trust the Vault SSH CA

On each Ubuntu target server:

1. Create a file for the trusted CA key:

```bash
sudo mkdir -p /etc/ssh
sudo tee /etc/ssh/trusted-user-ca-keys.pem > /dev/null <<'EOF'
<PASTE_PUBLIC_KEY_FROM_VAULT_HERE>
EOF
sudo chmod 0644 /etc/ssh/trusted-user-ca-keys.pem
```

1. Update sshd config to trust it:

```bash
sudo bash -lc 'printf "\nTrustedUserCAKeys /etc/ssh/trusted-user-ca-keys.pem\n" >> /etc/ssh/sshd_config'
sudo systemctl restart ssh
```

> [!NOTE]
> You still need Linux users/groups on the servers (Vault is not a Linux user directory).
> Vault controls who can obtain a valid certificate for a given principal.

#### 8.4 Create SSH signing roles and policies

Create a signing role (example `lab`):

```bash
vault write ssh-client-signer/roles/lab \
  allowed_users="*" \
  allow_user_certificates=true \
  default_extensions="permit-pty" \
  ttl="8h" \
  max_ttl="24h"
```

Create a policy allowing students to sign only via this role:

`ssh-lab-sign.hcl`

```hcl
path "ssh-client-signer/sign/lab" {

  capabilities = ["update"]
}
```

Apply:

```bash
vault policy write ssh-lab-sign ssh-lab-sign.hcl
```

Assign to the appropriate users/groups.

### 9) Operating procedures (day-2 ops)

#### 9.1 Service management

```bash
sudo systemctl status vault --no-pager
sudo systemctl restart vault
```

#### 9.2 Seal / unseal

- Seal (emergency / maintenance):

```bash
vault operator seal
```

- Unseal requires `key-threshold` different key holders:

```bash
vault operator unseal
```

#### 9.3 Backups (Raft snapshot)

Create snapshot:

```bash
vault operator raft snapshot save /opt/vault/backup/vault.snap
```

Store snapshots securely (encrypted storage + restricted access).

#### 9.4 Rotation and revocation

- Revoke a compromised token:

```bash
vault token revoke <token>
```

- Rotate shared credentials stored in KV (example): update the KV secret and restart affected services.

#### 9.5 Audit review

- Audit logs are sensitive and must be access-restricted.
- Periodically review for unusual access patterns (off-hours, repeated denials, large reads).

### 10) Repo project rules (mandatory)

1. No credentials in source code (including test files).
2. Provide `.env.example` with variable names only (no values).
3. Devs fetch secrets at runtime via Vault (Pattern A preferred).
4. CI uses AppRole (or equivalent) with minimal scope.
5. Access rights are managed through Vault policies (per project + per environment).

### 11) Minimal quick start checklist

#### Admin (quick start)

- [ ] Install Vault service with TLS
- [ ] Initialize/unseal; store keys safely
- [ ] Enable audit log
- [ ] Enable KV v2 at `secret/`
- [ ] Create project policies
- [ ] Configure SSH signer + server trust (optional but recommended)


#### Lab member (quick start)

- [ ] Set `VAULT_ADDR`
- [ ] Login (OIDC/userpass)
- [ ] Fetch app secrets into env vars
- [ ] Obtain SSH cert and connect

