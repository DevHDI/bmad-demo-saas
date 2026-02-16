---
status: done
completedAt: "2026-01-22"
epic: "Epic 1: Platform Foundation"
storyPoints: 8
---

# Story 1-1: Platform Initialization

## Description

Establish the foundational infrastructure and development environment for CloudMetrics. This story encompasses the complete AWS infrastructure setup using Terraform, Next.js 14 application scaffolding with App Router, and CI/CD pipeline configuration. The infrastructure follows a multi-environment strategy (dev, staging, production) with isolated AWS accounts for security and compliance.

The platform initialization is critical as it sets the architectural patterns for the entire system. We're using AWS ECS Fargate for container orchestration to avoid EC2 management overhead, RDS PostgreSQL 16 with TimescaleDB extension for time-series data, and ElastiCache Redis for caching and session management. The infrastructure-as-code approach with Terraform ensures reproducibility and version control for all infrastructure changes.

This story also establishes development workflows including Git branching strategy, code review processes, and deployment pipelines. The Next.js 14 App Router provides server components for optimal performance and SEO, which is essential for the marketing site and dashboard initial load times.

## Acceptance Criteria

- AC-1: Terraform modules created for VPC, ECS cluster, RDS, ElastiCache, and CloudWatch with proper tagging strategy
- AC-2: Next.js 14 application initialized with TypeScript, ESLint, Prettier, and Tailwind CSS v4 configuration
- AC-3: Development environment runs locally with Docker Compose (PostgreSQL, Redis, Kafka)
- AC-4: GitHub Actions CI/CD pipeline builds, tests, and deploys to dev environment on merge to main
- AC-5: AWS environments (dev, staging, production) provisioned with identical Terraform configurations
- AC-6: CloudWatch dashboards created for infrastructure monitoring (CPU, memory, database connections, cache hit rate)
- AC-7: Application health check endpoints implemented and integrated with ECS health checks
- AC-8: Environment variable management configured using AWS Systems Manager Parameter Store
- AC-9: Database migration framework (node-pg-migrate) configured with rollback capabilities
- AC-10: Documentation updated with setup instructions, architecture diagrams, and deployment runbook

## Technical Notes

**Infrastructure Stack**:
- AWS VPC with public/private subnets across 3 availability zones
- ECS Fargate with Application Load Balancer (ALB) for container orchestration
- RDS PostgreSQL 16 Multi-AZ with automated backups (7-day retention)
- ElastiCache Redis 7 cluster mode (3 shards for high availability)
- S3 buckets for Terraform state (with versioning and encryption)
- CloudFront for static asset delivery
- Route 53 for DNS management

**Next.js Configuration**:
- App Router with server components for dashboard pages
- Tailwind CSS v4 with `@theme inline` in globals.css (no tailwind.config file)
- TypeScript strict mode enabled
- Absolute imports configured (@/components, @/lib, @/app)
- Environment-specific .env files (.env.local, .env.development, .env.production)

**Security Considerations**:
- All traffic encrypted with TLS 1.3
- Database credentials stored in AWS Secrets Manager
- IAM roles with least-privilege policies for ECS tasks
- Security groups restrict traffic to necessary ports only
- VPC Flow Logs enabled for network monitoring

**Edge Cases**:
- Terraform state locking with DynamoDB to prevent concurrent modifications
- ECS task definition rollback strategy if deployment fails health checks
- Database connection pooling with PgBouncer to handle connection limits
- Redis failover testing to ensure automatic promotion of replicas

## Tasks

- [x] Set up AWS organization with separate accounts for dev, staging, production
- [x] Create Terraform modules for VPC, networking, and security groups
- [x] Provision RDS PostgreSQL 16 with TimescaleDB extension enabled
- [x] Configure ElastiCache Redis cluster with cluster mode enabled
- [x] Set up ECS cluster with Fargate launch type and auto-scaling policies
- [x] Initialize Next.js 14 application with TypeScript and Tailwind CSS v4
- [x] Configure Docker Compose for local development environment
- [x] Create GitHub Actions workflow for CI/CD (build, test, deploy)
- [x] Implement database migration framework with seed data
- [x] Set up CloudWatch dashboards and alarms for infrastructure monitoring
- [x] Configure environment variable management with AWS Parameter Store
- [x] Create application health check endpoints (/health, /ready)
- [x] Document deployment process and runbook in README.md
- [x] Test deployment pipeline with sample application
- [x] Conduct infrastructure security review with AWS Security Hub

## Dependencies

- None (foundational story)

## Estimation

**Story Points**: 8

**Breakdown**:
- AWS infrastructure setup: 3 points
- Next.js application scaffolding: 2 points
- CI/CD pipeline configuration: 2 points
- Documentation and testing: 1 point
