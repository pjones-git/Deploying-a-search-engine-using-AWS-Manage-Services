# Troubleshooting Guide

This guide documents common issues encountered during the project and their solutions.

## Table of Contents
1. [Lambda-OpenSearch Communication Issues](#lambda-opensearch-communication-issues)
2. [CI/CD Pipeline Failures](#cicd-pipeline-failures)
3. [OpenSearch Indexing Problems](#opensearch-indexing-problems)
4. [PDF Processing Issues](#pdf-processing-issues)
5. [VPC and Networking](#vpc-and-networking)
6. [IAM and Permissions](#iam-and-permissions)

---

## Lambda-OpenSearch Communication Issues

### Problem: Lambda Function Times Out When Accessing OpenSearch

**Symptoms:**
- Lambda execution times out after 3+ seconds
- CloudWatch logs show no connection to OpenSearch
- Error: "Task timed out after X seconds"

**Root Causes:**
1. Lambda not in the same VPC as OpenSearch
2. Security group rules blocking traffic
3. Subnet has no route to OpenSearch

**Solutions:**

**Step 1: Verify VPC Configuration**
```bash
# Check Lambda VPC configuration
aws lambda get-function-configuration --function-name lextech-pdf-to-txt \
  --query 'VpcConfig'

# Check OpenSearch VPC
aws opensearch describe-domain --domain-name lextech-search \
  --query 'DomainStatus.VPCOptions'
```

**Step 2: Update Security Groups**
```bash
# Add inbound rule to OpenSearch security group
aws ec2 authorize-security-group-ingress \
  --group-id <opensearch-sg-id> \
  --protocol tcp \
  --port 443 \
  --source-group <lambda-sg-id>
```

**Step 3: Attach Lambda to VPC**
```bash
aws lambda update-function-configuration \
  --function-name lextech-upload-to-search \
  --vpc-config SubnetIds=<subnet1>,<subnet2>,SecurityGroupIds=<sg-id>
```

**Step 4: Verify NAT Gateway (if accessing internet)**
- Ensure private subnets have route to NAT Gateway
- Check route table: `0.0.0.0/0 → nat-xxxxxx`

---

## CI/CD Pipeline Failures

### Problem: PLACEHOLDER_TOKEN Error

**Symptoms:**
- CodePipeline fails at Source stage
- Error: "Could not access GitHub repository"
- Logs show "PLACEHOLDER_TOKEN" instead of actual token

**Solution:**

**Step 1: Create GitHub Personal Access Token**
1. Go to GitHub → Settings → Developer settings → Personal access tokens
2. Click "Generate new token (classic)"
3. Select scopes: `repo` (all), `admin:repo_hook`
4. Copy token immediately (won't be shown again)

**Step 2: Update CodePipeline Source Configuration**
```bash
# Update pipeline with new token
aws codepipeline update-pipeline --cli-input-json file://pipeline-config.json
```

**Step 3: Store Token in Secrets Manager (Recommended)**
```bash
aws secretsmanager create-secret \
  --name github-token \
  --secret-string "ghp_xxxxxxxxxxxx"

# Update pipeline to use Secrets Manager
```

---

### Problem: CloudFormation Stack Update Fails

**Symptoms:**
- CodeBuild succeeds but CloudFormation fails
- Error: "Insufficient permissions" or "Stack rollback"

**Solutions:**

**Check IAM Permissions:**
```bash
# Verify CodeBuild service role has CloudFormation permissions
aws iam get-role-policy --role-name CodeBuildServiceRole \
  --policy-name CloudFormationAccess
```

**Required Permissions:**
```json
{
  "Effect": "Allow",
  "Action": [
    "cloudformation:CreateStack",
    "cloudformation:UpdateStack",
    "cloudformation:DescribeStacks",
    "lambda:UpdateFunctionCode",
    "lambda:UpdateFunctionConfiguration",
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "*"
}
```

**Check Stack Events:**
```bash
aws cloudformation describe-stack-events \
  --stack-name PDFtoTxt-pipeline-stack \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`]'
```

---

## OpenSearch Indexing Problems

### Problem: Lambda Can't Index Documents in OpenSearch

**Symptoms:**
- Lambda executes without errors
- Documents don't appear in OpenSearch
- Error: "AuthorizationException" or "403 Forbidden"

**Root Cause:** OpenSearch fine-grained access control blocking Lambda role

**Solution:**

**Step 1: Access OpenSearch Dashboard via EC2 Jump Host**
1. RDP into EC2 instance (restricted to your IP)
2. Open browser to OpenSearch dashboard URL
3. Login with master credentials

**Step 2: Map IAM Role in OpenSearch**
1. Navigate to: Security → Roles
2. Select role: `all_access` or create custom role
3. Click "Mapped users"
4. Add Lambda execution role ARN:
   ```
   arn:aws:iam::219342442719:role/lambda-opensearch-role
   ```

**Step 3: Verify Index Permissions**
```python
# Test from Lambda or locally
import boto3
from opensearchpy import OpenSearch, RequestsHttpConnection
from requests_aws4auth import AWS4Auth

region = 'us-east-1'
service = 'es'
credentials = boto3.Session().get_credentials()
awsauth = AWS4Auth(credentials.access_key, credentials.secret_key, 
                   region, service, session_token=credentials.token)

client = OpenSearch(
    hosts=[{'host': 'vpc-lextech-search-....aos.us-east-1.on.aws', 'port': 443}],
    http_auth=awsauth,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection
)

# Test connection
print(client.info())
```

**Step 4: Create Index if Missing**
```bash
# Via OpenSearch dashboard or API
PUT /lextech-documents
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text" },
      "filename": { "type": "keyword" },
      "upload_date": { "type": "date" }
    }
  }
}
```

---

## PDF Processing Issues

### Problem: PDFs Not Being Processed

**Symptoms:**
- PDF uploaded to S3 but no text file created
- PDFtoTxt Lambda not triggered
- No CloudWatch logs for function

**Solutions:**

**Step 1: Verify S3 Event Notification**
```bash
aws s3api get-bucket-notification-configuration \
  --bucket lextech-content-store-219342442719
```

**Expected Output:**
```json
{
  "LambdaFunctionConfigurations": [
    {
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:219342442719:function:lextech-pdf-to-txt",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [
            {"Name": "suffix", "Value": ".pdf"}
          ]
        }
      }
    }
  ]
}
```

**Step 2: Add Event Notification if Missing**
```bash
aws s3api put-bucket-notification-configuration \
  --bucket lextech-content-store-219342442719 \
  --notification-configuration file://s3-notification.json
