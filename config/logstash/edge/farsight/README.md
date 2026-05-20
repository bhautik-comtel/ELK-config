# Logstash Edge - Farsight Configuration

This directory contains Logstash configuration for the **Farsight** edge location.

## Purpose
Farsight advanced filtering and processing before forwarding to Central Logstash.

## Server Details
- **Location:** Farsight
- **Role:** Advanced Filtering & Processing
- **Input:** Farsight data sources
- **Output:** Logstash Central (localhost:5000)

## Data Flow
```
Farsight Data Sources
    ↓
Logstash Edge - Farsight
    ├─ Advanced filtering
    ├─ Complex transformations
    └─ Forward to Central
    ↓
Logstash Central (5000)
```

## Configuration Checklist
- [ ] Advanced filters configured
- [ ] Transformation rules optimized
- [ ] Conditional processing setup
- [ ] Performance tuned for filters
- [ ] Monitoring enabled

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)