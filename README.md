# Konecta ERP – Cloud-Native Infrastructure

**Graduation Capstone Project | Konecta Internship Program**

Production-ready Kubernetes infrastructure for Konecta ERP on Google Cloud Platform using GKE Autopilot and Terraform, with secure networking, managed secrets, and integrated CI/CD for multi-environment deployments.

## Overview

This repository contains the complete cloud-native infrastructure and deployment automation for Konecta ERP, a microservices-based enterprise resource planning system. The solution leverages Google Kubernetes Engine (GKE) Autopilot for managed Kubernetes operations, Terraform for infrastructure-as-code provisioning, and integrated CI/CD pipelines for continuous deployment across development, staging, and production environments.

The architecture is designed for production workloads, emphasizing security, observability, scalability, and operational resilience through industry-standard cloud-native practices and Google Cloud Platform managed services.

## Architecture Overview

The infrastructure adopts a layered, service-oriented architecture deployed across a managed GKE Autopilot cluster in Google Cloud Platform.

**Platform Layer:**
- GKE Autopilot Cluster: Fully managed Kubernetes with automatic node provisioning, scaling, and security updates
- Global HTTPS Load Balancer: TLS-terminated external ingress with geographic load distribution
- Artifact Registry: Private container image repository with built-in vulnerability scanning
- Cloud NAT: Secure outbound internet access for private cluster nodes
- Secret Manager: Centralized, encrypted credential and configuration management

**Application Layer:**
- Frontend: Angular single-page application served via Nginx reverse proxy
- API Gateway: Spring Boot-based API gateway handling routing, aggregation, and cross-cutting concerns
- Microservices: Independent, domain-focused services implementing Authentication, Human Resources, Inventory, Finance, User Management, and Reporting domains
- Service Discovery: Consul-based service registry and configuration management

**Data Layer:**
- SQL Server: Primary relational database deployed as a Kubernetes StatefulSet
- RabbitMQ: Message broker for asynchronous inter-service communication, also deployed as a StatefulSet
- MailHog: Development and staging email testing tool

## Components

### Infrastructure (Terraform)

The infrastructure-as-code foundation includes:

- **GKE Autopilot Cluster**: Managed Kubernetes cluster with auto-scaling node pools, automatic security patching, and workload identity integration
- **VPC Networking**: Custom VPC with dedicated subnets for nodes and pods, isolated from public internet
- **Artifact Registry**: Private Docker registry for containerized services with image scanning and vulnerability detection
- **Cloud NAT**: Managed NAT gateway enabling private nodes to initiate outbound internet connections
- **Secret Manager**: Encrypted credential storage for database passwords, API keys, and service account tokens
- **Global Load Balancer**: HTTPS-terminated external load balancer with DNS failover and geographic routing

### Application Services

- **Frontend Service**: Angular single-page application containerized with Nginx
- **API Gateway**: Spring Boot service providing centralized API routing, request aggregation, and protocol translation
- **Authentication Service**: .NET Core microservice managing identity, authorization, JWT token issuance, and access control
- **Human Resources Service**: .NET Core microservice for employee management, organizational hierarchy, and payroll integration
- **Inventory Service**: .NET Core microservice handling warehouse management, stock tracking, and SKU operations
- **Finance Service**: .NET Core microservice for accounting, ledger management, and financial reporting
- **User Management Service**: .NET Core microservice for user account provisioning and role-based access control
- **Reporting Service**: Spring Boot service aggregating metrics and generating business analytics

### Infrastructure Services

- **Consul**: Service discovery, health checking, and distributed configuration management
- **SQL Server**: Production relational database with persistent volume claims and point-in-time recovery
- **RabbitMQ**: Message broker providing asynchronous communication patterns between microservices
- **MailHog**: Email capture service for development and staging environments (non-production only)

## Deployment Environments

The platform defines three distinct deployment environments with tailored configurations:

| Environment | Replicas | CPU Allocation | Memory Allocation | Domain | Purpose |
|-------------|----------|---|---|---|---|
| Development | 1 | 100m | 256Mi | dev.konecta.local | Active development and unit testing |
| Staging | 2 | 250m | 512Mi | staging.konecta.com | Pre-production validation and QA testing |
| Production | 3–20 | 500m–1000m | 1Gi–2Gi | konecta.com | Live production workloads |

