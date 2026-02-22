# Migrations Guide

Complete guide to database migrations, API versioning, and upgrade paths in ResonantGenesis.

## Overview

This guide covers:
- Database schema migrations
- API version migrations
- SDK upgrade paths
- Data migration strategies
- Rollback procedures

## Database Migrations

### Using Alembic

ResonantGenesis uses Alembic for database migrations.

#### Running Migrations

```bash
# Apply all pending migrations
alembic upgrade head

# Apply specific migration
alembic upgrade abc123

# Check current version
alembic current

# Show migration history
alembic history
```

#### Creating Migrations

```bash
# Auto-generate migration from model changes
alembic revision --autogenerate -m "Add user preferences table"

# Create empty migration
alembic revision -m "Custom data migration"
```

#### Migration File Structure

```python
"""Add user preferences table

Revision ID: abc123
Revises: def456
Create Date: 2026-02-21 10:00:00.000000
"""
from alembic import op
import sqlalchemy as sa

revision = 'abc123'
down_revision = 'def456'
branch_labels = None
depends_on = None

def upgrade():
    op.create_table(
        'user_preferences',
        sa.Column('id', sa.UUID(), primary_key=True),
        sa.Column('user_id', sa.UUID(), sa.ForeignKey('users.id')),
        sa.Column('theme', sa.String(50), default='light'),
        sa.Column('notifications', sa.Boolean(), default=True),
        sa.Column('created_at', sa.DateTime(), server_default=sa.func.now())
    )
    op.create_index('ix_user_preferences_user_id', 'user_preferences', ['user_id'])

def downgrade():
    op.drop_index('ix_user_preferences_user_id')
    op.drop_table('user_preferences')
```

### Migration Best Practices

1. **Always test locally first**
   ```bash
   # Test upgrade
   alembic upgrade head
   
   # Test downgrade
   alembic downgrade -1
   
   # Test upgrade again
   alembic upgrade head
   ```

2. **Use transactions**
   ```python
   def upgrade():
       with op.batch_alter_table('users') as batch_op:
           batch_op.add_column(sa.Column('new_field', sa.String(100)))
   ```

3. **Handle data migrations carefully**
   ```python
   def upgrade():
       # Schema change
       op.add_column('agents', sa.Column('version', sa.Integer(), default=1))
       
       # Data migration
       connection = op.get_bind()
       connection.execute(
           sa.text("UPDATE agents SET version = 1 WHERE version IS NULL")
       )
   ```

4. **Make migrations reversible**
   - Always implement `downgrade()`
   - Test both directions

### Production Migrations

```bash
# Backup database first
pg_dump -h localhost -U resonant resonantgenesis > backup.sql

# Run migrations
alembic upgrade head

# Verify
alembic current
```

## API Versioning

### Version Format

API versions follow the format: `v{major}.{minor}`

| Version | Status | End of Life |
|---------|--------|-------------|
| v1.0 | Deprecated | March 2026 |
| v1.1 | Supported | June 2026 |
| v1.2 | Current | - |

### Using API Versions

**In URL:**
```bash
curl https://resonantgenesis.xyz/api/v1.2/agents
```

**In Header:**
```bash
curl https://resonantgenesis.xyz/api/agents \
  -H "X-API-Version: 1.2"
```

**In SDK:**
```python
client = ResonantClient(
    api_key="...",
    api_version="1.2"
)
```

### Version Deprecation

When a version is deprecated:

1. **Deprecation notice** - 6 months before EOL
2. **Warning headers** - Added to responses
3. **Documentation** - Migration guide published
4. **EOL** - Version returns 410 Gone

**Deprecation Header:**
```http
X-API-Deprecation-Date: 2026-03-01
X-API-Sunset-Date: 2026-06-01
X-API-Upgrade-To: v1.2
```

### Breaking Changes

Breaking changes require a new major version:

- Removing endpoints
- Removing required fields
- Changing field types
- Changing authentication
- Changing error formats

Non-breaking changes can be in minor versions:

- Adding endpoints
- Adding optional fields
- Adding new error codes
- Performance improvements

## SDK Migrations

### Python SDK

#### Upgrading

```bash
pip install --upgrade resonantgenesis
```

#### Version Compatibility

| SDK Version | API Version | Python |
|-------------|-------------|--------|
| 1.0.x | v1.0 | 3.8+ |
| 1.1.x | v1.1 | 3.9+ |
| 2.0.x | v1.2 | 3.10+ |

#### Migration: 1.x to 2.x

**Breaking Changes:**

