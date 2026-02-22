# Terraform Infrastructure Guide

Complete guide to Terraform infrastructure as code, modules, and cloud provisioning for ResonantGenesis.

## Overview

ResonantGenesis provides official Terraform modules for deploying infrastructure on AWS, GCP, and Azure. This guide covers Terraform setup, module usage, and best practices for infrastructure as code.

## Prerequisites

| Requirement | Version |
|-------------|---------|
| Terraform | 1.5+ |
| AWS CLI | 2.0+ (for AWS) |
| gcloud CLI | Latest (for GCP) |
| Azure CLI | Latest (for Azure) |

## Quick Start

### Install Terraform

```bash
# macOS
brew install terraform

# Linux
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/

# Verify
terraform version
```

### Initialize Project

```bash
# Create project directory
mkdir resonant-infra && cd resonant-infra

# Create main.tf
cat > main.tf << 'EOF'
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

module "resonant" {
  source  = "resonantgenesis/resonant/aws"
  version = "1.0.0"
  
  environment = var.environment
  vpc_cidr    = var.vpc_cidr
}
EOF

# Initialize
terraform init
```

## AWS Module

### Basic Usage

```hcl
# main.tf
module "resonant" {
  source  = "resonantgenesis/resonant/aws"
  version = "1.0.0"
  
  # Required
  environment = "production"
  
  # Networking
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
  
  # Compute
  api_instance_type     = "t3.large"
  api_min_instances     = 3
  api_max_instances     = 20
  worker_instance_type  = "c5.xlarge"
  worker_min_instances  = 2
  worker_max_instances  = 50
  
  # Database
  db_instance_class     = "db.r6g.large"
  db_allocated_storage  = 100
  db_multi_az           = true
  
  # Cache
  cache_node_type       = "cache.r6g.large"
  cache_num_nodes       = 3
  
  # Tags
  tags = {
    Project     = "ResonantGenesis"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}
```

### VPC Configuration

```hcl
# vpc.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "resonant-${var.environment}"
  cidr = var.vpc_cidr
  
  azs             = var.availability_zones
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway     = true
  single_nat_gateway     = false
  one_nat_gateway_per_az = true
  
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = var.tags
}
```

### EKS Cluster

```hcl
# eks.tf
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.0.0"
  
  cluster_name    = "resonant-${var.environment}"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  cluster_endpoint_public_access = true
  
  eks_managed_node_groups = {
    api = {
      name           = "api-nodes"
      instance_types = ["t3.large"]
      min_size       = 3
      max_size       = 20
      desired_size   = 3
      
      labels = {
        role = "api"
      }
    }
    
    worker = {
      name           = "worker-nodes"
      instance_types = ["c5.xlarge"]
      min_size       = 2
      max_size       = 50
      desired_size   = 5
      
      labels = {
        role = "worker"
      }
    }
    
    session = {
      name           = "session-nodes"
      instance_types = ["c5.2xlarge"]
      min_size       = 5
      max_size       = 100
      desired_size   = 10
      
      labels = {
        role = "session"
      }
    }
  }
  
  tags = var.tags
}
```

### RDS Database

```hcl
# rds.tf
module "rds" {
  source  = "terraform-aws-modules/rds/aws"
  version = "6.0.0"
  
  identifier = "resonant-${var.environment}"
  
  engine               = "postgres"
  engine_version       = "15.4"
  family               = "postgres15"
  major_engine_version = "15"
  instance_class       = var.db_instance_class
  
  allocated_storage     = var.db_allocated_storage
  max_allocated_storage = var.db_allocated_storage * 2
  
  db_name  = "resonant"
  username = "resonant"
  port     = 5432
  
  multi_az               = var.db_multi_az
  db_subnet_group_name   = module.vpc.database_subnet_group_name
  vpc_security_group_ids = [module.security_group_rds.security_group_id]
  
  maintenance_window      = "Mon:00:00-Mon:03:00"
  backup_window           = "03:00-06:00"
  backup_retention_period = 30
  
  performance_insights_enabled = true
  
  deletion_protection = true
  
  tags = var.tags
}
```

### ElastiCache

```hcl
# elasticache.tf
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id = "resonant-${var.environment}"
  description          = "Redis cluster for ResonantGenesis"
  
  node_type            = var.cache_node_type
  num_cache_clusters   = var.cache_num_nodes
  port                 = 6379
  
  engine               = "redis"
  engine_version       = "7.0"
  parameter_group_name = "default.redis7"
  
  automatic_failover_enabled = true
  multi_az_enabled           = true
  
  subnet_group_name  = aws_elasticache_subnet_group.redis.name
  security_group_ids = [module.security_group_redis.security_group_id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  
  snapshot_retention_limit = 7
  snapshot_window          = "05:00-09:00"
  
  tags = var.tags
}
```

