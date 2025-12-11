# Serverless 3-Tier Application

A fully serverless AWS Cloud Development Kit (CDK v2) project that deploys a three-tier Task Manager web application using AWS managed services.

## Architecture

![Serverless 3-Tier Architecture](https://gitlab.aws.dev/rwarokar/serverless-3-tier-application/-/raw/main/AWS_Serverless_Three-tier_architecture.png)

### Presentation Tier
- **S3**: Static website hosting
- **CloudFront**: Global CDN with HTTPS
- **Cognito**: User authentication

### Logic Tier (VPC Isolated)
- **API Gateway**: RESTful API endpoints
- **Lambda**: Serverless compute in private subnets (AZ-A, AZ-B)
- **VPC**: Network isolation with private subnets across 2 AZs
- **NAT Gateway**: Outbound internet access for Lambda
- **VPC Gateway Endpoint**: Private connectivity to DynamoDB
- **X-Ray**: Distributed tracing

### Data Tier
- **DynamoDB**: NoSQL database with auto-scaling (Multi-AZ)
- **AWS Backup**: Daily backups with 30-day retention
- **Point-in-time recovery**: 35-day continuous backups

## Key Features

✅ **Zero server management** - No EC2 instances  
✅ **Auto-scaling** - Scales automatically with demand  
✅ **Pay-per-use** - Only pay for what you use  
✅ **HTTPS by default** - CloudFront SSL/TLS  
✅ **Cognito authentication** - Enforced on all API endpoints  
✅ **Network isolation** - Lambda in private VPC subnets  
✅ **Multi-AZ deployment** - High availability across availability zones  
✅ **VPC Gateway endpoint** - Private connectivity to DynamoDB  
✅ **Structured logging** - JSON logs searchable in CloudWatch  
✅ **Distributed tracing** - X-Ray for performance analysis  
✅ **Automated backups** - DynamoDB daily backups  
✅ **Unit tests** - Comprehensive test coverage  

## Prerequisites

- Node.js 16+ and npm
- AWS CDK v2 CLI: `npm i -g aws-cdk`
- AWS credentials configured (`aws configure`)

## Quick Start

```bash
# Clone repository
git clone <repository-url>
cd serverless-3-tier-application

# Install dependencies
npm install
cd lambda && npm install && cd ..

# Bootstrap CDK (first time only)
cdk bootstrap aws://ACCOUNT-ID/us-east-1

# Deploy
cdk deploy ServerlessThreeTierStack
```

## Access Application

After deployment, get the CloudFront URL from outputs:

```
ServerlessThreeTierStack.ApplicationUrl = https://xxx.cloudfront.net
```

1. Visit the URL
2. **Sign up** with email and password
3. Check email for verification code
4. **Verify** account
5. **Login** and start managing tasks

## Testing

```bash
# Run unit tests
cd lambda
npm test

# Test coverage
npm run test:coverage
```

## API Endpoints

All endpoints require Cognito authentication (JWT token in `Authorization` header):

- `GET /health` - Health check (no auth required)
- `GET /tasks` - List all tasks
- `POST /tasks` - Create new task
- `GET /tasks/{id}` - Get task by ID
- `PUT /tasks/{id}` - Update task
- `DELETE /tasks/{id}` - Delete task

## Monitoring

### CloudWatch Logs
- Lambda execution logs with structured JSON
- API Gateway access logs
- Searchable via CloudWatch Logs Insights

### X-Ray Tracing
- Request flow: CloudFront → API Gateway → Lambda → DynamoDB
- Performance bottleneck identification

### Example Log Query
```
fields @timestamp, level, event, taskId
| filter level = "ERROR"
| sort @timestamp desc
```

## Backup & Recovery

### DynamoDB Backups
- **Point-in-Time Recovery**: 35 days continuous backups
- **AWS Backup**: Daily at 2:00 AM UTC, 30-day retention

### Restore from Backup
```bash
# List backups
aws backup list-recovery-points-by-backup-vault \
  --backup-vault-name ServerlessThreeTierStack-tasks-vault

# Restore to point in time
aws dynamodb restore-table-to-point-in-time \
  --source-table-name ServerlessThreeTierStack-TasksTable \
  --target-table-name tasks-restored \
  --restore-date-time 2024-01-15T10:00:00Z
```

## Common Commands

```bash
# List stacks
cdk ls

# Build project
npm run build

# Deploy
cdk deploy ServerlessThreeTierStack

# View differences
cdk diff ServerlessThreeTierStack

# Destroy
cdk destroy ServerlessThreeTierStack
```

## Project Structure

```
serverless-3-tier-application/
├── bin/
│   └── ttp_rwarokar.ts          # CDK app entry point
├── lib/
│   └── serverless-stack.ts      # Main stack definition
├── lambda/
│   ├── index.js                 # Lambda function handler
│   ├── index.test.js            # Unit tests
│   └── package.json             # Lambda dependencies
├── assets/
│   └── web/
│       ├── index.html           # Task Manager UI
│       ├── auth.html            # Login/Signup page
│       └── config.js            # Auto-generated config
├── README.md                    # This file
└── TEST-INSTRUCTIONS.md         # Testing guide
```

## Clean Up

```bash
cdk destroy ServerlessThreeTierStack
```

This will delete all resources including:
- CloudFront distribution
- S3 bucket
- API Gateway
- Lambda function
- DynamoDB table
- Cognito User Pool
- Backup vault

