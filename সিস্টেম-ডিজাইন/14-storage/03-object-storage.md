# Object Storage (S3)

## 🎯 S3 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    S3 Concepts                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Bucket: Container for objects                                 │
│  ┌─────────────────────────────────────────────────┐           │
│  │  Bucket: my-company-images                       │           │
│  │                                                  │           │
│  │  Objects:                                        │           │
│  │  ├── photos/vacation/beach.jpg                  │           │
│  │  ├── photos/vacation/mountain.jpg               │           │
│  │  ├── documents/report.pdf                       │           │
│  │  └── backups/db-2024-01-26.sql                 │           │
│  │                                                  │           │
│  │  (Prefix "photos/" acts like folder)            │           │
│  └─────────────────────────────────────────────────┘           │
│                                                                 │
│  Object = Key + Data + Metadata                                │
│                                                                 │
│  Key: photos/vacation/beach.jpg                                │
│  Data: [binary image content]                                  │
│  Metadata:                                                      │
│    Content-Type: image/jpeg                                    │
│    x-amz-meta-photographer: John                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 📊 S3 Storage Classes

```
┌─────────────────────────────────────────────────────────────────┐
│                 S3 Storage Classes                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Frequent Access:                                              │
│  ├── S3 Standard        (most common, highest cost)           │
│  └── S3 Intelligent     (auto-tiering)                        │
│                                                                 │
│  Infrequent Access:                                            │
│  ├── S3 Standard-IA     (cheaper, retrieval fee)              │
│  └── S3 One Zone-IA     (single AZ, even cheaper)             │
│                                                                 │
│  Archive:                                                       │
│  ├── S3 Glacier Instant  (milliseconds retrieval)             │
│  ├── S3 Glacier Flexible (minutes to hours)                   │
│  └── S3 Glacier Deep     (12-48 hours, cheapest)              │
│                                                                 │
│  Cost (approx):                                                │
│  Standard:     $0.023/GB                                       │
│  Glacier Deep: $0.00099/GB                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 🔧 S3 API Examples

```python
import boto3

s3 = boto3.client('s3')

# Upload
s3.upload_file(
    'local_file.jpg',
    'my-bucket',
    'photos/image.jpg',
    ExtraArgs={'ContentType': 'image/jpeg'}
)

# Download
s3.download_file('my-bucket', 'photos/image.jpg', 'local.jpg')

# Generate presigned URL (temporary access)
url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'photos/image.jpg'},
    ExpiresIn=3600  # 1 hour
)

# List objects
response = s3.list_objects_v2(
    Bucket='my-bucket',
    Prefix='photos/'
)
for obj in response['Contents']:
    print(obj['Key'], obj['Size'])
```

## 💡 S3 Best Practices

```
Naming:
✓ Unique bucket names globally
✓ Use prefixes for organization
✓ Avoid sequential keys (use random prefix for high throughput)

Security:
✓ Block public access by default
✓ Use IAM policies
✓ Enable versioning
✓ Encrypt at rest (SSE-S3, SSE-KMS)

Performance:
✓ Use multipart upload for large files (>100MB)
✓ Use Transfer Acceleration for global uploads
✓ CloudFront for reads
```

## 📚 পরবর্তী টপিক

[Data Replication Strategies →](./04-replication-strategies.md)
