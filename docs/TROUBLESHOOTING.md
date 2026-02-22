# Troubleshooting Guide

Common issues and solutions for ResonantGenesis.

## Quick Diagnostics

### Check System Status

```bash
# Check API health
curl https://resonantgenesis.xyz/api/v1/health

# Check node status
curl https://resonantgenesis.xyz/api/v1/node/status

# Check your authentication
curl -H "Authorization: Bearer $TOKEN" \
  https://resonantgenesis.xyz/api/v1/auth/me
```

---

## Authentication Issues

### "Invalid or expired token"

**Symptoms:**
- API returns 401 Unauthorized
- "Invalid or expired token" error message

**Solutions:**

1. **Refresh your token:**
   ```python
   new_token = client.auth.refresh()
   ```

2. **Re-authenticate:**
   ```bash
   curl -X POST https://resonantgenesis.xyz/api/v1/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email": "you@example.com", "password": "your-password"}'
   ```

3. **Check token format:**
   - Ensure "Bearer " prefix (with space)
   - Token should start with "eyJ"

### "Account not verified"

**Symptoms:**
- Cannot create agents
- Limited API access

**Solutions:**

1. Check email for verification link
2. Request new verification email:
   ```bash
   curl -X POST https://resonantgenesis.xyz/api/v1/auth/resend-verification \
     -H "Authorization: Bearer $TOKEN"
   ```

---

## Agent Issues

### Agent Not Responding

**Symptoms:**
- Sessions hang indefinitely
- No output from agent

**Solutions:**

1. **Check agent status:**
   ```python
   agent = client.agents.get(agent_id)
   print(f"Status: {agent.status}")
   ```

2. **Verify agent is active:**
   - Go to Agent Studio
   - Check agent status is "Active"

3. **Check for stuck sessions:**
   ```python
   sessions = client.agents.get(agent_id).sessions.list(status="running")
   for session in sessions:
       session.cancel()
   ```

4. **Review agent logs:**
   ```python
   logs = client.agents.get(agent_id).logs.list(limit=10)
   for log in logs:
       print(f"{log.timestamp}: {log.message}")
   ```

### Agent Giving Wrong Answers

**Symptoms:**
- Responses don't match expectations
- Agent ignores instructions

**Solutions:**

1. **Review system prompt:**
   - Be more specific
   - Add examples of desired output
   - Include constraints

2. **Adjust temperature:**
   ```python
   agent = client.agents.update(
       agent_id,
       temperature=0.3  # Lower = more focused
   )
   ```

3. **Check tool configuration:**
   - Ensure needed tools are enabled
   - Verify tool permissions

4. **Test with simple prompts:**
   - Isolate the issue
   - Gradually add complexity

### High Token Usage

**Symptoms:**
- Unexpected costs
- Sessions using many tokens

**Solutions:**

1. **Set token limits:**
   ```python
   session = agent.execute(
       goal="Your task",
       max_tokens=2000
   )
   ```

2. **Shorten system prompt:**
   - Remove unnecessary instructions
   - Be concise

3. **Limit session steps:**
   ```python
   session = agent.execute(
       goal="Your task",
       max_steps=5
   )
   ```

4. **Use cheaper models for simple tasks:**
   - GPT-3.5 Turbo instead of GPT-4

---

## Session Issues

### Session Timeout

**Symptoms:**
- "Session timed out" error
- Incomplete results

**Solutions:**

1. **Increase timeout:**
   ```python
   session = agent.execute(
       goal="Your task",
       timeout_seconds=300
   )
   ```

2. **Break into smaller tasks:**
   - Split complex goals
   - Use multiple sessions

3. **Check external dependencies:**
   - API endpoints responding?
   - Network connectivity?

### Session Stuck in "Running"

**Symptoms:**
- Session never completes
- Status remains "running"

**Solutions:**

1. **Cancel the session:**
   ```python
   session.cancel()
   ```

2. **Check for blocking tools:**
   - External API timeouts
   - Long-running code execution

3. **Review session steps:**
   ```python
   steps = session.steps.list()
   for step in steps:
       print(f"{step.action}: {step.status}")
   ```

---

## API Issues

### Rate Limiting (429 Error)

**Symptoms:**
- "Rate limit exceeded" error
- 429 status code

**Solutions:**

1. **Check your limits:**
   ```python
   limits = client.rate_limits.get()
   print(f"Remaining: {limits.remaining}")
   print(f"Resets at: {limits.reset_at}")
   ```

2. **Implement backoff:**
   ```python
   import time
   
   def api_call_with_retry(func, max_retries=3):
       for i in range(max_retries):
           try:
               return func()
           except RateLimitError as e:
               wait = e.retry_after or (2 ** i)
               time.sleep(wait)
       raise Exception("Max retries exceeded")
   ```

3. **Upgrade your plan:**
   - Plus: 300 requests/min
   - Enterprise: 1000 requests/min

### Connection Errors

**Symptoms:**
- "Connection refused"
- "Network unreachable"

**Solutions:**

