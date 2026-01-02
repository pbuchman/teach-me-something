# Token Exchange Pattern (RFC 8693)

*Learned: 2026-01-02*

## Context

Building a web app with Auth0 for user authentication but needing Firebase Firestore real-time listeners, which require Firebase Auth. Created a `/auth/firebase-token` endpoint to exchange Auth0 JWT for Firebase custom token.

## Insight

Token exchange is when you trade a token from one identity system for a token in another. It bridges auth systems without duplicating user databases.

### The Flow

```
[User] → Auth0 JWT → [Your API] → Firebase Custom Token → [Firebase SDK]
                          ↓
                    Validates JWT
                    Extracts userId
                    Calls createCustomToken(userId)
```

### Why not just use the original token everywhere?

- Firebase Security Rules need `request.auth.uid` — only Firebase tokens have this
- Different systems have different token formats/claims
- Principle of least privilege — the exchanged token can have narrower scope

### Three Exchange Patterns

1. **Delegation** (what we did) — Service creates token on behalf of user
2. **Impersonation** — Token lets you act AS that user (more dangerous)
3. **Federation** — External IdP token accepted directly (OIDC federation)

### Security Considerations

- The exchange endpoint must validate the incoming token fully
- Exchanged tokens should be short-lived
- Log all exchanges for audit trail

## Sources

- [RFC 8693 OAuth 2.0 Token Exchange](https://datatracker.ietf.org/doc/html/rfc8693)
- [Firebase Custom Tokens](https://firebase.google.com/docs/auth/admin/create-custom-tokens)
