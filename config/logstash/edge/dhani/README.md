# Logstash Edge - Dhani Configuration

This directory contains Logstash configuration for the **Dhani** edge location.

## Purpose
Dhani region distributed log processing before forwarding to Central Logstash.

## Server Details
- **Location:** Dhani Region
- **Role:** Distributed Log Processing
- **Input:** Dhani region data sources
- **Output:** Logstash Central (localhost:5000)

## Data Flow
```
Dhani Data Sources
    ↓
Logstash Edge - Dhani
    ├─ Parse distributed logs
    ├─ Regional enrichment
    └─ Forward to Central
    ↓
Logstash Central (5000)
```

## Configuration Checklist
- [ ] Regional data sources connected
- [ ] Distributed processing optimized
- [ ] Pipelines configured for throughput
- [ ] Failover mechanisms in place
- [ ] Monitoring enabled

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)