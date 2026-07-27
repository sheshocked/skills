---
name: serverless-deployment
description: 
category: devops
tags: [serverless-deployment]
---

## When to Use
Deploy serverless applications on AWS Lambda, Cloudflare Workers, Vercel, or Google Cloud Functions. Covers function deployment, API Gateway configuration, cold starts, VPC access, and cost optimization.

## Core Concepts
- **Function-as-a-Service**: Execute code without managing servers
- **Cold starts**: Latency when a new container is provisioned
- **API Gateway**: HTTP routing, auth, and throttling in front of functions
- **Lambda layers**: Shared dependencies and runtimes
- **SnapStart**: JVM cold start optimization (AWS Lambda)
- **Edge functions**: Run at CDN nodes (Cloudflare Workers, Vercel Edge)

## Workflow
1. Package function with dependencies (bundled or layer)
2. Configure API Gateway or function URL
3. Set environment variables and secrets
4. Configure VPC access if needed (for RDS, ElastiCache)
5. Set up monitoring with CloudWatch/Dashbird
6. Optimize cold starts and costs

## Key Patterns
```yaml
# SAM template — API Gateway + Lambda
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Runtime: python3.12
    MemorySize: 256
    Timeout: 30
    Environment:
      Variables:
        TABLE_NAME: !Ref DynamoTable
        STAGE: production

Resources:
  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.handler
      CodeUri: src/
      Architectures: [arm64]
      SnapStart:
        ApplyOn: PublishedVersions
      Layers:
        - !Ref DependenciesLayer
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref DynamoTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /{proxy+}
            Method: ANY

  DependenciesLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: python-deps
      ContentUri: layers/python/
      CompatibleRuntimes: [python3.12]
      CompatibleArchitectures: [arm64]

  DynamoTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: my-table
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: pk
          AttributeType: S
      KeySchema:
        - AttributeName: pk
          KeyType: HASH

Outputs:
  ApiUrl:
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/"
```

```python
# Lambda handler with connection reuse (critical for performance)
import os
import json
import boto3
from functools import lru_cache

# Reuse connection across invocations
@lru_cache(maxsize=1)
def get_dynamodb():
    return boto3.resource('dynamodb')

TABLE = get_dynamodb().Table(os.environ['TABLE_NAME'])

def handler(event, context):
    # Main Lambda handler
    try:
        path = event.get('path', '/')

        if event['httpMethod'] == 'GET' and path == '/items':
            return list_items()
        elif event['httpMethod'] == 'POST' and path == '/items':
            body = json.loads(event['body'])
            return create_item(body)
        else:
            return response(404, {'error': 'Not found'})

    except Exception as e:
        print(json.dumps({
            'error': str(e),
            'trace_id': context.aws_request_id
        }))
        return response(500, {'error': 'Internal server error'})

def list_items():
    result = TABLE.scan()
    return response(200, {'items': result['Items']})

def create_item(data):
    TABLE.put_item(Item={'pk': data['id'], **data})
    return response(201, {'id': data['id']})

def response(status_code, body):
    return {
        'statusCode': status_code,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
        },
        'body': json.dumps(body, default=str),
    }
```

```yaml
# Cloudflare Worker
name: api-worker
main: src/index.ts
compatibility_date: 2024-01-01

[env.production]
vars = { ENVIRONMENT = "production" }
kv_namespaces = [
  { binding = "CACHE", id = "abc123" }
]

[[routes]]
pattern = "api.example.com/*"
zone_name = "example.com"
```

## Pitfalls
- **Cold start latency**: Use provisioned concurrency or SnapStart for latency-sensitive paths
- **VPC NAT costs**: Lambda in VPC needs NAT Gateway for internet — adds $32/mo minimum
- **Memory allocation**: More memory = more CPU; profile and right-size
- **Connection limits**: Don't create DB connections per invocation — use RDS Proxy
- **Timeout**: Default 3s, max 15m (Lambda); set appropriate for workload
- **Deployment size**: Keep packages small; use layers for shared deps

## Verification
```bash
# Test Lambda locally
sam local invoke ApiFunction -e events/api-get.json

# Deploy and test
sam deploy --guided
curl https://API_ID.execute-api.region.amazonaws.com/Prod/items

# Check logs
sam logs -n ApiFunction --tail

# Check cold start duration
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=ApiFunction \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 --statistics Average
```