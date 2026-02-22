# Authentication Documentation

Complete guide to authentication methods in ResonantGenesis.

## Overview

ResonantGenesis supports multiple authentication methods:

| Method | Use Case | Security Level |
|--------|----------|----------------|
| API Keys | Server-to-server | High |
| JWT Tokens | User sessions | High |
| OAuth2 | Third-party apps | High |
| Session Cookies | Web browser | Medium |

## API Keys

### Creating an API Key

**Via Dashboard:**
1. Go to **Settings** > **API Keys**
2. Click **Generate New Key**
3. Set permissions and expiration
4. Copy key (shown only once)

**Via API:**
```bash
curl -X POST https://resonantgenesis.xyz/api/v1/auth/api-keys \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Server",
    "permissions": ["agents:read", "agents:write", "sessions:*"],
    "expires_in_days": 365
  }'
```

**Response:**
```json
{
  "id": "key_abc123",
  "name": "Production Server",
  "key": "rg_live_sk_1234567890abcdef",
  "permissions": ["agents:read", "agents:write", "sessions:*"],
  "created_at": "2026-02-21T00:00:00Z",
  "expires_at": "2027-02-21T00:00:00Z"
}
```

### Using API Keys

Include the key in the `Authorization` header:

```bash
curl https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer rg_live_sk_1234567890abcdef"
```

**Python:**
```python
from resonantgenesis import ResonantClient

client = ResonantClient(api_key="rg_live_sk_1234567890abcdef")
agents = client.agents.list()
```

**JavaScript:**
```typescript
import { ResonantClient } from '@resonantgenesis/sdk';

const client = new ResonantClient({
  apiKey: 'rg_live_sk_1234567890abcdef'
});

const agents = await client.agents.list();
```

### API Key Permissions

| Permission | Description |
|------------|-------------|
| `agents:read` | List and get agents |
| `agents:write` | Create, update, delete agents |
| `sessions:read` | List and get sessions |
| `sessions:write` | Create and manage sessions |
| `teams:read` | List and get teams |
| `teams:write` | Create and manage teams |
| `webhooks:*` | Full webhook access |
| `*` | Full access (admin) |

### Revoking API Keys

```bash
curl -X DELETE https://resonantgenesis.xyz/api/v1/auth/api-keys/{key_id} \
  -H "Authorization: Bearer $JWT_TOKEN"
```

## JWT Tokens

### Obtaining a Token

**Login:**
```bash
curl -X POST https://resonantgenesis.xyz/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "your_password"
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Token Structure

JWT tokens contain:

```json
{
  "sub": "user_123",
  "email": "user@example.com",
  "tier": "plus",
  "permissions": ["agents:*", "sessions:*"],
  "iat": 1708531200,
  "exp": 1708534800
}
```

### Using JWT Tokens

```bash
curl https://resonantgenesis.xyz/api/v1/agents \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

### Refreshing Tokens

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 3600
}
```

### Token Expiration

| Token Type | Default Expiration | Max Expiration |
|------------|-------------------|----------------|
| Access Token | 1 hour | 24 hours |
| Refresh Token | 7 days | 30 days |
| API Key | 1 year | Never |

## OAuth2

### Supported Providers

| Provider | Scopes |
|----------|--------|
| Google | `email`, `profile` |
| GitHub | `user:email`, `read:user` |
| Discord | `identify`, `email` |

### OAuth2 Flow

1. **Redirect to provider:**
```
https://resonantgenesis.xyz/api/v1/auth/oauth/google?
  redirect_uri=https://yourapp.com/callback&
  state=random_state_string
```

2. **Handle callback:**
```bash
# User is redirected to:
https://yourapp.com/callback?code=auth_code&state=random_state_string
```

3. **Exchange code for token:**
```bash
curl -X POST https://resonantgenesis.xyz/api/v1/auth/oauth/token \
  -H "Content-Type: application/json" \
  -d '{
    "code": "auth_code",
    "redirect_uri": "https://yourapp.com/callback"
  }'
```

### Implementing OAuth2

**Python:**
```python
from resonantgenesis import ResonantClient

# Generate OAuth URL
auth_url = ResonantClient.get_oauth_url(
    provider="google",
    redirect_uri="https://yourapp.com/callback",
    state="random_state"
)

# After callback, exchange code
client = ResonantClient.from_oauth_code(
    code="auth_code",
    redirect_uri="https://yourapp.com/callback"
)
```

**JavaScript:**
```typescript
import { ResonantClient } from '@resonantgenesis/sdk';

// Generate OAuth URL
const authUrl = ResonantClient.getOAuthUrl({
  provider: 'google',
  redirectUri: 'https://yourapp.com/callback',
  state: 'random_state'
});

