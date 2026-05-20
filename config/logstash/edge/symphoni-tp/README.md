# Logstash Edge - Symphoni TP Configuration

This directory contains Logstash configuration for the **Symphoni TP** edge location.

## Purpose
TP region log processing before forwarding to Central Logstash.

## Server Details
- **Location:** TP Region
- **Role:** Regional Log Processing (TP)
- **Upstream:** TP region data sources
- **Downstream:** Logstash Central (5000)

## Contents
- **logstash.yml** - Edge Logstash configuration
- **pipelines/** - Processing pipelines for TP region
- **patterns/** - Regional grok patterns

## Region Information
- **Region Code:** TP
- **Data Centers:** TP region infrastructure
- **Traffic Volume:** Regional scale

## Data Flow
```
Symphoni TP Data Sources
    ↓
Logstash Edge - Symphoni TP
    ├─ Parse regional logs
    ├─ Regional enrichment
    └─ Forward to Central
    ↓
Logstash Central (5000)
    ↓
Elasticsearch
```

## Configuration Checklist
- [ ] Regional data source connections set
- [ ] Network optimization configured
- [ ] Pipelines tuned for TP traffic
- [ ] Regional backup configured
- [ ] Monitoring enabled

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
