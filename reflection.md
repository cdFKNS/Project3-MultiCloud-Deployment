# Project 3 Reflection: Multi-Cloud App Deployment (AWS Lead)

**Author:** KB (AWS Deployment Lead)  
**Application:** MercuryAI (Containerized Node.js Backend & Static Frontend)

The MercuryAI application has been successfully deployed and validated across both AWS and Azure environments, demonstrating architectural parity and functional consistency. This reflection documents the AWS deployment strategy, key security implementations, and critical multi-cloud insights gained, specifically addressing the technical challenges faced.

---

## 1. AWS Deployment Strategy and Process

**Objective:** Deploy the containerized backend and static frontend using managed, scalable AWS services to meet enterprise standards for high availability and performance.

### Components & Services

| Component       | AWS Services Used                | Rationale                                                                 |
|-----------------|----------------------------------|---------------------------------------------------------------------------|
| Backend API     | Elastic Beanstalk (EB) w/ Docker | EB abstracts infra (EC2, ALB, Auto Scaling) while leveraging Docker image. |
| Frontend Assets | Amazon S3 + CloudFront CDN       | Durable storage + global CDN for HTTPS, low latency, and DDoS protection. |
| Container Image | Amazon ECR                       | Secure, private registry integrated with CI/CD pipeline.                  |

### Key Deployment Artifacts
- **server/Dockerfile**: Defines Node.js API environment, dependencies, and container port (3001).  
- **Dockerrun.aws.json**: Specifies ECR image, port mapping, and volume mounts for logging.

---

## 2. Security Implementations and Challenges

Security followed the **Principle of Least Privilege (PoLP)**, with restrictive IAM roles and network controls.

### A. IAM and Credential Management
- **EC2 Instance Profile**  
  - Push runtime logs to CloudWatch  
  - Read `GEMINI_API_KEY` from AWS Secrets Manager (avoids hard-coded keys)  
- **CI/CD Deployment Role**  
  - Permissions limited to deployment tasks (e.g., `ecr:PushImage`, `s3:PutObject`, `elasticbeanstalk:UpdateEnvironment`)

### B. Network and CORS Configuration
- **Backend Security**: EC2 instances in private subnet, accessible only via ALB.  
- **Frontend Security**: CloudFront Origin Access Control (OAC) prevents direct S3 bucket access.  
- **CORS Challenge**:  
  - Modified `server.js` to read `PRODUCTION_CORS_ORIGINS` from environment variable.  
  - Allowed CloudFront URL dynamically while maintaining security.

> ⚠️ **Note on Security Warnings:** Low-severity issues (default security group settings, missing advanced logging) remain. In production, these require immediate remediation.

---

## 3. Multi-Cloud Insights and Consistency

Consistency between AWS and Azure deployments relied on **containerization**.

| Comparison Point         | AWS (Elastic Beanstalk)                  | Azure (App Service/AKS)                  | Insight                                                                 |
|---------------------------|------------------------------------------|------------------------------------------|-------------------------------------------------------------------------|
| Compute Abstraction       | High Control (Managed EC2/ALB)           | Higher Abstraction (Managed Service)      | AWS offers deeper visibility; Azure provides faster deployment path.    |
| Identity Management       | IAM (Resource-centric, fine-grained)     | Azure AD (User/Group-centric)             | Azure AD is unified for Microsoft users; IAM excels at fine-grained PoLP. |
| Consistency Challenge     | Env Variable Management                  | Configuration Drift                       | Risk in frontend API URL substitution; each cloud needs specific env vars. |
| Power of Parity           | Docker, Node.js                          | Docker, Node.js                          | Core app unchanged; only deployment artifacts differ.                   |

---

## 4. Conclusion

This project successfully demonstrated **multi-cloud readiness**.  
By leveraging **standardized container technology** and a disciplined IAM approach, MercuryAI can be reliably deployed and operated across different cloud providers.
