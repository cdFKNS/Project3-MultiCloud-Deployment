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
## Project Reflection: Nomdumiso's Azure Deployment Phase

This reflection focuses on the configuration of the hybrid deployment environment in Azure, utilizing Terraform (IaC) and Azure Pipelines (CI/CD), which was my primary area of focus.

### My Role: Azure Configuration and CI/CD Setup

My responsibility was preparing the three critical configuration files for the Azure deployment to ensure the pipeline could successfully provision the infrastructure and deploy the application code.

#### 1. Configuration Tasks Completed:

| File | Change Made | Rationale |
| :--- | :--- | :--- |
| `rbac_deployer_role.json` | Replaced the placeholder with the **Azure Subscription ID**. | Granted the Azure DevOps Service Connection the minimum required permissions (via RBAC) to create resources in the subscription. |
| `main.tf` (Terraform) | Inputted the IaC and set the **globally unique App Service Name**. | Defined the server infrastructure (App Service Plan and App Service) and ensured the name was unique across Azure. |
| `azure-pipeline.yml` | Set the final unique names for the **App Service** and the **Terraform State Storage Account**. | Finalized the CI/CD pipeline variables, linking the deployment process to the unique infrastructure defined in `main.tf`. |

#### 2. Key Challenges and Learning Points

| Challenge Encountered | Resolution and Lesson Learned |
| :--- | :--- |
| **Git Command Misplacement** | Repeatedly attempted to type Git commands (like `git push`) inside the code files (`.json`, `.tf`). | **LESSON:** Git commands are for the **Terminal** (command line), not the code editor. The editor must contain only valid code syntax (like JSON or Terraform HCL). |
| **Git Synchronization Error** | Encountered a rejected push (`error: failed to push some refs...`) because my local repository was behind the remote. | **LESSON:** When pushing fails due to "behind remote counterpart," the solution is always to **`git pull origin main`** first, and then push.  |
| **Azure Quota Block** | The deployment was halted by Azure's system due to the **"Current Limit (Free VMs): 0"** error. | **MOST IMPORTANT LESSON:** Infrastructure as Code (IaC) is limited by administrative settings. The solution required submitting a manual support request to increase the Linux Free VM quota to 1.  |

### Conclusion

Successfully configuring the three files and understanding the unique requirements of the Azure platform (like naming constraints and managing service quotas) were the defining achievements of this phase. The experience reinforced the separation of concerns between code editing and command execution, which is fundamental to professional cloud development workflows.

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