```

**Step 3: Grant S3 Permission to Invoke Lambda**
```bash
aws lambda add-permission \
  --function-name lextech-pdf-to-txt \
  --statement-id s3-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::lextech-content-store-219342442719
```

**Step 4: Test Lambda Function Manually**
```bash
# Create test event
aws lambda invoke \
  --function-name lextech-pdf-to-txt \
  --payload file://test-event.json \
  response.json
```

---

### Problem: PDF Text Extraction Fails

**Symptoms:**
- Lambda executes but produces empty text files
- Error: "Unable to extract text from PDF"

**Solutions:**

**Check PDF Format:**
- Ensure PDF contains text (not scanned images)
- Test with known good PDF first

**Verify PyPDF2/pdfplumber Library:**
```python
# In Lambda function
import PyPDF2  # or pdfplumber
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

try:
    # Download from S3
    s3_client.download_file(bucket, key, '/tmp/input.pdf')
    
    # Extract text
    with open('/tmp/input.pdf', 'rb') as file:
        reader = PyPDF2.PdfReader(file)
        text = ""
        for page in reader.pages:
            text += page.extract_text()
    
    logger.info(f"Extracted {len(text)} characters")
    
except Exception as e:
    logger.error(f"Error: {str(e)}")
    raise
```

---

## VPC and Networking

### Problem: Lambda Can't Access Internet (pip install fails)

**Symptoms:**
- Lambda deployment fails
- Error: "Unable to download package"
- pip install timeouts

**Solution:**

**Option 1: Use Lambda Layers**
```bash
# Package dependencies locally
pip install -r requirements.txt -t python/
zip -r layer.zip python/

# Upload as Lambda layer
aws lambda publish-layer-version \
  --layer-name opensearch-layer \
  --zip-file fileb://layer.zip \
  --compatible-runtimes python3.9
```

**Option 2: Add NAT Gateway**
1. Create NAT Gateway in public subnet
2. Update private subnet route table:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway ID

**Option 3: Package Dependencies in Deployment**
```yaml
# In buildspec.yml
phases:
  build:
    commands:
      - pip install -r requirements.txt -t .
      - zip -r function.zip .
artifacts:
  files:
    - function.zip
```

---

## IAM and Permissions

### Problem: Access Denied Errors

**Symptoms:**
- Lambda can't read from S3
- Lambda can't write to CloudWatch Logs
- Error: "User is not authorized to perform: xxx"

**Solutions:**

**Lambda Execution Role Must Include:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::lextech-content-store-219342442719/*",
        "arn:aws:s3:::lextech-intermediary-store-219342442719/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:CreateNetworkInterface",
        "ec2:DescribeNetworkInterfaces",
        "ec2:DeleteNetworkInterface"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "es:ESHttpGet",
        "es:ESHttpPost",
        "es:ESHttpPut"
      ],
      "Resource": "arn:aws:es:us-east-1:219342442719:domain/lextech-search/*"
    }
  ]
}
```

---

## Debugging Tips

### Enable Detailed CloudWatch Logging

```python
import logging
import json

logger = logging.getLogger()
logger.setLevel(logging.DEBUG)

def lambda_handler(event, context):
    logger.debug(f"Event: {json.dumps(event)}")
    
    try:
        # Your code
        result = process_data()
        logger.info(f"Success: {result}")
        return result
    except Exception as e:
        logger.error(f"Error: {str(e)}", exc_info=True)
        raise
```

### Test Components Independently

1. **Test S3 Trigger:** Upload test file
2. **Test Lambda Locally:** Use SAM CLI
3. **Test OpenSearch:** Use curl or Postman
4. **Test API Gateway:** Use AWS Console test feature

### Check CloudWatch Metrics

```bash
# Lambda invocations
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=lextech-pdf-to-txt \
  --start-time 2025-08-04T00:00:00Z \
  --end-time 2025-08-04T23:59:59Z \
  --period 3600 \
  --statistics Sum

# Lambda errors
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=lextech-pdf-to-txt \
  --start-time 2025-08-04T00:00:00Z \
  --end-time 2025-08-04T23:59:59Z \
  --period 3600 \
  --statistics Sum
```

---

## Additional Resources

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- [Amazon OpenSearch Service Guide](https://docs.aws.amazon.com/opensearch-service/)
- [AWS CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
- [VPC Troubleshooting](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-troubleshooting.html)

---

*For additional help, review CloudWatch logs or open an issue on the GitHub repository.*
