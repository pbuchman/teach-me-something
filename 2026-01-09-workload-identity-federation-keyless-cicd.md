# Workload Identity Federation for Keyless CI/CD

*Learned: 2026-01-09*

## Context

Exploring GitHub Actions deploy workflow and Cloud Build triggers in a monorepo, discovered WIF pattern for secure authentication.

## Insight

Google Cloud's Workload Identity Federation (WIF) replaces static service account keys with short-lived, automatically managed credentials. This is the modern standard for cloud CI/CD authentication.

### 1. The Problem with Service Account Keys

Traditional approach: store a JSON key file as a GitHub secret.

Issues:
- Keys don't expire (unless manually rotated)
- If leaked, attacker has permanent access until revoked
- Keys are long-lived credentials that violate zero-trust principles

### 2. How WIF Works

Instead of keys, WIF establishes trust between identity providers:

```
GitHub Actions requests OIDC token from GitHub
    ↓
Presents token to GCP
    ↓
GCP validates token against configured trust rules
    ↓
Issues short-lived GCP credentials
```

In GitHub Actions:

```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/123/locations/global/workloadIdentityPools/pool/providers/github'
    service_account: 'deploy@project.iam.gserviceaccount.com'
```

No secrets stored — authentication happens via OIDC token exchange.

### 3. Attribute Conditions as Security Boundary

Terraform configures which GitHub repos/branches can authenticate:

```hcl
resource "google_iam_workload_identity_pool_provider" "github" {
  attribute_condition = "assertion.repository == 'owner/repo'"
  # or more restrictive:
  # attribute_condition = "assertion.repository == 'owner/repo' && assertion.ref == 'refs/heads/main'"
}
```

Even if someone forks your repo, their Actions can't authenticate — the repository claim in the OIDC token won't match.

### 4. Short-Lived Tokens

WIF tokens typically expire in 1 hour. Benefits:
- Even if intercepted, blast radius is limited
- No rotation needed — every CI run gets fresh credentials
- Follows principle of least privilege temporally
