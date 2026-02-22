# Versioning Guide

Complete guide to API versioning, SDK versioning, and backward compatibility in ResonantGenesis.

## Overview

ResonantGenesis uses semantic versioning and maintains backward compatibility to ensure stable integrations. This guide covers versioning policies, migration strategies, and best practices.

## Versioning Scheme

### Semantic Versioning

We follow [Semantic Versioning 2.0.0](https://semver.org/):

```
MAJOR.MINOR.PATCH

Examples:
- 1.0.0 → 1.0.1 (patch: bug fixes)
- 1.0.0 → 1.1.0 (minor: new features, backward compatible)
- 1.0.0 → 2.0.0 (major: breaking changes)
```

### Version Components

| Component | When Incremented | Compatibility |
|-----------|------------------|---------------|
| **MAJOR** | Breaking changes | Not backward compatible |
| **MINOR** | New features | Backward compatible |
| **PATCH** | Bug fixes | Backward compatible |

## API Versioning

### URL-Based Versioning

API versions are included in the URL:

```
https://api.resonantgenesis.xyz/v1/agents
https://api.resonantgenesis.xyz/v2/agents
```

### Current Versions

| Version | Status | Support Until |
|---------|--------|---------------|
| v1 | Active | Dec 2027 |
| v2 | Beta | - |

### Version Header

You can also specify version via header:

```bash
curl -H "X-API-Version: 2026-02-01" \
  https://api.resonantgenesis.xyz/agents
```

### Default Version

If no version is specified:
- New accounts default to latest stable version
- Existing accounts keep their current version

## Deprecation Policy

### Timeline

| Phase | Duration | Description |
|-------|----------|-------------|
| **Announcement** | - | Deprecation notice in changelog |
| **Warning** | 6 months | Deprecation headers in responses |
| **Migration** | 6 months | Both versions supported |
| **Sunset** | - | Old version removed |

### Deprecation Headers

When using deprecated features:

```http
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Dec 2027 00:00:00 GMT
Link: <https://docs.resonantgenesis.xyz/migrations/v2>; rel="deprecation"
```

### Deprecation Warnings

```python
# SDK will log deprecation warnings
import warnings

# Enable deprecation warnings
warnings.filterwarnings("always", category=DeprecationWarning)

# Example warning:
# DeprecationWarning: 'agent.run()' is deprecated, use 'agent.execute()' instead
```

## Breaking Changes

### What Constitutes Breaking Changes

| Change Type | Breaking? |
|-------------|-----------|
| Removing endpoint | ✅ Yes |
| Removing field | ✅ Yes |
| Changing field type | ✅ Yes |
| Renaming field | ✅ Yes |
| Adding required parameter | ✅ Yes |
| Adding optional parameter | ❌ No |
| Adding new endpoint | ❌ No |
| Adding new field | ❌ No |
| Bug fixes | ❌ No |

### Breaking Change Process

1. **Announce** - 6 months before removal
2. **Document** - Migration guide published
3. **Warn** - Deprecation headers added
4. **Support** - Both versions available
5. **Remove** - Old version sunset

## SDK Versioning

### Python SDK

```bash
# Install specific version
pip install resonantgenesis==1.2.3

# Install latest
pip install resonantgenesis

# Upgrade
pip install --upgrade resonantgenesis
```

### JavaScript SDK

```bash
# Install specific version
npm install @resonantgenesis/sdk@1.2.3

# Install latest
npm install @resonantgenesis/sdk

# Upgrade
npm update @resonantgenesis/sdk
```

### SDK Compatibility Matrix

| SDK Version | API v1 | API v2 | Python | Node.js |
|-------------|--------|--------|--------|---------|
| 1.x | ✅ | ❌ | 3.8+ | 16+ |
| 2.x | ✅ | ✅ | 3.9+ | 18+ |

## Backward Compatibility

### Guarantees

Within a major version, we guarantee:

1. **Existing endpoints** remain available
2. **Response fields** are not removed
3. **Request parameters** remain optional
4. **Error codes** are not changed
5. **Authentication** methods work

### Best Practices

```python
# Good: Handle unknown fields gracefully
response = client.agents.get(agent_id)
name = response.get("name", "Unknown")

# Good: Check for optional fields
if hasattr(response, "new_field"):
    process_new_field(response.new_field)

# Good: Use SDK methods, not raw API
agent = client.agents.get(agent_id)  # SDK handles versioning
```

## Migration Guides

### v1 to v2 Migration

#### Endpoint Changes

| v1 Endpoint | v2 Endpoint |
|-------------|-------------|
| `/v1/agents/{id}/run` | `/v2/agents/{id}/execute` |
| `/v1/sessions/{id}/messages` | `/v2/sessions/{id}/steps` |
| `/v1/teams/{id}/run` | `/v2/teams/{id}/execute` |

#### Field Renames

| v1 Field | v2 Field |
|----------|----------|
| `agent.prompt` | `agent.system_prompt` |
| `session.messages` | `session.steps` |
| `result.response` | `result.output` |

#### Code Migration

```python
# v1 code
result = agent.run(prompt="Hello")
print(result.response)

# v2 code
result = agent.execute(goal="Hello")
print(result.output)
```

### Automated Migration

```bash
# Use migration tool
resonantgenesis migrate --from v1 --to v2 --dry-run

# Apply migration
resonantgenesis migrate --from v1 --to v2
```

## Version Pinning

### API Version Pinning

```python
# Pin to specific API version
client = ResonantGenesis(
    api_key="your_key",
    api_version="2026-02-01"
)
```

### SDK Version Pinning

```txt
# requirements.txt
resonantgenesis>=1.0.0,<2.0.0
```

```json
// package.json
{
  "dependencies": {
    "@resonantgenesis/sdk": "^1.0.0"
  }
}
```

## Changelog

### Format

```markdown
## [1.2.0] - 2026-02-21

### Added
- New `agent.clone()` method
- Support for custom tools

### Changed
- Improved error messages

### Deprecated
- `agent.run()` - use `agent.execute()` instead

### Fixed
- Session timeout handling
```

### Accessing Changelog

```python
# Get API changelog
changelog = client.get_changelog(
    since="2026-01-01",
    until="2026-02-21"
)

for entry in changelog:
    print(f"{entry.version}: {entry.summary}")
```

## Feature Flags

### Beta Features

```python
# Enable beta features
client = ResonantGenesis(
    api_key="your_key",
    beta_features=["streaming_v2", "agent_teams_v2"]
)
```

### Feature Availability

```python
# Check feature availability
features = client.get_features()

if features.is_available("streaming_v2"):
    # Use new streaming
    async for event in agent.execute_stream_v2(goal="Task"):
        print(event)
else:
    # Use stable streaming
    async for event in agent.execute_stream(goal="Task"):
        print(event)
```

## Testing Versions

### Sandbox Environment

```python
# Test against specific version in sandbox
client = ResonantGenesis(
    api_key="sandbox_key",
    base_url="https://sandbox.resonantgenesis.xyz",
    api_version="v2"
)
```

### Version Comparison

```python
# Compare behavior between versions
from resonantgenesis.testing import VersionComparator

comparator = VersionComparator(
    api_key="your_key",
    versions=["v1", "v2"]
)

results = comparator.compare(
    endpoint="/agents",
    method="GET"
)

print(f"Differences: {results.differences}")
```

## Notifications

### Version Announcements

Subscribe to version announcements:

```python
# Subscribe to announcements
client.notifications.subscribe(
    email="dev@company.com",
    topics=["api_versions", "deprecations", "breaking_changes"]
)
```

### Webhook Notifications

```python
# Configure version webhooks
client.webhooks.create(
    url="https://company.com/webhooks/resonant",
    events=["api.version.deprecated", "api.version.sunset"]
)
```

## Best Practices

### For Developers

1. **Pin versions** - Don't use latest in production
2. **Monitor deprecations** - Watch for warning headers
3. **Test upgrades** - Use sandbox before production
4. **Read changelogs** - Stay informed of changes
5. **Handle unknowns** - Gracefully handle new fields

### For API Consumers

1. **Use SDKs** - They handle versioning for you
2. **Subscribe to notifications** - Get advance warning
3. **Plan migrations** - Don't wait until sunset
4. **Test thoroughly** - Verify behavior after upgrades

### For Enterprise

1. **Version lock** - Pin specific versions
2. **Staged rollout** - Test in staging first
3. **Rollback plan** - Know how to revert
4. **Support contracts** - Extended support available

## API Reference

### Get API Version

```bash
GET /api/version
```

### Get Changelog

```bash
GET /api/changelog
```

### Get Deprecations

```bash
GET /api/deprecations
```

### Get Feature Flags

```bash
GET /api/features
```

---

**Need help with migration?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
