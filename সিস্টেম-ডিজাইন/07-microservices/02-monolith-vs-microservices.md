# মনোলিথ vs মাইক্রোসার্ভিস

## 🎯 মনোলিথ আর্কিটেকচার

```
Monolith = একটি বড় অ্যাপ্লিকেশন

┌──────────────────────────────────────────────────────┐
│                  Monolith Application                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────┬──────────┬──────────┬─────────────┐    │
│  │  User   │  Order   │ Product  │   Payment   │    │
│  │ Module  │  Module  │  Module  │   Module    │    │
│  └─────────┴──────────┴──────────┴─────────────┘    │
│                         │                            │
│                         ↓                            │
│              ┌──────────────────┐                   │
│              │   Single DB      │                   │
│              └──────────────────┘                   │
│                                                      │
│  • Single codebase                                  │
│  • Single deployment                                │
│  • Shared memory                                    │
│  • In-process calls                                 │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## 🔀 মাইক্রোসার্ভিস আর্কিটেকচার

```
Microservices = অনেক ছোট ছোট সার্ভিস

┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│   User   │ │  Order   │ │ Product  │ │ Payment  │
│ Service  │ │ Service  │ │ Service  │ │ Service  │
├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤
│ Node.js  │ │   Java   │ │  Python  │ │   Go     │
├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤
│ MongoDB  │ │ Postgres │ │  Redis   │ │ Postgres │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
     │             │             │           │
     └─────────────┴──────┬──────┴───────────┘
                          │
                   ┌──────┴──────┐
                   │   Network   │
                   │   (HTTP/gRPC)│
                   └─────────────┘
```

## 📊 তুলনামূলক বিশ্লেষণ

```
┌─────────────────────┬──────────────────┬──────────────────────┐
│       Aspect        │    Monolith      │    Microservices     │
├─────────────────────┼──────────────────┼──────────────────────┤
│ Deployment          │ একসাথে সব       │ আলাদা আলাদা          │
│ Scaling             │ পুরোটা scale    │ নির্দিষ্ট service     │
│ Technology          │ একই stack       │ বিভিন্ন stack        │
│ Team structure      │ একটি বড় টিম    │ ছোট ছোট টিম          │
│ Development speed   │ শুরুতে দ্রুত    │ শুরুতে ধীর           │
│ Debugging           │ সহজ             │ কঠিন                 │
│ Testing             │ সহজ             │ Integration কঠিন     │
│ Database            │ একটি shared     │ প্রতি service আলাদা  │
│ Communication       │ In-process      │ Network calls        │
│ Fault isolation     │ কম              │ বেশি                 │
└─────────────────────┴──────────────────┴──────────────────────┘
```

## ✅ কখন মনোলিথ?

```
✓ ছোট টিম (< ১০ জন)
✓ নতুন স্টার্টআপ / MVP
✓ Domain সম্পর্কে অনিশ্চিত
✓ দ্রুত market-এ যেতে হবে
✓ Simple application

"Start with monolith, 
 evolve to microservices when needed."
                    - Martin Fowler
```

## ✅ কখন মাইক্রোসার্ভিস?

```
✓ বড় টিম (২০+ developers)
✓ Independent deployment দরকার
✓ Different scaling needs
✓ Technology diversity দরকার
✓ Well-defined domain boundaries
```

## 🔄 Migration Path

```
Monolith → Microservices

Step 1: Identify boundaries
┌────────────────────────────┐
│         Monolith           │
│  ┌─────┬─────┬─────┬─────┐│
│  │User │Order│Prod │Pay  ││
│  └─────┴─────┴─────┴─────┘│
└────────────────────────────┘

Step 2: Extract one service
┌────────────────────┐  ┌──────────┐
│     Monolith       │  │  User    │
│  ┌─────┬─────┬────┐│  │ Service  │
│  │Order│Prod │Pay ││  └────┬─────┘
│  └─────┴─────┴────┘│       │
└──────────┬─────────┘       │
           └────── API ──────┘

Step 3: Continue extracting
┌──────────┐ ┌──────────┐
│  Order   │ │  User    │
│ Service  │ │ Service  │
└──────────┘ └──────────┘
┌──────────┐ ┌──────────┐
│ Product  │ │ Payment  │
│ Service  │ │ Service  │
└──────────┘ └──────────┘
```

## 📚 পরবর্তী টপিক

[সার্ভিস কমিউনিকেশন →](./03-service-communication.md)