### ALB

```hcl
# alb.tf
module "alb" {
  source  = "terraform-aws-modules/alb/aws"
  version = "8.0.0"
  
  name = "resonant-${var.environment}"
  
  load_balancer_type = "application"
  
  vpc_id          = module.vpc.vpc_id
  subnets         = module.vpc.public_subnets
  security_groups = [module.security_group_alb.security_group_id]
  
  target_groups = [
    {
      name             = "api"
      backend_protocol = "HTTP"
      backend_port     = 8000
      target_type      = "ip"
      
      health_check = {
        enabled             = true
        interval            = 30
        path                = "/health"
        port                = "traffic-port"
        healthy_threshold   = 2
        unhealthy_threshold = 3
        timeout             = 5
        protocol            = "HTTP"
        matcher             = "200"
      }
    }
  ]
  
  https_listeners = [
    {
      port               = 443
      protocol           = "HTTPS"
      certificate_arn    = var.certificate_arn
      target_group_index = 0
    }
  ]
  
  http_tcp_listeners = [
    {
      port        = 80
      protocol    = "HTTP"
      action_type = "redirect"
      redirect = {
        port        = "443"
        protocol    = "HTTPS"
        status_code = "HTTP_301"
      }
    }
  ]
  
  tags = var.tags
}
```

## GCP Module

### Basic Usage

```hcl
# main.tf
module "resonant" {
  source  = "resonantgenesis/resonant/google"
  version = "1.0.0"
  
  project_id  = var.project_id
  region      = var.region
  environment = var.environment
  
  # GKE
  gke_num_nodes = 3
  machine_type  = "e2-standard-4"
  
  # Cloud SQL
  db_tier = "db-custom-4-16384"
  
  # Memorystore
  redis_memory_size_gb = 5
}
```

### GKE Cluster

```hcl
# gke.tf
module "gke" {
  source  = "terraform-google-modules/kubernetes-engine/google"
  version = "27.0.0"
  
  project_id = var.project_id
  name       = "resonant-${var.environment}"
  region     = var.region
  
  network    = module.vpc.network_name
  subnetwork = module.vpc.subnets_names[0]
  
  ip_range_pods     = "pods"
  ip_range_services = "services"
  
  node_pools = [
    {
      name         = "api-pool"
      machine_type = "e2-standard-4"
      min_count    = 3
      max_count    = 20
      auto_upgrade = true
    },
    {
      name         = "worker-pool"
      machine_type = "c2-standard-8"
      min_count    = 2
      max_count    = 50
      auto_upgrade = true
    }
  ]
}
```

### Cloud SQL

```hcl
# cloudsql.tf
module "cloudsql" {
  source  = "GoogleCloudPlatform/sql-db/google//modules/postgresql"
  version = "15.0.0"
  
  project_id       = var.project_id
  name             = "resonant-${var.environment}"
  database_version = "POSTGRES_15"
  region           = var.region
  
  tier = var.db_tier
  
  availability_type = "REGIONAL"
  
  ip_configuration = {
    ipv4_enabled    = false
    private_network = module.vpc.network_self_link
  }
  
  backup_configuration = {
    enabled                        = true
    start_time                     = "03:00"
    point_in_time_recovery_enabled = true
    retained_backups               = 30
  }
  
  database_flags = [
    {
      name  = "max_connections"
      value = "500"
    }
  ]
}
```

## Azure Module

### Basic Usage

```hcl
# main.tf
module "resonant" {
  source  = "resonantgenesis/resonant/azurerm"
  version = "1.0.0"
  
  resource_group_name = var.resource_group_name
  location            = var.location
  environment         = var.environment
  
  # AKS
  aks_node_count = 3
  aks_vm_size    = "Standard_D4s_v3"
  
  # PostgreSQL
  postgresql_sku_name = "GP_Gen5_4"
  
  # Redis
  redis_capacity = 2
  redis_family   = "C"
}
```

### AKS Cluster

```hcl
# aks.tf
module "aks" {
  source  = "Azure/aks/azurerm"
  version = "7.0.0"
  
  resource_group_name = var.resource_group_name
  cluster_name        = "resonant-${var.environment}"
  location            = var.location
  
  kubernetes_version = "1.28"
  
  vnet_subnet_id = module.vnet.vnet_subnets[0]
  
  agents_pool_name = "api"
  agents_size      = "Standard_D4s_v3"
  agents_count     = 3
  agents_max_count = 20
  agents_min_count = 3
  
  enable_auto_scaling = true
  
  network_plugin = "azure"
  network_policy = "calico"
  
  tags = var.tags
}
```

