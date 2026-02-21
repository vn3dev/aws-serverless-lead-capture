# Serverless Static Website & Lead Capture on AWS

A fully managed serverless architecture delivering static content via CloudFront and processing lead submissions using API Gateway, Lambda, DynamoDB, and SES

This project implements a scalable serverless architecture on AWS. Static assets are delivered through CloudFront from a private S3 origin, while form submissions are processed through API Gateway and AWS Lambda. Leads are persisted in DynamoDB and email notifications are sent via Amazon SES. Operational monitoring and cost controls are implemented using CloudWatch and AWS Budgets

## 📚 Documentation

- [Architecture Decision Records](./adr/)
- [Build Process](./build-process/)

## Architecture Diagram

![Architecture Diagram](docs/img/serverless-lead-capture-diagramv2.png)

## Demo

### Form Submission Flow

![Input form test](docs/img/get-ebook-example.gif)

### DynamoDB Persistence

![Table item registry](docs/img/tableitem-example.png)

### CloudWatch Logs

![Cloudwatch log test](docs/img/cloudwatch-log-example.png)

### Owner Email

![Owner email inbox test](docs/img/email-example.png)
![Owner email message test](docs/img/email2-example.png)

## Request Flow

1. User → DNS → CloudFront  
2. CloudFront → S3
3. Browser → API Gateway  
4. API Gateway → Lambda  
5. Lambda → DynamoDB  
6. Lambda → SES  
7. Lambda → CloudWatch  
8. CloudWatch → Alarm → SNS → Email  
9. AWS Budgets → Alert → Email

## AWS Services Used

- Amazon CloudFront
- Amazon S3
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon SES
- Amazon CloudWatch
- Amazon SNS
- AWS Budgets
- AWS Certificate Manager

## Security & Hardening

- Private S3 bucket with CloudFront origin access control
- TLS termination via ACM
- CORS restricted to specific domains
- Input validation and sanitization
- Payload size limits
- API Gateway throttling
- Least-privilege IAM roles
- PII removed from logs
- Email masking for notifications

## Scalability

- Fully serverless architecture
- Automatic horizontal scaling
- Pay-per-use compute model
- API throttling controls
- Optional async processing: SQS-ready architecture

## Cost Management

- AWS Budgets alerts configured
- Free-tier conscious architecture
- No always-on compute resources
- Pay-per-request billing model

## Future Improvements

- Introduce SQS for async email processing
- Implement AWS WAF for advanced protection
- Add CI/CD pipeline
- Infrastructure as Code: AWS SAM / Terraform
- Add tracing with AWS X-Ray

## Author

Vinicius Costa  
Serverless Architecture Design February 2026