# Elasticsearch ML (Machine Learning) Configuration

This directory contains **Elasticsearch ML** (Machine Learning) configuration.

## Purpose
ML tier enables advanced analytics, anomaly detection, and forecasting capabilities.

## Server Details
- **Tier:** ML (Machine Learning dedicated node)
- **Role:** ML jobs, anomaly detection, forecasting
- **Port:** 9200 (REST API)
- **Characteristics:** CPU-intensive, requires sufficient memory

## Contents
- **elasticsearch.yml** - ML node configuration
- **ml-jobs/** - ML job definitions
- **anomaly-detection/** - Anomaly detection configurations
- **forecasting/** - Forecasting models
- **docker-compose.yml** - Docker deployment

## Node Configuration Example
```yaml
# Node identification
node.name: elasticsearch-ml-1
node.roles: [ml, remote_cluster_client]

# ML settings
node.ml: true
xpack.ml.enabled: true
xpack.ml.max_open_jobs: 20
xpack.ml.memory.auto_cap: true

# JVM settings for ML (high CPU/memory)
jvm.options: -Xms12g -Xmx12g

# Network
network.host: 0.0.0.0
http.port: 9200
```

## ML Capabilities
- 🤖 **Anomaly Detection** - Identify unusual patterns
- 📊 **Forecasting** - Predict future trends
- 🏷️ **Log Categorization** - Auto-group similar logs
- 🎯 **Outlier Detection** - Find statistical outliers
- 📈 **Trend Analysis** - Analyze patterns over time

## Anomaly Detection Job Configuration
```json
{
  "job_id": "cpu-anomaly-detection",
  "description": "Detect anomalies in CPU usage",
  "groups": ["system-metrics"],
  "analysis_config": {
    "bucket_span": "15m",
    "detectors": [
      {
        "function": "high_mean",
        "field_name": "cpu_usage",
        "by_field_name": "hostname",
        "partition_field_name": "datacenter"
      }
    ],
    "influencers": ["hostname", "datacenter"]
  },
  "data_description": {
    "time_field": "@timestamp",
    "time_format": "epoch_ms"
  },
  "model_memory_limit": "512mb",
  "results_retention_days": 30
}
```

## Forecasting Job Configuration
```json
{
  "job_id": "traffic-forecast",
  "description": "Forecast network traffic trends",
  "analysis_config": {
    "bucket_span": "1h",
    "detectors": [
      {
        "function": "mean",
        "field_name": "network_bytes",
        "partition_field_name": "interface"
      }
    ]
  },
  "data_description": {
    "time_field": "@timestamp"
  },
  "model_memory_limit": "1024mb"
}
```

## Detector Functions
| Function | Use Case | Example |
|----------|----------|----------|
| high_mean | Upper threshold anomaly | CPU usage spike |
| low_mean | Lower threshold anomaly | Traffic drop |
| high_count | Spike in event count | Error rate increase |
| low_count | Decrease in event count | Service down |
| rare | Rare values | New error type |
| distinct_count | Unique value changes | New source IPs |

## Create ML Job
```bash
# Create anomaly detection job
curl -X PUT "localhost:9200/_ml/anomaly_detectors/cpu-anomaly" \
  -H 'Content-Type: application/json' \
  -d @anomaly-job.json

# Open job to process data
curl -X POST "localhost:9200/_ml/anomaly_detectors/cpu-anomaly/_open"

# Create datafeed to supply data
curl -X PUT "localhost:9200/_ml/datafeeds/cpu-anomaly-feed" \
  -H 'Content-Type: application/json' \
  -d '{
    "job_id": "cpu-anomaly",
    "indices": ["metrics-*"],
    "query": {"match_all": {}}
  }'

# Start datafeed
curl -X POST "localhost:9200/_ml/datafeeds/cpu-anomaly-feed/_start"
```

## Forecasting Examples
```bash
# Create forecast
curl -X POST "localhost:9200/_ml/anomaly_detectors/traffic-forecast/_forecast" \
  -H 'Content-Type: application/json' \
  -d '{
    "duration": "7d",
    "expires_in": "30d"
  }'

# Get forecast results
curl -X GET "localhost:9200/_ml/results/forecasts/traffic-forecast-1234567890"
```

## Log Categorization Job
```json
{
  "job_id": "error-categorization",
  "analysis_config": {
    "categorization_field_name": "message",
    "categorization_analyzer": {
      "tokenizer": "standard",
      "filter": ["lowercase", "stop"]
    }
  }
}
```

## Configuration Checklist
- [ ] ML node configured with sufficient resources
- [ ] ML license enabled
- [ ] Anomaly detection jobs created
- [ ] Forecasting models trained
- [ ] Datafeeds configured and running
- [ ] Results retention policies set
- [ ] Kibana ML UI accessible
- [ ] Alerts configured for anomalies

## Resource Requirements
| Component | CPU | Memory | Storage |
|-----------|-----|--------|----------|
| ML Node | 8+ cores | 12-16GB | 100GB |
| Per Job | 1-2 cores | 512MB-2GB | Variable |
| Forecasts | 2-4 cores | 1GB | 10GB |

## Performance Tuning
```yaml
# ML memory management
xpack.ml.memory.auto_cap: true
xpack.ml.max_open_jobs: 20
xpack.ml.per_partition_categorization.enabled: true

# Bucket span (data aggregation)
bucket_span: 15m   # for real-time detection
bucket_span: 1h    # for trend analysis
bucket_span: 1d    # for long-term patterns

# Model memory limits
model_memory_limit: 512mb  # anomaly detection
model_memory_limit: 1gb    # forecasting
model_memory_limit: 2gb    # complex patterns
```

## Monitoring ML Jobs
```bash
# List ML jobs
curl -X GET "localhost:9200/_ml/anomaly_detectors/_stats"

# Check job status
curl -X GET "localhost:9200/_ml/anomaly_detectors/cpu-anomaly/_stats"

# Monitor datafeeds
curl -X GET "localhost:9200/_ml/datafeeds/_stats"

# Get anomaly results
curl -X GET "localhost:9200/_ml/results/records?job_id=cpu-anomaly&sort=-anomaly_score"
```

## Kibana ML Integration
- 📊 **Anomaly Explorer** - Visualize anomalies
- 📈 **Single Metric Viewer** - Monitor trends
- 🔔 **Anomaly Alerts** - Create alerting rules
- 📉 **Forecasting** - View predictions
- 🏷️ **Log Categorization** - Analyze message patterns

## Common Detectors
```json
{
  "detectors": [
    {"function": "high_mean", "field_name": "response_time"},
    {"function": "high_count", "by_field_name": "error_type"},
    {"function": "rare", "field_name": "user_id", "by_field_name": "action"},
    {"function": "distinct_count", "field_name": "ip_address", "by_field_name": "host"}
  ]
}
```

## Troubleshooting
- **Job won't start:** Check memory limits and data availability
- **Slow processing:** Increase bucket_span or reduce data volume
- **High memory usage:** Reduce model_memory_limit or job count
- **No anomalies detected:** Adjust detector functions and thresholds

## Important Links
- [ML Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/ml-overview.html)
- [Anomaly Detection](https://www.elastic.co/guide/en/elasticsearch/reference/current/ml-ad-overview.html)
- [Forecasting](https://www.elastic.co/guide/en/elasticsearch/reference/current/ml-forecasting.html)
- [Log Categorization](https://www.elastic.co/guide/en/elasticsearch/reference/current/ml-log-categorization.html)