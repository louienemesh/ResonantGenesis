# Error Codes Reference

Complete reference for all error codes and messages in ResonantGenesis.

## Error Response Format

All API errors follow this format:

```json
{
  "error": {
    "code": "error_code",
    "message": "Human-readable error message",
    "details": {},
    "request_id": "req_abc123"
  }
}
```

## HTTP Status Codes

| Status | Description |
|--------|-------------|
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Authentication required |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource already exists |
| 422 | Unprocessable Entity - Validation failed |
| 429 | Too Many Requests - Rate limited |
| 500 | Internal Server Error |
| 502 | Bad Gateway - Upstream service error |
| 503 | Service Unavailable - Maintenance |

---

## Authentication Errors (401)

### `invalid_credentials`

**Message:** Invalid email or password

**Cause:** Login attempt with wrong credentials

**Solution:**
- Verify email address
- Reset password if forgotten
- Check for typos

```python
try:
    client.auth.login(email, password)
except AuthenticationError as e:
    if e.code == "invalid_credentials":
        print("Wrong email or password")
```

### `token_expired`

**Message:** Access token has expired

**Cause:** JWT token past expiration time

**Solution:**
- Refresh the token
- Re-authenticate if refresh fails

```python
try:
    result = client.agents.list()
except TokenExpiredError:
    client.auth.refresh()
    result = client.agents.list()
```

### `token_invalid`

**Message:** Invalid or malformed token

**Cause:** Token is corrupted or tampered

**Solution:**
- Re-authenticate to get new token
- Check token format (should start with "eyJ")

### `api_key_invalid`

**Message:** Invalid API key

**Cause:** API key doesn't exist or was revoked

**Solution:**
- Generate new API key in dashboard
- Check for copy/paste errors

### `api_key_expired`

**Message:** API key has expired

**Cause:** API key past expiration date

**Solution:**
- Generate new API key
- Set longer expiration next time

### `2fa_required`

**Message:** Two-factor authentication required

**Cause:** Account has 2FA enabled

**Solution:**
- Submit 2FA code with temp token
- Use backup code if device unavailable

---

## Authorization Errors (403)

### `insufficient_permissions`

**Message:** You don't have permission to perform this action

**Cause:** API key or token lacks required scope

**Solution:**
- Check required permissions for endpoint
- Generate new API key with correct scopes
- Upgrade account tier if needed

```python
try:
    client.agents.delete(agent_id)
except PermissionError as e:
    print(f"Need permission: {e.required_permission}")
```

### `tier_required`

**Message:** This feature requires {tier} tier or higher

**Cause:** Feature not available on current tier

**Solution:**
- Upgrade to required tier
- Use alternative approach for current tier

### `account_suspended`

**Message:** Your account has been suspended

**Cause:** Policy violation or billing issue

**Solution:**
- Contact support
- Resolve billing issues
- Appeal if incorrect

### `resource_not_owned`

**Message:** You don't own this resource

**Cause:** Attempting to modify another user's resource

**Solution:**
- Verify resource ID
- Check you're using correct account

---

## Validation Errors (400/422)

### `validation_error`

**Message:** Validation failed

**Cause:** Request body failed validation

**Details:**
```json
{
  "error": {
    "code": "validation_error",
    "message": "Validation failed",
    "details": {
      "fields": {
        "name": ["Name is required"],
        "system_prompt": ["Must be at least 10 characters"]
      }
    }
  }
}
```

**Solution:**
- Check field requirements in API docs
- Validate input before sending

### `invalid_parameter`

**Message:** Invalid value for parameter: {param}

**Cause:** Parameter has wrong type or format

**Solution:**
- Check parameter type (string, number, etc.)
- Verify format (UUID, email, URL)

### `missing_parameter`

**Message:** Required parameter missing: {param}

**Cause:** Required field not provided

**Solution:**
- Include all required fields
- Check API documentation

### `invalid_json`

**Message:** Request body is not valid JSON

**Cause:** Malformed JSON in request

**Solution:**
- Validate JSON syntax
- Check for trailing commas, quotes

---

## Resource Errors (404/409)

