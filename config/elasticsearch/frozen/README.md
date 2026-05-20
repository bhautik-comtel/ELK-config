# Elasticsearch FROZEN Tier Configuration

This directory contains the **Elasticsearch FROZEN Tier** configuration.

## Purpose
FROZEN tier is for long-term archival storage with minimal resource requirements but still searchable via searchable snapshots.

## Server Details
- **Tier:** FROZEN (Hot/Warm/Cold/Frozen architecture)
- **Role:** Long-term archive storage
- **Port:** 9200 (REST API)
- **Data Age:** 30+ days (archive)
- **Characteristics:** Low cost, minimal I/O, searchable via snapshots

## Data Lifecycle
```
New Data → HOT Tier → WARM Tier → COLD Tier → FROZEN Tier → Delete
(Recent)  (1-7d)    (7-30d)     (30d+)     (Archive)     (Cleanup)
```

## Contents
- **elasticsearch.yml** - FROZEN tier configuration
- **ilm-policies/** - Index Lifecycle Management policies
- **repository-settings/** - Snapshot repository configuration
- **docker-compose.yml** - Docker deployment

## Configuration Structure
```
frozen/
├── elasticsearch.yml           - FROZEN tier node configuration
├── ilm-policies/
│   ├── default-policy.json    - Default ILM policy
│   └── long-term-policy.json  - Long-term retention policy
├── repository-settings/
│   ├── snapshot-repo.json     - Snapshot repository config
│   └── s3-repository.json     - S3 backend example
└── docker-compose.yml         - Docker deployment
```

## Node Configuration
```yaml
# Node role - frozen tier
node.roles: [data_frozen]

# Reduced resource requirements
jvm.options: -Xms8g -Xmx8g

# Searchable snapshots
xpack.searchable.snapshots.shared_cache.size: "5gb"
xpack.searchable.snapshots.cache_fetch_async: true
```

## FROZEN Tier Features
- 📦 **Searchable Snapshots** - Query archived data without full restoration
- 💾 **Minimal Storage** - Compressed snapshots on S3/GCS/etc
- 🔍 **Transparent Search** - Query frozen data like hot data
- ⏱️ **Lazy Loading** - Data fetched on-demand from snapshot

## ILM Policy Configuration
```json
{
  "phases": {
    "frozen": {
      "min_age": "30d",
      "actions": {
        "searchable_snapshot": {
          "snapshot_repository": "archive-repo"
        },
        "set_priority": {
          "priority": 0
        }
      }
    },
    "delete": {
      "min_age": "90d",
      "actions": {
        "delete": {}
      }
    }
  }
}
```

## Snapshot Repository Configuration
```json
{
  "type": "s3",
  "settings": {
    "bucket": "elastic-snapshots",
    "base_path": "elasticsearch",
    "compress": true,
    "max_snapshot_bytes_per_sec": "40mb",
    "max_restore_bytes_per_sec": "40mb",
    "chunk_size": "1gb",
    "server_side_encryption": true,
    "storage_class": "GLACIER"
  }
}
```

## Configuration Checklist
- [ ] FROZEN node role configured
- [ ] Snapshot repository created and tested
- [ ] ILM policy configured for frozen phase
- [ ] Searchable snapshots enabled
- [ ] Cache size configured appropriately
- [ ] Backup retention policies defined
- [ ] Monitoring enabled
- [ ] Restore procedures tested

## Storage Backends Supported
| Backend | Use Case | Cost |
|---------|----------|------|
| S3 | AWS cloud storage | Low |
| GCS | Google Cloud storage | Low |
| Azure | Azure Blob storage | Low |
| Local FS | On-premises | Very Low |
| HDFS | Hadoop ecosystem | Varies |

## Lifecycle Example
```yaml
# 0-7 days: HOT tier (frequent reads/writes)
# 7-30 days: WARM tier (occasional reads)
# 30-90 days: COLD tier (rare reads)
# 90+ days: FROZEN tier (archived, searchable snapshots)
# 180+ days: DELETE (removed permanently)
```

## Creating Snapshots
```bash
# Register snapshot repository
curl -X PUT "localhost:9200/_snapshot/archive-repo" \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "s3",
    "settings": {"bucket": "my-backups"}
  }'

# Create snapshot
curl -X PUT "localhost:9200/_snapshot/archive-repo/snapshot-1"

# Monitor snapshot status
curl -X GET "localhost:9200/_snapshot/archive-repo/snapshot-1"
```

## Searchable Snapshots Commands
```bash
# Mount searchable snapshot
curl -X POST "localhost:9200/_snapshot/archive-repo/snapshot-1/_mount?pretty"

# Check searchable snapshot cache
curl -X GET "localhost:9200/_searchable_snapshots/cache/stats"

# Restore from snapshot
curl -X POST "localhost:9200/_snapshot/archive-repo/snapshot-1/_restore"
```

## Performance Tuning
```yaml
# Cache settings for searchable snapshots
xpack.searchable.snapshots.shared_cache.size: "5gb"
xpack.searchable.snapshots.cache_fetch_async: true
xpack.searchable.snapshots.cache_dir: "/mnt/cache"

# Snapshot upload/download limits
repository.s3.max_snapshot_bytes_per_sec: "40mb"
repository.s3.max_restore_bytes_per_sec: "40mb"
```

## Monitoring Storage
```bash
# Check index size and allocation
curl -X GET "localhost:9200/_cat/indices?v"

# Monitor snapshot repository
curl -X GET "localhost:9200/_snapshot/_status"

# Check disk usage
curl -X GET "localhost:9200/_nodes/stats/fs"
```

## Cost Optimization
- ✅ Compress snapshots (reduces by 50-80%)
- ✅ Use appropriate storage class (GLACIER for long-term)
- ✅ Delete expired snapshots regularly
- ✅ Monitor bandwidth for uploads/downloads
- ✅ Use searchable snapshots instead of restoration

## Troubleshooting
- **Snapshot fails:** Check repository connectivity and permissions
- **Restore slow:** Increase max_restore_bytes_per_sec
- **Cache misses:** Increase shared_cache.size
- **Quota exceeded:** Review retention policies and delete old indices

## Links
- [FROZEN Tier Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-tiers.html#frozen-tier)
- [Searchable Snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/current/searchable-snapshots.html)
- [Snapshot & Restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html)
- [ILM Frozen Phase](https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-index-lifecycle.html)
