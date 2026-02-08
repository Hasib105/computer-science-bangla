# Logging

## 🎯 Logging Best Practices

```
Log Levels:
├── TRACE: Most detailed, rarely used in prod
├── DEBUG: Development debugging
├── INFO: Normal operations
├── WARN: Something unexpected but handled
├── ERROR: Something failed
└── FATAL: System is crashing
```

## 📊 Structured Logging

```json
// Bad (unstructured)
"User 123 placed order 456 for $100"

// Good (structured JSON)
{
  "timestamp": "2024-01-26T10:30:45.123Z",
  "level": "INFO",
  "service": "order-service",
  "trace_id": "abc-123",
  "message": "Order placed",
  "user_id": "123",
  "order_id": "456",
  "amount": 100.00,
  "currency": "USD"
}
```

## 🏗️ ELK Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    ELK Stack                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Services → Filebeat → Logstash → Elasticsearch → Kibana   │
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐          │
│  │ Service  │───→│ Filebeat │───→│   Logstash   │          │
│  │   Logs   │    │  (Ship)  │    │  (Transform) │          │
│  └──────────┘    └──────────┘    └──────┬───────┘          │
│                                         │                   │
│                                         ↓                   │
│  ┌──────────────────────────────────────────────────┐      │
│  │              Elasticsearch                        │      │
│  │           (Store & Search)                        │      │
│  └───────────────────────┬──────────────────────────┘      │
│                          │                                  │
│                          ↓                                  │
│  ┌──────────────────────────────────────────────────┐      │
│  │                  Kibana                           │      │
│  │              (Visualize)                          │      │
│  └──────────────────────────────────────────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 🔧 Log Aggregation Pattern

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "service": "my-service",
            "message": record.getMessage(),
            "logger": record.name,
        }
        
        # Add extra fields
        if hasattr(record, 'trace_id'):
            log_entry['trace_id'] = record.trace_id
        if hasattr(record, 'user_id'):
            log_entry['user_id'] = record.user_id
            
        return json.dumps(log_entry)

# Usage
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

logger.info("Order processed", extra={
    'trace_id': 'abc-123',
    'user_id': '456',
    'order_id': '789'
})
```

## 💡 What to Log

```
✓ Log:
  - Request/Response (summary)
  - Errors with context
  - Business events
  - Performance metrics
  - Security events

✗ Don't Log:
  - Passwords, tokens
  - PII without masking
  - High-volume debug in prod
  - Sensitive data
```

## 📚 পরবর্তী টপিক

[Metrics →](./03-metrics.md)
