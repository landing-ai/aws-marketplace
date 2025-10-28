# Agentic Document Extraction API - AWS Installation Guide

## Contents

* [Overview](#overview)
* [Requirements](#requirements)
* [Deployment](#deployment)
  * [Deployment Process](#deployment-process)
* [Accessing Your Deployment](#accessing-your-deployment)
* [Configuration](#configuration)
* [Testing the API](#testing-the-api)
* [Updating Your Deployment](#updating-your-deployment)
* [Cleanup](#cleanup)

## Overview

The Agentic Document Extraction API is a cloud-native solution that provides intelligent document analysis capabilities. This guide walks you through deploying the service to your AWS account using CloudFormation and Helm charts on Amazon EKS.

The deployment typically takes about 10-15 minutes to complete, including:
- Creating IAM roles and policies
- Setting up AWS Secrets Manager
- Configuring EKS Pod Identity
- Deploying the Helm chart to your EKS cluster

## Requirements

Before you begin the deployment process, ensure you have the following prerequisites:

### AWS Account Requirements

1. **Active AWS Account** with appropriate permissions to create:
   - IAM Roles and Policies
   - AWS Secrets Manager Secrets
   - EKS Pod Identity Associations
   - Lambda Functions

2. **Existing EKS Cluster**: You must have an EKS cluster already running in your AWS account. Note the cluster name as you'll need it during deployment.

3. **AWS CLI Installed**: For post-deployment configuration and testing
   ```bash
   aws --version
   ```

4. **kubectl Installed**: To interact with your EKS cluster
   ```bash
   kubectl version --client
   ```

5. **Helm 3.x Installed**: Required for deploying the Helm chart
   ```bash
   helm version
   ```

### API Keys

You'll need API keys for the AI services used by the Agentic Document Extraction API. The service supports two configuration options:

**Option 1: ADE 1 Configuration (Azure OpenAI + Anthropic)**
- Azure OpenAI API Key
- Azure OpenAI Endpoint URL
- Anthropic API Key

**Option 2: ADE 2 Configuration (Google Vertex AI)**
- Gemini Vertex Auth credentials
- Gemini Vertex Project ID
- Gemini Vertex Location

You can configure one or both options based on your needs.

### AWS Marketplace Subscription

Subscribe to the Agentic Document Extraction API from the AWS Marketplace:
- [Agentic Document Extraction API](https://aws.amazon.com/marketplace/pp/prodview-aoolzvht3s6gq)

## Deployment

After subscribing to the service in AWS Marketplace and gathering all prerequisites, you're ready to deploy.

### Deployment Process

1. **Subscribe to the Marketplace Listing**
   
   Navigate to the AWS Marketplace and subscribe to the [Agentic Document Extraction API - Enterprise VPC](https://aws.amazon.com/marketplace/pp/prodview-aoolzvht3s6gq).

2. **Launch the CloudFormation Stack**

   **Quick Deploy Links** - Click to launch the CloudFormation stack in your preferred region:

   | Region | Region Name | Launch Stack |
   |--------|-------------|--------------|
   | us-east-1 | US East (N. Virginia) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | us-east-2 | US East (Ohio) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | us-west-1 | US West (N. California) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | us-west-2 | US West (Oregon) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | eu-west-1 | Europe (Ireland) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | eu-central-1 | Europe (Frankfurt) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | ap-southeast-1 | Asia Pacific (Singapore) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |
   | ap-northeast-1 | Asia Pacific (Tokyo) | [Launch Stack](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/quickcreate?templateURL=https://ade-marketplace-public.s3.us-east-1.amazonaws.com/ade-cloudformation.yaml) |

   **Other Regions:** For regions not listed above, click any "Launch Stack" link and change `region=<region-code>` in the URL to your desired region (e.g., `region=ca-central-1` for Canada, `region=ap-south-1` for Mumbai, etc.).
   
   After clicking the Launch Stack link:
   - Fill in the required parameters (EKS cluster name, API keys)
   - Acknowledge IAM resource creation
   - Click "Create Stack"

<img width="2984" height="2862" alt="image" src="https://github.com/user-attachments/assets/d6659b13-4037-4c89-b18c-0b152fb9162d" />

3. **Monitor Stack Creation**

   The stack typically takes 5-10 minutes to create. Monitor progress in the CloudFormation console and wait for the status to show `CREATE_COMPLETE`.


<img width="2984" height="2640" alt="image" src="https://github.com/user-attachments/assets/9fcdf5b0-f0b8-4983-93f9-e389a0d155e7" />

4. **Retrieve Stack Outputs**

   Once the stack is created, go to the CloudFormation console and view the **Outputs** tab for your stack. Note the following values which you'll need for the Helm deployment:
   - **StackName**: The Kubernetes namespace to use (same as CloudFormation stack name)
   - **SecretName**: The AWS Secrets Manager secret name containing your API keys

<img width="3024" height="1964" alt="image" src="https://github.com/user-attachments/assets/bf16983f-4c86-48fd-9b82-8201abc496d1" />


5. **Deploy the Helm Chart**

   First, configure kubectl to access your EKS cluster:

   ```bash
   aws eks update-kubeconfig --name your-cluster-name --region us-east-1
   ```

   Then, authenticate to the ECR registry:

   ```bash
   aws ecr get-login-password --region us-east-1 | \
     docker login --username AWS --password-stdin \
     709825985650.dkr.ecr.us-east-1.amazonaws.com
   ```

   Finally, install the Helm chart:

   ```bash
   helm upgrade --install landing-agentic-doc-enterprise \
     oci://709825985650.dkr.ecr.us-east-1.amazonaws.com/landingai/landing-agentic-doc-enterprise \
     --version <chart-version> \
     --namespace <StackName-from-outputs> \
     --create-namespace \
     --set app_config.api_keys_secret=<SecretName-from-outputs>
   ```

   Replace `<StackName-from-outputs>` and `<SecretName-from-outputs>` with the actual values from step 4.

   <img width="2219" height="743" alt="image" src="https://github.com/user-attachments/assets/df04d67b-9c54-4309-bc98-c65c0a0890b7" />
   


## Accessing Your Deployment

Once the Helm chart is deployed, you can access the API in several ways:

### Option 1: Port Forwarding (Development/Testing)

For quick testing, use kubectl port forwarding:

```bash
kubectl port-forward svc/landing-agentic-doc-rest 5000:5000 \
  -n <your-namespace>
```

The API will be available at `http://localhost:5000`.

### Option 2: LoadBalancer Service (Production)

The Helm chart creates LoadBalancer services by default. Get the external endpoints:

```bash
kubectl get svc -n <your-namespace>
```

Look for services with type `LoadBalancer` and note their `EXTERNAL-IP` values.

## Configuration

The deployment can be customized using Helm values. Common configuration options:

### API Keys Secret

```bash
--set app_config.api_keys_secret=<your-secret-name>
```

### Resource Limits

```bash
--set deployments.rest.resources.limits.cpu=2000m \
--set deployments.rest.resources.limits.memory=4Gi
```

### Enable GPU Support

```bash
--set useGPU=true
```

### Service Type

```bash
--set service.rest.type=LoadBalancer
```

For a complete list of configuration options, see `values.yaml`.

## Testing the API

Once deployed, test the API to ensure it's working correctly:

```bash
# Port forward if not using LoadBalancer
kubectl port-forward svc/landing-agentic-doc-rest 5000:5000 \
  -n <your-namespace> &

# Test with a PDF document
curl -X POST http://localhost:5000/v1/tools/agentic-document-analysis \
  --form "pdf=@/path/to/your/test.pdf"
```

You should receive a JSON response with the document analysis results.

## Updating Your Deployment

To update your deployment with a new version:

1. **Update the Helm chart:**

   ```bash
   helm upgrade landing-agentic-doc \
     oci://709825985650.dkr.ecr.us-east-1.amazonaws.com/landingai/landing-agentic-doc \
     --version <new-version> \
     --namespace <your-namespace> \
     --reuse-values
   ```

2. **Update API keys in Secrets Manager:**

   ```bash
   aws secretsmanager update-secret \
     --secret-id <SecretName-from-outputs> \
     --secret-string '{
       "AZURE_OPENAI_API_KEY": "new-key",
       "AZURE_OPENAI_ENDPOINT": "new-endpoint",
       "ANTHROPIC_API_KEY": "new-key"
     }'
   ```

3. **Restart pods to pick up new secrets:**

   ```bash
   kubectl rollout restart deployment -n <your-namespace>
   ```

## Cleanup

To completely remove the deployment:

1. **Uninstall the Helm chart:**

   ```bash
   helm uninstall landing-agentic-doc -n <your-namespace>
   ```

2. **Delete the CloudFormation stack:**

   Go to the CloudFormation console, select your stack, and click **Delete**. This will automatically remove:
   - IAM roles and policies
   - Secrets Manager secrets
   - EKS Pod Identity associations
   - Lambda functions

3. **Delete the namespace (optional):**

   ```bash
   kubectl delete namespace <your-namespace>
   ```

> **Note**: The CloudFormation stack is configured to preserve the EKS Pod Identity addon on deletion. This prevents accidental disruption to other workloads in your cluster. If you need to remove the addon, do so manually after stack deletion.

---

For additional support or questions, please refer to the [README.md](README.md) or contact your AWS Marketplace support channel.