## Variables

### variables.tf

```hcl
# variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}

variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Availability zones"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

variable "api_instance_type" {
  description = "API instance type"
  type        = string
  default     = "t3.large"
}

variable "api_min_instances" {
  description = "Minimum API instances"
  type        = number
  default     = 3
}

variable "api_max_instances" {
  description = "Maximum API instances"
  type        = number
  default     = 20
}

variable "db_instance_class" {
  description = "RDS instance class"
  type        = string
  default     = "db.r6g.large"
}

variable "db_allocated_storage" {
  description = "RDS allocated storage in GB"
  type        = number
  default     = 100
}

variable "db_multi_az" {
  description = "Enable Multi-AZ"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

### terraform.tfvars

```hcl
# terraform.tfvars
environment = "production"
region      = "us-east-1"

vpc_cidr           = "10.0.0.0/16"
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]

api_instance_type    = "t3.large"
api_min_instances    = 3
api_max_instances    = 20

db_instance_class    = "db.r6g.large"
db_allocated_storage = 100
db_multi_az          = true

tags = {
  Project     = "ResonantGenesis"
  Environment = "production"
  ManagedBy   = "Terraform"
  Owner       = "platform-team"
}
```

## Outputs

```hcl
# outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "eks_cluster_endpoint" {
  description = "EKS cluster endpoint"
  value       = module.eks.cluster_endpoint
}

output "rds_endpoint" {
  description = "RDS endpoint"
  value       = module.rds.db_instance_endpoint
}

output "redis_endpoint" {
  description = "Redis endpoint"
  value       = aws_elasticache_replication_group.redis.primary_endpoint_address
}

output "alb_dns_name" {
  description = "ALB DNS name"
  value       = module.alb.lb_dns_name
}
```

## State Management

### S3 Backend

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "resonant-terraform-state"
    key            = "production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "resonant-terraform-locks"
  }
}
```

### GCS Backend

```hcl
# backend.tf
terraform {
  backend "gcs" {
    bucket = "resonant-terraform-state"
    prefix = "production"
  }
}
```

### Azure Backend

```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "resonant-terraform"
    storage_account_name = "resonanttfstate"
    container_name       = "tfstate"
    key                  = "production.tfstate"
  }
}
```

## Workspaces

```bash
# Create workspace
terraform workspace new production
terraform workspace new staging

# List workspaces
terraform workspace list

# Switch workspace
terraform workspace select production

# Use in configuration
locals {
  environment = terraform.workspace
}
```

## Commands

### Basic Commands

```bash
# Initialize
terraform init

# Plan
terraform plan -out=tfplan

# Apply
terraform apply tfplan

# Destroy
terraform destroy

# Format
terraform fmt -recursive

# Validate
terraform validate
```

### Advanced Commands

```bash
# Import existing resource
terraform import aws_instance.example i-1234567890abcdef0

# State management
terraform state list
terraform state show aws_instance.example
terraform state mv aws_instance.old aws_instance.new

# Refresh state
terraform refresh

# Target specific resource
terraform apply -target=module.eks
```

## Best Practices

### For Code

1. **Use modules** - Reusable components
2. **Version pin** - Lock provider versions
3. **Use variables** - Parameterize everything
4. **Format code** - Run `terraform fmt`
5. **Validate** - Run `terraform validate`

### For State

1. **Remote backend** - Never local state in production
2. **State locking** - Prevent concurrent modifications
3. **Encryption** - Encrypt state at rest
4. **Backup** - Regular state backups
5. **Workspaces** - Separate environments

### For Security

1. **No secrets in code** - Use secrets manager
2. **Least privilege** - Minimal IAM permissions
3. **Audit logging** - Track changes
4. **Code review** - Review all changes
5. **Scan for issues** - Use tfsec, checkov

## Troubleshooting

### Common Issues

```bash
# Lock stuck
terraform force-unlock LOCK_ID

# State corruption
terraform state pull > backup.tfstate
terraform state push backup.tfstate

# Provider issues
rm -rf .terraform
terraform init -upgrade
```

### Debug Mode

```bash
# Enable debug logging
export TF_LOG=DEBUG
terraform apply

# Log to file
export TF_LOG_PATH=terraform.log
```

---

**Need Terraform help?** Contact infrastructure@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
