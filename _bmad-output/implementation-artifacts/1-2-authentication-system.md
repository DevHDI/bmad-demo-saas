---
status: done
completedAt: "2026-01-25"
epic: "Epic 1: Platform Foundation"
storyPoints: 8
---

# Story 1-2: Authentication & SSO System

## Description

Build a comprehensive authentication system supporting multiple authentication methods including traditional email/password, passwordless magic links, and enterprise Single Sign-On (SSO) via Google Workspace and Okta SAML. This system serves as the foundation for all user access control and must be secure, scalable, and user-friendly.

The authentication architecture follows industry best practices with bcrypt password hashing (cost factor 12), JWT tokens with RSA-256 signing for stateless authentication, and automatic refresh token rotation to prevent token theft. Magic links provide passwordless authentication for improved user experience, while SSO integration enables seamless enterprise adoption without requiring users to manage separate credentials.

Security is paramount in this implementation. All authentication flows are protected against common vulnerabilities including timing attacks, CSRF, session fixation, and brute force attacks. Rate limiting is applied to login endpoints, and audit logging captures all authentication events for security monitoring and compliance.

## Acceptance Criteria

- AC-1: Email/password authentication with bcrypt hashing (cost factor 12) and password complexity validation
- AC-2: Magic link generation and verification with time-limited tokens (15-minute expiry)
- AC-3: Google Workspace OAuth 2.0 integration with automatic user provisioning
- AC-4: Okta SAML 2.0 integration with identity federation and JIT (Just-In-Time) user creation
- AC-5: JWT access tokens (15-minute expiry) and refresh tokens (30-day expiry) with RSA-256 signing
- AC-6: Automatic refresh token rotation on use with token family tracking for breach detection
- AC-7: Multi-factor authentication (MFA) support with TOTP (Time-based One-Time Password)
- AC-8: Account lockout after 5 failed login attempts with exponential backoff
- AC-9: Password reset flow with secure token generation and email delivery
- AC-10: Session management with device tracking and revocation capabilities

## Technical Notes

**Authentication Stack**:
- NextAuth.js for authentication flow orchestration
- bcrypt for password hashing with salt rounds = 12
- jsonwebtoken for JWT signing/verification with RSA-256 keys
- @okta/saml2 for SAML assertion validation
- Google OAuth 2.0 client library for Workspace integration
- otplib for TOTP MFA token generation/validation

**Security Implementations**:
- Rate limiting: 5 attempts per 15 minutes per IP/email combination
- CSRF protection via double-submit cookies
- Secure cookie flags: HttpOnly, Secure, SameSite=Strict
- Token rotation: New refresh token issued on every use, old token invalidated
- Password requirements: Min 12 characters, uppercase, lowercase, number, special character

**Database Schema**:
```sql
users: id, email, password_hash, email_verified, mfa_enabled, created_at
auth_sessions: id, user_id, refresh_token_hash, device_info, last_used_at
sso_connections: id, tenant_id, provider, config, enabled
auth_audit_log: id, user_id, event_type, ip_address, user_agent, timestamp
```

**Edge Cases**:
- Concurrent login attempts with same credentials (prevent duplicate sessions)
- SSO user email changes (handle email conflicts and account linking)
- Refresh token reuse detection (token family breach detection)
- Magic link click after token expiry (graceful error with retry option)
- SAML assertion replay attacks (check NotBefore/NotOnOrAfter conditions)

## Tasks

- [x] Design authentication database schema with proper indexing
- [x] Implement email/password registration and login flow
- [x] Build password hashing service with bcrypt
- [x] Create magic link generation and email delivery system
- [x] Implement magic link verification and session creation
- [x] Integrate Google Workspace OAuth 2.0 with callback handling
- [x] Build Okta SAML connector with assertion parsing
- [x] Implement JWT token generation with RSA-256 signing
- [x] Create refresh token rotation mechanism with family tracking
- [x] Build password reset flow with secure token generation
- [x] Add TOTP-based MFA with QR code generation
- [x] Implement rate limiting on authentication endpoints
- [x] Create session management dashboard for users
- [x] Add comprehensive authentication audit logging
- [x] Write authentication integration tests covering all flows

## Dependencies

- Story 1-1: Platform Initialization (database, Next.js app)

## Estimation

**Story Points**: 8

**Breakdown**:
- Email/password authentication: 2 points
- Magic link implementation: 1 point
- Google Workspace SSO: 2 points
- Okta SAML integration: 2 points
- JWT and refresh token management: 1 point
