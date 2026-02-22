# Backup & Recovery Guide

Complete guide to backup procedures, disaster recovery, and business continuity for ResonantGenesis.

## Overview

ResonantGenesis implements comprehensive backup and recovery procedures to ensure data protection and business continuity. This guide covers backup strategies, recovery procedures, and disaster recovery planning.

## Backup Strategy

### Backup Types

| Type | Frequency | Retention | RPO |
|------|-----------|-----------|-----|
| **Full Backup** | Weekly | 90 days | 7 days |
| **Incremental** | Daily | 30 days | 24 hours |
| **Transaction Log** | Hourly | 7 days | 1 hour |
| **Real-time Replication** | Continuous | N/A | Minutes |

### What's Backed Up

| Data Type | Backup Method | Location |
|-----------|---------------|----------|
| Database | PostgreSQL pg_dump | S3 + Cross-region |
| File Storage | S3 replication | Multi-region |
| Configuration | Git + encrypted backup | S3 |
| Secrets | Vault snapshots | Encrypted S3 |
| Logs | Log aggregation | Elasticsearch |

### Backup Schedule

```yaml
# Backup schedule configuration
backups:
  database:
    full:
      schedule: "0 2 * * 0"  # Sunday 2 AM
      retention: 90d
    incremental:
      schedule: "0 2 * * 1-6"  # Mon-Sat 2 AM
      retention: 30d
    transaction_log:
      schedule: "0 * * * *"  # Every hour
      retention: 7d
  
  files:
    schedule: "0 3 * * *"  # Daily 3 AM
    retention: 30d
  
  configuration:
    schedule: "0 4 * * *"  # Daily 4 AM
    retention: 90d
```

## Database Backup

### PostgreSQL Backup

```bash
# Full backup
pg_dump -Fc -h localhost -U resonant resonantgenesis > backup_$(date +%Y%m%d).dump

# Incremental backup (WAL archiving)
archive_command = 'aws s3 cp %p s3://backups/wal/%f'

# Point-in-time recovery setup
restore_command = 'aws s3 cp s3://backups/wal/%f %p'
```

### Automated Backup Script

```python
import subprocess
from datetime import datetime
import boto3

def backup_database():
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = f"/tmp/backup_{timestamp}.dump"
    
    # Create backup
    subprocess.run([
        "pg_dump", "-Fc",
        "-h", "localhost",
        "-U", "resonant",
        "-d", "resonantgenesis",
        "-f", backup_file
    ], check=True)
    
    # Upload to S3
    s3 = boto3.client('s3')
    s3.upload_file(
        backup_file,
        "resonant-backups",
        f"database/{timestamp}.dump"
    )
    
    # Cleanup
    os.remove(backup_file)
    
    return f"database/{timestamp}.dump"
```

### Backup Verification

```python
def verify_backup(backup_path: str) -> bool:
    """Verify backup integrity"""
    
    # Download backup
    s3 = boto3.client('s3')
    local_path = "/tmp/verify_backup.dump"
    s3.download_file("resonant-backups", backup_path, local_path)
    
    # Verify with pg_restore
    result = subprocess.run([
        "pg_restore", "--list", local_path
    ], capture_output=True)
    
    # Cleanup
    os.remove(local_path)
    
    return result.returncode == 0
```

## File Storage Backup

### S3 Cross-Region Replication

```yaml
# S3 replication configuration
replication:
  role: arn:aws:iam::123456789:role/s3-replication
  rules:
    - id: ReplicateAll
      status: Enabled
      priority: 1
      destination:
        bucket: arn:aws:s3:::resonant-backup-dr
        storage_class: STANDARD_IA
      filter:
        prefix: ""
```

### File Backup Script

```python
def backup_files():
    """Backup file storage to DR region"""
    
    s3 = boto3.client('s3')
    
    # List all objects
    paginator = s3.get_paginator('list_objects_v2')
    
    for page in paginator.paginate(Bucket='resonant-files'):
        for obj in page.get('Contents', []):
            # Copy to DR bucket
            s3.copy_object(
                CopySource={'Bucket': 'resonant-files', 'Key': obj['Key']},
                Bucket='resonant-files-dr',
                Key=obj['Key']
            )
```

## Configuration Backup

### Git-Based Configuration

```bash
# Backup configuration to Git
git add config/
git commit -m "Configuration backup $(date +%Y%m%d)"
git push origin main

# Encrypt sensitive config
gpg --encrypt --recipient backup@resonantgenesis.xyz config/secrets.yaml
```

