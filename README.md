<div align="center">
  
![GitHub Actions](https://img.icons8.com/color/96/github.png)
![AWS ECS](https://img.icons8.com/color/96/amazon-web-services.png)
![OIDC](https://img.icons8.com/color/96/security-checked.png)

# AWS ECS Fargate Nginx - Unified CI/CD Pipeline with GitHub Actions & OIDC

**Updated: January 14, 2026**

[![Follow @nicoleepaixao](https://img.shields.io/github/followers/nicoleepaixao?label=Follow&style=social)](https://github.com/nicoleepaixao)
[![Star this repo](https://img.shields.io/github/stars/nicoleepaixao/aws-ecs-fargate-nginx-oidc-pipeline?style=social)](https://github.com/nicoleepaixao/aws-ecs-fargate-nginx-oidc-pipeline)
[![Medium Article](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://nicoleepaixao.medium.com/)

<p align="center">
  <a href="README-PT.md">🇧🇷</a>
  <a href="README.md">🇺🇸</a>
</p>

</div>

---

<p align="center">
  <img src="img/pipeline-ecs.png" alt="Pipeline ECS Architecture" width="1200">
</p>

## **The Problem**

Traditional CI/CD pipelines require storing **long-lived AWS credentials** (Access Keys) in GitHub Secrets, which creates security risks:

- **Credentials can be stolen** if the repository is compromised
- **No automatic rotation** of credentials
- **Difficult auditing** of who used credentials and when
- **Broad permissions** often granted for simplicity

This project implements **GitHub Actions OIDC** authentication, eliminating the need for stored credentials while maintaining secure, automated deployments to AWS ECS Fargate.

---

## **Solution**

This repository provides a **single, production-ready CI/CD pipeline** that:

✅ **Zero long-lived credentials** - Uses OIDC for temporary AWS access  
✅ **Automated builds** - Builds Docker images on every commit  
✅ **Intelligent multi-environment** - Automatically detects homol (develop) and prod (main)  
✅ **Secure deployments** - Push to ECR and deploy to ECS automatically  
✅ **Unified pipeline** - Single workflow that adapts to the environment  
✅ **GitOps** - Infrastructure as code with complete audit trail

---

## **Project Structure**

```text
aws-ecs-fargate-nginx-oidc-pipeline/
│
├── .github/
│   └── workflows/
│       ├── deploy.yml                # Unified pipeline (single)
│       └── pr-validation.yml         # PR validation
│
├── config/
│   ├── homol.env                     # Homologation configuration
│   └── prod.env                      # Production configuration
│
├── Dockerfile                        # Nginx container
├── nginx.conf                        # Nginx configuration
├── html/
│   └── index.html                    # Static content
└── scripts/
    └── deploy-helper.sh              # Helper scripts (optional)
```

---

## **Required GitHub Secrets**

Configure **only 2 secrets** in GitHub (`Settings > Secrets and variables > Actions`):

| **Secret** | **Description** | **Example** |
|------------|-----------------|-------------|
| `AWS_ROLE_ARN_HOMOL` | IAM role ARN for homologation | `arn:aws:iam::123456789012:role/GitHubActionsRole-nginx-homol` |
| `AWS_ROLE_ARN_PROD` | IAM role ARN for production | `arn:aws:iam::123456789012:role/GitHubActionsRole-nginx-prod` |

**That's it! No need for AWS_ACCOUNT_ID or Access Keys!**

---

## **How the Unified Pipeline Works**

### **1. Automatic Environment Detection**

```yaml
# Job: setup
# Detects branch and automatically defines all variables

develop → homologation
  ├── ECS Service: nginx-service-homol
  ├── Task Definition: nginx-homol
  └── Role ARN: AWS_ROLE_ARN_HOMOL

main → production
  ├── ECS Service: nginx-service-prod
  ├── Task Definition: nginx
  └── Role ARN: AWS_ROLE_ARN_PROD
```

### **2. Tagging Strategy**

The pipeline creates multiple tags for traceability:

```bash
# For HOMOLOGATION (develop)
{ECR_REPO}:{SHA}                    # Ex: ecr_nginx:a1b2c3d
{ECR_REPO}:homologation-{SHA}       # Ex: ecr_nginx:homologation-a1b2c3d

# For PRODUCTION (main)
{ECR_REPO}:{SHA}                    # Ex: ecr_nginx:a1b2c3d
{ECR_REPO}:production-{SHA}         # Ex: ecr_nginx:production-a1b2c3d
{ECR_REPO}:latest                   # Ex: ecr_nginx:latest (prod only)
```

### **3. Deployment Summary**

At the end of each deployment, the pipeline generates an automatic summary:

```text
🚀 Deployment Summary

| Item              | Value                           |
|-------------------|---------------------------------|
| Environment       | production                      |
| Service           | nginx-service-prod              |
| Cluster           | my-cluster                      |
| Image             | 123.ecr.../ecr_nginx:a1b2c3d   |
| Commit            | a1b2c3d                         |
| Deployed by       | @nicoleepaixao                  |
| Application URL   | http://my-alb-123.us-east-1.elb.amazonaws.com |

✅ Deployment completed successfully!
```

---

## **Quick Setup**

### **Step 1: Clone the Repository**

```bash
git clone https://github.com/nicoleepaixao/aws-ecs-fargate-nginx-oidc-pipeline.git
cd aws-ecs-fargate-nginx-oidc-pipeline
```

---

### **Step 2: Configure GitHub Environments**

Create two environments in GitHub (`Settings > Environments`):

1. **homologation** - For staging deployments
2. **production** - For production deployments

**Recommended protection rules for production:**
- ✅ Required reviewers (at least 1)
- ✅ Wait timer (5-10 minutes)
- ✅ Deployment branches: `main` only

---

### **Step 3: Update Variables**

Edit `.github/workflows/deploy.yml` and adjust variables at the top:

```yaml
env:
  AWS_REGION: us-east-1          # Your AWS region
  ECR_REPOSITORY: ecr_nginx       # Your ECR repository name
  ECS_CLUSTER: my-cluster         # Your ECS cluster name
  CONTAINER_NAME: nginx           # Container name in task definition
```

And in the `setup` job, adjust service and task definition names:

```yaml
# For PRODUCTION (main)
echo "ecs-service=nginx-service-prod" >> $GITHUB_OUTPUT
echo "task-definition=nginx" >> $GITHUB_OUTPUT

# For HOMOLOGATION (develop)
echo "ecs-service=nginx-service-homol" >> $GITHUB_OUTPUT
echo "task-definition=nginx-homol" >> $GITHUB_OUTPUT
```

---

### **Step 4: Add GitHub Secrets**

1. Go to `Settings > Secrets and variables > Actions`
2. Click `New repository secret`
3. Add the 2 secrets:

**Secret 1:**
- Name: `AWS_ROLE_ARN_HOMOL`
- Value: `arn:aws:iam::YOUR_ACCOUNT:role/GitHubActionsRole-nginx-homol`

**Secret 2:**
- Name: `AWS_ROLE_ARN_PROD`
- Value: `arn:aws:iam::YOUR_ACCOUNT:role/GitHubActionsRole-nginx-prod`

---

### **Step 5: Test the Pipeline**

**For Homologation:**
```bash
git checkout -b develop
git add .
git commit -m "test: homologation deployment"
git push origin develop
```

**For Production:**
```bash
git checkout main
git add .
git commit -m "feat: production deployment"
git push origin main
```

Monitor at `Actions` → See the pipeline automatically detect the environment!

---

## **Development Workflow**

```text
Complete Flow:
┌─────────────────────────────────────────────────────────────┐
│ 1. Create feature branch                                     │
│    git checkout -b feature/new-feature develop              │
│                                                              │
│ 2. Develop and commit                                       │
│    git add .                                                 │
│    git commit -m "feat: add feature X"                      │
│                                                              │
│ 3. Push and create PR to develop                            │
│    git push origin feature/new-feature                      │
│    → PR validation runs (builds + tests)                    │
│                                                              │
│ 4. Merge PR to develop                                      │
│    → Pipeline detects: branch=develop                       │
│    → AUTOMATIC Deploy to HOMOLOGATION                       │
│                                                              │
│ 5. Test in homologation environment                         │
│    → Validate features                                      │
│                                                              │
│ 6. Create PR from develop to main                           │
│    → PR validation runs again                               │
│    → Requires approval (if configured)                      │
│                                                              │
│ 7. Merge PR to main                                         │
│    → Pipeline detects: branch=main                          │
│    → AUTOMATIC Deploy to PRODUCTION                         │
│    → Wait timer + approval (if configured)                  │
└─────────────────────────────────────────────────────────────┘
```

---

## **Unified Pipeline Advantages**

### **Simplified Maintenance**
- **Single pipeline** to manage
- **Changes propagate** to all environments
- **Less code duplication**

### **Consistency**
- **Same process** for homol and prod
- **Same quality** in all environments
- **Reduces configuration** errors

### **Security**
- **OIDC for both** environments
- **Separate roles** with specific permissions
- **Complete audit** via CloudTrail

### **Flexibility**
- **Easy to add** new environments
- **Customization** per environment in GitHub
- **Intelligent tags** for traceability

---

## **Monitoring**

### **GitHub Actions Dashboard**

1. Go to `Actions` → `Deploy to AWS ECS`
2. See which environment was detected
3. Follow logs in real-time
4. Check deployment summary

### **AWS CLI Commands**

**View deployment status:**
```bash
aws ecs describe-services \
  --cluster my-cluster \
  --services nginx-service-prod \
  --query 'services[0].{status:status,events:events[0:3]}'
```

**List images in ECR:**
```bash
aws ecr list-images \
  --repository-name ecr_nginx \
  --query 'imageIds[*].[imageTag]' \
  --output table
```

---

## **Rollback**

### **Method 1: Git Revert (Recommended)**

```bash
# Revert last commit
git revert HEAD
git push origin main  # Triggers new deployment with previous code
```

### **Method 2: Redeploy Previous Task Definition**

```bash
# List revisions
aws ecs list-task-definitions --family-prefix nginx --sort DESC

# Update to previous revision
aws ecs update-service \
  --cluster my-cluster \
  --service nginx-service-prod \
  --task-definition nginx:5  # Previous revision number
```

---

## **Troubleshooting**

| **Error** | **Cause** | **Solution** |
|----------|-----------|-------------|
| `Branch not configured` | Push to unmapped branch | Add branch in `setup` job |
| `Unable to assume role` | Incorrect trust policy | Verify IAM role trust policy |
| `Permission denied` | Role missing permissions | Add necessary permissions |
| `Service didn't stabilize` | Health checks failing | Check ALB target group |

---

## **Best Practices**

### **Security**

✅ Use separate IAM roles for homol and prod  
✅ Enable branch protection on `main`  
✅ Require pull request reviews  
✅ Use GitHub Environments with approvals  
✅ Enable ECR image scanning  
✅ Rotate nothing (OIDC handles it!)

### **Deployment**

✅ Always test in homologation first  
✅ Use semantic versioning for tags  
✅ Tag production images with git SHA  
✅ Enable ECS deployment circuit breaker  
✅ Set appropriate health check grace period  
✅ Monitor CloudWatch metrics and alarms

### **Code Quality**

✅ Run linting on PRs  
✅ Include automated tests  
✅ Use multi-stage Docker builds  
✅ Scan for security vulnerabilities  
✅ Keep Docker images minimal

---

## **Prerequisites**

Before using this pipeline, you must have:

1. **AWS Infrastructure deployed** - Follow [aws-ecs-fargate-nginx-awscli](https://github.com/nicoleepaixao/aws-ecs-fargate-nginx-awscli)
2. **OIDC configured in AWS** - Follow [aws-github-oidc-pipeline](https://github.com/nicoleepaixao/aws-github-oidc-pipeline)
3. **GitHub repository** with appropriate permissions
4. **ECS cluster, service, and task definitions** already created

---

## **How OIDC Works**

**Traditional Method (Insecure):**
```text
GitHub Secrets → Static AWS Access Key → Stored forever → Security risk
```

**OIDC Method (Secure):**
```text
GitHub generates JWT token → AWS validates token → Temporary credentials (15 min) → Auto-expires
```

**Key Benefits:**
1. **No stored credentials** - Nothing to steal or rotate
2. **Temporary access** - Credentials expire automatically
3. **Fine-grained permissions** - Scoped to specific repos/branches
4. **Full audit trail** - CloudTrail logs every action
5. **Compliance friendly** - Meets security best practices

---

## **Required IAM Roles**

You need two IAM roles (created via [aws-github-oidc-pipeline](https://github.com/nicoleepaixao/aws-github-oidc-pipeline)):

1. **GitHubActionsRole-nginx-homol** - For homologation deployments
2. **GitHubActionsRole-nginx-prod** - For production deployments

**Required permissions for both roles:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:DescribeServices",
        "ecs:DescribeTaskDefinition",
        "ecs:DescribeTasks",
        "ecs:ListTasks",
        "ecs:RegisterTaskDefinition",
        "ecs:UpdateService"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": [
        "arn:aws:iam::*:role/ecsTaskExecutionRole",
        "arn:aws:iam::*:role/ecsTaskRole-nginx"
      ]
    }
  ]
}
```

---

## **Advanced Features**

### **Blue/Green Deployments**

Modify workflow to use ECS Blue/Green deployment:

```yaml
- name: Deploy with Blue/Green
  run: |
    aws deploy create-deployment \
      --application-name AppECS-my-cluster-nginx-service \
      --deployment-group-name DgpECS-my-cluster-nginx-service \
      --revision revisionType=S3,s3Location={bucket=my-codedeploy-bucket,key=task-def.json,bundleType=JSON}
```

### **Canary Deployments**

Use ECS deployment configuration for canary:

```yaml
- name: Deploy with Canary
  run: |
    aws ecs update-service \
      --cluster my-cluster \
      --service nginx-service-prod \
      --task-definition nginx:latest \
      --deployment-configuration "maximumPercent=200,minimumHealthyPercent=50,deploymentCircuitBreaker={enable=true,rollback=true}"
```

### **Slack Notifications**

Add Slack notification step:

```yaml
- name: Notify Slack
  if: always()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'Deployment to production: ${{ job.status }}'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## **Related Projects**

- [AWS ECS Infrastructure Setup](https://github.com/nicoleepaixao/aws-ecs-fargate-nginx-awscli) - Complete infrastructure deployment with AWS CLI
- [AWS GitHub OIDC Configuration](https://github.com/nicoleepaixao/aws-github-oidc-pipeline) - OIDC setup for zero long-lived credentials

---

## **Additional Resources**

- [GitHub Actions OIDC Documentation](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [AWS ECS Deployment Best Practices](https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/deployment.html)
- [GitHub Actions for AWS](https://github.com/aws-actions)

---

## **Contributing**

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## **License**

This project is licensed under the MIT License - see the LICENSE file for details.

---

## **Connect & Follow**

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/nicoleepaixao)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?logo=linkedin&logoColor=white&style=for-the-badge)](https://www.linkedin.com/in/nicolepaixao/)
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@nicoleepaixao)

</div>

---

<div align="center">

**Automate your deployments securely with OIDC!**

*Document Created: January 14, 2026*

Made with ❤️ by [Nicole Paixão](https://github.com/nicoleepaixao)

</div>
