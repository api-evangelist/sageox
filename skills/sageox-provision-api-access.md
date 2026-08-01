---
name: Provision programmatic API access
description: Confirm identity, mint a SageOx API key, and verify service health before driving the API from an agent or CI job.
api: openapi/sageox-openapi-original.json
operations:
  - getCurrentUser
  - createAPIKey
  - listAPIKeys
  - getReadiness
---

# Provision programmatic API access

Bootstrap a machine/agent integration against the SageOx API.

## Auth
All calls require `Authorization: Bearer <JWT>`. Interactive users authenticate
via OAuth (authorization-code + PKCE S256); agents/CLI use the device flow. See
`authentication/sageox-authentication.yml`.

## Steps
1. **Confirm identity** — `GET /api/v1/users/me` (`getCurrentUser`) to verify the
   token resolves to the expected user and tier.
2. **Mint a key** — `POST /api/v1/api-keys` (`createAPIKey`). Store the returned
   secret securely; it is shown once.
3. **Audit** — `GET /api/v1/api-keys` (`listAPIKeys`) to confirm the key exists
   and to review/rotate existing keys.
4. **Health-check** — `GET /ready` (`getReadiness`) before driving traffic; a 503
   means a dependency is degraded, so retry with backoff.

## Conventions
- Respect rate-limit signaling: `429` + `Retry-After` /
  `X-SageOx-Rate-Limit-Remaining` / `X-SageOx-Quota-Remaining`
  (`conventions/sageox-conventions.yml`).
- Errors use the `{ success:false, error }` envelope; branch on HTTP status.