### Environment Backup

```python
def backup_configuration():
    """Backup all configuration files"""
    
    config_files = [
        "/etc/resonant/config.yaml",
        "/etc/resonant/nginx.conf",
        "/etc/resonant/docker-compose.yaml"
    ]
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    archive = f"/tmp/config_{timestamp}.tar.gz"
    
    # Create archive
    with tarfile.open(archive, "w:gz") as tar:
        for f in config_files:
            tar.add(f)
    
    # Encrypt
    subprocess.run([
        "gpg", "--encrypt",
        "--recipient", "backup@resonantgenesis.xyz",
        archive
    ])
    
    # Upload
    s3 = boto3.client('s3')
    s3.upload_file(f"{archive}.gpg", "resonant-backups", f"config/{timestamp}.tar.gz.gpg")
```

## Recovery Procedures

### Database Recovery

#### Full Recovery

```bash
# Stop application
systemctl stop resonant-api

# Restore from backup
pg_restore -h localhost -U resonant -d resonantgenesis -c backup.dump

# Verify data
psql -h localhost -U resonant -d resonantgenesis -c "SELECT COUNT(*) FROM agents;"

# Start application
systemctl start resonant-api
```

#### Point-in-Time Recovery

```bash
# Stop PostgreSQL
systemctl stop postgresql

# Restore base backup
pg_restore -D /var/lib/postgresql/data base_backup.tar

# Configure recovery
cat > /var/lib/postgresql/data/recovery.conf << EOF
restore_command = 'aws s3 cp s3://backups/wal/%f %p'
recovery_target_time = '2026-02-21 03:00:00'
EOF

# Start PostgreSQL
systemctl start postgresql
```

### File Recovery

```python
def recover_files(timestamp: str):
    """Recover files from backup"""
    
    s3 = boto3.client('s3')
    
    # List backup objects
    response = s3.list_objects_v2(
        Bucket='resonant-files-dr',
        Prefix=f"backup/{timestamp}/"
    )
    
    for obj in response.get('Contents', []):
        # Restore to primary bucket
        s3.copy_object(
            CopySource={'Bucket': 'resonant-files-dr', 'Key': obj['Key']},
            Bucket='resonant-files',
            Key=obj['Key'].replace(f"backup/{timestamp}/", "")
        )
```

### Configuration Recovery

```bash
# Download encrypted config
aws s3 cp s3://resonant-backups/config/latest.tar.gz.gpg /tmp/

# Decrypt
gpg --decrypt /tmp/latest.tar.gz.gpg > /tmp/config.tar.gz

# Extract
tar -xzf /tmp/config.tar.gz -C /etc/resonant/

# Restart services
systemctl restart resonant-api
```

## Disaster Recovery

### DR Architecture

```
Primary Region (us-east-1)          DR Region (us-west-2)
┌─────────────────────┐             ┌─────────────────────┐
│   Load Balancer     │             │   Load Balancer     │
│         ↓           │             │         ↓           │
│   API Servers       │────────────→│   API Servers       │
│         ↓           │  Failover   │         ↓           │
│   PostgreSQL        │────────────→│   PostgreSQL        │
│   (Primary)         │  Replication│   (Standby)         │
│         ↓           │             │         ↓           │
│   S3 Storage        │────────────→│   S3 Storage        │
│                     │  Replication│                     │
└─────────────────────┘             └─────────────────────┘
```

### RTO and RPO

| Scenario | RTO | RPO |
|----------|-----|-----|
| Single server failure | 5 minutes | 0 |
| Availability zone failure | 15 minutes | 0 |
| Region failure | 1 hour | 15 minutes |
| Data corruption | 4 hours | 1 hour |

### Failover Procedure

