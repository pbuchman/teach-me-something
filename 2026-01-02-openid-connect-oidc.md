# OpenID Connect (OIDC) — Identity Layer on OAuth 2.0

*Learned: 2026-01-02*

## Context

Working with Auth0 authentication in a web app. Understanding how OIDC builds on OAuth 2.0 to provide user identity, not just authorization. The `sub` claim in the ID token is what we use as userId throughout the system.

## Insight

OIDC answers: "Who is this user?" while OAuth 2.0 only answers: "What can this token access?"

### The Key Addition: ID Token

```
OAuth 2.0:  access_token (opaque, for API calls)
OIDC:       access_token + id_token (JWT with user claims)
```

### ID Token Anatomy

```json
{
  "iss": "https://your-tenant.auth0.com/",  // Who issued it
  "sub": "auth0|abc123",                     // User ID (stable!)
  "aud": "your-client-id",                   // Intended recipient
  "exp": 1735812000,                         // Expiration
  "iat": 1735808400,                         // Issued at
  "name": "John Doe",                        // Profile claims
  "email": "john@example.com"
}
```

### Why `sub` Matters

- It's the ONLY stable user identifier
- Email can change, name can change — `sub` never changes
- This is what you store in your database as `userId`

### OIDC Flows

| Flow | Use Case |
|------|----------|
| Authorization Code | Web apps (most secure) |
| Authorization Code + PKCE | SPAs, mobile (what Auth0 recommends) |
| Implicit | Legacy SPAs (deprecated) |
| Client Credentials | Machine-to-machine (no user) |

### Discovery Endpoint

```
https://your-tenant.auth0.com/.well-known/openid-configuration
```

Returns all endpoints, supported scopes, signing keys (JWKS) — everything a client needs to integrate.

## Sources

- [OpenID Connect Core 1.0 Spec](https://openid.net/specs/openid-connect-core-1_0.html)
- [Auth0 OIDC Handbook](https://auth0.com/docs/authenticate/protocols/openid-connect-protocol)