Each environment maintains isolated configuration, secrets, and resource profiles through Kubernetes overlays and namespace separation.

## Scaling and Resource Management

### Horizontal Pod Autoscaling (HPA)

Services are configured with Horizontal Pod Autoscaler to dynamically adjust replica counts based on CPU and memory utilization:

- **Development**: Minimum 1 replica, maximum 10 replicas
- **Staging**: Minimum 2 replicas, maximum 10 replicas
- **Production**: Minimum 3 replicas, maximum 20 replicas

Scaling thresholds are configured to maintain service availability while optimizing resource utilization and cost.

### Vertical Pod Autoscaling (VPA)

Vertical Pod Autoscaler is available for recommendation-based right-sizing of resource requests and limits per workload. This feature is optional and can be enabled per deployment based on workload profiling data.

### Resource Requests and Limits

Resource allocation is tuned per environment to balance cost efficiency with performance requirements:

**Development Environment:**
requests:
  memory: 256Mi
  cpu: 100m
limits:
  memory: 512Mi
  cpu: 250m

**Production Environment:**
requests:
  memory: 1Gi
  cpu: 500m
limits:
  memory: 2Gi
  cpu: 1000m

## Monitoring, Logging, and Observability

### Metrics Collection

- **GKE Monitoring**: Automatic collection of node and pod metrics, integrated with Google Cloud Monitoring
- **Prometheus**: Metrics scraping through GKE-managed Prometheus for custom application metrics
- **Cloud Logging**: Centralized log aggregation with structured JSON logging and full-text search capabilities

### Log Access

Application logs are accessible through multiple interfaces:

kubectl logs -l app=api-gateway --tail=100 -f

Query logs by service label, deployment, or pod name for targeted troubleshooting.

### Performance Metrics

Monitor cluster and workload health using kubectl tooling:

kubectl top nodes     # Node CPU and memory utilization
kubectl top pods      # Pod resource consumption
kubectl get hpa       # Horizontal Pod Autoscaler status and scaling decisions

## Security Architecture

### Network Security

- **Private GKE Nodes**: All cluster nodes run in private subnets with no direct public IP addresses
- **Network Policies**: Kubernetes network policies enforce pod-to-pod communication boundaries
- **Cloud NAT**: Controlled outbound connectivity through managed NAT gateway, preventing unauthorized egress
- **TLS/SSL Termination**: HTTPS encryption at load balancer and optional inter-service mutual TLS via Consul Connect

### Authentication and Authorization

- **Workload Identity**: Services authenticate to Google Cloud APIs using short-lived, scoped service account tokens
- **Kubernetes RBAC**: Role-based access control limits administrative and operational permissions
- **JWT Authentication**: APIs authenticate requests using signed JWT tokens with role-based claims
- **Service-to-Service mTLS**: Optional mutual TLS for authenticated, encrypted microservice communication

### Secrets and Credential Management

- **Kubernetes Secrets**: Native Kubernetes secret objects with base64 encoding for pod environment variables
- **Google Secret Manager**: External secret storage with encryption at rest, audit logging, and version control
- **Sealed Secrets**: Optional encryption of secrets in version control using asymmetric key pairs

## Directory Structure

