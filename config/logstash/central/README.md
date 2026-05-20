# Logstash Central Configuration

This directory contains **Logstash Central** configuration for aggregating and processing data.

## Purpose
Central Logstash aggregates data from all edge instances, applies final transformation, and forwards to Elasticsearch.

## Server Details
- **Role:** Central data aggregation hub
- **Input Port:** 5000 (TCP/TLS from edge instances)
- **Output:** Elasticsearch (9200)
- **Features:** Pipeline coordination, load balancing, enrichment

## Data Flow
```
Edge Logstash Instances (7 regions)
    ↓
Logstash Central
    ├─ Input: TCP 5000 (from edges)
    ├─ Filter: Transform, enrich, deduplicate
    ├─ Output: Elasticsearch (9200)
    └─ DLQ: Dead letter queue for errors
```

## Contents
- **logstash.yml** - Main Logstash configuration
- **pipelines/** - Central processing pipelines
- **patterns/** - Grok patterns for parsing
- **plugins/** - Custom plugins
- **docker-compose.yml** - Docker deployment

## Logstash Configuration Example
```yaml
# Node settings
node.name: logstash-central-1
path.data: /var/lib/logstash
path.logs: /var/log/logstash

# Pipeline settings
pipeline.id: central-pipeline
pipeline.workers: 8
pipeline.batch.size: 5000
pipeline.batch.delay: 50

# Queue settings
queue.type: memory
queue.max_bytes: 1gb

# Monitoring
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.url: ["http://elasticsearch:9200"]
```

## Central Pipeline Configuration
```ruby
input {
  tcp {
    port => 5000
    codec => json
    type => "edge-logs"
    tags => ["from-edge"]
  }
}

filter {
  # Remove duplicate events
  if [_source][processed] == "true" {
    drop { }
  }
  
  # Standardize timestamp
  date {
    match => ["timestamp", "ISO8601"]
    target => "@timestamp"
  }
  
  # Add central processing tag
  mutate {
    add_field => {
      "processed_by" => "logstash-central"
      "processing_timestamp" => "%{@timestamp}"
    }
  }
  
  # Enrich with geographic data
  geoip {
    source => "source_ip"
    target => "geoip"
  }
  
  # Add metadata
  mutate {
    add_field => {
      "cluster" => "elk-production"
      "environment" => "prod"
    }
  }
}

output {
  # Main output to Elasticsearch
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    user => "logstash"
    password => "${LOGSTASH_PASSWORD}"
  }
  
  # Dead letter queue for failed events
  dead_letter_queue {
    enable => true
  }
  
  # Monitoring output
  stdout {
    codec => rubydebug
  }
}
```

## Pipeline Configuration Structure
```
pipelines/
├── central.conf         - Main aggregation pipeline
├── enrichment.conf      - Data enrichment
├── deduplication.conf   - Remove duplicates
├── filtering.conf       - Event filtering
└── error-handling.conf  - Error management
```

## Input Pipelines
```ruby
# Receive from Karna edge
input {
  tcp {
    port => 5001
    codec => json
    type => "karna-logs"
  }
}

# Receive from SSPL edge
input {
  tcp {
    port => 5002
    codec => json
    type => "sspl-logs"
  }
}

# Receive from Dhani edge
input {
  tcp {
    port => 5003
    codec => json
    type => "dhani-logs"
  }
}

# ... (continue for other edges)
```

## Enrichment Filter
```ruby
filter {
  # GeoIP enrichment
  geoip {
    source => "client_ip"
    target => "geoip"
  }
  
  # User agent parsing
  useragent {
    source => "user_agent"
    target => "ua"
  }
  
  # DNS lookup
  dns {
    reverse => ["client_ip"]
    action => "append"
  }
  
  # Custom enrichment
  if [environment] == "production" {
    mutate {
      add_field => {"priority" => "high"}
    }
  }
}
```

## Dead Letter Queue (DLQ) Processing
```ruby
input {
  dead_letter_queue {
    path => "/var/lib/logstash/dead-letter-queue"
    commit_offsets => true
    pipeline_id => "central-pipeline"
  }
}

filter {
  # Log failed events
  ruby {
    code => 'event.set("dlq_processing", Time.now)'
  }
}

output {
  # Send to DLQ index in Elasticsearch
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "dlq-logs-%{+YYYY.MM.dd}"
  }
  
  # Alert on critical DLQ events
  email {
    to => "ops@company.com"
    subject => "DLQ Alert: Failed to process event"
  }
}
```

## Configuration Checklist
- [ ] TCP input ports configured (5000+)
- [ ] TLS/SSL certificates installed
- [ ] Elasticsearch connectivity verified
- [ ] Pipeline parallelization tuned
- [ ] DLQ configured and monitored
- [ ] Backpressure handling configured
- [ ] Monitoring enabled
- [ ] Log rotation configured

## Performance Tuning
```yaml
# Worker threads (CPU cores = workers)
pipeline.workers: 8

# Batch processing
pipeline.batch.size: 5000    # events per batch
pipeline.batch.delay: 50     # milliseconds

# Queue settings
queue.type: memory
queue.max_bytes: 1gb
queue.drain: true

# JVM settings
jvm.options: -Xms4g -Xmx4g
```

## Monitoring Commands
```bash
# Check Logstash status
curl -X GET "localhost:9600"

# Pipeline stats
curl -X GET "localhost:9600/_node/pipelines"

# Performance metrics
curl -X GET "localhost:9600/_node/stats/pipelines"

# JVM stats
curl -X GET "localhost:9600/_node/stats/jvm"
```

## Load Balancing Strategy
```
Edge Instances (7 locations)
    ↓
┌─────────┬─────────┬─────────┐
│ Central │ Central │ Central │
│  Node 1 │  Node 2 │  Node 3 │
└────┬────┴────┬────┴────┬────┘
     └─────────┼─────────┘
              ↓
       Elasticsearch Cluster
```

## Error Handling
```ruby
# Capture errors
output {
  if "_grokparsefailure" in [tags] {
    file {
      path => "/var/log/logstash/parse-errors.log"
    }
  }
  
  if [@metadata][elasticsearch][error] {
    dead_letter_queue {
      enable => true
    }
  }
}
```

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Pipeline Configuration](https://www.elastic.co/guide/en/logstash/current/configuration.html)
- [Performance Tuning](https://www.elastic.co/guide/en/logstash/current/performance-troubleshooting.html)