### `not_found`

**Message:** Resource not found

**Cause:** Resource doesn't exist or was deleted

**Solution:**
- Verify resource ID
- Check resource wasn't deleted
- Ensure correct API version

```python
try:
    agent = client.agents.get(agent_id)
except NotFoundError:
    print("Agent doesn't exist")
```

### `agent_not_found`

**Message:** Agent not found: {agent_id}

**Cause:** Agent ID doesn't exist

**Solution:**
- List agents to find correct ID
- Check agent wasn't deleted

### `session_not_found`

**Message:** Session not found: {session_id}

**Cause:** Session ID doesn't exist

**Solution:**
- Sessions may expire after completion
- Check session ID is correct

### `already_exists`

**Message:** Resource already exists

**Cause:** Attempting to create duplicate

**Solution:**
- Use unique identifiers
- Update existing resource instead

### `name_taken`

**Message:** Name is already in use

**Cause:** Agent/team name not unique

**Solution:**
- Choose different name
- Add suffix or prefix

---

## Rate Limit Errors (429)

### `rate_limit_exceeded`

**Message:** Rate limit exceeded

**Cause:** Too many requests in time window

**Details:**
```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded",
    "details": {
      "limit": 60,
      "remaining": 0,
      "reset_at": "2026-02-21T10:05:00Z",
      "retry_after": 30
    }
  }
}
```

**Solution:**
- Wait for reset time
- Implement exponential backoff
- Upgrade tier for higher limits

```python
try:
    result = client.agents.list()
except RateLimitError as e:
    time.sleep(e.retry_after)
    result = client.agents.list()
```

### `concurrent_limit_exceeded`

**Message:** Too many concurrent sessions

**Cause:** Exceeded concurrent session limit

**Solution:**
- Wait for sessions to complete
- Cancel unused sessions
- Upgrade tier

---

## Agent Errors

### `agent_inactive`

**Message:** Agent is not active

**Cause:** Agent status is paused or deprecated

**Solution:**
- Activate agent in dashboard
- Check agent status before executing

### `agent_execution_failed`

**Message:** Agent execution failed

**Cause:** Error during agent execution

**Details:**
```json
{
  "error": {
    "code": "agent_execution_failed",
    "message": "Agent execution failed",
    "details": {
      "step": 3,
      "action": "web_search",
      "error": "Search service unavailable"
    }
  }
}
```

**Solution:**
- Check agent configuration
- Review execution logs
- Verify tool availability

### `model_unavailable`

**Message:** Model is temporarily unavailable

**Cause:** AI model service down

**Solution:**
- Retry after a few minutes
- Use fallback model
- Check status page

### `context_too_long`

**Message:** Context exceeds model limit

**Cause:** Input + history exceeds token limit

**Solution:**
- Reduce system prompt length
- Clear session history
- Use model with larger context

---

## Tool Errors

### `tool_not_available`

**Message:** Tool is not available: {tool}

**Cause:** Tool not enabled or doesn't exist

**Solution:**
- Enable tool in agent config
- Check tool name spelling
- Verify tier supports tool

### `tool_execution_failed`

**Message:** Tool execution failed: {tool}

**Cause:** Error during tool execution

**Details:**
```json
{
  "error": {
    "code": "tool_execution_failed",
    "message": "Tool execution failed",
    "details": {
      "tool": "code_exec",
      "error": "Execution timeout",
      "timeout_seconds": 30
    }
  }
}
```

**Solution:**
- Check tool configuration
- Increase timeout if needed
- Review tool input

### `domain_not_allowed`

**Message:** Domain not in allowlist: {domain}

**Cause:** API call to non-allowed domain

**Solution:**
- Add domain to allowlist
- Check domain spelling

### `code_execution_timeout`

**Message:** Code execution timed out

**Cause:** Code took too long to execute

**Solution:**
- Optimize code
- Increase timeout limit
- Break into smaller operations

---

## Webhook Errors

### `webhook_signature_invalid`

**Message:** Invalid webhook signature

**Cause:** Signature verification failed

**Solution:**
- Check secret matches
- Verify signature algorithm (SHA-256)
- Ensure payload not modified

