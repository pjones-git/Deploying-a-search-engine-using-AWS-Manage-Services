# Architecture Documentation

## System Architecture Diagram

The architecture diagram from the original project documentation (page 1 of project-documentation.pdf) shows the complete system design.

### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│                        Build Your Google                        │
│                     CI/CD Pipeline with IaC                     │
└─────────────────────────────────────────────────────────────────┘

DevOps User → GitHub → CodeBuild → CloudFormation → CodePipeline

┌─────────────────────────────────────────────────────────────────┐
│                     Document Processing Flow                     │
└─────────────────────────────────────────────────────────────────┘

User → EC2 (Web Access)
  ├─→ PDF → Document Store (S3) → Event → PDFtoTxt (Lambda)
  │                                          ↓
  │                               Intermediary Store (S3)
  │                                          ↓
  │                                  UploadToSearch (Lambda)
  │                                          ↓
  └─→ API Gateway ←─────────────→ OpenSearch Service
         │                              ↑
         ├─→ SearchGateway (Lambda)     │
         └─→ SearchFunction (Lambda) ───┘

CloudWatch (Event Monitoring)
```

## Data Flow

### 1. Document Upload Flow
1. User uploads PDF to **lextech-content-store** S3 bucket
2. S3 event notification triggers **PDFtoTxt** Lambda function
3. Lambda extracts text from PDF
4. Text file saved to **lextech-intermediary-store** S3 bucket
5. S3 event triggers **UploadToSearch** Lambda function
6. Lambda indexes text content in OpenSearch **lextech-documents** index

### 2. Search Query Flow
1. User accesses API Gateway URL
2. **SearchGateway** Lambda serves HTML interface
3. User submits search query via web form
4. API Gateway routes POST request to **SearchFunction** Lambda
5. Lambda queries OpenSearch cluster
6. Results returned to user interface

### 3. CI/CD Deployment Flow
1. Developer pushes code to GitHub repository
2. CodePipeline detects source code changes
3. CodePipeline triggers CodeBuild project
4. CodeBuild executes buildspec.yml:
   - Packages Lambda function code
   - Uploads artifacts to S3
   - Runs CloudFormation stack update
5. CloudFormation deploys/updates Lambda functions
6. CloudWatch logs all pipeline activity

## Network Architecture

### VPC Configuration
- **OpenSearch Domain:** Deployed in private VPC subnets
- **Lambda Functions:** VPC-enabled with ENIs in private subnets
- **EC2 Jump Host:** Windows instance in same VPC for dashboard access
- **Security Groups:** 
  - Lambda SG allows outbound to OpenSearch
  - OpenSearch SG allows inbound from Lambda SG
  - EC2 SG allows RDP (3389) from specific IP only

### Availability
- **OpenSearch:** 3-AZ deployment with standby
- **Lambda:** Multi-AZ by default
- **S3:** Multi-AZ replication by default

## Resource Naming Convention

All resources follow the naming pattern: `lextech-{service}-{account-id}`

**Examples:**
- S3: `lextech-content-store-219342442719`
- Lambda: `lextech-pdf-to-txt`
- OpenSearch: `lextech-search`
- API Gateway: `LexTech-Search-API`

## Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | HTML/CSS/JavaScript | User interface |
| **API Layer** | API Gateway | REST API endpoint |
| **Compute** | AWS Lambda (Python) | Serverless functions |
| **Search** | Amazon OpenSearch | Document indexing & search |
| **Storage** | Amazon S3 | PDF and text file storage |
| **CI/CD** | CodePipeline + CodeBuild | Automated deployment |
| **IaC** | CloudFormation | Infrastructure management |
| **Monitoring** | CloudWatch | Logs and metrics |
| **Networking** | VPC, Security Groups | Private network isolation |

## Security Architecture

### IAM Roles & Permissions

**Lambda Execution Roles:**
- S3 read/write permissions
- VPC network interface creation
- CloudWatch Logs write access
- OpenSearch domain access

**CodeBuild Service Role:**
- S3 artifact access
- CloudFormation stack updates
- Lambda function updates

**OpenSearch Role Mapping:**
- Lambda execution roles mapped to OpenSearch internal roles
- Fine-grained access control enabled

### Network Security
- OpenSearch not publicly accessible
- All communication within VPC
- EC2 jump host RDP restricted to single IP
- Lambda functions in private subnets

## Scalability Considerations

- **Lambda:** Auto-scales based on invocations
- **OpenSearch:** Cluster can scale horizontally (add nodes)
- **S3:** Unlimited storage, automatic scaling
- **API Gateway:** Handles 10,000 requests/second by default

## Cost Optimization

- **Serverless Architecture:** Pay only for execution time
- **No Idle Resources:** Lambda functions only run when triggered
- **S3 Lifecycle Policies:** Can archive old documents to Glacier
- **OpenSearch:** Right-sized instance types for workload

---

For the visual architecture diagram, please refer to `project-documentation.pdf` page 1, or the architecture image at the top of the main README.
