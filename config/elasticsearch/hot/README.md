# Elasticsearch HOT Tier Configuration

This directory contains the **Elasticsearch HOT Tier** configuration.

## Purpose
HOT tier is for actively indexed, frequently searched recent data (0-7 days).

## Server Details
- **Tier:** HOT (Hot/Warm/Cold/Frozen architecture)
- **Role:** Primary indexing and search
- **Port:** 9200 (REST API)
- **Data Age:** 0-7 days (recent)
- **Characteristics:** High I/O, high CPU, frequent reads/writes

## Contents
- **elasticsearch.yml** - HOT tier node configuration
- **mappings/** - Index field mappings
- **index-templates/** - Index template definitions
- **ingest-pipelines/** - Data processing pipelines
- **docker-compose.yml** - Docker deployment

## Data Lifecycle
```
New Data → HOT Tier → WARM Tier → COLD Tier → FROZEN Tier → Delete
(Recent)  (1-7d)    (7-30d)     (30d+)     (Archive)     (Cleanup)
```

## Node Configuration Example
```yaml
# Node identification
node.name: elasticsearch-hot-1
node.roles: [master, data_hot, ingest]

# Cluster settings
cluster.name: elk-cluster
cluster.initial_master_nodes: [elasticsearch-hot-1, elasticsearch-hot-2]

# Network
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

# JVM settings for HOT tier (high performance)
jvm.options: -Xms16g -Xmx16g

# Index settings
index.codec: best_compression
index.number_of_shards: 3
index.number_of_replicas: 1
index.refresh_interval: 1s
```

## HOT Tier Features
- ✨ **Real-time Indexing** - Immediate data availability
- 🔍 **Fast Searches** - Optimized for query performance
- 📝 **Bulk Ingestion** - Handle high throughput
- 💾 **Primary Storage** - All active data copies
- 🔄 **Rolling Indices** - Daily/hourly rotation

## Index Template Configuration
```json
{
  "index_patterns": ["logs-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index.lifecycle.name": "default-policy",
    "index.lifecycle.rollover_alias": "logs"
  },
  "mappings": {
    "properties": {
      "@timestamp": {"type": "date"},
      "level": {"type": "keyword"},
      "message": {"type": "text"},
      "host": {"type": "keyword"}
    }
  }
}
```

## Ingest Pipeline Configuration
```json
{
  "description": "Parse and enrich incoming logs",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{COMBINEDAPACHELOG}"]
      }
    },
    {
      "date": {
        "field": "timestamp",
        "target_field": "@timestamp"
      }
    },
    {
      "geoip": {
        "field": "clientip",
        "target_field": "geoip"
      }
    }
  ]
}
```

## ILM Policy Configuration
```json
{
  "phases": {
    "hot": {
      "min_age": "0d",
      "actions": {
        "rollover": {
          "max_size": "50gb",
          "max_age": "1d"
        },
        "set_priority": {"priority": 100}
      }
    },
    "warm": {
      "min_age": "7d",
      "actions": {
        "set_priority": {"priority": 50},
        "forcemerge": {"max_num_segments": 1}
      }
    },
    "delete": {
      "min_age": "30d",
      "actions": {"delete": {}}
    }
  }
}
```

## Configuration Checklist
- [ ] Node roles configured (data_hot)
- [ ] JVM heap settings appropriate (16GB recommended)
- [ ] Index templates created
- [ ] Ingest pipelines configured
- [ ] ILM policy configured
- [ ] Rollover alias configured
- [ ] Sharding strategy optimized
- [ ] Monitoring enabled

## Performance Tuning
```yaml
# Refresh interval (tradeoff: real-time vs performance)
index.refresh_interval: 1s  # for real-time
index.refresh_interval: 30s # for batch processing

# Thread pool settings
thread_pool.write.queue_size: 1000
thread_pool.search.queue_size: 1000

# Buffer settings
indices.memory.index_buffer_size: 30%

# Circuit breaker settings
indices.breaker.total.limit: 70%
indices.breaker.fielddata.limit: 50%
```

## Bulk Ingestion Commands
```bash
# Create index
curl -X PUT "localhost:9200/logs-2024.05.20" \
  -H 'Content-Type: application/json' \
  -d '{"settings": {"number_of_shards": 3}}')

# Bulk index documents
curl -X POST "localhost:9200/_bulk" \
  -H 'Content-Type: application/json' \
  -d @bulk-data.jsonl

# Create rollover alias
curl -X POST "localhost:9200/logs-000001/_alias/logs" \
  -H 'Content-Type: application/json' \
  -d '{"is_write_index": true}'
```

## Monitoring HOT Tier
```bash
# Check index stats
curl -X GET "localhost:9200/_cat/indices?v"

# Monitor indexing rate
curl -X GET "localhost:9200/_nodes/stats/indices/indexing"

# Check disk usage
curl -X GET "localhost:9200/_cat/shards?h=index,shard,prirep,state,docs,bytes"

# Monitor JVM memory
curl -X GET "localhost:9200/_nodes/stats/jvm"
```

## Troubleshooting
- **High CPU:** Reduce refresh_interval or increase shards
- **Memory issues:** Increase JVM heap or reduce doc size
- **Slow indexing:** Check thread_pool.write queue size
- **Search latency:** Optimize query patterns and filters

## Important Links
- [HOT Tier Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-tiers.html#hot-tier)
- [ILM Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)
- [Ingest Pipelines](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)
- [Index Templates](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-template.html)