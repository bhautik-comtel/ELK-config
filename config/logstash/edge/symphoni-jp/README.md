# Logstash Edge - Symphoni JP Configuration

This directory contains Logstash configuration for the **Symphoni JP** (Japan) edge location.

## Purpose
Japan region log processing before forwarding to Central Logstash.

## Server Details
- **Location:** Japan Region
- **Role:** APAC Regional Log Processing
- **Input:** Japan region data sources
- **Output:** Logstash Central (localhost:5000)

## Data Flow
```
Symphoni JP Data Sources
    ↓
Logstash Edge - Symphoni JP
    ├─ Parse APAC logs
    ├─ Regional enrichment
    └─ Forward to Central
    ↓
Logstash Central (5000)
```

## Configuration Checklist
- [ ] Japan region data sources connected
- [ ] Regional timezone handling configured
- [ ] Pipelines optimized for APAC traffic
- [ ] Local backup configured
- [ ] Monitoring enabled

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)