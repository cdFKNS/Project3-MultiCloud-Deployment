# Project 3 Reflection: Multi-Cloud App Deployment

**AWS Lead:** Kabelo Modimoeng (AWS Deployment Lead)  
**Application:** MercuryAI (Node.js Backend & Static Frontend)

The MercuryAI application has been successfully deployed and validated on AWS and Azure environments, demonstrating architectural parity and functional consistency. This reflection documents the finalized AWS deployment strategy, key security implementations, and critical multi-cloud insights gained, specifically addressing the technical challenges faced.

---

## 1. AWS Deployment Strategy and Process (Revised)

**Final Approach:** Simplified, single-tier architecture where both frontend assets (`index.html`, `script.js`, `style.css`) and the Node.js backend (`server.js`) were bundled into a single ZIP file (Source Bundle) and deployed to Elastic Beanstalk (EB).

### Components & Services

| Component          | AWS Service Used                  | Rationale                                                                 |
|--------------------|-----------------------------------|---------------------------------------------------------------------------|
| Entire Application | Elastic Beanstalk (Node.js)       | Consolidated hosting: EB runs Node.js server and serves static files via built-in web server (e.g., Nginx). |
| Container Registry | N/A                               | Bypassed containerization (Docker/ECR) to reduce complexity.              |
| CDN/Storage        | N/A                               | Frontend served directly from EB instance, no need for S3/CloudFront.     |

### Key Deployment Artifacts
- **ZIP Source Bundle**: Contains all application files (`index.html`, `server.js`, etc.) and `package.json`.  
- **package.json**: Defines dependencies and start script for EB environment.

---

## 2. Security Implementations and Challenges

The single-tier deployment simplified network security but maintained strict IAM and secret management.

### A. IAM and Credential Management
- **EC2 Instance Profile**  
  - Push runtime logs to CloudWatch  
  - Read `GEMINI_API_KEY` from AWS Secrets Manager (avoids hard-coded keys)  
- **CI/CD Deployment Role**  
  - Limited to deployment-specific actions (upload ZIP to EB S3 bucket, trigger environment updates)

### B. Network and Security Group Configuration
- **CORS Simplification**:  
  - Frontend and backend deployed on same EB instance → automatically satisfies same-origin policy.  
  - Eliminated need for dynamic CORS configuration or CloudFront whitelisting.  
- **Inbound Access**:  
  - Load Balancer Security Group terminates HTTP/S traffic (ports 80/443).  
  - Instances only accept traffic from Load Balancer on internal port.  

> ⚠️ **Note on Security Warnings:** Pending low-severity issues (default security group settings, missing advanced logging) remain. In production, these require immediate remediation.

---

## 3. Multi-Cloud Insights and Consistency

Deployment parity achieved through **code consistency**, despite architectural differences.

| Comparison Point     | AWS (Elastic Beanstalk Single-Tier) | Azure (App Service)                  | Insight                                                                 |
|----------------------|--------------------------------------|--------------------------------------|-------------------------------------------------------------------------|
| Deployment Model     | Single Source Bundle (Node.js)       | Single App Service Unit               | Both used single-environment deployment, proving monolithic hosting works. |
| Service Consistency  | EB web server config for static files | App Service requires startup commands | Challenge shifted from CORS to internal file serving configuration.     |
| Consistency Strategy | Core app logic unchanged             | Core app logic unchanged              | Portable Node.js code → infra setup differs, app remains cloud-agnostic. |
| Multi-Cloud Risk     | Dependency on EB’s native features   | Requires custom config for parity     | Mild vendor lock-in risk due to AWS-specific behavior.                  |

---

## 4. Conclusion

The final single-tier AWS deployment strategy successfully hosted the application, offering **simpler setup** and **same-origin communication benefits** compared to a decoupled S3/CloudFront approach.  

Alongside Azure App Service deployment, this validates MercuryAI’s **multi-cloud readiness**.  
Key takeaway: strategic choices—like using a single source bundle—can simplify deployment, shifting focus from cross-origin issues to proper cloud platform configuration.