infrastructure/
├── kubernetes/
│   ├── base/
│   │   ├── sqlserver/              # SQL Server StatefulSet and service definitions
│   │   ├── rabbitmq/               # RabbitMQ broker StatefulSet configuration
│   │   ├── config-server/          # Spring Cloud Config server for centralized configuration
│   │   ├── api-gateway/            # API Gateway deployment and routing rules
│   │   ├── auth-service/           # Authentication service deployment
│   │   ├── hr-service/             # Human Resources service deployment
│   │   ├── inventory-service/      # Inventory management service deployment
│   │   ├── finance-service/        # Finance and accounting service deployment
│   │   ├── user-service/           # User management service deployment
│   │   ├── reporting-service/      # Business analytics and reporting service
│   │   ├── frontend/               # Angular frontend deployment and ingress
│   │   └── networking/             # Network policies, ingress controllers, TLS certificates
│   └── overlays/
│       ├── dev/                    # Development environment overrides (1 replica, minimal resources)
│       ├── staging/                # Staging environment overrides (2 replicas, medium resources)
│       └── prod/                   # Production environment overrides (3+ replicas, optimized resources)
├── helm/
│   └── consul/
│       └── custom-values.yaml      # Consul Helm chart configuration values
├── terraform/
│   ├── gke/                        # GKE cluster definition and configuration
│   ├── artifact-registry/          # Artifact Registry repository provisioning
│   ├── networking/                 # VPC, subnets, firewall rules, load balancer configuration
│   ├── secrets/                    # Secret Manager resources and KMS keys
│   └── main.tf                     # Root Terraform module
└── ci-cd/
    ├── cloudbuild.yaml             # Google Cloud Build pipeline definition
    └── github-actions/
        ├── build.yml               # GitHub Actions build workflow
        └── deploy.yml              # GitHub Actions deployment workflow

## Continuous Integration and Continuous Deployment

### Automated Pipelines

**Google Cloud Build:**
- Triggered automatically on git push to remote repository
- Builds all Docker images in parallel to minimize build time
- Scans container images for known vulnerabilities before pushing to registry
- Pushes validated images to Artifact Registry with semantic versioning tags
- Deploys to appropriate environment based on branch (dev, staging, main/prod)

**GitHub Actions:**
- Runs build workflow on pull requests for code validation before merge
- Executes deployment workflow on merge to main branch for production releases
- Maintains environment-specific deployment configurations and secrets

### Manual Deployment

For single-service or emergency deployments:

docker build -t us-central1-docker.pkg.dev/PROJECT/konecta-erp/api-gateway:v1.0.0 \
  -f konecta_erp/backend/ApiGateWay/Dockerfile \
  konecta_erp/backend/ApiGateWay

docker push us-central1-docker.pkg.dev/PROJECT/konecta-erp/api-gateway:v1.0.0

kubectl set image deployment/api-gateway \
  api-gateway=us-central1-docker.pkg.dev/PROJECT/konecta-erp/api-gateway:v1.0.0

## Getting Started

### Prerequisites

Install the following tools on your local development machine:

- Google Cloud SDK: `curl https://sdk.cloud.google.com | bash`
- kubectl: `gcloud components install kubectl`
- Terraform: `brew install terraform` (macOS) or download from terraform.io
- Helm: `brew install helm` (macOS) or download from helm.sh

### Initial Setup

Clone the repository and configure GCP project access:

git clone https://github.com/your-org/konecta-erp-infrastructure.git
cd konecta-erp-infrastructure

gcloud auth login
gcloud config set project YOUR_GCP_PROJECT_ID

### Provision Infrastructure

Deploy the GKE cluster and supporting resources:

cd terraform/gke
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your specific configuration
terraform init
terraform plan
terraform apply

cd ../artifact-registry
terraform init
terraform apply

### Deploy Applications

Retrieve cluster credentials and deploy services:

gcloud container clusters get-credentials konecta-erp-cluster --region us-central1

helm repo add hashicorp https://helm.releases.hashicorp.com
helm install consul hashicorp/consul -f helm/consul/custom-values.yaml

kubectl apply -k kubernetes/overlays/dev/

# Verify deployment status
kubectl get pods -A
kubectl get ingress -A

## Troubleshooting and Support

Detailed troubleshooting guides and common solutions are available in dedicated documentation:

- [Kubernetes Deployment Guide](kubernetes/README.md) – Cluster setup, pod scheduling, and resource management issues
- [CI/CD Pipeline Guide](ci-cd/README.md) – Build failures, deployment errors, and pipeline configuration
- [Terraform Modules Documentation](terraform/) – Infrastructure provisioning and GCP resource configuration

## Contributing

Follow this workflow for contributions:

1. Create a feature branch from the main development branch
2. Implement infrastructure changes or updates
3. Test changes in the development environment to verify functionality
4. Commit changes with descriptive messages documenting modifications
5. Create a pull request for code review and validation
6. Deploy validated changes to staging environment for integration testing
7. Merge to main branch upon approval for production deployment

## License

Copyright © 2024 Konecta ERP. All rights reserved.
