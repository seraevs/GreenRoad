## Azure DevOps CI/CD for EC2 Provisioning with Kubernetes
## This project demonstrates an automatic CI/CD workflow using Azure DevOps for provisioning an EC2 instance and deploying containerized services with Kubernetes. 
## The solution leverages Terraform for infrastructure provisioning and Kubernetes for orchestrating containerized web and monitoring services.

## Overview
The CI/CD pipeline automates the following tasks:

    Provisioning an EC2 instance using Terraform.
    Deploying an internet-facing containerized web site.
    Deploying a containerized monitoring solution to monitor the web site using Kubernetes.

    Path to the scripts:
        azure-pipelines.yml
        terraform/web-site-deployment.yaml
        terraform/monitoring-solution-deployment.yaml

## Table of Contents
    Prerequisites
    Azure DevOps Pipeline Configuration
    Terraform Configuration
    Kubernetes Configuration
    Project Structure
    Kubernetes Manifests
    Azure DevOps Pipeline YAML
    Usage
    Contributing
    License
## Prerequisites
    Before using this solution, i installed following prerequisites:

        Azure DevOps account.
        Azure subscription.
        Azure Service Connection in Azure DevOps for your Azure Subscription.
        Terraform installed on machine or included in the pipeline.
        Kubernetes configured for deploying services.

## Azure DevOps Pipeline Configuration
    ## Terraform Configuration
        Created a Terraform script for provisioning an EC2 instance.
        Configured Terraform backend settings for state storage.

## Kubernetes Configuration
    Created Kubernetes manifests for the web site and monitoring solution.
    Ensured Kubernetes is configured for deploying services.

## Project Structure
    The project follows a structured layout. Key directories include:

    azure-pipelines.yml:

    trigger:
        - main

        pool:
        vmImage: 'ubuntu-latest'

        variables:
        terraformPath: '$(System.DefaultWorkingDirectory)/terraform'
        terraformBackendStorageAccount: 'seryev345'
        terraformBackendContainer: 'tfstate'
        terraformBackendKey: 'terraform.tfstate'

        stages:
        - stage: TerraformProvisioning
        jobs:
        - job: ProvisionEC2
            displayName: 'Provision EC2 with Terraform'
            steps:
            - task: UseDotNet@2
            displayName: 'Use .NET Core sdk'
            inputs:
                packageType: 'sdk'
                version: '3.1.x'

            - checkout: self

            - script: |
                cd $(terraformPath)
                terraform init -backend-config="storage_account_name=$(terraformBackendStorageAccount)" -backend-config="container_name=$(terraformBackendContainer)" -backend-config="key=$(terraformBackendKey)"
                terraform apply -auto-approve
            displayName: 'Terraform Apply'

        - stage: KubernetesDeployment
        dependsOn: TerraformProvisioning
        jobs:
        - job: DeployServices
            displayName: 'Deploy Containerized Services with Kubernetes'
            steps:
            - task: UseDotNet@2
            displayName: 'Use .NET Core sdk'
            inputs:
                packageType: 'sdk'
                version: '3.1.x'

            - checkout: self

            - script: |
                # Use kubectl to apply Kubernetes manifests for your web site and monitoring solution
                # Update the paths based on your project structure
                kubectl apply -f terraform/web-site-deployment.yaml
                kubectl apply -f terraform/monitoring-solution-deployment.yaml
            displayName: 'Deploy Services with Kubernetes'

            # Add additional steps for testing, monitoring, etc.

        - stage: TerraformDestroy
        dependsOn: KubernetesDeployment
        jobs:
        - job: DestroyEC2
            displayName: 'Destroy EC2 with Terraform'
            steps:
            - task: UseDotNet@2
            displayName: 'Use .NET Core sdk'
            inputs:
                packageType: 'sdk'
                version: '3.1.x'

            - checkout: self

            - script: |
                cd $(terraformPath)
                terraform destroy -auto-approve
            displayName: 'Terraform Destroy'

    terraform/: Contains Terraform scripts.
    kubernetes/: Contains Kubernetes manifest files.
    Kubernetes Manifests
        web-site-deployment.yaml:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
        name: web-site
        spec:
        replicas: 3
        selector:
            matchLabels:
            app: web-site
        template:
            metadata:
            labels:
                app: web-site
            spec:
            containers:
            - name: web-site-container
                image: my-web-site-image:latest
                ports:
                - containerPort: 80

        monitoring-solution-deployment.yaml:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
        name: monitoring-solution
        spec:
        replicas: 1
        selector:
            matchLabels:
            app: monitoring-solution
        template:
            metadata:
            labels:
                app: monitoring-solution
            spec:
            containers:
            - name: monitoring-solution-container
                image: my-monitoring-solution-image:latest
                ports:
                - containerPort: 8080

## Azure DevOps Pipeline YAML
    Created Azure DevOps Pipeline YAML section above for a complete YAML template. 
    Customize it based on your project structure and requirements.

## Usage
    Fork or clone the repository.
    Configure Azure DevOps with your project.
    Update Terraform and Kubernetes configurations.
    Run the CI/CD pipeline in Azure DevOps.
    Contributing
    Contributions are welcome! Follow the guidelines in the Contributing file.

## License
    This project is licensed under the MIT License. 
    Feel free to use, modify, and distribute as per the terms of the license.