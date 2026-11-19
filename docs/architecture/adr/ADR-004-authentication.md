# ADR-004: Authentication & Identity Management Architecture

**Date:** 2026-10-12  
**Status:** proposed (Revised for Phase 1)  
**Deciders:** Principal Architect & Security Engineer  

---

## Context

Authentication and authorization in My Health App are critical safety and privacy boundaries. Requirements:
1. Support Email/Password authentication (registration, verification, login, password recovery).
2. Support native Google Sign-in on Android and iOS.
3. Cryptographically sound token lifecycle (short-lived access tokens, rotatable refresh tokens, revocation).
4. Strict multi-tenant authorization: A patient must never access another patient's data. Server must never trust client-provided `patient_profile_id`.
5. Product Rule: **Never ship placeholder or non-functional authentication controls that imply working functionality.**

We must evaluate managed vs. self-hosted options across security, cost, Google sign-in integration, email verification, recovery, session revocation, portability, and operational burden before finalizing the provider implementation, and decouple through a formal `AuthProviderAdapter`.

---

## Evaluation of Authentication Options

| Evaluation Dimension | Supabase Auth (Managed BaaS) | Firebase Auth (Google Identity) | Clerk / Auth0 (Auth SaaS) | Keycloak (Self-Hosted OIDC) | Custom Node.js (Argon2id + JWT in Monolith) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Security & Cryptography** | High (Managed bcrypt/Argon2, PKCE, secure token rotation) | High (Google security infrastructure, PKCE) | Very High (Enterprise security, SOC2, WebAuthn) | High (OpenID Certified, enterprise hardened) | Medium-High (Requires self-maintaining crypto, rate limiting, and salt rotation) |
| **Cost & Scaling** | Generous free tier (50k MAU), predictable tiers | Free tier (50k MAU), pay-as-you-go | Expensive at scale ($0.02 - $0.05/MAU after free tier) | Free software, but compute/DB costs apply | Free (Compute/DB bundled in monolith) |
| **Google Sign-In** | Built-in native token verification & OAuth | Industry standard, native React Native SDK | High, easy social connection setup | Moderate (Requires configuring Google OIDC identity broker) | Requires manual Google JWKS verification endpoint |
| **Email Verification & Recovery** | Out-of-the-box templates, rate limits, custom SMTP support | Out-of-the-box email links/codes | Best-in-class email templates and passwordless flows | Robust built-in email workflows | Must implement custom tokens, hashing, expiration, and SMTP logic |
| **Session Revocation** | Native refresh token rotation and instant revocation APIs | Revocation via Firebase Admin SDK / token invalidation | Instant dashboard & API session revocation | Backchannel logout and token revocation | Must maintain token revocation blocklist / DB session table |
| **Data Sovereignty & Portability** | High (Direct PostgreSQL `auth.users` tables; easily exportable) | Low (Google proprietary identity store; passwords cannot be exported) | Low-Medium (User data in proprietary SaaS cloud) | High (100% self-hosted, full database ownership) | Highest (100% owned in application PostgreSQL database) |
| **Operational Burden** | Very Low (Zero infra maintenance; managed service) | Very Low (Managed service) | Zero (Pure SaaS) | Very High (Complex Java service, heavy memory footprint, requires separate updates) | Medium (Must maintain rate limiters, token rotation, migration scripts) |

---

## Architectural Recommendation & Decision

### 1. Retention of `AuthProviderAdapter`
To isolate application business logic and the mobile client from the specific identity backend, we implement a decoupled adapter pattern:

```typescript
export interface AuthSession {
  userId: string;
  patientProfileId: string;
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

export interface AuthProviderAdapter {
  signUpWithEmail(email: string, password: string): Promise<{ userId: string; requiresVerification: boolean }>;
  verifyEmail(token: string): Promise<boolean>;
  signInWithEmail(email: string, password: string): Promise<AuthSession>;
  signInWithGoogle(idToken: string): Promise<AuthSession>;
  refreshToken(refreshToken: string): Promise<AuthSession>;
  signOut(session: AuthSession): Promise<void>;
  requestPasswordReset(email: string): Promise<void>;
  resetPassword(token: string, newPassword: string): Promise<void>;
  verifySessionToken(token: string): Promise<{ userId: string; email: string }>;
}
```

### 2. Provider Recommendation: **Supabase Auth or Node.js Modular Engine**
- **Primary Recommendation:** **Supabase Auth (Cloud or Self-Hosted)**
  - *Rationale:* Combines managed security (PKCE, token rotation, automated rate-limiting, Google Sign-In) with high data portability (identities live in PostgreSQL and can be queried or migrated directly). Eliminates the operational burden of building custom cryptographic reset routines while keeping infrastructure costs minimal.
- **Secondary Portable Alternative:** **Embedded Node.js Engine (Argon2id + JWT)**
  - *Rationale:* Zero external dependencies, ideal for fully offline test suites and air-gapped environments.
- **Policy:** The backend will support the `AuthProviderAdapter` interface. During Phase 1 foundation work, the adapter contract, session token verification middleware, and domain models (`users`, `patient_profiles`) will be established. The final provider implementation will be wired based on stakeholder sign-off prior to releasing authentication screens to users. **No placeholder or non-functional auth buttons will be shown in the UI.**

### 3. Server-Side Ownership & Anti-Spoofing Invariant
- Every API endpoint that handles patient-owned data (`documents`, `prescriptions`, `medications`, `dose_events`, `reports`) resolves `patient_profile_id` exclusively from the verified session context (`req.authContext = { userId, patientProfileId }`).
- Any client-supplied `patient_profile_id` in request headers, bodies, or query params is strictly ignored.
- Automated cross-tenant tests must verify that User A receives `404 Not Found` or `403 Forbidden` when attempting to access User B's resources.

---

## Consequences

### Positive
- The mobile UI and backend domain services are decoupled from the specific authentication vendor.
- Security and session revocation are rigorously specified before writing code.
- No misleading UI mockups or nonfunctional login screens.

### Negative
- Requires implementing the adapter layer and mock adapter for unit tests.