// After callback, exchange code
const client = await ResonantClient.fromOAuthCode({
  code: 'auth_code',
  redirectUri: 'https://yourapp.com/callback'
});
```

## Two-Factor Authentication (2FA)

### Enabling 2FA

```bash
# Generate 2FA secret
curl -X POST https://resonantgenesis.xyz/api/v1/auth/2fa/setup \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "secret": "JBSWY3DPEHPK3PXP",
  "qr_code_url": "otpauth://totp/ResonantGenesis:user@example.com?secret=JBSWY3DPEHPK3PXP",
  "backup_codes": ["12345678", "23456789", "34567890"]
}
```

### Verifying 2FA

```bash
# Confirm 2FA setup
curl -X POST https://resonantgenesis.xyz/api/v1/auth/2fa/verify \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"code": "123456"}'
```

### Login with 2FA

```bash
# Step 1: Login
curl -X POST https://resonantgenesis.xyz/api/v1/auth/login \
  -d '{"email": "user@example.com", "password": "password"}'

# Response indicates 2FA required
{
  "requires_2fa": true,
  "temp_token": "temp_abc123"
}

# Step 2: Submit 2FA code
curl -X POST https://resonantgenesis.xyz/api/v1/auth/2fa/login \
  -d '{"temp_token": "temp_abc123", "code": "123456"}'
```

## Session Management

### List Active Sessions

```bash
curl https://resonantgenesis.xyz/api/v1/auth/sessions \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "sessions": [
    {
      "id": "session_123",
      "device": "Chrome on macOS",
      "ip_address": "192.168.1.1",
      "location": "San Francisco, CA",
      "created_at": "2026-02-20T10:00:00Z",
      "last_active": "2026-02-21T15:30:00Z",
      "current": true
    }
  ]
}
```

### Revoke Session

```bash
curl -X DELETE https://resonantgenesis.xyz/api/v1/auth/sessions/{session_id} \
  -H "Authorization: Bearer $TOKEN"
```

### Revoke All Sessions

```bash
curl -X POST https://resonantgenesis.xyz/api/v1/auth/sessions/revoke-all \
  -H "Authorization: Bearer $TOKEN"
```

## Security Best Practices

### API Key Security

1. **Never expose in client code**: Use server-side only
2. **Use environment variables**: Don't hardcode
3. **Rotate regularly**: Set expiration dates
4. **Limit permissions**: Use least privilege
5. **Monitor usage**: Check for anomalies

```python
import os

# Good: Use environment variable
client = ResonantClient(api_key=os.environ["RESONANT_API_KEY"])

# Bad: Hardcoded key
client = ResonantClient(api_key="rg_live_sk_...")  # Don't do this!
```

### Token Security

1. **Store securely**: Use secure storage (keychain, vault)
2. **Use HTTPS only**: Never transmit over HTTP
3. **Implement refresh**: Don't use long-lived access tokens
4. **Handle expiration**: Refresh before expiry
5. **Clear on logout**: Remove all stored tokens

```python
import keyring

# Store token securely
keyring.set_password("resonantgenesis", "access_token", token)

# Retrieve token
token = keyring.get_password("resonantgenesis", "access_token")
```

### OAuth Security

1. **Validate state**: Prevent CSRF attacks
2. **Use PKCE**: For public clients
3. **Verify redirect URI**: Exact match only
4. **Short-lived codes**: Exchange immediately

```python
import secrets

# Generate secure state
state = secrets.token_urlsafe(32)

# Store state for validation
session["oauth_state"] = state

# Validate on callback
if request.args.get("state") != session.get("oauth_state"):
    raise SecurityError("Invalid state parameter")
```

## Error Handling

### Authentication Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `invalid_credentials` | Wrong email/password | Check credentials |
| `token_expired` | Token has expired | Refresh token |
| `token_invalid` | Token is malformed | Re-authenticate |
| `insufficient_permissions` | Missing required scope | Request additional permissions |
| `2fa_required` | 2FA code needed | Submit 2FA code |
| `account_locked` | Too many failed attempts | Wait or contact support |

### Handling Errors

```python
from resonantgenesis import (
    ResonantClient,
    AuthenticationError,
    TokenExpiredError,
    PermissionError
)

try:
    client = ResonantClient(api_key="...")
    agents = client.agents.list()
except TokenExpiredError:
    # Refresh token and retry
    client.auth.refresh()
    agents = client.agents.list()
except PermissionError as e:
    print(f"Missing permission: {e.required_permission}")
except AuthenticationError as e:
    print(f"Auth failed: {e.message}")
```

## API Reference

### Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/login` | POST | Login with email/password |
| `/auth/logout` | POST | Logout current session |
| `/auth/refresh` | POST | Refresh access token |
| `/auth/register` | POST | Create new account |
| `/auth/verify-email` | POST | Verify email address |
| `/auth/reset-password` | POST | Request password reset |
| `/auth/api-keys` | GET | List API keys |
| `/auth/api-keys` | POST | Create API key |
| `/auth/api-keys/{id}` | DELETE | Revoke API key |
| `/auth/sessions` | GET | List active sessions |
| `/auth/sessions/{id}` | DELETE | Revoke session |
| `/auth/2fa/setup` | POST | Set up 2FA |
| `/auth/2fa/verify` | POST | Verify 2FA code |
| `/auth/oauth/{provider}` | GET | Start OAuth flow |
| `/auth/oauth/token` | POST | Exchange OAuth code |

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
