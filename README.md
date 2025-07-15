# SimpleTimeService - DevOps Challenge

A simple Python web service that returns current timestamp and visitor's IP address, deployed on AWS using ECS Fargate with Terraform.

## Overview

This project demonstrates:
- **Containerized Python Flask application**
- **Infrastructure as Code** using Terraform
- **AWS ECS Fargate** deployment
- **Application Load Balancer** for high availability
- **VPC with public/private subnets** for security
- **Docker best practices** (non-root user, minimal image)

## Architecture

```
Internet → ALB (Public Subnets) → ECS Tasks (Private Subnets)
```

- **VPC**: 10.0.0.0/16 with 2 AZs
- **Public Subnets**: 10.0.1.0/24, 10.0.2.0/24 (ALB)
- **Private Subnets**: 10.0.10.0/24, 10.0.20.0/24 (ECS Tasks)
- **NAT Gateways**: For private subnet internet access
- **ECS Fargate**: Serverless container hosting
- **Application Load Balancer**: Traffic distribution

## API Response

```json
{
  "timestamp": "2025-07-13T10:30:45Z",
  "ip": "203.0.113.1"
}
```

## Repository Structure

```
.
├── app/
│   ├── app.py              # Flask application
│   ├── requirements.txt    # Python dependencies
│   └── Dockerfile         # Container configuration
├── terraform/
│   ├── main.tf            # Main infrastructure
│   ├── variables.tf       # Input variables
│   ├── outputs.tf         # Output values
│   └── terraform.tfvars   # Variable values
└── README.md              # This file
```

## Prerequisites

### Required Tools

1. **Docker** - [Install Docker](https://docs.docker.com/get-docker/)
2. **AWS CLI** - [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
3. **Terraform** - [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
4. **Git** - [Install Git](https://git-scm.com/downloads)

### AWS Account Setup

1. **Create AWS Account** - [AWS Free Tier](https://aws.amazon.com/free/)
2. **Create IAM User** with programmatic access
3. **Attach Policies**:
   - `AmazonECSFullAccess`
   - `AmazonVPCFullAccess`
   - `AmazonEC2FullAccess`
   - `IAMFullAccess`
   - `AmazonRoute53FullAccess`
   - `CloudWatchLogsFullAccess`

## Part 1: Application Setup

### 1. Clone Repository

```bash
git clone <your-repository-url>
cd simpletimeservice
```

### 2. Build and Test Docker Image

```bash
cd app

# Build the image
docker build -t simpletimeservice:latest .

# Run locally for testing
docker run -p 5000:5000 simpletimeservice:latest

# Test the service
curl http://localhost:5000
```

### 3. Push to Docker Hub

```bash
# Login to Docker Hub
docker login

# Tag the image
docker tag simpletimeservice:latest YOUR_DOCKERHUB_USERNAME/simpletimeservice:latest

# Push to registry
docker push YOUR_DOCKERHUB_USERNAME/simpletimeservice:latest
```

## Part 2: Infrastructure Deployment

### 1. Configure AWS Credentials

```bash
# Option 1: AWS CLI
aws configure
# Enter your Access Key ID, Secret Access Key, Region (us-east-1), and output format (json)

# Option 2: Environment Variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

### 2. Update Terraform Configuration

Edit `terraform/terraform.tfvars`:

```hcl
container_image = "YOUR_DOCKERHUB_USERNAME/simpletimeservice:latest"
```

### 3. Deploy Infrastructure

```bash
cd terraform

# Initialize Terraform
terraform init

# Plan the deployment
terraform plan

# Apply the changes
terraform apply
```

Type `yes` when prompted to confirm deployment.

### 4. Get Application URL

After deployment completes:

```bash
# Get the load balancer URL
terraform output application_url
```

### 5. Test Deployed Application

```bash
# Test the service
curl http://YOUR_ALB_DNS_NAME

# Expected response:
# {
#   "timestamp": "2025-07-13T10:30:45Z",
#   "ip": "203.0.113.1"
# }
```

## Cleanup

**Important**: Clean up resources to avoid charges!

```bash
cd terraform
terraform destroy
```

Type `yes` when prompted to confirm deletion.

## Configuration Details

### Application Configuration

- **Port**: 5000
- **Health Check**: `/health` endpoint
- **Production Server**: Gunicorn with 2 workers
- **Security**: Runs as non-root user
- **Logging**: CloudWatch integration

### Infrastructure Configuration

- **ECS Cluster**: Fargate launch type
- **Task Resources**: 256 CPU, 512 MB memory
- **Service**: 2 running tasks for high availability
- **Load Balancer**: Application Load Balancer (ALB)
- **Health Checks**: 30-second intervals
- **Logging**: CloudWatch logs with 7-day retention

## Security Features

- **Non-root container**: Application runs as `appuser`
- **Private subnets**: ECS tasks isolated from internet
- **Security groups**: Minimal required access
- **NAT gateways**: Secure outbound internet access
- **No hardcoded secrets**: Uses AWS IAM roles

## Monitoring

- **CloudWatch Logs**: `/ecs/simpletimeservice` log group
- **Health Checks**: Load balancer monitors task health
- **ECS Service**: Auto-restarts failed tasks

## Troubleshooting

### Common Issues

1. **Docker build fails**:
   ```bash
   # Check Dockerfile syntax
   docker build -t test .
   ```

2. **Terraform fails**:
   ```bash
   # Check AWS credentials
   aws sts get-caller-identity
   
   # Check terraform syntax
   terraform validate
   ```

3. **Service not accessible**:
   ```bash
   # Check ECS service status
   aws ecs describe-services --cluster simpletimeservice-cluster --services simpletimeservice-service
   ```

4. **Tasks failing**:
   ```bash
   # Check CloudWatch logs
   aws logs describe-log-groups --log-group-name-prefix "/ecs/simpletimeservice"
   ```

### Useful Commands

```bash
# Check ECS cluster status
aws ecs list-clusters

# Check running tasks
aws ecs list-tasks --cluster simpletimeservice-cluster

# View task logs
aws logs tail /ecs/simpletimeservice --follow

# Check load balancer health
aws elbv2 describe-target-health --target-group-arn <target-group-arn>
```

## Cost Estimation

Estimated monthly costs (us-east-1):
- **ECS Fargate**: ~$15-20 (2 tasks, 256 CPU, 512 MB)
- **Application Load Balancer**: ~$16-20
- **NAT Gateways**: ~$45-50 (2 gateways)
- **Data Transfer**: ~$5-10
- **CloudWatch Logs**: ~$1-2

**Total**: ~$80-100/month

## Next Steps / Improvements

- **HTTPS**: Add SSL certificate and Route 53 domain
- **Auto Scaling**: Configure ECS service auto scaling
- **CI/CD**: GitHub Actions for automated deployments
- **Monitoring**: CloudWatch alarms and dashboards
- **Security**: WAF, VPC endpoints, secrets management
- **Multi-region**: Deploy across multiple AWS regions

## Author

Created for Particle41 DevOps Challenge - [Your Name]

## License

This project is for assessment purposes only.
