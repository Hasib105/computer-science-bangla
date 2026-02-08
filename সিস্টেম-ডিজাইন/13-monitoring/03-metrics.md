# Metrics

## 🎯 Metrics Types

```
Counter: Only increases
  - http_requests_total
  - errors_total
  - orders_processed_total

Gauge: Can increase/decrease
  - active_connections
  - cpu_usage
  - memory_bytes

Histogram: Distribution in buckets
  - request_duration_seconds_bucket{le="0.1"}
  - request_duration_seconds_bucket{le="0.5"}
  - request_duration_seconds_bucket{le="1.0"}

Summary: Percentiles
  - request_duration_seconds{quantile="0.5"}
  - request_duration_seconds{quantile="0.99"}
```

## 📊 Prometheus + Grafana

```
┌─────────────────────────────────────────────────────────────┐
│               Prometheus Architecture                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│     ┌─────────────────────────────────────┐                │
│     │         Prometheus Server           │                │
│     │    ┌─────────────────────────┐     │                │
│     │    │    Time Series DB       │     │                │
│     │    └─────────────────────────┘     │                │
│     └─────────────────┬───────────────────┘                │
│                       │                                     │
│          ┌────────────┼────────────┐                       │
│          │            │            │                       │
│          │ Pull       │ Pull       │ Pull                  │
│          ↓            ↓            ↓                       │
│    ┌──────────┐ ┌──────────┐ ┌──────────┐                 │
│    │Service A │ │Service B │ │Service C │                 │
│    │ /metrics │ │ /metrics │ │ /metrics │                 │
│    └──────────┘ └──────────┘ └──────────┘                 │
│                                                             │
│     Prometheus pulls metrics from /metrics endpoints       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 🔧 Metrics Implementation

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Define metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint'],
    buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 5.0]
)

ACTIVE_USERS = Gauge(
    'active_users',
    'Number of active users'
)

# Usage
def handle_request():
    start_time = time.time()
    
    # Process request...
    status = 200
    
    # Record metrics
    REQUEST_COUNT.labels(
        method='GET',
        endpoint='/api/orders',
        status=status
    ).inc()
    
    REQUEST_LATENCY.labels(
        method='GET',
        endpoint='/api/orders'
    ).observe(time.time() - start_time)

# Start metrics server
start_http_server(8000)  # /metrics endpoint
```

## 📈 RED Method

```
Rate:    Requests per second
Errors:  Error rate
Duration: Latency

For request-driven services (APIs):

rate(http_requests_total[5m])
rate(http_requests_total{status=~"5.."}[5m])
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

## 📊 USE Method

```
Utilization: % time busy
Saturation:  Queue depth
Errors:      Error count

For resources (CPU, Disk, Network):

CPU Utilization: avg(rate(node_cpu_seconds_total{mode!="idle"}[5m]))
Disk Saturation: node_disk_io_time_seconds_total
Network Errors:  node_network_receive_errs_total
```

## 📚 পরবর্তী টপিক

[Distributed Tracing →](./04-distributed-tracing.md)