```python
def initiate_failover():
    """Initiate DR failover"""
    
    # 1. Verify DR readiness
    if not verify_dr_health():
        raise Exception("DR region not healthy")
    
    # 2. Promote DR database
    promote_dr_database()
    
    # 3. Update DNS
    update_dns_to_dr()
    
    # 4. Verify services
    verify_dr_services()
    
    # 5. Notify team
    send_failover_notification()

def promote_dr_database():
    """Promote DR database to primary"""
    
    # Connect to DR database
    conn = psycopg2.connect(DR_DATABASE_URL)
    cursor = conn.cursor()
    
    # Promote to primary
    cursor.execute("SELECT pg_promote()")
    conn.commit()

def update_dns_to_dr():
    """Update DNS to point to DR region"""
    
    route53 = boto3.client('route53')
    
    route53.change_resource_record_sets(
        HostedZoneId='ZONE_ID',
        ChangeBatch={
            'Changes': [{
                'Action': 'UPSERT',
                'ResourceRecordSet': {
                    'Name': 'api.resonantgenesis.xyz',
                    'Type': 'A',
                    'AliasTarget': {
                        'HostedZoneId': 'DR_ALB_ZONE',
                        'DNSName': 'dr-alb.us-west-2.elb.amazonaws.com',
                        'EvaluateTargetHealth': True
                    }
                }
            }]
        }
    )
```

### Failback Procedure

```python
def initiate_failback():
    """Return to primary region after DR event"""
    
    # 1. Verify primary region health
    verify_primary_health()
    
    # 2. Sync data from DR to primary
    sync_data_to_primary()
    
    # 3. Verify data consistency
    verify_data_consistency()
    
    # 4. Update DNS back to primary
    update_dns_to_primary()
    
    # 5. Demote DR database back to standby
    demote_dr_database()
    
    # 6. Verify services
    verify_primary_services()
```

## Business Continuity

### BCP Overview

| Component | Strategy | Recovery Time |
|-----------|----------|---------------|
| API Services | Multi-AZ, Auto-scaling | < 5 min |
| Database | Multi-AZ, Read replicas | < 15 min |
| File Storage | Cross-region replication | < 5 min |
| DNS | Route 53 health checks | < 1 min |

### Communication Plan

| Severity | Notification | Channels |
|----------|--------------|----------|
| P1 | Immediate | PagerDuty, Slack, Email |
| P2 | 15 minutes | Slack, Email |
| P3 | 1 hour | Email |
| P4 | 24 hours | Email |

### Runbook

```markdown
## DR Runbook

### Pre-Failover Checklist
- [ ] Verify DR region health
- [ ] Confirm data replication lag < 15 min
- [ ] Notify on-call team
- [ ] Prepare rollback plan

### Failover Steps
1. [ ] Initiate database failover
2. [ ] Update DNS records
3. [ ] Verify API health
4. [ ] Test critical paths
5. [ ] Update status page
6. [ ] Notify customers

### Post-Failover
- [ ] Monitor error rates
- [ ] Check performance metrics
- [ ] Document timeline
- [ ] Schedule post-mortem
```

## Testing

### Backup Testing

```python
def test_backup_restore():
    """Monthly backup restore test"""
    
    # Get latest backup
    backup = get_latest_backup()
    
    # Restore to test environment
    restore_to_test_env(backup)
    
    # Run verification queries
    results = run_verification_queries()
    
    # Generate report
    generate_test_report(results)
```

### DR Testing

| Test Type | Frequency | Scope |
|-----------|-----------|-------|
| Tabletop exercise | Quarterly | Full team |
| Failover test | Semi-annual | DR region |
| Full DR drill | Annual | Complete failover |

### Test Schedule

```yaml
# DR test schedule
tests:
  backup_restore:
    frequency: monthly
    scope: database, files
    
  failover_test:
    frequency: semi-annual
    scope: dr_region
    notification: 48h_advance
    
  full_dr_drill:
    frequency: annual
    scope: complete_failover
    notification: 1w_advance
```

## Monitoring

### Backup Monitoring

```python
# Monitor backup status
def check_backup_status():
    metrics = {
        "last_backup_time": get_last_backup_time(),
        "backup_size_gb": get_backup_size(),
        "backup_duration_min": get_backup_duration(),
        "replication_lag_sec": get_replication_lag()
    }
    
    # Alert if backup is stale
    if metrics["last_backup_time"] > timedelta(hours=25):
        send_alert("Backup is stale", severity="high")
    
    return metrics
```

### Replication Monitoring

```sql
-- Check replication lag
SELECT
    client_addr,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;
```

## API Reference

### Trigger Backup

```bash
POST /api/v1/admin/backup
```

### Get Backup Status

```bash
GET /api/v1/admin/backup/status
```

### List Backups

```bash
GET /api/v1/admin/backups
```

### Restore Backup

```bash
POST /api/v1/admin/restore
```

### Initiate Failover

```bash
POST /api/v1/admin/failover
```

---

**Need help with recovery?** Contact operations@resonantgenesis.xyz or call the emergency hotline.
