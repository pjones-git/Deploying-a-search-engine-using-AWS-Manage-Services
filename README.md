# Deploying a Search Engine Using AWS Managed Services

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=flat-square&logo=amazon-aws)
![Lambda](https://img.shields.io/badge/Lambda-Serverless-blue?style=flat-square&logo=aws-lambda)
![OpenSearch](https://img.shields.io/badge/OpenSearch-Search-green?style=flat-square)
![CI/CD](https://img.shields.io/badge/CI%2FCD-Pipeline-yellow?style=flat-square&logo=github-actions)

## 📋 Project Overview

A fully automated, serverless search engine built on AWS managed services that processes PDF documents, extracts text content, and provides full-text search capabilities through a web interface. This project demonstrates enterprise-grade cloud architecture with automated CI/CD pipelines, Infrastructure as Code (IaC), and event-driven processing.

**Project Completion:** August 4, 2025  
**Author:** Patrick M. Jones

---

## 🏗️ Architecture

<img width="689" height="513" alt="Screenshot 2025-10-29 at 7 00 12 PM" src="https://github.com/user-attachments/assets/a9c5425c-d8c0-4e2e-87f5-ce8d7a3d6cb1" />


### Architecture Flow
1. **Source Control** → GitHub repository with automated CI/CD integration
2. **Build & Deploy** → CodePipeline triggers CodeBuild on code changes
3. **Infrastructure** → CloudFormation manages all AWS resources as code
4. **Document Processing** → S3 event triggers Lambda for PDF-to-text conversion
5. **Indexing** → Text files automatically indexed in OpenSearch
6. **Search Interface** → API Gateway + Lambda serves web interface and processes queries
7. **Monitoring** → CloudWatch logs all Lambda executions and system events

---

## ✨ Key Features

### Document Processing Pipeline
- **Automated PDF Processing:** Upload PDFs to S3 bucket to trigger automatic text extraction
- **Multi-stage Processing:** PDF → Text → OpenSearch indexing pipeline
- **Event-driven Architecture:** S3 event notifications trigger Lambda functions
- **Intermediary Storage:** Separate buckets for PDFs and extracted text

### Search Capabilities
- **Full-text Search:** Query documents using OpenSearch's powerful search engine
- **Web Interface:** User-friendly HTML/CSS/JavaScript interface served via API Gateway
- **Real-time Results:** Search results returned instantly from OpenSearch cluster
- **Relevance Scoring:** Results ranked by relevance to search query

### DevOps & Automation
- **CI/CD Pipeline:** Automated deployment using CodePipeline and CodeBuild
- **Infrastructure as Code:** All resources defined in CloudFormation templates
- **Automated Testing:** Pipeline validation before deployment
- **Version Control:** GitHub integration with personal access token authentication

### Security & Networking
- **VPC Isolation:** OpenSearch cluster deployed in private VPC
- **Security Groups:** Configured for Lambda-to-OpenSearch communication
- **IAM Roles:** Least-privilege access for all services
- **EC2 Jump Host:** Secure access to OpenSearch dashboard via RDP (restricted to specific IP)

---

## 🛠️ Skills & Technologies

### AWS Services
| Service | Purpose | Configuration |
|---------|---------|---------------|
| **Lambda** | Serverless compute for PDF processing, text indexing, and search | 4 functions with VPC access |
| **OpenSearch** | Document indexing and full-text search engine | VPC deployment, 3-AZ with standby |
| **S3** | Storage for PDFs, text files, and CI/CD artifacts | 3 buckets with event notifications |
| **API Gateway** | REST API for search interface and query processing | Lambda proxy integration |
| **CodePipeline** | Automated CI/CD orchestration | Source → Build stages |
| **CodeBuild** | Build and deployment automation | buildspec.yml for each Lambda |
| **CloudFormation** | Infrastructure as Code (IaC) | 4 stacks for modular deployment |
| **EC2** | Jump host for OpenSearch dashboard access | Windows instance in same VPC |
| **CloudWatch** | Logging and monitoring | Automatic Lambda log groups |
| **IAM** | Security and access management | Custom roles for each service |

### Development Skills
- **Python/Lambda Development:** Serverless function development
- **REST API Design:** API Gateway configuration and Lambda integration
- **HTML/CSS/JavaScript:** Frontend web interface development
- **YAML Configuration:** buildspec.yml and CloudFormation templates
- **Event-driven Architecture:** S3 triggers and asynchronous processing
- **VPC Networking:** Security groups, subnets, and private networking
- **OpenSearch/Elasticsearch:** Index management and search query optimization

### DevOps Practices
- **CI/CD Pipelines:** Automated testing and deployment
- **Infrastructure as Code (IaC):** CloudFormation template development
- **Version Control:** Git workflows and GitHub integration
- **Troubleshooting:** CloudWatch logs analysis and debugging
- **Security Configuration:** IAM policies, role mapping, and VPC security

---

## 📁 Project Structure

```
.
├── README.md                          # This file
├── docs/
│   ├── architecture-diagram.png       # System architecture diagram
│   ├── setup-guide.md                # Detailed setup instructions
│   └── troubleshooting.md            # Common issues and solutions
├── PDFtoTxt/
│   ├── lambda_function.py            # PDF to text extraction Lambda
│   ├── requirements.txt              # Python dependencies
│   └── buildspec.yml                 # CodeBuild specification
├── UploadToSearch/
│   ├── lambda_function.py            # Text to OpenSearch indexing Lambda
│   ├── requirements.txt
│   └── buildspec.yml
├── SearchGateway/
│   ├── lambda_function.py            # Web interface serving Lambda
│   ├── requirements.txt
│   └── buildspec.yml
├── SearchFunction/
│   ├── lambda_function.py            # Search query processing Lambda
│   ├── requirements.txt
│   └── buildspec.yml
└── cloudformation/
    ├── api-gateway-stack.yaml        # API Gateway infrastructure
    ├── opensearch-stack.yaml         # OpenSearch cluster configuration
    └── s3-buckets-stack.yaml         # S3 buckets and policies
```

---

## 🚀 AWS Resources

### S3 Buckets
- **lextech-content-store-219342442719** - PDF upload bucket (triggers PDFtoTxt Lambda)
- **lextech-intermediary-store-219342442719** - Text file storage (triggers UploadToSearch Lambda)
- **lextech-artifacts-bucket-219342442719** - CI/CD artifacts storage

### Lambda Functions
| Function Name | Trigger | Purpose |
|---------------|---------|------|
| **lextech-pdf-to-txt** | S3 .pdf upload | Extract text from PDF files |
| **lextech-upload-to-search** | S3 .txt upload | Index text in OpenSearch |
| **lextech-search-gateway** | API Gateway GET / | Serve web interface |
| **lextech-search-function** | API Gateway POST /search | Process search queries |

### OpenSearch Domain
- **Domain:** lextech-search
- **Version:** OpenSearch 2.19
- **Deployment:** 3-AZ with standby
- **Index:** lextech-documents
- **Access:** VPC-based with EC2 jump host

### API Gateway
- **API Name:** LexTech-Search-API
- **Endpoint:** REST API with Lambda proxy integration
- **Routes:**
  - `GET /` - Web interface (SearchGateway Lambda)
  - `POST /api/search` - Search queries (SearchFunction Lambda)

---

## 📊 Project Summary

This project implements a production-ready, serverless search engine on AWS that automatically processes PDF documents and makes them searchable through a web interface. The architecture leverages AWS managed services to eliminate server management while providing scalability, reliability, and cost-efficiency.

### Key Accomplishments
✅ **Fully Automated Pipeline:** PDF upload → Text extraction → Indexing → Searchable in seconds  
✅ **Event-Driven Architecture:** S3 events trigger Lambda functions for seamless processing  
✅ **CI/CD Integration:** GitHub pushes automatically deploy to AWS via CodePipeline  
✅ **Infrastructure as Code:** All resources managed through CloudFormation templates  
✅ **Secure VPC Design:** Private OpenSearch cluster with controlled access  
✅ **Production-Ready:** Comprehensive error handling, logging, and monitoring  

### Technical Highlights
- **Zero Server Management:** 100% serverless architecture using Lambda and managed services
- **Scalable Design:** Auto-scaling Lambda and OpenSearch cluster handles varying loads
- **Cost-Effective:** Pay-per-use model with no idle resource costs
- **Resilient:** Multi-AZ OpenSearch deployment with automatic failover
- **Maintainable:** Modular CloudFormation stacks for easy updates

---

## 🎓 Skills Demonstrated

### Cloud Architecture
- Designing serverless, event-driven systems
- Implementing multi-tier application architecture
- VPC networking and security group configuration
- High availability and fault tolerance design

### AWS Expertise
- Lambda function development and optimization
- OpenSearch cluster deployment and management
- API Gateway REST API design
- S3 event notification configuration
- CloudFormation Infrastructure as Code
- IAM security policy creation

### DevOps Engineering
- CI/CD pipeline implementation
- Automated build and deployment workflows
- Infrastructure as Code (IaC) practices
- Log aggregation and monitoring
- Troubleshooting complex distributed systems

### Software Development
- Python serverless application development
- REST API design and implementation
- Frontend web development (HTML/CSS/JS)
- Asynchronous event processing
- Error handling and retry logic

---

## 🔧 Setup Instructions

### Prerequisites
- AWS Account with administrative access
- GitHub account
- AWS CLI configured
- Python 3.9+
- Git

### Deployment Steps

1. **Clone Repository**
   ```bash
   git clone https://github.com/pjones-git/Deploying-a-search-engine-using-AWS-Manage-Services.git
   cd Deploying-a-search-engine-using-AWS-Manage-Services
   ```

2. **Configure GitHub Token**
   - Create GitHub Personal Access Token (Classic)
   - Store in AWS Secrets Manager or update buildspec.yml

3. **Deploy CloudFormation Stacks**
   ```bash
   # Deploy S3 buckets
   aws cloudformation create-stack --stack-name lextech-s3-buckets \
     --template-body file://cloudformation/s3-buckets-stack.yaml
   
   # Deploy OpenSearch cluster
   aws cloudformation create-stack --stack-name lextech-opensearch \
     --template-body file://cloudformation/opensearch-stack.yaml \
     --capabilities CAPABILITY_IAM
   
   # Deploy API Gateway
   aws cloudformation create-stack --stack-name lextech-api-gateway \
     --template-body file://cloudformation/api-gateway-stack.yaml \
     --capabilities CAPABILITY_IAM
   ```

4. **Configure OpenSearch**
   - RDP into EC2 jump host (restricted to your IP)
   - Access OpenSearch dashboard
   - Create `lextech-documents` index

5. **Deploy Lambda Functions**
   - CodePipeline automatically deploys on GitHub push
   - Or manually upload Lambda packages

6. **Test the System**
   - Upload a PDF to the content S3 bucket
   - Check CloudWatch logs for processing
   - Access API Gateway URL to search documents

For detailed setup instructions, see [docs/setup-guide.md](./docs/setup-guide.md)

---

## 🐛 Troubleshooting

### Common Issues Encountered & Resolved

#### 1. Lambda-OpenSearch VPC Communication
**Problem:** Lambda functions couldn't reach OpenSearch cluster  
**Solution:** Configured security groups and subnets to allow Lambda VPC access

#### 2. CI/CD Pipeline Authentication
**Problem:** "PLACEHOLDER_TOKEN" error preventing GitHub access  
**Solution:** Created valid GitHub Personal Access Token and updated pipeline

#### 3. OpenSearch Permission Denied
**Problem:** Lambda couldn't index documents in OpenSearch  
**Solution:** Mapped Lambda execution role in OpenSearch security settings

#### 4. PDF Processing Not Triggering Search
**Problem:** Uploaded PDFs weren't appearing in search results  
**Solution:** Debugged S3 event chain: upload → PDF processing → text extraction → indexing

#### 5. Lambda Deployment Failures
**Problem:** Inconsistent Lambda deployments via CodeBuild  
**Solution:** Fixed buildspec.yml configurations and IAM permissions

For more troubleshooting tips, see [docs/troubleshooting.md](./docs/troubleshooting.md)

---

## 📈 Lessons Learned

This project presented significant complexity and required nearly two weeks of dedicated effort. Key insights gained:

### Technical Lessons
- **VPC Networking is Critical:** Proper security group and subnet configuration is essential for Lambda-OpenSearch communication
- **IAM Permissions are Complex:** Role mapping in OpenSearch requires careful configuration of both AWS IAM and OpenSearch internal roles
- **Event-Driven Debugging:** Tracing events through multiple services (S3 → Lambda → OpenSearch) requires systematic CloudWatch log analysis
- **CI/CD Takes Time:** Pipeline setup involves many moving parts and requires iterative troubleshooting

### Personal Growth
- **Persistence Pays Off:** Overcame multiple "brick wall" moments through systematic debugging
- **Incremental Progress:** Breaking down complex problems into smaller steps was key to success
- **Documentation Matters:** Recording configurations and errors saved time when revisiting issues
- **Cloud Complexity:** Real-world cloud architecture involves many interconnected services that must work together

### Professional Skills
- **Problem-Solving:** Developed systematic approach to debugging distributed systems
- **AWS Expertise:** Deep hands-on experience with 10+ AWS services
- **DevOps Mindset:** Understanding the importance of automation and Infrastructure as Code
- **Resilience:** Learning to push through difficult technical challenges

---

## 🎯 Use Cases

This architecture pattern can be applied to:
- **Document Management Systems:** Legal firms, medical records, enterprise knowledge bases
- **Content Discovery Platforms:** News archives, research paper repositories
- **E-Learning Platforms:** Course materials, lecture notes, study guides
- **Compliance & Audit:** Searchable document repositories for regulatory requirements
- **Technical Documentation:** API docs, user manuals, troubleshooting guides

---

## 🔒 Security Considerations

- **VPC Isolation:** OpenSearch cluster not publicly accessible
- **IAM Least Privilege:** Each service has minimal required permissions
- **Encryption:** S3 buckets use server-side encryption
- **Access Control:** EC2 jump host RDP restricted to specific IP (108.91.0.80/32)
- **API Security:** API Gateway can be configured with authentication (future enhancement)

---

## 🚧 Future Enhancements

- [ ] Add API authentication (API keys or Cognito)
- [ ] Implement document versioning
- [ ] Support additional file formats (DOCX, TXT, HTML)
- [ ] Add document preview in search results
- [ ] Implement advanced search filters (date, author, category)
- [ ] Add usage analytics and metrics dashboard
- [ ] Implement document deletion workflow
- [ ] Create mobile-responsive UI improvements
- [ ] Add batch upload capability
- [ ] Implement search autocomplete

---

## 📞 Contact

**Patrick M. Jones**  
GitHub: [@pjones-git](https://github.com/pjones-git)  
Project Link: [https://github.com/pjones-git/Deploying-a-search-engine-using-AWS-Manage-Services](https://github.com/pjones-git/Deploying-a-search-engine-using-AWS-Manage-Services)

---

## 📄 License

This project is available for portfolio review and educational purposes.

---

## 🙏 Acknowledgments

- AWS Documentation and Best Practices
- OpenSearch Community
- Great Learning Post Graduate Program in Cloud Computing (August 2024 Cohort)

---

*This project demonstrates enterprise-level cloud architecture skills suitable for Cloud Engineer, DevOps Engineer, and Solutions Architect roles.*
