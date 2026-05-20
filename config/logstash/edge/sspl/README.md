# Logstash Edge - SSPL Configuration

This directory contains Logstash configuration for the **SSPL** edge location.

## Purpose
SSPL services log processing before forwarding to Central Logstash.

## Server Details
- **Location:** SSPL Services
- **Role:** Service-specific Log Processing
- **Input:** SSPL services and applications
- **Output:** Logstash Central (localhost:5000)

## Data Flow
```
SSPL Services & Applications
    ↓
Logstash Edge - SSPL
    ├─ Parse service logs
    ├─ Service enrichment
    └─ Forward to Central
    ↓
Logstash Central (5000)
```

## Configuration Checklist
- [ ] Service data sources connected
- [ ] Service-specific pipelines configured
- [ ] Error handling optimized
- [ ] Metrics collection enabled
- [ ] Monitoring configured

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)