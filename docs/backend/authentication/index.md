# Authentication System

This document provides an overview of the authentication system used in the Quillium backend.

## Overview

The Quillium authentication system is built on JWT (JSON Web Tokens) with support for refresh tokens to enhance security and user experience.

## Authentication Flow

1. **User Login**: User provides credentials (email/password)
2. **Token Generation**: Server validates credentials and generates two tokens:
   - Access Token: Short-lived JWT for API access
   - Refresh Token: Long-lived token for obtaining new access tokens
3. **API Access**: Client includes access token in Authorization header
4. **Token Refresh**: When access token expires, client uses refresh token to get a new one

## JWT Structure

### Access Token

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_id",
    "name": "user_email",
    "admin": false,
    "exp": 1617120000,
    "iat": 1617110000
  },
  "signature": "..."
}
```

### Token Expiration

- Access Token: 15 minutes (configurable)
- Refresh Token: 7 days (configurable)

## Refresh Token Implementation

The refresh token pattern enhances security by:

1. Limiting the lifetime of access tokens to minimize risk if compromised
2. Providing a seamless user experience through automatic token refresh
3. Allowing token revocation without requiring user re-authentication

### Database Schema

Refresh tokens are stored in the database with the following schema:

```sql
CREATE TABLE refresh_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    revoked BOOLEAN NOT NULL DEFAULT FALSE
);
```

### Refresh Token Endpoint

```go
// RefreshTokenHandler exchanges a valid refresh token for a new access token
func RefreshTokenHandler(w http.ResponseWriter, r *http.Request) {
    // Extract refresh token from request
    // Validate refresh token against database
    // Check if token is expired or revoked
    // Generate new access token
    // Return new access token to client
}
```

## Security Considerations

### Token Storage

- **Client-side**: 
  - Access token: Memory or short-lived cookie
  - Refresh token: HttpOnly, Secure cookie

- **Server-side**:
  - Access token: Not stored (stateless)
  - Refresh token: Database with expiration and revocation support

### Token Revocation

Refresh tokens can be revoked in the following scenarios:

1. User logout
2. Password change
3. Security breach
4. Admin intervention

### CSRF Protection

For cookie-based token storage, CSRF protection is implemented through:

1. SameSite cookie attribute
2. CSRF tokens for sensitive operations

## Implementation Details

### Middleware

The authentication middleware:

1. Extracts the JWT from the Authorization header
2. Validates the token signature and expiration
3. Sets user information in the request context

```go
func WithAuth(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Extract token from Authorization header
        // Validate token
        // Set user ID and admin status in context
        // Call next handler
    }
}
```

### Token Generation

```go
func GenerateJWT(userID int, isAdmin bool) (string, error) {
    // Create token with claims
    // Sign token with secret key
    // Return token string
}
```

## Future Enhancements

### OAuth Integration

Support for OAuth providers like Google, GitHub, etc.

### Multi-factor Authentication

Implementation of 2FA using TOTP or similar methods.

### Device Management

Tracking and managing active sessions across multiple devices.