1. **Async by default**
   ```python
   # Old (1.x)
   result = client.agents.list()
   
   # New (2.x)
   result = await client.agents.list()
   
   # Or use sync wrapper
   from resonantgenesis import SyncClient
   client = SyncClient(api_key="...")
   result = client.agents.list()
   ```

2. **Response objects**
   ```python
   # Old (1.x)
   agent = client.agents.get(id)
   print(agent['name'])
   
   # New (2.x)
   agent = await client.agents.get(id)
   print(agent.name)
   ```

3. **Error handling**
   ```python
   # Old (1.x)
   from resonantgenesis.exceptions import APIError
   
   # New (2.x)
   from resonantgenesis import ResonantError
   ```

### JavaScript SDK

#### Upgrading

```bash
npm install @resonantgenesis/sdk@latest
```

#### Version Compatibility

| SDK Version | API Version | Node.js |
|-------------|-------------|---------|
| 1.0.x | v1.0 | 16+ |
| 1.1.x | v1.1 | 18+ |
| 2.0.x | v1.2 | 20+ |

#### Migration: 1.x to 2.x

**Breaking Changes:**

1. **ESM only**
   ```typescript
   // Old (1.x) - CommonJS
   const { ResonantClient } = require('@resonantgenesis/sdk');
   
   // New (2.x) - ESM
   import { ResonantClient } from '@resonantgenesis/sdk';
   ```

2. **TypeScript strict mode**
   ```typescript
   // Old (1.x)
   const agent = await client.agents.get(id);
   agent.unknownField; // No error
   
   // New (2.x)
   const agent = await client.agents.get(id);
   agent.unknownField; // TypeScript error
   ```

## Data Migrations

### Exporting Data

```bash
# Export agents
curl https://resonantgenesis.xyz/api/v1/agents/export \
  -H "Authorization: Bearer $TOKEN" \
  -o agents.json

# Export sessions
curl https://resonantgenesis.xyz/api/v1/sessions/export \
  -H "Authorization: Bearer $TOKEN" \
  -o sessions.json
```

### Importing Data

```bash
# Import agents
curl -X POST https://resonantgenesis.xyz/api/v1/agents/import \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @agents.json
```

### Bulk Operations

```python
# Export all data
export = await client.data.export_all()

# Save to file
with open('backup.json', 'w') as f:
    json.dump(export, f)

# Import to new account
with open('backup.json', 'r') as f:
    data = json.load(f)

await new_client.data.import_all(data)
```

## Rollback Procedures

### Database Rollback

```bash
# Rollback last migration
alembic downgrade -1

# Rollback to specific version
alembic downgrade abc123

# Rollback all
alembic downgrade base
```

### Application Rollback

```bash
# Docker rollback
docker pull resonantgenesis/backend:v1.2.3
docker-compose up -d

# Kubernetes rollback
kubectl rollout undo deployment/backend
```

### Data Rollback

```bash
# Restore from backup
psql -h localhost -U resonant resonantgenesis < backup.sql

# Point-in-time recovery (if configured)
pg_restore --target-time="2026-02-21 10:00:00" backup.dump
```

## Migration Checklist

### Before Migration

- [ ] Backup database
- [ ] Review changelog
- [ ] Test in staging
- [ ] Notify users
- [ ] Schedule maintenance window

### During Migration

- [ ] Enable maintenance mode
- [ ] Run database migrations
- [ ] Deploy new code
- [ ] Run smoke tests
- [ ] Monitor logs

### After Migration

- [ ] Disable maintenance mode
- [ ] Verify functionality
- [ ] Monitor metrics
- [ ] Update documentation
- [ ] Notify users of completion

## Troubleshooting

### Migration Failed

1. **Check logs**
   ```bash
   alembic history --verbose
   ```

2. **Identify failed migration**
   ```bash
   alembic current
   ```

3. **Fix and retry**
   ```bash
   # Fix the migration file
   # Then retry
   alembic upgrade head
   ```

### Version Mismatch

```python
# Check versions
print(f"SDK: {resonantgenesis.__version__}")
print(f"API: {client.api_version}")

# Force specific version
client = ResonantClient(
    api_key="...",
    api_version="1.2"
)
```

### Data Corruption

1. **Stop application**
2. **Restore from backup**
3. **Investigate cause**
4. **Fix and re-migrate**

---

## API Changelog

### v1.2 (Current)

- Added Agent Teams API
- Added Memory System API
- Enhanced webhook triggers
- Improved error responses

### v1.1

- Added custom tools
- Added session streaming
- Added rate limit headers

### v1.0

- Initial release
- Core agents API
- Sessions API
- Basic webhooks

---

**Need help?** Contact support@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
