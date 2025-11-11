# Requirements Document

## Introduction

This document defines the requirements for implementing a GitHub Actions CI/CD workflow for the retail store sample application. The workflow will automate the process of detecting code changes in microservices, building and pushing Docker images to Amazon ECR (Elastic Container Registry), and updating Helm chart values with new image tags to enable GitOps-based deployments via ArgoCD.

## Glossary

- **GitHub Actions Workflow**: An automated CI/CD pipeline defined in YAML that executes on specific GitHub events
- **ECR (Elastic Container Registry)**: AWS-managed Docker container registry for storing and managing container images
- **Helm Chart**: A package manager for Kubernetes that uses YAML templates to define application deployments
- **ArgoCD**: A GitOps continuous delivery tool for Kubernetes that automatically syncs applications based on Git repository state
- **Microservice**: An independent application component (ui, catalog, cart, checkout, orders) with its own codebase and Dockerfile
- **Image Tag**: A version identifier for Docker images, using Git commit hash (7 characters) for traceability
- **Change Detection**: Logic to identify which microservices have modified source code to optimize build processes
- **values.yaml**: Helm chart configuration file containing deployment parameters including container image repository and tag

## Requirements

### Requirement 1: Change Detection

**User Story:** As a developer, I want the CI/CD pipeline to automatically detect which microservices have changed, so that only modified services are rebuilt and deployed.

#### Acceptance Criteria

1. WHEN a commit is pushed to the gitops branch with changes in the src/ directory, THE GitHub Actions Workflow SHALL identify all modified microservices based on file paths
2. WHEN changes are detected in src/ui/ directory, THE GitHub Actions Workflow SHALL mark the ui service for building
3. WHEN changes are detected in src/catalog/ directory, THE GitHub Actions Workflow SHALL mark the catalog service for building
4. WHEN changes are detected in src/cart/ directory, THE GitHub Actions Workflow SHALL mark the cart service for building
5. WHEN changes are detected in src/checkout/ directory, THE GitHub Actions Workflow SHALL mark the checkout service for building
6. WHEN changes are detected in src/orders/ directory, THE GitHub Actions Workflow SHALL mark the orders service for building
7. WHEN the workflow is triggered manually via workflow_dispatch, THE GitHub Actions Workflow SHALL build all five microservices regardless of changes

### Requirement 2: Docker Image Building

**User Story:** As a DevOps engineer, I want Docker images to be built automatically for changed microservices, so that the latest code is containerized and ready for deployment.

#### Acceptance Criteria

1. WHEN a microservice is marked for building, THE GitHub Actions Workflow SHALL execute docker build using the Dockerfile in the service's root directory
2. WHEN building a Docker image, THE GitHub Actions Workflow SHALL tag the image with the AWS account ID, region, repository name, and 7-character Git commit hash
3. WHEN building multiple services, THE GitHub Actions Workflow SHALL build images in parallel to reduce total pipeline execution time
4. IF a Docker build fails for any service, THEN THE GitHub Actions Workflow SHALL fail the entire workflow and report the error
5. THE GitHub Actions Workflow SHALL authenticate with AWS ECR before building images

### Requirement 3: ECR Image Push

**User Story:** As a DevOps engineer, I want built Docker images to be automatically pushed to Amazon ECR, so that they are available for Kubernetes deployments.

#### Acceptance Criteria

1. WHEN a Docker image build completes successfully, THE GitHub Actions Workflow SHALL push the image to the corresponding ECR repository
2. WHEN pushing to ECR, THE GitHub Actions Workflow SHALL create the ECR repository if it does not already exist
3. WHEN authenticating with ECR, THE GitHub Actions Workflow SHALL use AWS credentials stored as GitHub secrets (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, AWS_ACCOUNT_ID)
4. IF an ECR push fails for any service, THEN THE GitHub Actions Workflow SHALL fail the entire workflow and report the error
5. THE GitHub Actions Workflow SHALL use the image tag format: {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com/{SERVICE_NAME}:{COMMIT_HASH}

### Requirement 4: Helm Chart Updates

**User Story:** As a DevOps engineer, I want Helm chart values to be automatically updated with new image tags, so that ArgoCD can detect and deploy the latest versions.

#### Acceptance Criteria

1. WHEN a Docker image is successfully pushed to ECR, THE GitHub Actions Workflow SHALL update the corresponding Helm chart values.yaml file with the new image repository and tag
2. WHEN updating values.yaml, THE GitHub Actions Workflow SHALL modify only the image.repository and image.tag fields under the main image section
3. WHEN updating values.yaml, THE GitHub Actions Workflow SHALL preserve all infrastructure image configurations (mysql, redis, postgresql, rabbitmq, dynamodb-local)
4. WHEN updating values.yaml, THE GitHub Actions Workflow SHALL preserve all other configuration values including resources, ingress, and service settings
5. THE GitHub Actions Workflow SHALL commit the updated values.yaml files back to the gitops branch with a descriptive commit message including the service name and commit hash
6. THE GitHub Actions Workflow SHALL configure Git user identity before committing changes

### Requirement 5: Workflow Triggers and Configuration

**User Story:** As a developer, I want the CI/CD pipeline to trigger automatically on code changes and support manual execution, so that deployments are both automated and controllable.

#### Acceptance Criteria

1. THE GitHub Actions Workflow SHALL trigger automatically when commits are pushed to the gitops branch with changes in the src/** path pattern
2. THE GitHub Actions Workflow SHALL support manual triggering via workflow_dispatch event
3. THE GitHub Actions Workflow SHALL execute on GitHub-hosted ubuntu-latest runners
4. THE GitHub Actions Workflow SHALL require four GitHub secrets: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, and AWS_ACCOUNT_ID
5. THE GitHub Actions Workflow SHALL be defined in the .github/workflows/ directory with a descriptive filename

### Requirement 6: Error Handling and Logging

**User Story:** As a DevOps engineer, I want clear error messages and logs from the CI/CD pipeline, so that I can quickly diagnose and fix deployment issues.

#### Acceptance Criteria

1. WHEN any step in the workflow fails, THE GitHub Actions Workflow SHALL output detailed error messages to the workflow logs
2. WHEN building Docker images, THE GitHub Actions Workflow SHALL display build progress and layer information
3. WHEN pushing to ECR, THE GitHub Actions Workflow SHALL log the full image URI for each pushed image
4. WHEN updating Helm charts, THE GitHub Actions Workflow SHALL log which files were modified
5. THE GitHub Actions Workflow SHALL use descriptive step names that clearly indicate the action being performed