1. **Check API status:**
   - Visit [status.resonantgenesis.xyz](https://status.resonantgenesis.xyz)

2. **Verify network:**
   ```bash
   ping resonantgenesis.xyz
   curl -v https://resonantgenesis.xyz/api/v1/health
   ```

3. **Check firewall/proxy:**
   - Ensure outbound HTTPS allowed
   - Configure proxy if needed

4. **Try alternative endpoint:**
   ```python
   client = ResonantClient(
       api_key="...",
       base_url="https://api.resonantgenesis.xyz"
   )
   ```

---

## Tool Issues

### Web Search Not Working

**Symptoms:**
- "Tool not available" error
- Empty search results

**Solutions:**

1. **Verify tool is enabled:**
   ```python
   agent = client.agents.get(agent_id)
   print(agent.tools)  # Should include "web_search"
   ```

2. **Check trust tier:**
   - Web search requires T1+

3. **Review tool config:**
   ```python
   agent = client.agents.update(
       agent_id,
       tool_config={
           "web_search": {
               "max_results": 10,
               "safe_search": True
           }
       }
   )
   ```

### Code Execution Failing

**Symptoms:**
- "Execution failed" error
- Timeout during code run

**Solutions:**

1. **Check trust tier:**
   - Code execution requires T2+

2. **Review resource limits:**
   ```python
   tool_config={
       "code_exec": {
           "timeout_seconds": 60,
           "memory_mb": 512
       }
   }
   ```

3. **Check allowed packages:**
   - Only allowlisted packages work
   - Request additional packages via support

4. **Test code locally first:**
   - Ensure code works before agent runs it

### API Call Blocked

**Symptoms:**
- "Domain not allowed" error
- External API calls fail

**Solutions:**

1. **Check domain allowlist:**
   ```python
   tool_config={
       "api_call": {
           "allowed_domains": ["api.example.com"]
       }
   }
   ```

2. **Verify URL format:**
   - Must include protocol (https://)
   - Check for typos

3. **Check external API:**
   - Is the API up?
   - Are credentials valid?

---

## Webhook Issues

### Webhook Not Triggering

**Symptoms:**
- External events don't start sessions
- No webhook logs

**Solutions:**

1. **Verify webhook URL:**
   ```python
   trigger = client.agents.get(agent_id).triggers.get(trigger_id)
   print(trigger.webhook_url)
   ```

2. **Check trigger status:**
   - Ensure not paused
   - Verify configuration

3. **Test manually:**
   ```bash
   curl -X POST https://resonantgenesis.xyz/api/v1/agents/triggers/webhook/{trigger_id} \
     -H "Content-Type: application/json" \
     -H "X-Webhook-Signature: sha256=..." \
     -d '{"test": "data"}'
   ```

4. **Review webhook logs:**
   ```python
   logs = trigger.logs.list(limit=10)
   for log in logs:
       print(f"{log.received_at}: {log.status}")
   ```

### Invalid Signature Error

**Symptoms:**
- 401 "Invalid signature" response
- Webhook rejected

**Solutions:**

1. **Verify secret matches:**
   - Check secret in trigger config
   - Check secret in external service

2. **Check signature format:**
   - Should be `sha256=<hex_digest>`
   - Computed over raw body

3. **Verify payload encoding:**
   - Use UTF-8
   - Don't modify payload before signing

---

## Blockchain Issues

### Transaction Failed

**Symptoms:**
- "Transaction reverted" error
- Agent not published

**Solutions:**

1. **Check wallet balance:**
   - Ensure sufficient ETH for gas
   - Base network uses ETH

2. **Verify network:**
   - Mainnet vs Sepolia testnet
   - Check RPC endpoint

3. **Review transaction:**
   ```python
   tx = client.blockchain.get_transaction(tx_hash)
   print(f"Status: {tx.status}")
   print(f"Error: {tx.error}")
   ```

### DSID Not Found

**Symptoms:**
- "Identity not registered" error
- Cannot verify agent

**Solutions:**

1. **Check registration:**
   ```python
   is_registered = client.blockchain.is_registered(dsid)
   ```

2. **Verify network:**
   - Same network as registration?

3. **Wait for confirmation:**
   - Allow 1-2 minutes after registration

---

## Performance Issues

### Slow Response Times

**Symptoms:**
- API calls taking long
- Sessions slow to complete

**Solutions:**

1. **Check system status:**
   - Visit status page
   - Check for incidents

2. **Optimize requests:**
   - Use pagination
   - Request only needed fields

3. **Use caching:**
   ```python
   from functools import lru_cache
   
   @lru_cache(maxsize=100)
   def get_agent(agent_id):
       return client.agents.get(agent_id)
   ```

4. **Use async operations:**
   ```python
   import asyncio
   
   async def parallel_requests():
       tasks = [
           client.agents.get_async(id1),
           client.agents.get_async(id2)
       ]
       return await asyncio.gather(*tasks)
   ```

### High Memory Usage

**Symptoms:**
- Application crashes
- Out of memory errors

**Solutions:**

1. **Stream large responses:**
   ```python
   async for chunk in session.stream():
       process(chunk)
   ```

2. **Paginate results:**
   ```python
   for page in client.agents.list_pages(limit=50):
       for agent in page:
           process(agent)
   ```

3. **Clean up sessions:**
   ```python
   # Close sessions when done
   session.close()
   ```

---

## Getting Help

### Before Contacting Support

1. **Check this guide** for common solutions
2. **Review [FAQ](./FAQ.md)** for answers
3. **Search [GitHub Issues](https://github.com/louienemesh/ResonantGenesis/issues)**
4. **Check [Discord](https://discord.gg/resonantgenesis)** for community help

### When Contacting Support

Include:
- **Error message** (exact text)
- **Request ID** (from response headers)
- **Steps to reproduce**
- **Expected vs actual behavior**
- **Environment** (SDK version, language, OS)

### Support Channels

- **Discord**: [discord.gg/resonantgenesis](https://discord.gg/resonantgenesis)
- **GitHub Issues**: [Report bugs](https://github.com/louienemesh/ResonantGenesis/issues)
- **Email**: support@resonantgenesis.xyz
- **Help Center**: [resonantgenesis.xyz/help](https://resonantgenesis.xyz/help)

---

**Still stuck?** Join our [Discord](https://discord.gg/resonantgenesis) for real-time help from the community and team.
