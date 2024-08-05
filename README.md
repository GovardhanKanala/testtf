# Terraform Deployment with GitHub Actions

This repository contains the infrastructure code for deploying using Terraform from GitHub Actions. It supports multiple environments: dev, beta, stg, and prod. This document outlines the branching strategy and the GitHub Actions workflow used to deploy changes to these environments.

## Branching Strategy

### Main Branches

- **main**: Reflects the production environment (prod).
- **develop**: Reflects the development environment (dev).

### Environment Branches

- **beta**: Reflects the beta environment.
- **staging**: Reflects the staging environment (stg).

### Feature Branches

- **feature/feature-name**: Used for developing new features or making changes.

## Workflow for Changes and Pull Requests

### Developing New Features

1. Create a new feature branch from the **develop** branch.
2. Make changes and commit them to the feature branch.
3. Open a pull request (PR) from the feature branch to the **develop** branch for review and testing.

### Testing in Development

1. Merge the feature branch into the **develop** branch after PR approval.
2. This triggers a GitHub Action to apply the Terraform changes to the dev environment.

### Promoting to Beta

1. After testing in the dev environment, create a PR from the **develop** branch to the **beta** branch.
2. Merge the PR to deploy the changes to the beta environment.

### Promoting to Staging

1. After testing in the beta environment, create a PR from the **beta** branch to the **staging** branch.
2. Merge the PR to deploy the changes to the staging environment.

### Promoting to Production

1. After testing in the staging environment, create a PR from the **staging** branch to the **main** branch.
2. Merge the PR to deploy the changes to the production environment.

## GitHub Actions Workflow

### Setup Terraform

Use the `hashicorp/setup-terraform` action to set up Terraform.

### Validate and Plan

Use `terraform init`, `terraform validate`, and `terraform plan` for each environment to ensure the code is correct and changes are reviewed before applying.

### Apply Changes

Use `terraform apply` to apply the changes to the respective environment after the PR is merged.

Here’s a sample GitHub Actions workflow for the **develop** branch:

```yaml
name: Terraform Deploy

on:
  push:
    branches:
      - develop
      - beta
      - staging
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.1.3

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan

      - name: Terraform Apply
        if: github.ref == 'refs/heads/develop' || github.ref == 'refs/heads/beta' || github.ref == 'refs/heads/staging' || github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve
        env:
          TF_VAR_environment: ${{ github.ref_name }}
          # Add other environment variables here

