# OpenID Connect (OIDC) — The Identity Layer on OAuth 2.0

*Learned: 2026-01-09*

## Context

Learning about Workload Identity Federation in CI/CD led to deeper exploration of how OIDC enables keyless authentication between identity providers.

## Insight

### 1. OAuth 2.0 vs OIDC: Authorization vs Authentication

OAuth 2.0 answers "what can this client do?" (authorization). OIDC answers "who is this user?" (authentication). OIDC is literally OAuth 2.0 + an identity layer.

When you see "Login with Google" — that's OIDC.

### 2. The ID Token — OIDC's Key Innovation

OAuth 2.0 returns an access token (opaque, for API calls). OIDC adds an **ID token** — a signed JWT containing user claims:

| Claim | Description |
|-------|-------------|
| `iss` | Issuer (who issued the token) |
| `sub` | Subject (unique user ID) |
| `email` | User's email |
| `name` | User's display name |
| `iat` | Issued at (timestamp) |
| `exp` | Expiration (timestamp) |

The relying party (your app) can verify this JWT locally without calling the IdP again.

### 3. The Authorization Code Flow (Most Common)

```
User → App: "Login"
App → IdP: Redirect to /authorize?response_type=code&client_id=...&scope=openid email
User → IdP: Authenticates (password, MFA, etc.)
IdP → App: Redirect with ?code=abc123
App → IdP: POST /token (code + client_secret)
IdP → App: { access_token, id_token, refresh_token }
```

The `openid` scope is required — it signals "this is OIDC, give me an ID token."

### 4. PKCE Extension — Public Client Security

SPAs and mobile apps can't keep secrets. PKCE (Proof Key for Code Exchange) adds a challenge:

1. App generates random `code_verifier`, hashes it to `code_challenge`
2. Sends `code_challenge` with `/authorize`
3. Sends `code_verifier` with `/token`
4. IdP verifies hash matches — proves the same app initiated and completed the flow

This is why Workload Identity Federation works for GitHub Actions — GitHub issues OIDC tokens with claims like `repository` and `ref`, and GCP validates them without shared secrets.

## Sources

- [OpenID Foundation - How Connect Works](https://openid.net/developers/how-connect-works/)
- [Microsoft - What Is OIDC](https://www.microsoft.com/en-us/security/business/security-101/what-is-openid-connect-oidc)
- [FusionAuth - What Is OIDC](https://fusionauth.io/articles/identity-basics/what-is-oidc)
