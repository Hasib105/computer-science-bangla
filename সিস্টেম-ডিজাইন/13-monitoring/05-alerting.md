# Alerting

## 🎯 Alerting Best Practices

```
Good Alerts:
✓ Actionable (someone needs to do something)
✓ Symptoms-based (not causes)
✓ Low noise (no alert fatigue)
✓ Documented runbook

Bad Alerts:
✗ CPU > 80% (so what?)
✗ Too many alerts (ignored)
✗ No context (what to do?)
```

## 📊 SLI/SLO Based Alerting

```
SLI (Service Level Indicator):
- Measurable metric
- Example: Request success rate

SLO (Service Level Objective):
- Target for SLI
- Example: 99.9% success rate

SLA (Service Level Agreement):
- Contract with consequences
- Example: 99.9% uptime or refund

Error Budget:
- Allowed failures
- 99.9% = 0.1% error budget = ~43 min/month downtime
```

## 🔔 Alert Severity

```
┌─────────────────────────────────────────────────────────────┐
│               Alert Severity Levels                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  P1 (Critical):                                            │
│  - Service is DOWN                                         │
│  - Users cannot complete core actions                      │
│  - Revenue impact                                          │
│  - Wake up on-call immediately                             │
│                                                             │
│  P2 (High):                                                │
│  - Service degraded                                        │
│  - Some users affected                                     │
│  - Page during business hours                              │
│                                                             │
│  P3 (Medium):                                              │
│  - Performance issues                                      │
│  - Workaround available                                    │
│  - Ticket, fix next business day                           │
│                                                             │
│  P4 (Low):                                                 │
│  - Minor issues                                            │
│  - No user impact                                          │
│  - Track and fix when convenient                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 🏗️ Alerting Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 Alerting Pipeline                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐                                          │
│  │  Prometheus  │                                          │
│  │   (Metrics)  │                                          │
│  └──────┬───────┘                                          │
│         │ Alert rules                                       │
│         ↓                                                   │
│  ┌──────────────┐                                          │
│  │ Alertmanager │                                          │
│  │              │                                          │
│  │ - Dedup      │                                          │
│  │ - Group      │                                          │
│  │ - Route      │                                          │
│  │ - Silence    │                                          │
│  └──────┬───────┘                                          │
│         │                                                   │
│    ┌────┴────┬────────┬────────┐                          │
│    ↓         ↓        ↓        ↓                          │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                      │
│ │Slack │ │Email │ │Pager │ │Ticket│                      │
│ │      │ │      │ │Duty  │ │System│                      │
│ └──────┘ └──────┘ └──────┘ └──────┘                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 🔧 Prometheus Alert Rules

```yaml
groups:
  - name: service_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m])) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"
          runbook: "https://wiki/runbooks/high-error-rate"
      
      # Slow responses
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99,
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1.0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "P99 latency is high"
          description: "P99 latency is {{ $value }}s"
```

## 💡 On-Call Best Practices

```
✓ Clear escalation path
✓ Runbooks for each alert
✓ Blameless postmortems
✓ Rotate on-call fairly
✓ Compensate on-call time
✓ Limit alerts per shift
✓ Review and tune alerts regularly
```

---

🎉 মনিটরিং সেকশন সম্পূর্ণ!

[স্টোরেজ সিস্টেম →](../14-storage/README.md)
