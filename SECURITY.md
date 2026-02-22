# Security Policy

## Supported Versions

We actively support the following versions with security updates:

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security issue, please report it responsibly.

### How to Report

**DO NOT** create a public GitHub issue for security vulnerabilities.

Instead, please report security issues via one of these methods:

1. **Email**: security@resonantgenesis.xyz
2. **GitHub Security Advisories**: [Create a private security advisory](https://github.com/louienemesh/ResonantGenesis/security/advisories/new)

### What to Include

Please provide as much information as possible:

- **Description**: Clear description of the vulnerability
- **Impact**: What could an attacker achieve?
- **Steps to Reproduce**: Detailed steps to reproduce the issue
- **Affected Components**: Which parts of the system are affected?
- **Suggested Fix**: If you have ideas for remediation
- **Your Contact Info**: So we can follow up

### Response Timeline

| Action | Timeline |
|--------|----------|
| Initial Response | Within 48 hours |
| Vulnerability Assessment | Within 7 days |
| Fix Development | Depends on severity |
| Public Disclosure | After fix is deployed |

### Severity Levels

| Level | Description | Response Time |
|-------|-------------|---------------|
| **Critical** | Remote code execution, data breach | 24-48 hours |
| **High** | Authentication bypass, privilege escalation | 3-5 days |
| **Medium** | Information disclosure, XSS | 7-14 days |
| **Low** | Minor issues, hardening | 30 days |

## Security Best Practices

### For Users

#### API Keys & Secrets

```bash
# NEVER commit secrets to version control
# Use environment variables instead

# Bad - hardcoded secret
API_KEY = "sk-1234567890abcdef"

# Good - environment variable
API_KEY = os.environ.get("RESONANT_API_KEY")
```

#### Authentication

- Use strong, unique passwords
- Enable two-factor authentication (2FA) when available
- Rotate API keys regularly
- Use the minimum required permissions

#### Agent Security

- Review agent permissions before execution
- Use trust tiers appropriately
- Monitor agent execution logs
- Set appropriate rate limits

### For Developers

#### Input Validation

```python
# Always validate and sanitize user input
from pydantic import BaseModel, validator

class AgentCreate(BaseModel):
    name: str
    
    @validator('name')
    def validate_name(cls, v):
        if len(v) > 100:
            raise ValueError('Name too long')
        if not v.strip():
            raise ValueError('Name cannot be empty')
        return v.strip()
```

#### SQL Injection Prevention

```python
# Bad - vulnerable to SQL injection
query = f"SELECT * FROM agents WHERE name = '{user_input}'"

# Good - parameterized query
query = "SELECT * FROM agents WHERE name = :name"
result = db.execute(query, {"name": user_input})
```

#### XSS Prevention

```typescript
// Bad - vulnerable to XSS
element.innerHTML = userInput;

// Good - use safe methods
element.textContent = userInput;

// Or sanitize HTML
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

#### Authentication & Authorization

```python
# Always verify authentication
@router.get("/agents/{agent_id}")
async def get_agent(
    agent_id: str,
    current_user: User = Depends(get_current_user)
):
    agent = await get_agent_by_id(agent_id)
    
    # Always verify authorization
    if agent.owner_id != current_user.id:
        raise HTTPException(status_code=403, detail="Not authorized")
    
    return agent
```

#### Secrets Management

```yaml
# Use environment variables for secrets
# .env.example (commit this)
DATABASE_URL=postgresql://user:pass@localhost:5432/db
JWT_SECRET=your-secret-here
API_KEY=your-api-key

# .env (NEVER commit this)
# Add to .gitignore
```

## Smart Contract Security

### Audit Status

| Contract | Audited | Auditor | Date |
|----------|---------|---------|------|
| IdentityRegistry.sol | Pending | - | - |
| AgentRegistry.sol | Pending | - | - |
| MemoryAnchors.sol | Pending | - | - |

### Security Measures

1. **Access Control**: Role-based permissions using OpenZeppelin
2. **Reentrancy Protection**: ReentrancyGuard on all state-changing functions
3. **Integer Overflow**: Solidity 0.8+ built-in overflow checks
4. **Upgradability**: Transparent proxy pattern for upgrades

### Known Limitations

- Contracts are currently on testnet (Base Sepolia)
- Mainnet deployment pending security audit
- Gas optimization ongoing

## Infrastructure Security

### Network Security

- All traffic encrypted via TLS 1.3
- DDoS protection via Cloudflare
- Rate limiting on all API endpoints
- IP allowlisting for admin endpoints

### Data Protection

- Database encryption at rest
- Encrypted backups
- Regular security scans
- Access logging and monitoring

### Deployment Security

- CI/CD pipeline security checks
- Container image scanning
- Dependency vulnerability scanning
- Infrastructure as Code (IaC) security

## Bug Bounty Program

We are planning to launch a bug bounty program. Details coming soon.

### Scope (Planned)

**In Scope:**
- resonantgenesis.xyz web application
- API endpoints (api.resonantgenesis.xyz)
- Smart contracts on Base mainnet
- Authentication and authorization systems

**Out of Scope:**
- Third-party services
- Social engineering attacks
- Physical security
- Denial of service attacks

### Rewards (Planned)

| Severity | Reward Range |
|----------|--------------|
| Critical | $1,000 - $5,000 |
| High | $500 - $1,000 |
| Medium | $100 - $500 |
| Low | $25 - $100 |

## Security Contacts

- **Security Team**: security@resonantgenesis.xyz
- **PGP Key**: Available on request
- **Response Hours**: 24/7 for critical issues

## Acknowledgments

We thank the following security researchers for responsibly disclosing vulnerabilities:

*No vulnerabilities reported yet. Be the first!*

---

## Changelog

### 2026-02-21
- Initial security policy created
- Vulnerability reporting process established
- Security best practices documented

---

*Last updated: February 21, 2026*