### `webhook_payload_invalid`

**Message:** Invalid webhook payload

**Cause:** Payload doesn't match expected format

**Solution:**
- Check payload structure
- Verify content type is JSON

### `trigger_not_found`

**Message:** Trigger not found: {trigger_id}

**Cause:** Webhook trigger doesn't exist

**Solution:**
- Verify trigger ID
- Check trigger wasn't deleted

---

## Blockchain Errors

### `transaction_failed`

**Message:** Blockchain transaction failed

**Cause:** Transaction reverted or failed

**Details:**
```json
{
  "error": {
    "code": "transaction_failed",
    "message": "Transaction failed",
    "details": {
      "tx_hash": "0x...",
      "reason": "insufficient funds"
    }
  }
}
```

**Solution:**
- Check wallet balance
- Verify gas settings
- Review transaction parameters

### `dsid_not_registered`

**Message:** DSID not registered

**Cause:** Identity not registered on Base (Ethereum L2)

**Solution:**
- Register identity on Base first
- Check correct network (Base Mainnet or Sepolia)

### `already_registered`

**Message:** Already registered on chain

**Cause:** Attempting duplicate registration

**Solution:**
- Use existing registration
- Check if already published

---

## Server Errors (500+)

### `internal_error`

**Message:** An internal error occurred

**Cause:** Unexpected server error

**Solution:**
- Retry request
- Contact support with request_id
- Check status page

### `service_unavailable`

**Message:** Service temporarily unavailable

**Cause:** Maintenance or overload

**Solution:**
- Retry after a few minutes
- Check status page
- Use fallback if critical

### `upstream_error`

**Message:** Upstream service error

**Cause:** Third-party service failed

**Solution:**
- Retry request
- Check if specific tool/service is down

---

## Error Handling Best Practices

### Python

```python
from resonantgenesis import (
    ResonantClient,
    ResonantError,
    AuthenticationError,
    RateLimitError,
    ValidationError,
    NotFoundError
)

client = ResonantClient(api_key="...")

try:
    agent = client.agents.create(
        name="My Agent",
        system_prompt="You are helpful."
    )
except ValidationError as e:
    print(f"Validation failed: {e.details}")
except RateLimitError as e:
    print(f"Rate limited. Retry after {e.retry_after}s")
except NotFoundError as e:
    print(f"Resource not found: {e.resource_id}")
except AuthenticationError as e:
    print(f"Auth failed: {e.message}")
except ResonantError as e:
    print(f"API error: {e.code} - {e.message}")
    print(f"Request ID: {e.request_id}")
```

### JavaScript

```typescript
import {
  ResonantClient,
  ResonantError,
  AuthenticationError,
  RateLimitError,
  ValidationError,
  NotFoundError
} from '@resonantgenesis/sdk';

const client = new ResonantClient({ apiKey: '...' });

try {
  const agent = await client.agents.create({
    name: 'My Agent',
    systemPrompt: 'You are helpful.'
  });
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Validation failed:', error.details);
  } else if (error instanceof RateLimitError) {
    console.log(`Rate limited. Retry after ${error.retryAfter}s`);
  } else if (error instanceof NotFoundError) {
    console.log('Resource not found:', error.resourceId);
  } else if (error instanceof AuthenticationError) {
    console.log('Auth failed:', error.message);
  } else if (error instanceof ResonantError) {
    console.log(`API error: ${error.code} - ${error.message}`);
    console.log('Request ID:', error.requestId);
  }
}
```

---

## Getting Help

When contacting support, include:

1. **Error code** and message
2. **Request ID** from error response
3. **Timestamp** of the error
4. **Steps to reproduce**
5. **Request payload** (sanitized)

**Support channels:**
- Discord: [discord.gg/resonantgenesis](https://discord.gg/resonantgenesis)
- Email: support@resonantgenesis.xyz
- GitHub: [Report issues](https://github.com/louienemesh/ResonantGenesis/issues)

---

**See also:** [Troubleshooting Guide](./TROUBLESHOOTING.md) | [API Reference](./API_REFERENCE.md)
