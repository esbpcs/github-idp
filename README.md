# GitHub Actions IDP

This repository provides a comprehensive and secure framework for managing multi-account AWS environments using GitHub Actions and OpenID Connect (OIDC). It enables automated, secure, and repeatable deployments of applications across various environments such as development, staging, and production using **AWS CloudFormation**.

-----

## 🏛️ Architecture

1. **GitHub Actions Workflows**: These are the entry points for all operations, including onboarding, deployment, and decommissioning.
2. **Tooling AWS Account**: This is a central AWS account that establishes the trust relationship with your GitHub organization.
      * **GitHub OIDC Provider**: A one-time setup that allows AWS to trust GitHub as an identity provider.
      * **GitHubActionsToolingRole**: A central IAM role that GitHub Actions assumes to interact with your AWS environments.
3. **Target AWS Account(s)**: These are the AWS accounts where your applications will be deployed (e.g., development, staging, production).
      * **ManagementConnectorRole**: A role with elevated permissions used for initial environment setup.
      * **InternalToolingDeploymentRole**: A least-privilege role created for each application, with just enough permissions to deploy and manage that application.
      * **Application Resources**: The actual resources that make up your application.

-----

## 🚀 Getting Started

### Prerequisites

* An AWS account to serve as the central **Tooling Account**.
* One or more AWS accounts to serve as **Target Accounts**.
* A GitHub organization and a repository to house your workflows and CloudFormation templates.
* The AWS CLI installed and configured locally for the initial setup.

### Setup

1. **Tooling Account Setup**:

      * Deploy the `tooling.yaml` CloudFormation stack in your **Tooling Account**.
      * Take note of the `GitHubActionsToolingRoleArn` output.

2. **Target Account Setup**:

      * In each **Target Account**, deploy the `management-connector.yaml` CloudFormation stack.
      * Take note of the `ManagementConnectorRoleArn` output.

3. **GitHub Secrets Configuration**:

      * In your GitHub repository, configure the following secrets:
          * `TOOLING_ACCOUNT_OIDC_ROLE_ARN`: The ARN of the `GitHubActionsToolingRole` from the tooling account setup.
          * `MANAGEMENT_CONNECTOR_ROLE_ARN`: The ARN of the `ManagementConnectorRole` from the target account setup.

-----

## ⚙️ Workflows

### Onboard New Environment

* **File**: `.github/workflows/onboard-environment.yml`
* **Description**: Provisions a new environment by deploying the `client.yaml` stack, creating a dedicated, least-privilege IAM role for the application. The ARN of this new role is then stored in SSM Parameter Store.
* **Inputs**: `company_name`, `environment_name`, `application_repo`, `aws_region`

### Deploy Application

* **File**: `.github/workflows/main.yml`
* **Description**: Deploys your application by retrieving the appropriate deployment role ARN from SSM, assuming that role, and then deploying the application's CloudFormation stack.
* **Inputs**: `company_name`, `environment_name`, `application_repo`

### Decommission Environment

* **File**: `.github/workflows/decomission-environment.yml`
* **Description**: Tears down an application environment by deleting the application stack, the client IAM role stack, and the associated SSM parameter.
* **Inputs**: `company_name`, `environment_name`, `application_repo`, `aws_region`

### Debug OIDC

* **File**: `.github/workflows/debug-oidc.yml`
* **Description**: A utility workflow to help diagnose issues with the OIDC configuration. It assumes the management connector role and prints the AWS caller identity to verify that authentication is working correctly.
* **Inputs**: `target_environment`, `aws_region`

-----

## 🔒 Security

* **Least Privilege**: Each application gets its own IAM role with only the permissions it needs.
* **Separation of Concerns**: A high-privilege IAM role is used for environment setup, but a separate, low-privilege role is used for day-to-day deployments.
* **OIDC Authentication**: Avoids the need to store long-lived AWS credentials as GitHub secrets.
* **Runner Hardening**: The deployment workflow uses `step-security/harden-runner` to block all outbound traffic from the GitHub Actions runner, except to a few essential endpoints.

-----

## ⚖️ License

This project is licensed under the MIT License. See the `LICENSE` file for details.
