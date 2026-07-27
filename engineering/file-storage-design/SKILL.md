---
name: file-storage-design
description: - Application needs to store user uploads (images, documents, videos)
category: engineering
tags: [file-storage-design]
---

## When to Use

- Application needs to store user uploads (images, documents, videos)
- Generated files (reports, exports, backups) need durable storage
- Static assets need CDN delivery
- Large binary objects that don't belong in a database

## Core Concepts

- **Object Storage vs Block Storage**: Object (S3, GCS) for unstructured files, block (EBS) for databases. Object is cheaper, more durable, infinitely scalable.
- **Presigned URLs**: Time-limited URLs that grant direct upload/download access without routing through your server. Reduces server load.
- **Content-Addressable Storage**: File name = hash of content. Same content = same path. Built-in deduplication.
- **Multipart Upload**: For large files (>100MB), upload in chunks. Resumable, parallelized.
- **Lifecycle Policies**: Auto-delete old versions, transition to cheaper storage tiers.

## Workflow

1. **Choose storage backend** — S3 (AWS), GCS (GCP), Azure Blob, MinIO (self-hosted)
2. **Design access patterns** — public read, private with presigned URL, server-side proxy
3. **Implement upload flow** — direct to S3 via presigned URL (preferred) vs through your server
4. **Add processing pipeline** — image resize, virus scan, metadata extraction
5. **Set lifecycle policies** — auto-delete temp files, archive old versions
6. **Monitor** — storage size, bandwidth costs, error rates

## Key Patterns

```python
# Presigned URL generation for direct S3 upload
import boto3
from datetime import timedelta

s3_client = boto3.client("s3")

def create_presigned_upload_url(
    bucket: str,
    key: str,
    content_type: str,
    max_size_mb: int = 10,
    expires_in: int = 3600,
) -> dict: