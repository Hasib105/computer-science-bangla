# Distributed Tracing

## 🎯 কেন Tracing দরকার?

```
Microservices-এ একটি request অনেক services-এ যায়।
কোথায় সময় বেশি লাগছে বোঝা কঠিন।

User Request ────→ API Gateway
                      │
         ┌────────────┼────────────┐
         ↓            ↓            ↓
    Service A    Service B    Service C
         │            │            │
         ↓            ↓            ↓
    Database     Cache         Service D
                                   │
                                   ↓
                              External API

কোথায় ৫০০ms লাগছে?
```

## 📊 Trace Structure

```
Trace: Complete request journey
├── Span: Single operation
│   ├── Trace ID: abc-123 (same for all spans)
│   ├── Span ID: span-1
│   ├── Parent Span ID: null (root)
│   ├── Operation: "HTTP GET /orders"
│   ├── Start Time: 10:00:00.000
│   ├── Duration: 500ms
│   └── Tags: {user_id: 123, status: 200}
│
└── Child Spans:
    ├── Span: "Order Service"
    │   ├── Span ID: span-2
    │   ├── Parent: span-1
    │   └── Duration: 200ms
    │
    └── Span: "Database Query"
        ├── Span ID: span-3
        ├── Parent: span-2
        └── Duration: 50ms
```

## 🏗️ Tracing Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 Distributed Tracing                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Request with Trace ID in headers                               │
│                                                                 │
│  ┌──────────┐  X-Trace-ID   ┌──────────┐  X-Trace-ID           │
│  │ Service  │──────────────→│ Service  │──────────────→ ...    │
│  │    A     │               │    B     │                        │
│  └────┬─────┘               └────┬─────┘                        │
│       │                          │                              │
│       │ Report spans             │ Report spans                 │
│       ↓                          ↓                              │
│  ┌────────────────────────────────────────────────────┐        │
│  │              Tracing Collector                      │        │
│  │                (Jaeger/Zipkin)                      │        │
│  └────────────────────────┬───────────────────────────┘        │
│                           │                                     │
│                           ↓                                     │
│  ┌────────────────────────────────────────────────────┐        │
│  │                   Storage                           │        │
│  │            (Elasticsearch/Cassandra)                │        │
│  └────────────────────────┬───────────────────────────┘        │
│                           │                                     │
│                           ↓                                     │
│  ┌────────────────────────────────────────────────────┐        │
│  │                    UI                               │        │
│  │              (Jaeger UI/Zipkin)                     │        │
│  └────────────────────────────────────────────────────┘        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🔧 OpenTelemetry Implementation

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter

# Setup
trace.set_tracer_provider(TracerProvider())
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

tracer = trace.get_tracer(__name__)

# Usage
def process_order(order_id):
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order_id", order_id)
        
        # Child span
        with tracer.start_as_current_span("validate_order"):
            validate(order_id)
        
        with tracer.start_as_current_span("charge_payment"):
            charge(order_id)
        
        with tracer.start_as_current_span("update_inventory"):
            update_inventory(order_id)
```

## 💡 Sampling Strategies

```
Head-based Sampling:
- Decide at request start
- 1% of all requests
- Simple but might miss errors

Tail-based Sampling:
- Decide after request complete
- Keep slow/error requests
- More resource intensive
- Better for debugging
```

## 📚 পরবর্তী টপিক

[Alerting →](./05-alerting.md)
