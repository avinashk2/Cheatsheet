# Microsoft AI-102 Azure AI Engineer Associate - Comprehensive Exam Preparation Cheatsheet (2025)

## Table of Contents
1. [Azure AI Services Overview](#1-azure-ai-services-overview)
2. [Azure AI Vision](#2-azure-ai-vision)
3. [Azure AI Document Intelligence](#3-azure-ai-document-intelligence)
4. [Azure AI Language](#4-azure-ai-language)
5. [Azure AI Speech](#5-azure-ai-speech)
6. [Azure OpenAI Service](#6-azure-openai-service)
7. [Azure AI Search](#7-azure-ai-search)
8. [Azure Bot Service & Bot Framework](#8-azure-bot-service--bot-framework)
9. [Azure AI Content Safety](#9-azure-ai-content-safety)
10. [Applied AI Services](#10-applied-ai-services)
11. [Responsible AI & Governance](#11-responsible-ai--governance)
12. [Security, Networking & Identity](#12-security-networking--identity)
13. [Monitoring, Diagnostics & Optimization](#13-monitoring-diagnostics--optimization)
14. [Deployment & Infrastructure as Code](#14-deployment--infrastructure-as-code)
15. [Python Code Examples & Patterns](#15-python-code-examples--patterns)
16. [REST API Reference](#16-rest-api-reference)
17. [Limits, Quotas & Pricing](#17-limits-quotas--pricing)
18. [Troubleshooting Guide](#18-troubleshooting-guide)
19. [Migration Guides](#19-migration-guides)
20. [Exam Preparation Addendum](#20-exam-preparation-addendum)

---

## 1. AZURE AI SERVICES OVERVIEW

### 1.1 Multi-Service Resource vs Individual Services

#### Multi-Service Resource
A single Azure resource that provides access to multiple AI services through one endpoint and one set of keys.

**Benefits:**
- Single API key for all services
- Simplified management and billing
- Consolidated cost tracking
- Easier to manage access control
- Single endpoint for multiple services

**Limitations:**
- Cannot use custom subdomains (required for some features)
- All services must be in the same region
- Less granular access control

**Creating Multi-Service Resource:**

```bash
# Azure CLI - Create multi-service resource
az cognitiveservices account create \
  --name "my-multiservice-resource" \
  --resource-group "ai-resources-rg" \
  --kind "CognitiveServices" \
  --sku "S0" \
  --location "eastus" \
  --yes
```

#### Individual Service Resources
Separate resources for each AI service (Vision, Language, Speech, etc.).

**Benefits:**
- Custom subdomain support (required for some features)
- Granular access control per service
- Different regions per service
- Independent scaling
- Service-specific features enabled

**Limitations:**
- Multiple keys to manage
- More resources to track
- Separate billing per service

```bash
# Azure CLI - Create individual service resource
az cognitiveservices account create \
  --name "my-vision-service" \
  --resource-group "ai-resources-rg" \
  --kind "ComputerVision" \
  --sku "S1" \
  --location "eastus" \
  --custom-domain "my-vision-service" \
  --yes
```

**ARM Template for Multi-Service:**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "accountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Cognitive Services account"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "sku": {
      "type": "string",
      "defaultValue": "S0",
      "allowedValues": ["F0", "S0"]
    }
  },
  "resources": [
    {
      "type": "Microsoft.CognitiveServices/accounts",
      "apiVersion": "2024-10-01",
      "name": "[parameters('accountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('sku')]"
      },
      "kind": "CognitiveServices",
      "properties": {
        "publicNetworkAccess": "Enabled",
        "networkAcls": {
          "defaultAction": "Allow"
        }
      }
    }
  ],
  "outputs": {
    "endpoint": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.CognitiveServices/accounts', parameters('accountName'))).endpoint]"
    },
    "key": {
      "type": "string",
      "value": "[listKeys(resourceId('Microsoft.CognitiveServices/accounts', parameters('accountName')), '2024-10-01').key1]"
    }
  }
}
```

**Bicep Template (Preferred):**

```bicep
// Multi-service Cognitive Services resource
param accountName string
param location string = resourceGroup().location
param sku string = 'S0'
param tags object = {}

resource cognitiveServices 'Microsoft.CognitiveServices/accounts@2024-10-01' = {
  name: accountName
  location: location
  sku: {
    name: sku
  }
  kind: 'CognitiveServices'
  tags: tags
  properties: {
    publicNetworkAccess: 'Enabled'
    networkAcls: {
      defaultAction: 'Allow'
      ipRules: []
      virtualNetworkRules: []
    }
    disableLocalAuth: false
    restore: false
  }
}

output endpoint string = cognitiveServices.properties.endpoint
output resourceId string = cognitiveServices.id
output name string = cognitiveServices.name
```

### 1.2 Authentication Methods

#### 1.2.1 API Keys (Subscription Keys)

**Characteristics:**
- Simplest authentication method
- Two keys provided (key1, key2) for rotation
- Included in request headers
- No expiration (until manually regenerated)
- ⚠️ Less secure than Azure AD authentication

**Retrieve Keys:**

```bash
# Get primary key
az cognitiveservices account keys list \
  --name "my-ai-service" \
  --resource-group "ai-resources-rg" \
  --query "key1" -o tsv

# Regenerate key1
az cognitiveservices account keys regenerate \
  --name "my-ai-service" \
  --resource-group "ai-resources-rg" \
  --key-name key1
```

**Python SDK with API Key:**

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Load from environment variable (best practice)
endpoint = os.environ["AZURE_AI_ENDPOINT"]
key = os.environ["AZURE_AI_KEY"]

# Create client with key credential
credential = AzureKeyCredential(key)
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

# Example usage
documents = ["This is a great service!"]
response = client.analyze_sentiment(documents)
print(f"Sentiment: {response[0].sentiment}")
```

**REST API with Key:**

```bash
# Example with curl
curl -X POST "https://YOUR-RESOURCE.cognitiveservices.azure.com/text/analytics/v3.1/sentiment" \
  -H "Ocp-Apim-Subscription-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "documents": [
      {
        "id": "1",
        "language": "en",
        "text": "This is a great service!"
      }
    ]
  }'
```

#### 1.2.2 Azure Active Directory (Azure AD / Entra ID)

**Benefits:**
- ✅ More secure than API keys
- ✅ Integration with Azure RBAC
- ✅ Token expiration and automatic refresh
- ✅ Conditional Access policies support
- ✅ Audit logging

**Required RBAC Roles:**
- `Cognitive Services User` - Read-only access
- `Cognitive Services Contributor` - Full access
- `Cognitive Services OpenAI User` - OpenAI read access
- `Cognitive Services OpenAI Contributor` - OpenAI full access

**Enable Azure AD Authentication:**

```bash
# Assign role to user
az role assignment create \
  --role "Cognitive Services User" \
  --assignee "user@example.com" \
  --scope "/subscriptions/SUBSCRIPTION_ID/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-ai-service"

# Assign role to service principal
az role assignment create \
  --role "Cognitive Services Contributor" \
  --assignee "APP_ID" \
  --scope "/subscriptions/SUBSCRIPTION_ID/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-ai-service"
```

**Python SDK with Azure AD:**

```python
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient
import os

endpoint = os.environ["AZURE_AI_ENDPOINT"]

# DefaultAzureCredential tries multiple authentication methods:
# 1. Environment variables
# 2. Managed Identity
# 3. Visual Studio Code
# 4. Azure CLI
# 5. Azure PowerShell
# 6. Interactive browser
credential = DefaultAzureCredential()

client = TextAnalyticsClient(endpoint=endpoint, credential=credential)

# Usage is identical to API key authentication
documents = ["Analyzing with Azure AD authentication"]
response = client.analyze_sentiment(documents)
```

**Different Credential Types:**

```python
from azure.identity import (
    DefaultAzureCredential,
    ClientSecretCredential,
    ManagedIdentityCredential,
    AzureCliCredential,
    InteractiveBrowserCredential,
    ChainedTokenCredential
)

# 1. Client Secret (Service Principal)
credential = ClientSecretCredential(
    tenant_id="YOUR_TENANT_ID",
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET"
)

# 2. Managed Identity (preferred in Azure)
credential = ManagedIdentityCredential()

# 3. Azure CLI (for local development)
credential = AzureCliCredential()

# 4. Interactive Browser
credential = InteractiveBrowserCredential()

# 5. Chained (try multiple methods in order)
credential = ChainedTokenCredential(
    ManagedIdentityCredential(),
    AzureCliCredential(),
    InteractiveBrowserCredential()
)
```

#### 1.2.3 Managed Identity

**System-Assigned Managed Identity:**
- Tied to the lifecycle of the Azure resource
- Automatically cleaned up when resource is deleted
- One identity per resource

**User-Assigned Managed Identity:**
- Independent lifecycle
- Can be shared across multiple resources
- Managed separately from resources

**Enable System-Assigned Managed Identity:**

```bash
# Enable on Azure App Service
az webapp identity assign \
  --name "my-web-app" \
  --resource-group "app-rg"

# Enable on Azure Function
az functionapp identity assign \
  --name "my-function" \
  --resource-group "app-rg"

# Enable on Virtual Machine
az vm identity assign \
  --name "my-vm" \
  --resource-group "vm-rg"
```

**Create User-Assigned Managed Identity:**

```bash
# Create identity
az identity create \
  --name "my-user-identity" \
  --resource-group "identity-rg"

# Get principal ID
PRINCIPAL_ID=$(az identity show \
  --name "my-user-identity" \
  --resource-group "identity-rg" \
  --query "principalId" -o tsv)

# Assign to resource
az webapp identity assign \
  --name "my-web-app" \
  --resource-group "app-rg" \
  --identities "/subscriptions/SUB_ID/resourcegroups/identity-rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/my-user-identity"
```

**Grant Managed Identity Access:**

```bash
# Get the managed identity's principal ID
PRINCIPAL_ID=$(az webapp identity show \
  --name "my-web-app" \
  --resource-group "app-rg" \
  --query "principalId" -o tsv)

# Grant access to AI service
az role assignment create \
  --role "Cognitive Services User" \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type "ServicePrincipal" \
  --scope "/subscriptions/SUB_ID/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-ai-service"
```

**Python with Managed Identity:**

```python
from azure.identity import ManagedIdentityCredential
from azure.ai.textanalytics import TextAnalyticsClient
import os

endpoint = os.environ["AZURE_AI_ENDPOINT"]

# System-assigned managed identity
credential = ManagedIdentityCredential()

# User-assigned managed identity (specify client_id)
# credential = ManagedIdentityCredential(client_id="USER_ASSIGNED_CLIENT_ID")

client = TextAnalyticsClient(endpoint=endpoint, credential=credential)
```

**Bicep Template with Managed Identity:**

```bicep
param webAppName string
param aiServiceName string
param location string = resourceGroup().location

// Create AI Service
resource aiService 'Microsoft.CognitiveServices/accounts@2024-10-01' = {
  name: aiServiceName
  location: location
  kind: 'TextAnalytics'
  sku: {
    name: 'S'
  }
  properties: {
    publicNetworkAccess: 'Enabled'
    disableLocalAuth: false  // Set to true to require Azure AD only
  }
}

// Create App Service with managed identity
resource appService 'Microsoft.Web/sites@2023-01-01' = {
  name: webAppName
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    // ... other properties
  }
}

// Grant App Service access to AI Service
resource roleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(aiService.id, appService.id, 'CognitiveServicesUser')
  scope: aiService
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'a97b65f3-24c7-4388-baec-2e87135dc908') // Cognitive Services User
    principalId: appService.identity.principalId
    principalType: 'ServicePrincipal'
  }
}

output aiServiceEndpoint string = aiService.properties.endpoint
output appServicePrincipalId string = appService.identity.principalId
```

### 1.3 Python SDK Installation

#### Core AI Services SDKs (Latest versions as of Dec 2025)

```bash
# Azure AI Vision
pip install azure-ai-vision-imageanalysis==1.0.0b3
pip install azure-cognitiveservices-vision-computervision==0.9.0
pip install azure-cognitiveservices-vision-customvision==3.1.0
pip install azure-cognitiveservices-vision-face==0.6.0

# Azure AI Language
pip install azure-ai-textanalytics==5.3.0
pip install azure-ai-language-questionanswering==1.1.0
pip install azure-ai-language-conversations==1.1.0

# Azure AI Speech
pip install azure-cognitiveservices-speech==1.37.0

# Azure AI Document Intelligence (formerly Form Recognizer)
pip install azure-ai-formrecognizer==3.3.3

# Azure OpenAI
pip install openai==1.51.0  # Official OpenAI library with Azure support
pip install tiktoken==0.7.0  # Token counting

# Azure AI Search
pip install azure-search-documents==11.6.0b4
pip install azure-search-documents[aio]  # For async support

# Azure AI Translator
pip install azure-ai-translation-text==1.0.0
pip install azure-ai-translation-document==1.0.0

# Azure Content Safety
pip install azure-ai-contentsafety==1.0.0

# Bot Framework
pip install botbuilder-core==4.16.1
pip install botbuilder-dialogs==4.16.1
pip install botbuilder-ai==4.16.1
pip install botbuilder-applicationinsights==4.16.1
pip install botbuilder-azure==4.16.1

# Azure Identity (for authentication)
pip install azure-identity==1.17.1

# Azure Key Vault (for secrets)
pip install azure-keyvault-secrets==4.8.0

# Azure Monitor (for logging)
pip install azure-monitor-opentelemetry==1.6.0
pip install opencensus-ext-azure==1.1.13
```

**Requirements.txt Template:**

```text
# Azure AI Services - Latest versions (Dec 2025)
azure-ai-vision-imageanalysis==1.0.0b3
azure-ai-textanalytics==5.3.0
azure-ai-formrecognizer==3.3.3
azure-cognitiveservices-speech==1.37.0
azure-ai-language-questionanswering==1.1.0
azure-ai-language-conversations==1.1.0
azure-search-documents==11.6.0b4
azure-ai-contentsafety==1.0.0
openai==1.51.0
tiktoken==0.7.0

# Authentication
azure-identity==1.17.1
azure-keyvault-secrets==4.8.0

# Bot Framework
botbuilder-core==4.16.1
botbuilder-dialogs==4.16.1
botbuilder-ai==4.16.1

# Monitoring
opencensus-ext-azure==1.1.13
azure-monitor-opentelemetry==1.6.0

# Additional utilities
python-dotenv==1.0.0
aiohttp==3.9.1
requests==2.31.0
Pillow==10.3.0  # Image processing
numpy==1.26.4
pandas==2.2.1
```

**Environment Setup Script:**

```python
# setup_environment.py
import os
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

def load_secrets_from_keyvault(vault_url: str):
    """Load secrets from Azure Key Vault"""
    credential = DefaultAzureCredential()
    client = SecretClient(vault_url=vault_url, credential=credential)
    
    secrets = {
        "AZURE_AI_ENDPOINT": client.get_secret("ai-endpoint").value,
        "AZURE_AI_KEY": client.get_secret("ai-key").value,
        "AZURE_OPENAI_ENDPOINT": client.get_secret("openai-endpoint").value,
        "AZURE_OPENAI_KEY": client.get_secret("openai-key").value,
        "AZURE_SEARCH_ENDPOINT": client.get_secret("search-endpoint").value,
        "AZURE_SEARCH_KEY": client.get_secret("search-key").value,
    }
    
    return secrets

def load_from_env_file():
    """Load from .env file for local development"""
    from dotenv import load_dotenv
    load_dotenv()
    
    return {
        "AZURE_AI_ENDPOINT": os.getenv("AZURE_AI_ENDPOINT"),
        "AZURE_AI_KEY": os.getenv("AZURE_AI_KEY"),
        "AZURE_OPENAI_ENDPOINT": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "AZURE_OPENAI_KEY": os.getenv("AZURE_OPENAI_KEY"),
        "AZURE_SEARCH_ENDPOINT": os.getenv("AZURE_SEARCH_ENDPOINT"),
        "AZURE_SEARCH_KEY": os.getenv("AZURE_SEARCH_KEY"),
    }

# Example .env file structure
"""
AZURE_AI_ENDPOINT=https://your-resource.cognitiveservices.azure.com/
AZURE_AI_KEY=your_api_key_here
AZURE_OPENAI_ENDPOINT=https://your-openai.openai.azure.com/
AZURE_OPENAI_KEY=your_openai_key_here
AZURE_OPENAI_DEPLOYMENT_NAME=gpt-4
AZURE_SEARCH_ENDPOINT=https://your-search.search.windows.net
AZURE_SEARCH_KEY=your_search_key_here
"""
```

### 1.4 Regional Availability and Data Residency

#### Key Considerations
- Not all services available in all regions
- Data residency compliance (GDPR, HIPAA, etc.)
- Latency optimization (choose nearest region)
- Pricing differences by region
- Model availability (especially for OpenAI)

#### Major Regions with AI Services (2025)

| Region | Vision | Language | Speech | OpenAI | Document Intel | Search |
|--------|--------|----------|--------|---------|----------------|--------|
| East US | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| East US 2 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| West US | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| West US 2 | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| West US 3 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| North Europe | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| West Europe | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| UK South | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| France Central | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Switzerland North | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Southeast Asia | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ |
| Australia East | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Japan East | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Canada Central | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

**Check Service Availability:**

```bash
# List all available locations for a service
az cognitiveservices account list-skus \
  --kind "TextAnalytics" \
  --query "[].locations" -o table

# List all available kinds (service types)
az cognitiveservices account list-kinds -o table

# Check OpenAI model availability by region
az cognitiveservices account list-models \
  --kind "OpenAI" \
  --location "eastus" -o table
```

**Data Residency Configuration:**

```bicep
// Bicep template with data residency controls
param location string = 'switzerlandnorth'  // GDPR-compliant region
param enableDataResidency bool = true

resource aiService 'Microsoft.CognitiveServices/accounts@2024-10-01' = {
  name: 'gdpr-compliant-ai'
  location: location
  kind: 'CognitiveServices'
  sku: {
    name: 'S0'
  }
  properties: {
    publicNetworkAccess: 'Disabled'  // Enhanced security
    restrictOutboundNetworkAccess: true
    allowedFqdnList: []  // Whitelist only approved endpoints
    networkAcls: {
      defaultAction: 'Deny'
      virtualNetworkRules: [
        {
          id: '/subscriptions/SUB_ID/resourceGroups/RG/providers/Microsoft.Network/virtualNetworks/VNET/subnets/SUBNET'
          ignoreMissingVnetServiceEndpoint: false
        }
      ]
    }
  }
}
```

### 1.5 Container Deployment

#### Supported Containers (2025)
- **Vision**: Read OCR, Spatial Analysis
- **Language**: Sentiment Analysis, Key Phrase Extraction, Language Detection, NER
- **Speech**: Speech-to-Text, Text-to-Speech, Neural TTS, Custom Speech
- **Decision**: Anomaly Detector
- **Translator**: Text Translation

#### Container Benefits
- ✅ On-premises deployment
- ✅ Disconnected/air-gapped environments
- ✅ Data stays within network perimeter
- ✅ Low latency for local processing
- ✅ Compliance with data locality requirements

#### Container Requirements
- Docker engine
- Azure subscription (for billing)
- Internet connection for billing reporting (can be proxied)
- Minimum 4 CPU cores, 8GB RAM (varies by service)

**Pull Container Image:**

```bash
# Login to Microsoft Container Registry
docker login mcr.microsoft.com

# Pull Read OCR container
docker pull mcr.microsoft.com/azure-cognitive-services/vision/read:3.2

# Pull Language Detection container
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/language:3.1

# Pull Speech-to-Text container
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:4.0.0
```

**Run Container with Docker:**

```bash
# Run Read OCR container
docker run --rm -it -p 5000:5000 \
  --memory 16g \
  --cpus 8 \
  -v /path/to/output:/output \
  mcr.microsoft.com/azure-cognitive-services/vision/read:3.2 \
  Eula=accept \
  Billing=https://YOUR_RESOURCE.cognitiveservices.azure.com/ \
  ApiKey=YOUR_API_KEY \
  Logging:Console:LogLevel:Default=Information

# Run with disconnected billing (requires purchased commitment)
docker run --rm -it -p 5000:5000 \
  --memory 16g \
  --cpus 8 \
  mcr.microsoft.com/azure-cognitive-services/vision/read:3.2 \
  Eula=accept \
  Billing=https://YOUR_RESOURCE.cognitiveservices.azure.com/ \
  ApiKey=YOUR_API_KEY \
  DownloadLicense=True \
  Mounts:License=/path/to/license
```

**Kubernetes Deployment (AKS):**

```yaml
# cognitive-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: read-ocr-deployment
  namespace: ai-services
spec:
  replicas: 3
  selector:
    matchLabels:
      app: read-ocr
  template:
    metadata:
      labels:
        app: read-ocr
    spec:
      containers:
      - name: read-ocr
        image: mcr.microsoft.com/azure-cognitive-services/vision/read:3.2
        ports:
        - containerPort: 5000
        env:
        - name: Eula
          value: "accept"
        - name: Billing
          valueFrom:
            secretKeyRef:
              name: ai-service-secret
              key: billing-endpoint
        - name: ApiKey
          valueFrom:
            secretKeyRef:
              name: ai-service-secret
              key: api-key
        resources:
          requests:
            memory: "8Gi"
            cpu: "4"
          limits:
            memory: "16Gi"
            cpu: "8"
        livenessProbe:
          httpGet:
            path: /status
            port: 5000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /status
            port: 5000
          initialDelaySeconds: 15
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: read-ocr-service
  namespace: ai-services
spec:
  type: LoadBalancer
  selector:
    app: read-ocr
  ports:
  - port: 80
    targetPort: 5000
---
apiVersion: v1
kind: Secret
metadata:
  name: ai-service-secret
  namespace: ai-services
type: Opaque
stringData:
  billing-endpoint: "https://YOUR_RESOURCE.cognitiveservices.azure.com/"
  api-key: "YOUR_API_KEY"
```

**Deploy to AKS:**

```bash
# Create namespace
kubectl create namespace ai-services

# Apply deployment
kubectl apply -f cognitive-service-deployment.yaml

# Check status
kubectl get pods -n ai-services
kubectl get services -n ai-services

# View logs
kubectl logs -f deployment/read-ocr-deployment -n ai-services

# Scale deployment
kubectl scale deployment read-ocr-deployment --replicas=5 -n ai-services
```

**Azure Container Instances (ACI):**

```bash
# Deploy to ACI
az container create \
  --name "read-ocr-aci" \
  --resource-group "ai-containers-rg" \
  --image "mcr.microsoft.com/azure-cognitive-services/vision/read:3.2" \
  --cpu 4 \
  --memory 8 \
  --ports 5000 \
  --ip-address Public \
  --environment-variables \
    Eula=accept \
    Billing=https://YOUR_RESOURCE.cognitiveservices.azure.com/ \
  --secure-environment-variables \
    ApiKey=YOUR_API_KEY
```

**Bicep for ACI Deployment:**

```bicep
param containerName string = 'read-ocr-aci'
param location string = resourceGroup().location
param cpuCores int = 4
param memoryInGb int = 8
param billingEndpoint string
@secure()
param apiKey string

resource containerGroup 'Microsoft.ContainerInstance/containerGroups@2023-05-01' = {
  name: containerName
  location: location
  properties: {
    containers: [
      {
        name: 'read-ocr'
        properties: {
          image: 'mcr.microsoft.com/azure-cognitive-services/vision/read:3.2'
          ports: [
            {
              port: 5000
              protocol: 'TCP'
            }
          ]
          resources: {
            requests: {
              cpu: cpuCores
              memoryInGB: memoryInGb
            }
          }
          environmentVariables: [
            {
              name: 'Eula'
              value: 'accept'
            }
            {
              name: 'Billing'
              value: billingEndpoint
            }
            {
              name: 'ApiKey'
              secureValue: apiKey
            }
          ]
        }
      }
    ]
    osType: 'Linux'
    ipAddress: {
      type: 'Public'
      ports: [
        {
          port: 5000
          protocol: 'TCP'
        }
      ]
    }
  }
}

output containerIPv4Address string = containerGroup.properties.ipAddress.ip
```

**Python Client for Container:**

```python
import requests
from PIL import Image
import io

# Container endpoint (local or ACI/AKS)
container_endpoint = "http://localhost:5000"  # or public IP

def analyze_image_with_container(image_path: str):
    """Send image to containerized Read API"""
    
    # Read image
    with open(image_path, 'rb') as image_file:
        image_data = image_file.read()
    
    # Submit image for analysis
    headers = {'Content-Type': 'application/octet-stream'}
    analyze_url = f"{container_endpoint}/vision/v3.2/read/analyze"
    
    response = requests.post(analyze_url, headers=headers, data=image_data)
    response.raise_for_status()
    
    # Get operation location
    operation_location = response.headers['Operation-Location']
    
    # Poll for results
    import time
    max_attempts = 30
    for i in range(max_attempts):
        result_response = requests.get(operation_location)
        result = result_response.json()
        
        if result['status'] == 'succeeded':
            return result
        elif result['status'] == 'failed':
            raise Exception(f"Analysis failed: {result}")
        
        time.sleep(1)
    
    raise TimeoutError("Analysis timed out")

# Example usage
if __name__ == "__main__":
    result = analyze_image_with_container("document.jpg")
    
    for page in result['analyzeResult']['readResults']:
        print(f"Page {page['page']}:")
        for line in page['lines']:
            print(f"  {line['text']}")
```

### 1.6 Custom Subdomain and DNS Configuration

Custom subdomains are required for:
- ✅ Azure AD authentication
- ✅ Virtual network integration
- ✅ Private endpoints
- ✅ Custom DNS scenarios
- ✅ Branding and organization

**Create Resource with Custom Subdomain:**

```bash
# Custom subdomain must be globally unique
az cognitiveservices account create \
  --name "my-company-ai-prod" \
  --resource-group "ai-resources-rg" \
  --kind "TextAnalytics" \
  --sku "S" \
  --location "eastus" \
  --custom-domain "my-company-ai-prod" \
  --yes

# Endpoint will be: https://my-company-ai-prod.cognitiveservices.azure.com/
```

**DNS Configuration for Private Endpoint:**

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group "networking-rg" \
  --name "privatelink.cognitiveservices.azure.com"

# Link to VNet
az network private-dns link vnet create \
  --resource-group "networking-rg" \
  --zone-name "privatelink.cognitiveservices.azure.com" \
  --name "ai-dns-link" \
  --virtual-network "my-vnet" \
  --registration-enabled false

# Create private endpoint
az network private-endpoint create \
  --name "ai-private-endpoint" \
  --resource-group "networking-rg" \
  --vnet-name "my-vnet" \
  --subnet "private-endpoints-subnet" \
  --private-connection-resource-id "/subscriptions/SUB_ID/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-company-ai-prod" \
  --group-id "account" \
  --connection-name "ai-private-connection"

# Create DNS record
az network private-endpoint dns-zone-group create \
  --resource-group "networking-rg" \
  --endpoint-name "ai-private-endpoint" \
  --name "ai-dns-zone-group" \
  --private-dns-zone "privatelink.cognitiveservices.azure.com" \
  --zone-name "cognitiveservices"
```

### 1.7 Virtual Networks and Private Endpoints

**Network Security Options:**
1. Public access with IP firewall
2. Service endpoints (VNet integration)
3. Private endpoints (Private Link)

**Configure IP Firewall:**

```bash
# Add IP rule
az cognitiveservices account network-rule add \
  --name "my-ai-service" \
  --resource-group "ai-resources-rg" \
  --ip-address "203.0.113.0/24"

# Set default action to Deny
az cognitiveservices account update \
  --name "my-ai-service" \
  --resource-group "ai-resources-rg" \
  --properties '{
    "networkAcls": {
      "defaultAction": "Deny"
    }
  }'

# Allow specific VNet subnet
az cognitiveservices account network-rule add \
  --name "my-ai-service" \
  --resource-group "ai-resources-rg" \
  --subnet "/subscriptions/SUB_ID/resourceGroups/networking-rg/providers/Microsoft.Network/virtualNetworks/my-vnet/subnets/app-subnet"
```

**Bicep with Private Endpoint:**

```bicep
param aiServiceName string
param location string = resourceGroup().location
param vnetName string
param subnetName string

// Create AI Service
resource aiService 'Microsoft.CognitiveServices/accounts@2024-10-01' = {
  name: aiServiceName
  location: location
  kind: 'TextAnalytics'
  sku: {
    name: 'S'
  }
  properties: {
    customSubDomain: aiServiceName
    publicNetworkAccess: 'Disabled'  // Force private endpoint usage
    networkAcls: {
      defaultAction: 'Deny'
    }
  }
}

// Reference existing VNet
resource vnet 'Microsoft.Network/virtualNetworks@2023-05-01' existing = {
  name: vnetName
}

resource subnet 'Microsoft.Network/virtualNetworks/subnets@2023-05-01' existing = {
  parent: vnet
  name: subnetName
}

// Create Private Endpoint
resource privateEndpoint 'Microsoft.Network/privateEndpoints@2023-05-01' = {
  name: '${aiServiceName}-pe'
  location: location
  properties: {
    subnet: {
      id: subnet.id
    }
    privateLinkServiceConnections: [
      {
        name: '${aiServiceName}-connection'
        properties: {
          privateLinkServiceId: aiService.id
          groupIds: [
            'account'
          ]
        }
      }
    ]
  }
}

// Private DNS Zone
resource privateDnsZone 'Microsoft.Network/privateDnsZones@2020-06-01' = {
  name: 'privatelink.cognitiveservices.azure.com'
  location: 'global'
}

// Link DNS Zone to VNet
resource dnsZoneLink 'Microsoft.Network/privateDnsZones/virtualNetworkLinks@2020-06-01' = {
  parent: privateDnsZone
  name: '${aiServiceName}-link'
  location: 'global'
  properties: {
    virtualNetwork: {
      id: vnet.id
    }
    registrationEnabled: false
  }
}

// DNS Zone Group
resource privateEndpointDnsGroup 'Microsoft.Network/privateEndpoints/privateDnsZoneGroups@2023-05-01' = {
  parent: privateEndpoint
  name: 'default'
  properties: {
    privateDnsZoneConfigs: [
      {
        name: 'config1'
        properties: {
          privateDnsZoneId: privateDnsZone.id
        }
      }
    ]
  }
}

output privateEndpointIP string = privateEndpoint.properties.customDnsConfigs[0].ipAddresses[0]
```

### 1.8 Diagnostic Logging and Application Insights

**Enable Diagnostic Settings:**

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group "monitoring-rg" \
  --workspace-name "ai-logs-workspace" \
  --location "eastus"

# Get workspace ID
WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group "monitoring-rg" \
  --workspace-name "ai-logs-workspace" \
  --query "id" -o tsv)

# Enable diagnostics for AI service
az monitor diagnostic-settings create \
  --name "ai-diagnostics" \
  --resource "/subscriptions/SUB_ID/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-ai-service" \
  --workspace "$WORKSPACE_ID" \
  --logs '[
    {
      "category": "Audit",
      "enabled": true
    },
    {
      "category": "RequestResponse",
      "enabled": true
    },
    {
      "category": "Trace",
      "enabled": true
    }
  ]' \
  --metrics '[
    {
      "category": "AllMetrics",
      "enabled": true
    }
  ]'
```

**Application Insights Integration:**

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
from opencensus.ext.azure import metrics_exporter
from opencensus.stats import aggregation as aggregation_module
from opencensus.stats import measure as measure_module
from opencensus.stats import stats as stats_module
from opencensus.stats import view as view_module
from opencensus.tags import tag_map as tag_map_module
import logging
import os

# Setup logging with App Insights
logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler(
    connection_string=os.environ['APPLICATIONINSIGHTS_CONNECTION_STRING']
))
logger.setLevel(logging.INFO)

# Setup metrics
stats = stats_module.stats
view_manager = stats.view_manager
stats_recorder = stats.stats_recorder

# Define custom metrics
ai_request_measure = measure_module.MeasureInt(
    "ai_requests",
    "Number of AI service requests",
    "requests"
)

ai_latency_measure = measure_module.MeasureFloat(
    "ai_latency",
    "AI service request latency",
    "ms"
)

# Create metrics exporter
exporter = metrics_exporter.new_metrics_exporter(
    connection_string=os.environ['APPLICATIONINSIGHTS_CONNECTION_STRING']
)
view_manager.register_exporter(exporter)

# Example usage with AI service
from azure.ai.textanalytics import TextAnalyticsClient
from azure.identity import DefaultAzureCredential
import time

def analyze_with_monitoring(text: str):
    """Analyze text with full monitoring"""
    start_time = time.time()
    
    try:
        # Log request
        logger.info("Starting text analysis", extra={
            "custom_dimensions": {
                "text_length": len(text),
                "operation": "sentiment_analysis"
            }
        })
        
        # Make AI service call
        endpoint = os.environ["AZURE_AI_ENDPOINT"]
        credential = DefaultAzureCredential()
        client = TextAnalyticsClient(endpoint, credential)
        
        result = client.analyze_sentiment([text])[0]
        
        # Calculate latency
        latency = (time.time() - start_time) * 1000
        
        # Log metrics
        mmap = stats_recorder.new_measurement_map()
        tmap = tag_map_module.TagMap()
        tmap.insert("operation", "sentiment_analysis")
        tmap.insert("status", "success")
        
        mmap.measure_int_put(ai_request_measure, 1)
        mmap.measure_float_put(ai_latency_measure, latency)
        mmap.record(tmap)
        
        # Log success
        logger.info("Analysis completed", extra={
            "custom_dimensions": {
                "sentiment": result.sentiment,
                "confidence": result.confidence_scores.positive,
                "latency_ms": latency
            }
        })
        
        return result
        
    except Exception as e:
        # Log error
        logger.error(f"Analysis failed: {str(e)}", extra={
            "custom_dimensions": {
                "error_type": type(e).__name__,
                "operation": "sentiment_analysis"
            }
        })
        raise
```

### 1.9 Cost Management and Optimization

**Cost Monitoring:**

```bash
# View costs for resource group
az consumption usage list \
  --resource-group "ai-resources-rg" \
  --start-date "2025-12-01" \
  --end-date "2025-12-31" \
  --query "[].{Date:usageStart, Cost:pretaxCost, Service:instanceName}" -o table

# Create budget
az consumption budget create \
  --resource-group "ai-resources-rg" \
  --budget-name "ai-monthly-budget" \
  --amount 1000 \
  --category "Cost" \
  --time-grain "Monthly" \
  --start-date "2025-12-01" \
  --end-date "2026-12-01"
```

**Cost Optimization Strategies:**

1. **Use Commitment Tiers** (Save up to 30%)
2. **Batch Processing** (Reduce per-request costs)
3. **Caching** (Avoid redundant calls)
4. **Request Optimization** (Minimize token usage)
5. **Auto-scaling** (Match demand)
6. **Free Tier** (For development/testing)

**Python Cost Tracking:**

```python
import tiktoken
from typing import Dict

class CostTracker:
    """Track AI service costs"""
    
    # Pricing (as of Dec 2025 - example rates)
    PRICING = {
        "gpt-4-turbo": {
            "input": 0.01 / 1000,   # $0.01 per 1K input tokens
            "output": 0.03 / 1000   # $0.03 per 1K output tokens
        },
        "gpt-35-turbo": {
            "input": 0.0015 / 1000,
            "output": 0.002 / 1000
        },
        "text-embedding-3-large": {
            "input": 0.00013 / 1000
        },
        "vision": {
            "per_image": 0.002
        },
        "speech-to-text": {
            "per_minute": 0.024
        }
    }
    
    def __init__(self):
        self.total_cost = 0.0
        self.usage_breakdown = {}
    
    def estimate_openai_cost(self, model: str, input_tokens: int, 
                            output_tokens: int) -> float:
        """Estimate OpenAI API cost"""
        pricing = self.PRICING.get(model, {})
        cost = (
            pricing.get("input", 0) * input_tokens +
            pricing.get("output", 0) * output_tokens
        )
        self.total_cost += cost
        self.usage_breakdown[model] = self.usage_breakdown.get(model, 0) + cost
        return cost
    
    def count_tokens(self, text: str, model: str = "gpt-4") -> int:
        """Count tokens in text"""
        encoding = tiktoken.encoding_for_model(model)
        return len(encoding.encode(text))
    
    def estimate_text_api_cost(self, service: str, 
                               num_requests: int, text_units: int = 0) -> float:
        """
        Estimate Text Analytics API costs
        service: 'sentiment', 'ner', 'key_phrases', etc.
        text_units: Number of 1000-character units
        """
        # Example pricing (S tier)
        price_per_1k_text_records = 2.00
        cost = (num_requests / 1000) * price_per_1k_text_records
        self.total_cost += cost
        return cost
    
    def get_report(self) -> Dict:
        """Get cost report"""
        return {
            "total_cost": round(self.total_cost, 4),
            "breakdown": {k: round(v, 4) for k, v in self.usage_breakdown.items()}
        }

# Example usage
tracker = CostTracker()

# OpenAI cost
prompt = "Analyze this customer feedback..."
response_text = "The sentiment is positive..."
input_tokens = tracker.count_tokens(prompt, "gpt-4")
output_tokens = tracker.count_tokens(response_text, "gpt-4")
cost = tracker.estimate_openai_cost("gpt-4-turbo", input_tokens, output_tokens)
print(f"OpenAI call cost: ${cost:.6f}")

# Text Analytics cost
cost = tracker.estimate_text_api_cost("sentiment", num_requests=1000)
print(f"Text Analytics cost: ${cost:.2f}")

# Get report
print(tracker.get_report())
```

### 1.10 Service Level Agreements (SLAs)

**Azure AI Services SLA: 99.9%**

| Service | SLA | Monthly Downtime | Condition |
|---------|-----|------------------|-----------|
| Vision | 99.9% | 43.2 minutes | Standard/Premium tier |
| Language | 99.9% | 43.2 minutes | Standard tier |
| Speech | 99.9% | 43.2 minutes | Standard tier |
| OpenAI | 99.9% | 43.2 minutes | Provisioned throughput |
| Document Intelligence | 99.9% | 43.2 minutes | Standard tier |
| Translator | 99.9% | 43.2 minutes | Standard tier |
| Bot Service | 99.9% | 43.2 minutes | Standard tier |
| Content Safety | 99.9% | 43.2 minutes | Standard tier |

**Free tier has NO SLA**

**Composite SLA Calculation:**
```
Composite SLA = SLA1 × SLA2 × SLA3
Example: Vision (99.9%) × Language (99.9%) × Search (99.9%)
= 0.999 × 0.999 × 0.999 = 0.997 = 99.7%
```

### 1.11 Migration from Cognitive Services to Azure AI Services

**Key Changes (2023-2025):**
- Rebranding: "Cognitive Services" → "Azure AI Services"
- No breaking API changes for most services
- New unified branding in portal
- Some services renamed (Form Recognizer → Document Intelligence)
- Enhanced responsible AI features
- Better integration with Azure AI Studio

**No Action Required for:**
- ✅ Existing API endpoints continue to work
- ✅ SDKs remain backward compatible
- ✅ Authentication mechanisms unchanged
- ✅ Pricing tiers remain the same

**Recommended Actions:**
1. Update documentation references
2. Use latest SDK versions
3. Adopt new naming conventions
4. Enable responsible AI features
5. Review security best practices

---

## 2. AZURE AI VISION

### 2.1 Image Analysis 4.0 API

The latest Image Analysis API (version 4.0) provides comprehensive computer vision capabilities with improved accuracy and new features.

**Key Capabilities:**
- Dense captions (multiple detailed descriptions)
- Object detection with bounding boxes
- People detection
- Smart cropping suggestions
- Tags and categories
- Adult/racy/gory content detection
- Brand detection
- Color analysis
- Image type classification
- Background removal (NEW in 4.0)

#### 2.1.1 Image Analysis Setup

**Install SDK:**

```bash
pip install azure-ai-vision-imageanalysis==1.0.0b3
pip install Pillow==10.3.0  # For image processing
```

**Create Vision Resource:**

```bash
# Create Computer Vision resource
az cognitiveservices account create \
  --name "my-vision-service" \
  --resource-group "ai-resources-rg" \
  --kind "ComputerVision" \
  --sku "S1" \
  --location "eastus" \
  --custom-domain "my-vision-service" \
  --yes

# Get endpoint and key
az cognitiveservices account show \
  --name "my-vision-service" \
  --resource-group "ai-resources-rg" \
  --query "properties.endpoint" -o tsv

az cognitiveservices account keys list \
  --name "my-vision-service" \
  --resource-group "ai-resources-rg" \
  --query "key1" -o tsv
```

#### 2.1.2 Comprehensive Image Analysis

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import os

# Initialize client
endpoint = os.environ["VISION_ENDPOINT"]
key = os.environ["VISION_KEY"]

client = ImageAnalysisClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(key)
)

def analyze_image_comprehensive(image_path: str):
    """
    Comprehensive image analysis with all features
    """
    
    # Read image
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    # Analyze with all visual features
    result = client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,           # Image caption
            VisualFeatures.DENSE_CAPTIONS,    # Multiple detailed captions
            VisualFeatures.OBJECTS,           # Object detection
            VisualFeatures.PEOPLE,            # People detection
            VisualFeatures.TAGS,              # Image tags
            VisualFeatures.READ,              # OCR text
            VisualFeatures.SMART_CROPS        # Smart crop suggestions
        ],
        gender_neutral_caption=True,          # Use gender-neutral language
        language="en",                        # Output language
        smart_crops_aspect_ratios=[0.9, 1.33] # Aspect ratios for cropping
    )
    
    # Process results
    analysis_results = {
        "caption": None,
        "dense_captions": [],
        "objects": [],
        "people": [],
        "tags": [],
        "text": [],
        "smart_crops": [],
        "metadata": {}
    }
    
    # Main caption
    if result.caption:
        analysis_results["caption"] = {
            "text": result.caption.text,
            "confidence": result.caption.confidence
        }
        print(f"Caption: {result.caption.text} (confidence: {result.caption.confidence:.2f})")
    
    # Dense captions (multiple detailed descriptions)
    if result.dense_captions:
        print("\nDense Captions:")
        for caption in result.dense_captions.list:
            analysis_results["dense_captions"].append({
                "text": caption.text,
                "confidence": caption.confidence,
                "bounding_box": {
                    "x": caption.bounding_box.x,
                    "y": caption.bounding_box.y,
                    "width": caption.bounding_box.width,
                    "height": caption.bounding_box.height
                }
            })
            print(f"  - {caption.text} (confidence: {caption.confidence:.2f})")
            print(f"    Region: [{caption.bounding_box.x}, {caption.bounding_box.y}, "
                  f"{caption.bounding_box.width}, {caption.bounding_box.height}]")
    
    # Objects
    if result.objects:
        print("\nObjects detected:")
        for obj in result.objects.list:
            analysis_results["objects"].append({
                "name": obj.tags[0].name if obj.tags else "unknown",
                "confidence": obj.tags[0].confidence if obj.tags else 0,
                "bounding_box": {
                    "x": obj.bounding_box.x,
                    "y": obj.bounding_box.y,
                    "width": obj.bounding_box.width,
                    "height": obj.bounding_box.height
                }
            })
            if obj.tags:
                print(f"  - {obj.tags[0].name} (confidence: {obj.tags[0].confidence:.2f})")
                print(f"    Location: [{obj.bounding_box.x}, {obj.bounding_box.y}, "
                      f"{obj.bounding_box.width}, {obj.bounding_box.height}]")
    
    # People
    if result.people:
        print(f"\nPeople detected: {len(result.people.list)}")
        for person in result.people.list:
            analysis_results["people"].append({
                "confidence": person.confidence,
                "bounding_box": {
                    "x": person.bounding_box.x,
                    "y": person.bounding_box.y,
                    "width": person.bounding_box.width,
                    "height": person.bounding_box.height
                }
            })
            print(f"  - Person (confidence: {person.confidence:.2f}) at "
                  f"[{person.bounding_box.x}, {person.bounding_box.y}]")
    
    # Tags
    if result.tags:
        print("\nTags:")
        for tag in result.tags.list:
            analysis_results["tags"].append({
                "name": tag.name,
                "confidence": tag.confidence
            })
            print(f"  - {tag.name} (confidence: {tag.confidence:.2f})")
    
    # OCR Text
    if result.read:
        print("\nText detected:")
        for block in result.read.blocks:
            for line in block.lines:
                analysis_results["text"].append({
                    "text": line.text,
                    "bounding_polygon": [
                        {"x": point.x, "y": point.y} 
                        for point in line.bounding_polygon
                    ]
                })
                print(f"  - {line.text}")
    
    # Smart crops
    if result.smart_crops:
        print("\nSmart crop suggestions:")
        for crop in result.smart_crops.list:
            analysis_results["smart_crops"].append({
                "aspect_ratio": crop.aspect_ratio,
                "bounding_box": {
                    "x": crop.bounding_box.x,
                    "y": crop.bounding_box.y,
                    "width": crop.bounding_box.width,
                    "height": crop.bounding_box.height
                }
            })
            print(f"  - Aspect ratio {crop.aspect_ratio}: "
                  f"[{crop.bounding_box.x}, {crop.bounding_box.y}, "
                  f"{crop.bounding_box.width}, {crop.bounding_box.height}]")
    
    # Metadata
    analysis_results["metadata"] = {
        "width": result.metadata.width,
        "height": result.metadata.height
    }
    
    return analysis_results

# Example usage
if __name__ == "__main__":
    results = analyze_image_comprehensive("sample_image.jpg")
```

#### 2.1.3 Background Removal (NEW Feature)

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
from PIL import Image
import io
import os

def remove_background(image_path: str, output_path: str):
    """
    Remove background from image and save result
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    # Read image
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    # Remove background
    result = client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.REMOVE_BACKGROUND]
    )
    
    # Save result (PNG with transparency)
    if result.segmentation:
        # The result contains the image with transparent background
        output_image_data = result.segmentation.image_data
        
        with open(output_path, "wb") as f:
            f.write(output_image_data)
        
        print(f"Background removed. Saved to: {output_path}")
        
        # Get metadata
        if result.metadata:
            print(f"Original size: {result.metadata.width}x{result.metadata.height}")
    
    return result

def remove_background_with_mode(image_path: str, mode: str = "backgroundRemoval"):
    """
    Remove background with different modes
    
    Modes:
    - backgroundRemoval: Remove background completely (transparent PNG)
    - foregroundMatting: Generate alpha matte for foreground
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    result = client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.REMOVE_BACKGROUND],
        segmentation_mode=mode  # "backgroundRemoval" or "foregroundMatting"
    )
    
    return result

# Example usage
if __name__ == "__main__":
    remove_background("person.jpg", "person_no_bg.png")
```

#### 2.1.4 Adult Content Detection

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import os

def detect_adult_content(image_path: str) -> dict:
    """
    Detect adult, racy, and gory content in images
    
    Returns:
        dict with flags and confidence scores for:
        - adult: Sexually explicit content
        - racy: Sexually suggestive content
        - gory: Violence/gore content
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    # Note: Adult content detection is part of the standard analyze call
    # but requires specific visual features
    result = client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.TAGS
        ]
    )
    
    # In the full API, adult content scores are returned
    # This is a simplified example
    content_safety = {
        "is_adult_content": False,
        "is_racy_content": False,
        "is_gory_content": False,
        "adult_score": 0.0,
        "racy_score": 0.0,
        "gore_score": 0.0
    }
    
    # Process results (actual implementation would check result.adult)
    # if hasattr(result, 'adult'):
    #     content_safety.update({
    #         "is_adult_content": result.adult.is_adult_content,
    #         "is_racy_content": result.adult.is_racy_content,
    #         "is_gory_content": result.adult.is_gory_content,
    #         "adult_score": result.adult.adult_score,
    #         "racy_score": result.adult.racy_score,
    #         "gore_score": result.adult.gore_score
    #     })
    
    return content_safety

def filter_unsafe_images(image_paths: list, threshold: float = 0.5) -> list:
    """
    Filter out images with adult/racy/gory content above threshold
    """
    safe_images = []
    
    for image_path in image_paths:
        safety = detect_adult_content(image_path)
        
        is_safe = (
            safety["adult_score"] < threshold and
            safety["racy_score"] < threshold and
            safety["gore_score"] < threshold
        )
        
        if is_safe:
            safe_images.append(image_path)
        else:
            print(f"Filtered out: {image_path} "
                  f"(adult: {safety['adult_score']:.2f}, "
                  f"racy: {safety['racy_score']:.2f}, "
                  f"gore: {safety['gore_score']:.2f})")
    
    return safe_images
```

#### 2.1.5 Brand Detection

```python
def detect_brands(image_path: str) -> list:
    """
    Detect known brands/logos in images
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    result = client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.TAGS]  # Brands may be detected as tags
    )
    
    # Brand detection results
    brands = []
    
    # In full implementation, result would have brands property
    # if hasattr(result, 'brands') and result.brands:
    #     for brand in result.brands.list:
    #         brands.append({
    #             "name": brand.name,
    #             "confidence": brand.confidence,
    #             "bounding_box": {
    #                 "x": brand.rectangle.x,
    #                 "y": brand.rectangle.y,
    #                 "width": brand.rectangle.w,
    #                 "height": brand.rectangle.h
    #             }
    #         })
    #         print(f"Brand detected: {brand.name} (confidence: {brand.confidence:.2f})")
    
    return brands
```

#### 2.1.6 Image Metadata and Color Analysis

```python
def analyze_image_metadata(image_path: str) -> dict:
    """
    Extract comprehensive metadata and color information
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    result = client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.TAGS
        ]
    )
    
    metadata = {
        "dimensions": {
            "width": result.metadata.width,
            "height": result.metadata.height
        },
        "format": None,  # Would be extracted from full API
        "colors": {
            "dominant_color_foreground": None,
            "dominant_color_background": None,
            "dominant_colors": [],
            "accent_color": None,
            "is_black_and_white": False
        },
        "image_type": {
            "clip_art_type": None,  # 0=non-clip-art, 1=ambiguous, 2=normal, 3=good
            "line_drawing_type": None  # 0=non-line-drawing, 1=line-drawing
        }
    }
    
    # In full implementation:
    # if hasattr(result, 'color'):
    #     metadata["colors"] = {
    #         "dominant_color_foreground": result.color.dominant_color_foreground,
    #         "dominant_color_background": result.color.dominant_color_background,
    #         "dominant_colors": result.color.dominant_colors,
    #         "accent_color": result.color.accent_color,
    #         "is_black_and_white": result.color.is_bw_img
    #     }
    
    # if hasattr(result, 'image_type'):
    #     metadata["image_type"] = {
    #         "clip_art_type": result.image_type.clip_art_type,
    #         "line_drawing_type": result.image_type.line_drawing_type
    #     }
    
    return metadata
```

### 2.2 Optical Character Recognition (OCR)

#### 2.2.1 Read API (Latest Version 4.0)

The Read API is the most advanced OCR solution for extracting text from images and documents.

**Features:**
- 100+ language support
- Handwritten and printed text
- Mixed languages in same document
- PDF and TIFF support (up to 2000 pages)
- Layout analysis (tables, paragraphs, lines)
- Page orientation detection
- Async operation for large documents

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import time
import os

def extract_text_read_api(image_path: str) -> dict:
    """
    Extract text using Read API (synchronous for images)
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    # Analyze with READ feature
    result = client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ],
        language="en"  # Specify language or use "auto" for detection
    )
    
    extracted_data = {
        "text": "",
        "lines": [],
        "words": []
    }
    
    if result.read:
        print(f"Detected {len(result.read.blocks)} text blocks\n")
        
        for block_idx, block in enumerate(result.read.blocks):
            print(f"Block {block_idx + 1}:")
            
            for line in block.lines:
                # Collect line text
                extracted_data["lines"].append({
                    "text": line.text,
                    "bounding_polygon": [
                        {"x": point.x, "y": point.y} 
                        for point in line.bounding_polygon
                    ]
                })
                
                print(f"  Line: '{line.text}'")
                print(f"  Bounding polygon: {[f'({p.x},{p.y})' for p in line.bounding_polygon]}")
                
                # Collect words
                for word in line.words:
                    extracted_data["words"].append({
                        "text": word.text,
                        "confidence": word.confidence,
                        "bounding_polygon": [
                            {"x": point.x, "y": point.y} 
                            for point in word.bounding_polygon
                        ]
                    })
                    print(f"    Word: '{word.text}' (confidence: {word.confidence:.4f})")
                
                # Append to full text
                extracted_data["text"] += line.text + "\n"
            
            print()
    
    return extracted_data
```

#### 2.2.2 Asynchronous Read API for PDFs

```python
import requests
import time
import os

def extract_text_from_pdf_async(pdf_path: str) -> dict:
    """
    Extract text from PDF using async Read API
    Supports multi-page PDFs up to 2000 pages
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    # Construct URL
    analyze_url = f"{endpoint}/vision/v3.2/read/analyze"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"
    }
    
    # Read PDF file
    with open(pdf_path, "rb") as f:
        pdf_data = f.read()
    
    # Submit PDF for analysis
    print("Submitting PDF for analysis...")
    response = requests.post(
        analyze_url,
        headers=headers,
        data=pdf_data,
        params={
            "language": "en",  # Or "auto" for automatic detection
            "readingOrder": "natural"  # "basic" or "natural"
        }
    )
    response.raise_for_status()
    
    # Get operation location from headers
    operation_location = response.headers["Operation-Location"]
    print(f"Operation ID: {operation_location.split('/')[-1]}")
    
    # Poll for results
    print("Waiting for results...")
    max_attempts = 60
    poll_interval = 2  # seconds
    
    for attempt in range(max_attempts):
        result_response = requests.get(
            operation_location,
            headers={"Ocp-Apim-Subscription-Key": key}
        )
        result_response.raise_for_status()
        result = result_response.json()
        
        status = result["status"]
        print(f"Status: {status}")
        
        if status == "succeeded":
            return process_read_results(result)
        elif status == "failed":
            raise Exception(f"OCR failed: {result.get('analyzeResult', {}).get('errors', 'Unknown error')}")
        
        time.sleep(poll_interval)
    
    raise TimeoutError("OCR operation timed out")

def process_read_results(result: dict) -> dict:
    """Process and structure Read API results"""
    
    analyze_result = result["analyzeResult"]
    
    extracted_data = {
        "pages": [],
        "full_text": "",
        "model_version": analyze_result.get("modelVersion"),
        "language": analyze_result.get("language")
    }
    
    # Process each page
    for page in analyze_result["readResults"]:
        page_data = {
            "page_number": page["page"],
            "angle": page["angle"],
            "width": page["width"],
            "height": page["height"],
            "unit": page["unit"],
            "lines": [],
            "text": ""
        }
        
        # Process lines
        for line in page["lines"]:
            line_data = {
                "text": line["text"],
                "bounding_box": line["boundingBox"],
                "words": []
            }
            
            # Process words
            if "words" in line:
                for word in line["words"]:
                    line_data["words"].append({
                        "text": word["text"],
                        "confidence": word.get("confidence", 1.0),
                        "bounding_box": word["boundingBox"]
                    })
            
            page_data["lines"].append(line_data)
            page_data["text"] += line["text"] + "\n"
        
        extracted_data["pages"].append(page_data)
        extracted_data["full_text"] += page_data["text"]
    
    return extracted_data

def extract_with_layout_analysis(pdf_path: str) -> dict:
    """
    Extract text with advanced layout analysis
    Includes tables, paragraphs, and structural information
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    analyze_url = f"{endpoint}/vision/v3.2/read/analyze"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"
    }
    
    with open(pdf_path, "rb") as f:
        pdf_data = f.read()
    
    # Submit with pages parameter to extract specific pages
    response = requests.post(
        analyze_url,
        headers=headers,
        data=pdf_data,
        params={
            "language": "auto",
            "pages": "1-5",  # Extract only pages 1-5
            "readingOrder": "natural"
        }
    )
    response.raise_for_status()
    
    operation_location = response.headers["Operation-Location"]
    
    # Poll for results
    for _ in range(60):
        result_response = requests.get(
            operation_location,
            headers={"Ocp-Apim-Subscription-Key": key}
        )
        result = result_response.json()
        
        if result["status"] == "succeeded":
            return result
        elif result["status"] == "failed":
            raise Exception(f"Analysis failed: {result}")
        
        time.sleep(2)
    
    raise TimeoutError("Operation timed out")

# Example usage
if __name__ == "__main__":
    # Extract from image
    image_result = extract_text_read_api("document.jpg")
    print("Extracted text:")
    print(image_result["text"])
    
    # Extract from PDF
    pdf_result = extract_text_from_pdf_async("multi_page_document.pdf")
    print(f"\nExtracted {len(pdf_result['pages'])} pages from PDF")
    print("First page text:")
    print(pdf_result["pages"][0]["text"][:500])
```

#### 2.2.3 Handwriting Recognition

```python
def extract_handwritten_text(image_path: str) -> dict:
    """
    Specialized function for handwritten text extraction
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    client = ImageAnalysisClient(
        endpoint=endpoint,
        credential=AzureKeyCredential(key)
    )
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    result = client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ],
        language="en"  # Handwriting currently supports limited languages
    )
    
    handwriting_results = {
        "lines": [],
        "confidence_scores": []
    }
    
    if result.read:
        for block in result.read.blocks:
            for line in block.lines:
                avg_confidence = sum(word.confidence for word in line.words) / len(line.words)
                
                handwriting_results["lines"].append({
                    "text": line.text,
                    "average_confidence": avg_confidence,
                    "word_count": len(line.words)
                })
                
                handwriting_results["confidence_scores"].append(avg_confidence)
                
                # Flag low confidence lines for review
                if avg_confidence < 0.7:
                    print(f"⚠️ Low confidence: '{line.text}' ({avg_confidence:.2f})")
                else:
                    print(f"✅ '{line.text}' ({avg_confidence:.2f})")
    
    # Calculate overall quality
    if handwriting_results["confidence_scores"]:
        avg_confidence = sum(handwriting_results["confidence_scores"]) / len(handwriting_results["confidence_scores"])
        handwriting_results["overall_confidence"] = avg_confidence
        print(f"\nOverall confidence: {avg_confidence:.2f}")
    
    return handwriting_results
```

#### 2.2.4 Multi-Language OCR

```python
def extract_multilanguage_text(image_path: str, languages: list = None) -> dict:
    """
    Extract text from multi-language documents
    
    Supported languages: 100+ including:
    - Arabic, Chinese (Simplified/Traditional), Czech, Danish, Dutch,
    - English, Finnish, French, German, Greek, Hungarian, Italian,
    - Japanese, Korean, Norwegian, Polish, Portuguese, Russian,
    - Spanish, Swedish, Turkish, and many more
    """
    endpoint = os.environ["VISION_ENDPOINT"]
    key = os.environ["VISION_KEY"]
    
    analyze_url = f"{endpoint}/vision/v3.2/read/analyze"
    
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"
    }
    
    with open(image_path, "rb") as f:
        image_data = f.read()
    
    # Use "auto" for automatic language detection
    # Or specify language codes: "en", "es", "fr", "de", "ja", "ko", "zh-Hans", etc.
    params = {
        "language": "auto" if not languages else ",".join(languages),
        "readingOrder": "natural"
    }
    
    response = requests.post(
        analyze_url,
        headers=headers,
        data=image_data,
        params=params
    )
    response.raise_for_status()
    
    operation_location = response.headers["Operation-Location"]
    
    # Poll for results
    for _ in range(30):
        result_response = requests.get(
            operation_location,
            headers={"Ocp-Apim-Subscription-Key": key}
        )
        result = result_response.json()
        
        if result["status"] == "succeeded":
            # Process results with language information
            multilang_data = {
                "detected_languages": set(),
                "text_by_language": {},
                "full_text": ""
            }
            
            for page in result["analyzeResult"]["readResults"]:
                for line in page["lines"]:
                    text = line["text"]
                    # Language detection would be at line/word level
                    multilang_data["full_text"] += text + "\n"
            
            return multilang_data
        
        elif result["status"] == "failed":
            raise Exception(f"Analysis failed: {result}")
        
        time.sleep(1)
    
    raise TimeoutError("Operation timed out")

# Language code reference
SUPPORTED_LANGUAGES = {
    "en": "English",
    "es": "Spanish",
    "fr": "French",
    "de": "German",
    "it": "Italian",
    "pt": "Portuguese",
    "nl": "Dutch",
    "sv": "Swedish",
    "no": "Norwegian",
    "da": "Danish",
    "fi": "Finnish",
    "pl": "Polish",
    "cs": "Czech",
    "ru": "Russian",
    "ar": "Arabic",
    "ja": "Japanese",
    "ko": "Korean",
    "zh-Hans": "Chinese Simplified",
    "zh-Hant": "Chinese Traditional",
    "hi": "Hindi",
    "th": "Thai",
    "vi": "Vietnamese"
}
```

### 2.3 Face API

⚠️ **IMPORTANT: Limited Access Feature**

The Face API is now a **Limited Access** service requiring an application and approval from Microsoft. This is part of Microsoft's Responsible AI initiative.

**Application Required For:**
- Face identification
- Face verification
- Celebrity recognition
- Face grouping with large person groups

**No Application Required For:**
- Face detection
- Face landmarks
- Face attributes (age, emotion, etc.)

**How to Apply:**
1. Go to: https://aka.ms/facerecognition
2. Fill out intake form
3. Provide use case justification
4. Wait for Microsoft approval
5. Receive Limited Access credentials

```python
from azure.cognitiveservices.vision.face import FaceClient
from azure.cognitiveservices.vision.face.models import (
    FaceAttributeType,
    DetectionModel,
    RecognitionModel
)
from msrest.authentication import CognitiveServicesCredentials
import os

# Initialize Face client
endpoint = os.environ["FACE_ENDPOINT"]
key = os.environ["FACE_KEY"]

face_client = FaceClient(endpoint, CognitiveServicesCredentials(key))

def detect_faces_with_attributes(image_path: str) -> list:
    """
    Detect faces with comprehensive attributes
    
    Available attributes:
    - age, gender, smile, facialHair, glasses, emotion
    - headPose, hair, makeup, accessories, blur, exposure, noise
    """
    
    with open(image_path, "rb") as image:
        # Detect faces with all attributes
        faces = face_client.face.detect_with_stream(
            image,
            detection_model=DetectionModel.detection03,  # Latest model
            recognition_model=RecognitionModel.recognition04,  # Latest model
            return_face_id=True,
            return_face_landmarks=True,
            return_face_attributes=[
                FaceAttributeType.age,
                FaceAttributeType.gender,
                FaceAttributeType.smile,
                FaceAttributeType.facial_hair,
                FaceAttributeType.glasses,
                FaceAttributeType.emotion,
                FaceAttributeType.head_pose,
                FaceAttributeType.hair,
                FaceAttributeType.makeup,
                FaceAttributeType.accessories,
                FaceAttributeType.blur,
                FaceAttributeType.exposure,
                FaceAttributeType.noise
            ]
        )
    
    face_data = []
    
    for face in faces:
        data = {
            "face_id": face.face_id,
            "face_rectangle": {
                "top": face.face_rectangle.top,
                "left": face.face_rectangle.left,
                "width": face.face_rectangle.width,
                "height": face.face_rectangle.height
            }
        }
        
        # Attributes
        if face.face_attributes:
            attrs = face.face_attributes
            
            data["attributes"] = {
                "age": attrs.age,
                "gender": attrs.gender.value if attrs.gender else None,
                "smile": attrs.smile,
                "facial_hair": {
                    "moustache": attrs.facial_hair.moustache,
                    "beard": attrs.facial_hair.beard,
                    "sideburns": attrs.facial_hair.sideburns
                } if attrs.facial_hair else None,
                "glasses": attrs.glasses.value if attrs.glasses else None,
                "emotion": {
                    "anger": attrs.emotion.anger,
                    "contempt": attrs.emotion.contempt,
                    "disgust": attrs.emotion.disgust,
                    "fear": attrs.emotion.fear,
                    "happiness": attrs.emotion.happiness,
                    "neutral": attrs.emotion.neutral,
                    "sadness": attrs.emotion.sadness,
                    "surprise": attrs.emotion.surprise
                } if attrs.emotion else None,
                "head_pose": {
                    "pitch": attrs.head_pose.pitch,
                    "roll": attrs.head_pose.roll,
                    "yaw": attrs.head_pose.yaw
                } if attrs.head_pose else None,
                "hair": {
                    "bald": attrs.hair.bald,
                    "hair_color": [
                        {"color": c.color.value, "confidence": c.confidence}
                        for c in attrs.hair.hair_color
                    ] if attrs.hair.hair_color else []
                } if attrs.hair else None,
                "makeup": {
                    "eye_makeup": attrs.makeup.eye_makeup,
                    "lip_makeup": attrs.makeup.lip_makeup
                } if attrs.makeup else None,
                "accessories": [
                    {"type": acc.type.value, "confidence": acc.confidence}
                    for acc in attrs.accessories
                ] if attrs.accessories else [],
                "blur": {
                    "level": attrs.blur.blur_level.value,
                    "value": attrs.blur.value
                } if attrs.blur else None,
                "exposure": {
                    "level": attrs.exposure.exposure_level.value,
                    "value": attrs.exposure.value
                } if attrs.exposure else None,
                "noise": {
                    "level": attrs.noise.noise_level.value,
                    "value": attrs.noise.value
                } if attrs.noise else None
            }
            
            print(f"\nFace detected:")
            print(f"  Age: {attrs.age}")
            print(f"  Gender: {attrs.gender.value if attrs.gender else 'Unknown'}")
            print(f"  Smile: {attrs.smile:.2f}")
            
            # Dominant emotion
            if attrs.emotion:
                emotions = {
                    "anger": attrs.emotion.anger,
                    "contempt": attrs.emotion.contempt,
                    "disgust": attrs.emotion.disgust,
                    "fear": attrs.emotion.fear,
                    "happiness": attrs.emotion.happiness,
                    "neutral": attrs.emotion.neutral,
                    "sadness": attrs.emotion.sadness,
                    "surprise": attrs.emotion.surprise
                }
                dominant_emotion = max(emotions.items(), key=lambda x: x[1])
                print(f"  Dominant emotion: {dominant_emotion[0]} ({dominant_emotion[1]:.2f})")
        
        # Landmarks
        if face.face_landmarks:
            data["landmarks"] = {
                "pupil_left": {
                    "x": face.face_landmarks.pupil_left.x,
                    "y": face.face_landmarks.pupil_left.y
                },
                "pupil_right": {
                    "x": face.face_landmarks.pupil_right.x,
                    "y": face.face_landmarks.pupil_right.y
                },
                "nose_tip": {
                    "x": face.face_landmarks.nose_tip.x,
                    "y": face.face_landmarks.nose_tip.y
                },
                "mouth_left": {
                    "x": face.face_landmarks.mouth_left.x,
                    "y": face.face_landmarks.mouth_left.y
                },
                "mouth_right": {
                    "x": face.face_landmarks.mouth_right.x,
                    "y": face.face_landmarks.mouth_right.y
                }
                # ... 27 total landmarks available
            }
        
        face_data.append(data)
    
    return face_data
```

#### 2.3.1 Face Verification (Limited Access)

```python
def verify_faces(image_path1: str, image_path2: str) -> dict:
    """
    Verify if two faces belong to the same person
    ⚠️ Requires Limited Access approval
    """
    
    # Detect faces in both images
    with open(image_path1, "rb") as image1:
        faces1 = face_client.face.detect_with_stream(
            image1,
            detection_model=DetectionModel.detection03,
            recognition_model=RecognitionModel.recognition04,
            return_face_id=True
        )
    
    with open(image_path2, "rb") as image2:
        faces2 = face_client.face.detect_with_stream(
            image2,
            detection_model=DetectionModel.detection03,
            recognition_model=RecognitionModel.recognition04,
            return_face_id=True
        )
    
    if not faces1 or not faces2:
        raise ValueError("No faces detected in one or both images")
    
    # Verify the first face in each image
    face_id1 = faces1[0].face_id
    face_id2 = faces2[0].face_id
    
    # Perform verification
    verify_result = face_client.face.verify_face_to_face(face_id1, face_id2)
    
    result = {
        "is_identical": verify_result.is_identical,
        "confidence": verify_result.confidence
    }
    
    print(f"Faces are identical: {verify_result.is_identical}")
    print(f"Confidence: {verify_result.confidence:.4f}")
    
    return result
```

#### 2.3.2 Face Identification with PersonGroup (Limited Access)

```python
def create_person_group_and_train(group_id: str, group_name: str, 
                                  training_data: dict):
    """
    Create and train a PersonGroup for face identification
    ⚠️ Requires Limited Access approval
    
    training_data format:
    {
        "person1_name": ["path/to/image1.jpg", "path/to/image2.jpg"],
        "person2_name": ["path/to/image3.jpg", "path/to/image4.jpg"]
    }
    """
    
    # Create PersonGroup
    face_client.person_group.create(
        person_group_id=group_id,
        name=group_name,
        recognition_model=RecognitionModel.recognition04
    )
    print(f"Created PersonGroup: {group_name}")
    
    # Add persons and their faces
    person_ids = {}
    
    for person_name, image_paths in training_data.items():
        # Create person
        person = face_client.person_group_person.create(
            person_group_id=group_id,
            name=person_name
        )
        person_ids[person_name] = person.person_id
        print(f"Created person: {person_name}")
        
        # Add faces for this person
        for image_path in image_paths:
            with open(image_path, "rb") as image:
                face_client.person_group_person.add_face_from_stream(
                    person_group_id=group_id,
                    person_id=person.person_id,
                    image=image,
                    detection_model=DetectionModel.detection03
                )
            print(f"  Added face from: {image_path}")
    
    # Train the PersonGroup
    print("\nTraining PersonGroup...")
    face_client.person_group.train(group_id)
    
    # Wait for training to complete
    while True:
        training_status = face_client.person_group.get_training_status(group_id)
        print(f"Training status: {training_status.status}")
        
        if training_status.status == "succeeded":
            break
        elif training_status.status == "failed":
            raise Exception(f"Training failed: {training_status.message}")
        
        time.sleep(1)
    
    print("Training completed!")
    return person_ids

def identify_faces(group_id: str, image_path: str, confidence_threshold: float = 0.7):
    """
    Identify faces in an image using a trained PersonGroup
    ⚠️ Requires Limited Access approval
    """
    
    # Detect faces in the image
    with open(image_path, "rb") as image:
        faces = face_client.face.detect_with_stream(
            image,
            detection_model=DetectionModel.detection03,
            recognition_model=RecognitionModel.recognition04,
            return_face_id=True
        )
    
    if not faces:
        print("No faces detected")
        return []
    
    face_ids = [face.face_id for face in faces]
    print(f"Detected {len(faces)} face(s)")
    
    # Identify faces
    identify_results = face_client.face.identify(
        face_ids=face_ids,
        person_group_id=group_id,
        confidence_threshold=confidence_threshold
    )
    
    identified_faces = []
    
    for face, result in zip(faces, identify_results):
        face_data = {
            "face_rectangle": {
                "top": face.face_rectangle.top,
                "left": face.face_rectangle.left,
                "width": face.face_rectangle.width,
                "height": face.face_rectangle.height
            },
            "candidates": []
        }
        
        if result.candidates:
            for candidate in result.candidates:
                # Get person details
                person = face_client.person_group_person.get(
                    person_group_id=group_id,
                    person_id=candidate.person_id
                )
                
                face_data["candidates"].append({
                    "person_id": candidate.person_id,
                    "person_name": person.name,
                    "confidence": candidate.confidence
                })
                
                print(f"Identified: {person.name} (confidence: {candidate.confidence:.4f})")
        else:
            print("Face not recognized")
        
        identified_faces.append(face_data)
    
    return identified_faces

# Example usage
if __name__ == "__main__":
    # Create and train PersonGroup
    training_data = {
        "Alice": ["alice1.jpg", "alice2.jpg", "alice3.jpg"],
        "Bob": ["bob1.jpg", "bob2.jpg", "bob3.jpg"],
        "Charlie": ["charlie1.jpg", "charlie2.jpg"]
    }
    
    person_ids = create_person_group_and_train(
        group_id="employees",
        group_name="Company Employees",
        training_data=training_data
    )
    
    # Identify faces in new image
    results = identify_faces("employees", "group_photo.jpg")
```

### 2.4 Custom Vision

Custom Vision allows you to train custom image classification and object detection models with your own labeled data.

**Two Project Types:**
1. **Classification** - Categorize entire images
   - Multi-class (single label per image)
   - Multi-label (multiple labels per image)
2. **Object Detection** - Detect and locate multiple objects in images

```python
from azure.cognitiveservices.vision.customvision.training import CustomVisionTrainingClient
from azure.cognitiveservices.vision.customvision.prediction import CustomVisionPredictionClient
from azure.cognitiveservices.vision.customvision.training.models import (
    ImageFileCreateBatch,
    ImageFileCreateEntry,
    Region
)
from msrest.authentication import ApiKeyCredentials
import time
import os

# Initialize clients
training_endpoint = os.environ["CUSTOMVISION_TRAINING_ENDPOINT"]
training_key = os.environ["CUSTOMVISION_TRAINING_KEY"]
prediction_endpoint = os.environ["CUSTOMVISION_PREDICTION_ENDPOINT"]
prediction_key = os.environ["CUSTOMVISION_PREDICTION_KEY"]
prediction_resource_id = os.environ["CUSTOMVISION_PREDICTION_RESOURCE_ID"]

training_credentials = ApiKeyCredentials(in_headers={"Training-key": training_key})
trainer = CustomVisionTrainingClient(training_endpoint, training_credentials)

prediction_credentials = ApiKeyCredentials(in_headers={"Prediction-key": prediction_key})
predictor = CustomVisionPredictionClient(prediction_endpoint, prediction_credentials)

def create_classification_project(project_name: str, is_multilabel: bool = False):
    """
    Create a classification project
    """
    from azure.cognitiveservices.vision.customvision.training.models import (
        Project,
        ClassificationType,
        Domain
    )
    
    # Get available domains
    domains = trainer.get_domains()
    
    # Use "General (compact)" for exportable models
    # Use "General" for cloud-only (better accuracy)
    classification_domain = next(
        d for d in domains 
        if d.type == "Classification" and "General" in d.name
    )
    
    print(f"Creating project: {project_name}")
    print(f"Domain: {classification_domain.name}")
    print(f"Multi-label: {is_multilabel}")
    
    project = trainer.create_project(
        project_name,
        domain_id=classification_domain.id,
        classification_type=ClassificationType.multilabel if is_multilabel 
                           else ClassificationType.multiclass
    )
    
    print(f"Project created with ID: {project.id}")
    return project

def add_images_to_classification_project(project_id: str, training_images: dict):
    """
    Add labeled images to classification project
    
    training_images format:
    {
        "tag1": ["path/to/image1.jpg", "path/to/image2.jpg"],
        "tag2": ["path/to/image3.jpg", "path/to/image4.jpg"]
    }
    """
    
    # Create tags
    tags = {}
    for tag_name in training_images.keys():
        tag = trainer.create_tag(project_id, tag_name)
        tags[tag_name] = tag
        print(f"Created tag: {tag_name} (ID: {tag.id})")
    
    # Upload images in batches (max 64 per batch)
    batch_size = 64
    
    for tag_name, image_paths in training_images.items():
        print(f"\nUploading images for tag: {tag_name}")
        tag_id = tags[tag_name].id
        
        for i in range(0, len(image_paths), batch_size):
            batch_paths = image_paths[i:i + batch_size]
            image_entries = []
            
            for image_path in batch_paths:
                with open(image_path, "rb") as image:
                    image_data = image.read()
                    image_entries.append(
                        ImageFileCreateEntry(
                            name=os.path.basename(image_path),
                            contents=image_data,
                            tag_ids=[tag_id]
                        )
                    )
            
            # Upload batch
            upload_result = trainer.create_images_from_files(
                project_id,
                ImageFileCreateBatch(images=image_entries)
            )
            
            if upload_result.is_batch_successful:
                print(f"  Uploaded {len(batch_paths)} images successfully")
            else:
                print(f"  ⚠️ Some images failed to upload:")
                for image in upload_result.images:
                    if image.status != "OK":
                        print(f"    - {image.source_url}: {image.status}")
    
    print("\nImage upload completed!")

def train_classification_model(project_id: str, iteration_name: str = None):
    """
    Train the classification model
    """
    if not iteration_name:
        iteration_name = f"Iteration_{int(time.time())}"
    
    print(f"Starting training: {iteration_name}")
    iteration = trainer.train_project(project_id)
    
    # Wait for training to complete
    while iteration.status != "Completed":
        iteration = trainer.get_iteration(project_id, iteration.id)
        print(f"Training status: {iteration.status}")
        
        if iteration.status == "Failed":
            raise Exception("Training failed")
        
        time.sleep(10)
    
    print("Training completed!")
    
    # Print performance metrics
    print(f"\nPerformance Metrics:")
    print(f"  Precision: {iteration.precision:.2%}")
    print(f"  Recall: {iteration.recall:.2%}")
    print(f"  Average Precision: {iteration.average_precision:.2%}")
    
    return iteration

def publish_iteration(project_id: str, iteration_id: str, publish_name: str):
    """
    Publish trained iteration for prediction
    """
    trainer.publish_iteration(
        project_id,
        iteration_id,
        publish_name,
        prediction_resource_id
    )
    print(f"Iteration published as: {publish_name}")

def classify_image(project_id: str, publish_name: str, image_path: str):
    """
    Classify an image using the published model
    """
    with open(image_path, "rb") as image:
        results = predictor.classify_image(
            project_id,
            publish_name,
            image.read()
        )
    
    print(f"\nClassification results for: {image_path}")
    for prediction in results.predictions:
        print(f"  {prediction.tag_name}: {prediction.probability:.2%}")
    
    return results.predictions

# Object Detection functions

def create_object_detection_project(project_name: str):
    """
    Create an object detection project
    """
    from azure.cognitiveservices.vision.customvision.training.models import Domain
    
    domains = trainer.get_domains()
    obj_detection_domain = next(
        d for d in domains 
        if d.type == "ObjectDetection" and "General" in d.name
    )
    
    project = trainer.create_project(
        project_name,
        domain_id=obj_detection_domain.id
    )
    
    print(f"Object detection project created: {project.id}")
    return project

def add_images_with_regions(project_id: str, labeled_images: list):
    """
    Add images with bounding box annotations for object detection
    
    labeled_images format:
    [
        {
            "image_path": "path/to/image.jpg",
            "regions": [
                {
                    "tag_name": "dog",
                    "left": 0.1,  # Normalized coordinates (0-1)
                    "top": 0.2,
                    "width": 0.3,
                    "height": 0.4
                },
                {
                    "tag_name": "cat",
                    "left": 0.5,
                    "top": 0.1,
                    "width": 0.25,
                    "height": 0.35
                }
            ]
        }
    ]
    """
    
    # Create tags
    tag_names = set()
    for image_data in labeled_images:
        for region in image_data["regions"]:
            tag_names.add(region["tag_name"])
    
    tags = {}
    for tag_name in tag_names:
        tag = trainer.create_tag(project_id, tag_name)
        tags[tag_name] = tag
        print(f"Created tag: {tag_name}")
    
    # Upload images with regions
    print("\nUploading images with bounding boxes...")
    
    for image_data in labeled_images:
        image_path = image_data["image_path"]
        
        with open(image_path, "rb") as image_file:
            image_contents = image_file.read()
        
        # Create region objects
        regions = []
        for region_data in image_data["regions"]:
            tag_id = tags[region_data["tag_name"]].id
            region = Region(
                tag_id=tag_id,
                left=region_data["left"],
                top=region_data["top"],
                width=region_data["width"],
                height=region_data["height"]
            )
            regions.append(region)
        
        # Upload image with regions
        trainer.create_images_from_files(
            project_id,
            ImageFileCreateBatch(images=[
                ImageFileCreateEntry(
                    name=os.path.basename(image_path),
                    contents=image_contents,
                    regions=regions
                )
            ])
        )
        
        print(f"  Uploaded: {image_path} with {len(regions)} objects")
    
    print("Upload completed!")

def detect_objects(project_id: str, publish_name: str, image_path: str, 
                   threshold: float = 0.5):
    """
    Detect objects in an image
    """
    with open(image_path, "rb") as image:
        results = predictor.detect_image(
            project_id,
            publish_name,
            image.read()
        )
    
    print(f"\nObject detection results for: {image_path}")
    
    detected_objects = []
    for prediction in results.predictions:
        if prediction.probability >= threshold:
            detected_objects.append({
                "tag": prediction.tag_name,
                "probability": prediction.probability,
                "bounding_box": {
                    "left": prediction.bounding_box.left,
                    "top": prediction.bounding_box.top,
                    "width": prediction.bounding_box.width,
                    "height": prediction.bounding_box.height
                }
            })
            
            print(f"  {prediction.tag_name}: {prediction.probability:.2%}")
            print(f"    Location: ({prediction.bounding_box.left:.2f}, "
                  f"{prediction.bounding_box.top:.2f}) "
                  f"Size: {prediction.bounding_box.width:.2f} x "
                  f"{prediction.bounding_box.height:.2f}")
    
    return detected_objects

# Model export (for edge deployment)

def export_model(project_id: str, iteration_id: str, platform: str = "ONNX"):
    """
    Export model for edge deployment
    
    Supported platforms:
    - ONNX (Open Neural Network Exchange)
    - TensorFlow
    - TensorFlowLite
    - CoreML (iOS)
    - DockerFile
    """
    print(f"Exporting model as {platform}...")
    
    export_result = trainer.export_iteration(
        project_id,
        iteration_id,
        platform
    )
    
    # Wait for export to complete
    while export_result.status == "Exporting":
        export_result = trainer.get_exports(project_id, iteration_id)[0]
        print(f"Export status: {export_result.status}")
        time.sleep(5)
    
    if export_result.status == "Done":
        print(f"Export completed!")
        print(f"Download URL: {export_result.download_uri}")
        return export_result.download_uri
    else:
        raise Exception(f"Export failed: {export_result.status}")

# Example usage
if __name__ == "__main__":
    # Classification example
    project = create_classification_project("Product Classifier", is_multilabel=False)
    
    training_images = {
        "shoes": ["training/shoes/img1.jpg", "training/shoes/img2.jpg"],
        "shirts": ["training/shirts/img1.jpg", "training/shirts/img2.jpg"],
        "pants": ["training/pants/img1.jpg", "training/pants/img2.jpg"]
    }
    
    add_images_to_classification_project(project.id, training_images)
    iteration = train_classification_model(project.id)
    publish_iteration(project.id, iteration.id, "production")
    
    # Classify new image
    predictions = classify_image(project.id, "production", "test/new_product.jpg")
```

### 2.5 Video Analysis

Azure Video Indexer provides comprehensive video analysis capabilities including:
- Scene detection
- Shot detection
- Keyframe extraction
- OCR in video frames
- Object tracking
- Person tracking
- Transcript extraction
- Translation
- Topic and keyword extraction
- Sentiment analysis of transcript
- Custom brand and person models

**Note:** Video Indexer has its own portal and API, separate from the main Azure AI Vision API.

```python
import requests
import time
import os

class VideoIndexerClient:
    """Azure Video Indexer API client"""
    
    def __init__(self, account_id: str, api_key: str, location: str = "trial"):
        self.account_id = account_id
        self.api_key = api_key
        self.location = location
        self.base_url = f"https://api.videoindexer.ai"
    
    def get_access_token(self, permission_type: str = "Contributor") -> str:
        """
        Get access token for API calls
        permission_type: "Reader" or "Contributor"
        """
        url = f"{self.base_url}/Auth/{self.location}/Accounts/{self.account_id}/AccessToken"
        
        params = {
            "allowEdit": "true" if permission_type == "Contributor" else "false"
        }
        
        headers = {
            "Ocp-Apim-Subscription-Key": self.api_key
        }
        
        response = requests.get(url, params=params, headers=headers)
        response.raise_for_status()
        
        return response.json()
    
    def upload_video(self, video_path: str, video_name: str, 
                     language: str = "en-US", privacy: str = "Private") -> str:
        """
        Upload and index a video
        """
        access_token = self.get_access_token("Contributor")
        
        url = f"{self.base_url}/{self.location}/Accounts/{self.account_id}/Videos"
        
        params = {
            "accessToken": access_token,
            "name": video_name,
            "language": language,
            "privacy": privacy,
            "videoUrl": ""  # or provide URL instead of file upload
        }
        
        # Read video file
        with open(video_path, "rb") as video_file:
            files = {"file": video_file}
            response = requests.post(url, params=params, files=files)
            response.raise_for_status()
        
        result = response.json()
        video_id = result["id"]
        
        print(f"Video uploaded with ID: {video_id}")
        print("Indexing in progress...")
        
        return video_id
    
    def get_video_index(self, video_id: str) -> dict:
        """
        Get video index and insights
        """
        access_token = self.get_access_token("Reader")
        
        url = f"{self.base_url}/{self.location}/Accounts/{self.account_id}/Videos/{video_id}/Index"
        
        params = {
            "accessToken": access_token,
            "language": "en-US"
        }
        
        response = requests.get(url, params=params)
        response.raise_for_status()
        
        return response.json()
    
    def wait_for_indexing(self, video_id: str, max_wait: int = 600) -> dict:
        """
        Wait for video indexing to complete
        """
        start_time = time.time()
        
        while time.time() - start_time < max_wait:
            index = self.get_video_index(video_id)
            state = index["state"]
            
            print(f"Indexing state: {state}")
            
            if state == "Processed":
                print("Indexing completed!")
                return index
            elif state == "Failed":
                raise Exception("Indexing failed")
            
            time.sleep(10)
        
        raise TimeoutError("Indexing timed out")
    
    def extract_insights(self, index: dict) -> dict:
        """
        Extract structured insights from video index
        """
        insights = {
            "duration": index["videos"][0]["duration"],
            "faces": [],
            "keywords": [],
            "topics": [],
            "scenes": [],
            "shots": [],
            "transcript": [],
            "ocr": [],
            "labels": [],
            "sentiments": []
        }
        
        video_insights = index["videos"][0]["insights"]
        
        # Faces
        if "faces" in video_insights:
            for face in video_insights["faces"]:
                insights["faces"].append({
                    "id": face["id"],
                    "name": face.get("name", "Unknown"),
                    "confidence": face.get("confidence", 0),
                    "appearances": len(face["instances"])
                })
        
        # Keywords
        if "keywords" in video_insights:
            insights["keywords"] = [kw["text"] for kw in video_insights["keywords"]]
        
        # Topics
        if "topics" in video_insights:
            insights["topics"] = [topic["name"] for topic in video_insights["topics"]]
        
        # Scenes
        if "scenes" in video_insights:
            for scene in video_insights["scenes"]:
                insights["scenes"].append({
                    "id": scene["id"],
                    "start": scene["instances"][0]["start"],
                    "end": scene["instances"][0]["end"]
                })
        
        # Shots
        if "shots" in video_insights:
            for shot in video_insights["shots"]:
                keyframe = shot["keyFrames"][0] if shot.get("keyFrames") else None
                insights["shots"].append({
                    "id": shot["id"],
                    "start": shot["instances"][0]["start"],
                    "end": shot["instances"][0]["end"],
                    "keyframe_id": keyframe["instances"][0]["thumbnailId"] if keyframe else None
                })
        
        # Transcript
        if "transcript" in video_insights:
            for line in video_insights["transcript"]:
                insights["transcript"].append({
                    "text": line["text"],
                    "start": line["instances"][0]["start"],
                    "end": line["instances"][0]["end"],
                    "confidence": line.get("confidence", 0)
                })
        
        # OCR
        if "ocr" in video_insights:
            for ocr_item in video_insights["ocr"]:
                insights["ocr"].append({
                    "text": ocr_item["text"],
                    "confidence": ocr_item.get("confidence", 0),
                    "appearances": len(ocr_item["instances"])
                })
        
        # Labels (objects, activities)
        if "labels" in video_insights:
            for label in video_insights["labels"]:
                insights["labels"].append({
                    "name": label["name"],
                    "appearances": len(label["instances"])
                })
        
        # Sentiments
        if "sentiments" in video_insights:
            for sentiment in video_insights["sentiments"]:
                insights["sentiments"].append({
                    "sentiment_type": sentiment["sentimentType"],
                    "average_score": sentiment["averageScore"],
                    "start": sentiment["instances"][0]["start"],
                    "end": sentiment["instances"][0]["end"]
                })
        
        return insights
    
    def download_thumbnail(self, video_id: str, thumbnail_id: str, 
                          output_path: str):
        """
        Download a video thumbnail/keyframe
        """
        access_token = self.get_access_token("Reader")
        
        url = f"{self.base_url}/{self.location}/Accounts/{self.account_id}/Videos/{video_id}/Thumbnails/{thumbnail_id}"
        
        params = {
            "accessToken": access_token,
            "format": "Jpeg"  # or "Base64"
        }
        
        response = requests.get(url, params=params)
        response.raise_for_status()
        
        with open(output_path, "wb") as f:
            f.write(response.content)
        
        print(f"Thumbnail saved to: {output_path}")

# Example usage
if __name__ == "__main__":
    client = VideoIndexerClient(
        account_id=os.environ["VIDEO_INDEXER_ACCOUNT_ID"],
        api_key=os.environ["VIDEO_INDEXER_API_KEY"],
        location="eastus"  # or "trial" for trial account
    )
    
    # Upload and index video
    video_id = client.upload_video(
        video_path="sample_video.mp4",
        video_name="Sample Video Analysis",
        language="en-US"
    )
    
    # Wait for indexing
    index = client.wait_for_indexing(video_id)
    
    # Extract insights
    insights = client.extract_insights(index)
    
    print("\n=== Video Insights ===")
    print(f"Duration: {insights['duration']}")
    print(f"Faces detected: {len(insights['faces'])}")
    print(f"Keywords: {', '.join(insights['keywords'][:10])}")
    print(f"Topics: {', '.join(insights['topics'])}")
    print(f"Scenes: {len(insights['scenes'])}")
    print(f"Shots: {len(insights['shots'])}")
```

---

## 3. AZURE AI DOCUMENT INTELLIGENCE

Azure AI Document Intelligence (formerly Form Recognizer) extracts text, key-value pairs, tables, and structures from documents.

### 3.1 Prebuilt Models

Microsoft provides prebuilt models for common document types. No training required.

**Available Prebuilt Models (2025):**
- **read**: General text extraction (OCR)
- **layout**: Document structure (tables, paragraphs, headers)
- **general-document**: Key-value pairs from any document
- **invoice**: Invoices
- **receipt**: Receipts
- **business-card**: Business cards
- **identity-document**: Passports, driver's licenses, ID cards
- **w2**: US W-2 tax forms
- **health-insurance-card**: US health insurance cards
- **vaccination-card**: CDC vaccination cards
- **contract**: Contracts
- **marriage-certificate**: Marriage certificates

```python
from azure.ai.formrecognizer import DocumentAnalysisClient
from azure.core.credentials import AzureKeyCredential
import os

# Initialize client
endpoint = os.environ["DOCUMENT_INTELLIGENCE_ENDPOINT"]
key = os.environ["DOCUMENT_INTELLIGENCE_KEY"]

document_client = DocumentAnalysisClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(key)
)

def analyze_invoice(file_path: str) -> dict:
    """
    Analyze invoice using prebuilt model
    Extracts all standard invoice fields
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-invoice",
            document=f
        )
    
    result = poller.result()
    
    invoice_data = {
        "vendor": {},
        "customer": {},
        "invoice_details": {},
        "items": [],
        "totals": {}
    }
    
    # Process each invoice (can be multiple in one document)
    for document in result.documents:
        fields = document.fields
        
        # Vendor information
        if fields.get("VendorName"):
            invoice_data["vendor"]["name"] = fields["VendorName"].value
        if fields.get("VendorAddress"):
            invoice_data["vendor"]["address"] = fields["VendorAddress"].value
        if fields.get("VendorTaxId"):
            invoice_data["vendor"]["tax_id"] = fields["VendorTaxId"].value
        
        # Customer information
        if fields.get("CustomerName"):
            invoice_data["customer"]["name"] = fields["CustomerName"].value
        if fields.get("CustomerAddress"):
            invoice_data["customer"]["address"] = fields["CustomerAddress"].value
        if fields.get("CustomerTaxId"):
            invoice_data["customer"]["tax_id"] = fields["CustomerTaxId"].value
        
        # Invoice details
        if fields.get("InvoiceId"):
            invoice_data["invoice_details"]["id"] = fields["InvoiceId"].value
        if fields.get("InvoiceDate"):
            invoice_data["invoice_details"]["date"] = fields["InvoiceDate"].value
        if fields.get("DueDate"):
            invoice_data["invoice_details"]["due_date"] = fields["DueDate"].value
        if fields.get("PurchaseOrder"):
            invoice_data["invoice_details"]["po_number"] = fields["PurchaseOrder"].value
        
        # Line items
        if fields.get("Items"):
            for item in fields["Items"].value:
                item_fields = item.value
                item_data = {}
                
                if item_fields.get("Description"):
                    item_data["description"] = item_fields["Description"].value
                if item_fields.get("Quantity"):
                    item_data["quantity"] = item_fields["Quantity"].value
                if item_fields.get("UnitPrice"):
                    item_data["unit_price"] = item_fields["UnitPrice"].value
                if item_fields.get("Amount"):
                    item_data["amount"] = item_fields["Amount"].value
                if item_fields.get("Tax"):
                    item_data["tax"] = item_fields["Tax"].value
                if item_fields.get("ProductCode"):
                    item_data["product_code"] = item_fields["ProductCode"].value
                
                invoice_data["items"].append(item_data)
        
        # Totals
        if fields.get("SubTotal"):
            invoice_data["totals"]["subtotal"] = fields["SubTotal"].value
        if fields.get("TotalTax"):
            invoice_data["totals"]["tax"] = fields["TotalTax"].value
        if fields.get("InvoiceTotal"):
            invoice_data["totals"]["total"] = fields["InvoiceTotal"].value
        if fields.get("AmountDue"):
            invoice_data["totals"]["amount_due"] = fields["AmountDue"].value
    
    # Print summary
    print(f"Invoice Analysis Results:")
    print(f"  Vendor: {invoice_data['vendor'].get('name', 'N/A')}")
    print(f"  Invoice ID: {invoice_data['invoice_details'].get('id', 'N/A')}")
    print(f"  Date: {invoice_data['invoice_details'].get('date', 'N/A')}")
    print(f"  Total: {invoice_data['totals'].get('total', 'N/A')}")
    print(f"  Line items: {len(invoice_data['items'])}")
    
    return invoice_data

def analyze_receipt(file_path: str) -> dict:
    """
    Analyze receipt using prebuilt model
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-receipt",
            document=f,
            locale="en-US"  # Specify locale for better accuracy
        )
    
    result = poller.result()
    
    receipt_data = {
        "merchant": {},
        "transaction": {},
        "items": [],
        "totals": {}
    }
    
    for receipt in result.documents:
        fields = receipt.fields
        
        # Merchant information
        if fields.get("MerchantName"):
            receipt_data["merchant"]["name"] = fields["MerchantName"].value
        if fields.get("MerchantAddress"):
            receipt_data["merchant"]["address"] = fields["MerchantAddress"].value
        if fields.get("MerchantPhoneNumber"):
            receipt_data["merchant"]["phone"] = fields["MerchantPhoneNumber"].value
        
        # Transaction details
        if fields.get("TransactionDate"):
            receipt_data["transaction"]["date"] = fields["TransactionDate"].value
        if fields.get("TransactionTime"):
            receipt_data["transaction"]["time"] = fields["TransactionTime"].value
        
        # Items
        if fields.get("Items"):
            for item in fields["Items"].value:
                item_fields = item.value
                item_data = {}
                
                if item_fields.get("Description"):
                    item_data["description"] = item_fields["Description"].value
                if item_fields.get("Quantity"):
                    item_data["quantity"] = item_fields["Quantity"].value
                if item_fields.get("TotalPrice"):
                    item_data["total_price"] = item_fields["TotalPrice"].value
                
                receipt_data["items"].append(item_data)
        
        # Totals
        if fields.get("Subtotal"):
            receipt_data["totals"]["subtotal"] = fields["Subtotal"].value
        if fields.get("Tax"):
            receipt_data["totals"]["tax"] = fields["Tax"].value
        if fields.get("Tip"):
            receipt_data["totals"]["tip"] = fields["Tip"].value
        if fields.get("Total"):
            receipt_data["totals"]["total"] = fields["Total"].value
    
    print(f"Receipt Analysis Results:")
    print(f"  Merchant: {receipt_data['merchant'].get('name', 'N/A')}")
    print(f"  Date: {receipt_data['transaction'].get('date', 'N/A')}")
    print(f"  Total: {receipt_data['totals'].get('total', 'N/A')}")
    print(f"  Items: {len(receipt_data['items'])}")
    
    return receipt_data

def analyze_business_card(file_path: str) -> dict:
    """
    Analyze business card
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-businessCard",
            document=f
        )
    
    result = poller.result()
    
    card_data = {
        "contact_names": [],
        "job_titles": [],
        "departments": [],
        "emails": [],
        "websites": [],
        "mobile_phones": [],
        "work_phones": [],
        "faxes": [],
        "addresses": [],
        "company_names": []
    }
    
    for card in result.documents:
        fields = card.fields
        
        # Extract all fields
        if fields.get("ContactNames"):
            for name in fields["ContactNames"].value:
                card_data["contact_names"].append(name.value.get("FirstName", "") + " " + name.value.get("LastName", ""))
        
        if fields.get("JobTitles"):
            for title in fields["JobTitles"].value:
                card_data["job_titles"].append(title.value)
        
        if fields.get("Departments"):
            for dept in fields["Departments"].value:
                card_data["departments"].append(dept.value)
        
        if fields.get("Emails"):
            for email in fields["Emails"].value:
                card_data["emails"].append(email.value)
        
        if fields.get("Websites"):
            for website in fields["Websites"].value:
                card_data["websites"].append(website.value)
        
        if fields.get("MobilePhones"):
            for phone in fields["MobilePhones"].value:
                card_data["mobile_phones"].append(phone.value)
        
        if fields.get("WorkPhones"):
            for phone in fields["WorkPhones"].value:
                card_data["work_phones"].append(phone.value)
        
        if fields.get("Addresses"):
            for address in fields["Addresses"].value:
                card_data["addresses"].append(address.value)
        
        if fields.get("CompanyNames"):
            for company in fields["CompanyNames"].value:
                card_data["company_names"].append(company.value)
    
    print(f"Business Card Analysis:")
    print(f"  Name: {', '.join(card_data['contact_names'])}")
    print(f"  Title: {', '.join(card_data['job_titles'])}")
    print(f"  Company: {', '.join(card_data['company_names'])}")
    print(f"  Email: {', '.join(card_data['emails'])}")
    
    return card_data

def analyze_identity_document(file_path: str) -> dict:
    """
    Analyze identity document (passport, driver's license, ID card)
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-idDocument",
            document=f
        )
    
    result = poller.result()
    
    id_data = {
        "document_type": None,
        "personal_info": {},
        "document_info": {},
        "address": {}
    }
    
    for document in result.documents:
        fields = document.fields
        
        # Document type
        id_data["document_type"] = document.doc_type
        
        # Personal information
        if fields.get("FirstName"):
            id_data["personal_info"]["first_name"] = fields["FirstName"].value
        if fields.get("LastName"):
            id_data["personal_info"]["last_name"] = fields["LastName"].value
        if fields.get("DateOfBirth"):
            id_data["personal_info"]["date_of_birth"] = fields["DateOfBirth"].value
        if fields.get("Sex"):
            id_data["personal_info"]["sex"] = fields["Sex"].value
        
        # Document information
        if fields.get("DocumentNumber"):
            id_data["document_info"]["number"] = fields["DocumentNumber"].value
        if fields.get("DateOfExpiration"):
            id_data["document_info"]["expiration_date"] = fields["DateOfExpiration"].value
        if fields.get("DateOfIssue"):
            id_data["document_info"]["issue_date"] = fields["DateOfIssue"].value
        if fields.get("CountryRegion"):
            id_data["document_info"]["country"] = fields["CountryRegion"].value
        
        # Address
        if fields.get("Address"):
            id_data["address"]["full"] = fields["Address"].value
    
    print(f"ID Document Analysis:")
    print(f"  Type: {id_data['document_type']}")
    print(f"  Name: {id_data['personal_info'].get('first_name', '')} {id_data['personal_info'].get('last_name', '')}")
    print(f"  DOB: {id_data['personal_info'].get('date_of_birth', 'N/A')}")
    print(f"  Document #: {id_data['document_info'].get('number', 'N/A')}")
    
    return id_data

def analyze_w2_form(file_path: str) -> dict:
    """
    Analyze US W-2 tax form
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-tax.us.w2",
            document=f
        )
    
    result = poller.result()
    
    w2_data = {
        "employer": {},
        "employee": {},
        "wages": {},
        "tax_withheld": {}
    }
    
    for document in result.documents:
        fields = document.fields
        
        # Employer
        if fields.get("Employer"):
            employer_fields = fields["Employer"].value
            if employer_fields.get("Name"):
                w2_data["employer"]["name"] = employer_fields["Name"].value
            if employer_fields.get("IdNumber"):
                w2_data["employer"]["ein"] = employer_fields["IdNumber"].value
            if employer_fields.get("Address"):
                w2_data["employer"]["address"] = employer_fields["Address"].value
        
        # Employee
        if fields.get("Employee"):
            employee_fields = fields["Employee"].value
            if employee_fields.get("Name"):
                w2_data["employee"]["name"] = employee_fields["Name"].value
            if employee_fields.get("SocialSecurityNumber"):
                w2_data["employee"]["ssn"] = employee_fields["SocialSecurityNumber"].value
            if employee_fields.get("Address"):
                w2_data["employee"]["address"] = employee_fields["Address"].value
        
        # Wages and compensation
        if fields.get("WagesTipsAndOtherCompensation"):
            w2_data["wages"]["total"] = fields["WagesTipsAndOtherCompensation"].value
        if fields.get("SocialSecurityWages"):
            w2_data["wages"]["ss_wages"] = fields["SocialSecurityWages"].value
        if fields.get("MedicareWagesAndTips"):
            w2_data["wages"]["medicare_wages"] = fields["MedicareWagesAndTips"].value
        
        # Tax withheld
        if fields.get("FederalIncomeTaxWithheld"):
            w2_data["tax_withheld"]["federal"] = fields["FederalIncomeTaxWithheld"].value
        if fields.get("SocialSecurityTaxWithheld"):
            w2_data["tax_withheld"]["ss"] = fields["SocialSecurityTaxWithheld"].value
        if fields.get("MedicareTaxWithheld"):
            w2_data["tax_withheld"]["medicare"] = fields["MedicareTaxWithheld"].value
    
    print(f"W-2 Analysis:")
    print(f"  Employer: {w2_data['employer'].get('name', 'N/A')}")
    print(f"  Employee: {w2_data['employee'].get('name', 'N/A')}")
    print(f"  Total Wages: {w2_data['wages'].get('total', 'N/A')}")
    print(f"  Federal Tax Withheld: {w2_data['tax_withheld'].get('federal', 'N/A')}")
    
    return w2_data
```

### 3.2 Layout Analysis Model

Extract document structure: paragraphs, tables, selection marks, reading order.

```python
def analyze_layout(file_path: str) -> dict:
    """
    Analyze document layout and structure
    Extracts: text, tables, selection marks, paragraphs, sections
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-layout",
            document=f
        )
    
    result = poller.result()
    
    layout_data = {
        "pages": [],
        "tables": [],
        "paragraphs": [],
        "selection_marks": []
    }
    
    # Process pages
    for page in result.pages:
        page_data = {
            "page_number": page.page_number,
            "width": page.width,
            "height": page.height,
            "unit": page.unit,
            "angle": page.angle,
            "lines": [],
            "words": []
        }
        
        # Lines
        if page.lines:
            for line in page.lines:
                page_data["lines"].append({
                    "content": line.content,
                    "polygon": [(p.x, p.y) for p in line.polygon]
                })
        
        # Words
        if page.words:
            for word in page.words:
                page_data["words"].append({
                    "content": word.content,
                    "confidence": word.confidence,
                    "polygon": [(p.x, p.y) for p in word.polygon]
                })
        
        # Selection marks (checkboxes)
        if page.selection_marks:
            for mark in page.selection_marks:
                layout_data["selection_marks"].append({
                    "state": mark.state,  # "selected" or "unselected"
                    "confidence": mark.confidence,
                    "polygon": [(p.x, p.y) for p in mark.polygon],
                    "page": page.page_number
                })
        
        layout_data["pages"].append(page_data)
    
    # Process tables
    if result.tables:
        for table in result.tables:
            table_data = {
                "row_count": table.row_count,
                "column_count": table.column_count,
                "cells": [],
                "page": table.bounding_regions[0].page_number if table.bounding_regions else None
            }
            
            for cell in table.cells:
                table_data["cells"].append({
                    "row_index": cell.row_index,
                    "column_index": cell.column_index,
                    "content": cell.content,
                    "row_span": cell.row_span,
                    "column_span": cell.column_span,
                    "kind": cell.kind  # "content", "rowHeader", "columnHeader"
                })
            
            layout_data["tables"].append(table_data)
    
    # Process paragraphs
    if result.paragraphs:
        for paragraph in result.paragraphs:
            layout_data["paragraphs"].append({
                "content": paragraph.content,
                "role": paragraph.role,  # "title", "sectionHeading", "pageHeader", etc.
                "page": paragraph.bounding_regions[0].page_number if paragraph.bounding_regions else None
            })
    
    # Print summary
    print(f"Layout Analysis Summary:")
    print(f"  Pages: {len(layout_data['pages'])}")
    print(f"  Tables: {len(layout_data['tables'])}")
    print(f"  Paragraphs: {len(layout_data['paragraphs'])}")
    print(f"  Selection marks: {len(layout_data['selection_marks'])}")
    
    # Print table structure
    for i, table in enumerate(layout_data["tables"]):
        print(f"\nTable {i+1} ({table['row_count']}x{table['column_count']}):")
        
        # Create 2D array
        grid = [['' for _ in range(table['column_count'])] for _ in range(table['row_count'])]
        for cell in table['cells']:
            grid[cell['row_index']][cell['column_index']] = cell['content']
        
        # Print table
        for row in grid:
            print("  | " + " | ".join(row) + " |")
    
    return layout_data
```

### 3.3 Custom Models

Train custom models for your specific document types.

```python
from azure.ai.formrecognizer import DocumentModelAdministrationClient

# Initialize admin client for model management
admin_client = DocumentModelAdministrationClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(key)
)

def train_custom_template_model(training_data_url: str, model_id: str) -> str:
    """
    Train custom template model for structured documents
    
    Training data requirements:
    - At least 5 example documents (same layout)
    - Stored in Azure Blob Storage
    - Labeled using Document Intelligence Studio or FOTT tool
    - Must include .ocr.json and .labels.json files
    """
    
    print(f"Training custom template model: {model_id}")
    
    poller = admin_client.begin_build_document_model(
        build_mode="template",  # For structured documents with fixed layout
        blob_container_url=training_data_url,
        model_id=model_id,
        description="Custom template model for company-specific forms"
    )
    
    model = poller.result()
    
    print(f"Model trained successfully!")
    print(f"  Model ID: {model.model_id}")
    print(f"  Created: {model.created_on}")
    print(f"  Document types:")
    
    for doc_type_name, doc_type in model.doc_types.items():
        print(f"    {doc_type_name}:")
        print(f"      Description: {doc_type.description}")
        print(f"      Fields:")
        for field_name, field_schema in doc_type.field_schema.items():
            print(f"        - {field_name}: {field_schema.get('type', 'unknown')}")
    
    return model.model_id

def train_custom_neural_model(training_data_url: str, model_id: str) -> str:
    """
    Train custom neural model for unstructured/semi-structured documents
    
    Training data requirements:
    - At least 5 example documents
    - Can handle varying layouts
    - More flexible than template model
    - Better for complex scenarios
    """
    
    print(f"Training custom neural model: {model_id}")
    
    poller = admin_client.begin_build_document_model(
        build_mode="neural",  # For varying layouts
        blob_container_url=training_data_url,
        model_id=model_id,
        description="Custom neural model for varying document layouts"
    )
    
    model = poller.result()
    
    print(f"Neural model trained successfully!")
    print(f"  Model ID: {model.model_id}")
    
    return model.model_id

def train_custom_classification_model(training_data_url: str, model_id: str) -> str:
    """
    Train document classification model
    Classifies documents into categories
    
    Training data requirements:
    - At least 5 examples per category
    - Organized in subfolders by category in blob storage
    """
    
    print(f"Training document classification model: {model_id}")
    
    poller = admin_client.begin_build_document_classifier(
        blob_container_url=training_data_url,
        classifier_id=model_id,
        description="Document classifier for categorizing incoming documents"
    )
    
    classifier = poller.result()
    
    print(f"Classifier trained successfully!")
    print(f"  Classifier ID: {classifier.classifier_id}")
    print(f"  Document types:")
    for doc_type, details in classifier.doc_types.items():
        print(f"    - {doc_type}")
    
    return classifier.classifier_id

def analyze_with_custom_model(model_id: str, file_path: str) -> dict:
    """
    Analyze document with custom trained model
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            model_id=model_id,
            document=f
        )
    
    result = poller.result()
    
    extracted_data = {}
    
    # Process each document
    for document in result.documents:
        print(f"\nDocument type: {document.doc_type}")
        print(f"Confidence: {document.confidence}")
        
        # Extract all fields
        for field_name, field_value in document.fields.items():
            extracted_data[field_name] = {
                "value": field_value.value if field_value else None,
                "confidence": field_value.confidence if field_value else 0
            }
            
            if field_value:
                print(f"  {field_name}: {field_value.value} (confidence: {field_value.confidence:.2f})")
    
    return extracted_data

def compose_models(component_model_ids: list, composed_model_id: str) -> str:
    """
    Compose multiple custom models into a single model
    Useful for handling multiple document types with one API call
    """
    
    print(f"Composing models: {', '.join(component_model_ids)}")
    
    poller = admin_client.begin_compose_document_model(
        component_model_ids=component_model_ids,
        model_id=composed_model_id,
        description="Composed model for multiple document types"
    )
    
    composed_model = poller.result()
    
    print(f"Composed model created!")
    print(f"  Model ID: {composed_model.model_id}")
    print(f"  Component models: {len(component_model_ids)}")
    
    return composed_model.model_id

def list_models():
    """
    List all custom models in the resource
    """
    
    models = admin_client.list_document_models()
    
    print("Custom Models:")
    for model in models:
        print(f"  Model ID: {model.model_id}")
        print(f"    Created: {model.created_on}")
        print(f"    Description: {model.description}")
        print(f"    Tags: {model.tags}")
        print()

def delete_model(model_id: str):
    """
    Delete a custom model
    """
    admin_client.delete_document_model(model_id)
    print(f"Model deleted: {model_id}")

# Example usage
if __name__ == "__main__":
    # Train custom template model
    training_url = "https://mystorageaccount.blob.core.windows.net/training-data?sas_token"
    model_id = train_custom_template_model(training_url, "purchase-order-model")
    
    # Analyze document with custom model
    results = analyze_with_custom_model(model_id, "new_purchase_order.pdf")
    
    # Compose multiple models
    invoice_model_id = "invoice-model-v1"
    receipt_model_id = "receipt-model-v1"
    composed_id = compose_models(
        [invoice_model_id, receipt_model_id],
        "finance-documents-model"
    )
    
    # List all models
    list_models()
```

### 3.4 General Document Model

Extract key-value pairs from any document without training.

```python
def analyze_general_document(file_path: str) -> dict:
    """
    Analyze any document and extract key-value pairs
    No training required
    """
    
    with open(file_path, "rb") as f:
        poller = document_client.begin_analyze_document(
            "prebuilt-document",  # General document model
            document=f
        )
    
    result = poller.result()
    
    general_data = {
        "key_value_pairs": [],
        "tables": [],
        "entities": []
    }
    
    # Extract key-value pairs
    if result.key_value_pairs:
        for kv_pair in result.key_value_pairs:
            if kv_pair.key and kv_pair.value:
                general_data["key_value_pairs"].append({
                    "key": kv_pair.key.content,
                    "value": kv_pair.value.content,
                    "confidence": kv_pair.confidence
                })
                
                print(f"{kv_pair.key.content}: {kv_pair.value.content} "
                      f"(confidence: {kv_pair.confidence:.2f})")
    
    # Extract tables
    if result.tables:
        for table in result.tables:
            table_data = {
                "row_count": table.row_count,
                "column_count": table.column_count,
                "cells": []
            }
            
            for cell in table.cells:
                table_data["cells"].append({
                    "row": cell.row_index,
                    "column": cell.column_index,
                    "content": cell.content
                })
            
            general_data["tables"].append(table_data)
    
    return general_data
```

---

## 4. AZURE AI LANGUAGE

Azure AI Language provides comprehensive natural language processing capabilities.

### 4.1 Text Analytics API

#### 4.1.1 Sentiment Analysis with Opinion Mining

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

# Initialize client
endpoint = os.environ["LANGUAGE_ENDPOINT"]
key = os.environ["LANGUAGE_KEY"]

client = TextAnalyticsClient(endpoint=endpoint, credential=AzureKeyCredential(key))

def analyze_sentiment_with_opinions(documents: list) -> list:
    """
    Analyze sentiment with aspect-based sentiment analysis (opinion mining)
    
    Returns document-level and sentence-level sentiment
    Plus opinions about specific aspects
    """
    
    # Enable opinion mining for aspect-based sentiment
    response = client.analyze_sentiment(
        documents=documents,
        show_opinion_mining=True,
        language="en"
    )
    
    results = []
    
    for idx, doc in enumerate(response):
        if doc.is_error:
            print(f"Document {idx} error: {doc.error}")
            continue
        
        doc_result = {
            "document_sentiment": doc.sentiment,
            "confidence_scores": {
                "positive": doc.confidence_scores.positive,
                "neutral": doc.confidence_scores.neutral,
                "negative": doc.confidence_scores.negative
            },
            "sentences": []
        }
        
        print(f"\nDocument sentiment: {doc.sentiment}")
        print(f"  Positive: {doc.confidence_scores.positive:.2f}")
        print(f"  Neutral: {doc.confidence_scores.neutral:.2f}")
        print(f"  Negative: {doc.confidence_scores.negative:.2f}")
        
        # Sentence-level sentiment
        for sentence in doc.sentences:
            sentence_data = {
                "text": sentence.text,
                "sentiment": sentence.sentiment,
                "confidence_scores": {
                    "positive": sentence.confidence_scores.positive,
                    "neutral": sentence.confidence_scores.neutral,
                    "negative": sentence.confidence_scores.negative
                },
                "opinions": []
            }
            
            print(f"\n  Sentence: {sentence.text}")
            print(f"    Sentiment: {sentence.sentiment}")
            
            # Opinion mining (targets and assessments)
            if hasattr(sentence, 'mined_opinions'):
                for opinion in sentence.mined_opinions:
                    # Target (what is being discussed)
                    target = opinion.target
                    opinion_data = {
                        "target": {
                            "text": target.text,
                            "sentiment": target.sentiment,
                            "confidence_scores": {
                                "positive": target.confidence_scores.positive,
                                "negative": target.confidence_scores.negative
                            }
                        },
                        "assessments": []
                    }
                    
                    print(f"\n    Target: {target.text}")
                    print(f"      Sentiment: {target.sentiment}")
                    
                    # Assessments (opinions about the target)
                    for assessment in opinion.assessments:
                        assessment_data = {
                            "text": assessment.text,
                            "sentiment": assessment.sentiment,
                            "confidence_scores": {
                                "positive": assessment.confidence_scores.positive,
                                "negative": assessment.confidence_scores.negative
                            },
                            "is_negated": assessment.is_negated
                        }
                        
                        opinion_data["assessments"].append(assessment_data)
                        
                        print(f"      Assessment: {assessment.text}")
                        print(f"        Sentiment: {assessment.sentiment}")
                        print(f"        Negated: {assessment.is_negated}")
                    
                    sentence_data["opinions"].append(opinion_data)
            
            doc_result["sentences"].append(sentence_data)
        
        results.append(doc_result)
    
    return results

# Example usage
reviews = [
    "The food was delicious but the service was terrible.",
    "Great ambiance and friendly staff, but the prices are too high.",
    "The hotel room was spacious and clean. The bed was very comfortable."
]

results = analyze_sentiment_with_opinions(reviews)
```

#### 4.1.2 Named Entity Recognition (NER)

```python
def recognize_entities(documents: list) -> list:
    """
    Recognize named entities in text
    
    Entity categories:
    - Person, Location, Organization, DateTime, Quantity
    - Email, URL, PhoneNumber, IPAddress
    - Event, Product, Skill, Address
    """
    
    response = client.recognize_entities(
        documents=documents,
        language="en"
    )
    
    results = []
    
    for idx, doc in enumerate(response):
        if doc.is_error:
            print(f"Document {idx} error: {doc.error}")
            continue
        
        doc_entities = []
        
        print(f"\nDocument {idx + 1}:")
        for entity in doc.entities:
            entity_data = {
                "text": entity.text,
                "category": entity.category,
                "subcategory": entity.subcategory,
                "confidence_score": entity.confidence_score,
                "offset": entity.offset,
                "length": entity.length
            }
            
            doc_entities.append(entity_data)
            
            print(f"  Entity: {entity.text}")
            print(f"    Category: {entity.category}")
            if entity.subcategory:
                print(f"    Subcategory: {entity.subcategory}")
            print(f"    Confidence: {entity.confidence_score:.2f}")
        
        results.append(doc_entities)
    
    return results

def recognize_pii_entities(documents: list, domain: str = "phi") -> list:
    """
    Recognize Personally Identifiable Information (PII)
    
    Domains:
    - 'phi' (Protected Health Information) - HIPAA compliant
    - 'none' (general PII)
    
    PII categories:
    - SSN, Credit Card, Bank Account, Driver's License
    - Passport, Email, Phone, IP Address
    - Person Name, Address, Date of Birth
    """
    
    response = client.recognize_pii_entities(
        documents=documents,
        language="en",
        domain_filter=domain
    )
    
    results = []
    
    for idx, doc in enumerate(response):
        if doc.is_error:
            print(f"Document {idx} error: {doc.error}")
            continue
        
        doc_pii = {
            "redacted_text": doc.redacted_text,
            "entities": []
        }
        
        print(f"\nDocument {idx + 1}:")
        print(f"  Original: {documents[idx]}")
        print(f"  Redacted: {doc.redacted_text}")
        print(f"\n  PII Entities:")
        
        for entity in doc.entities:
            entity_data = {
                "text": entity.text,
                "category": entity.category,
                "subcategory": entity.subcategory,
                "confidence_score": entity.confidence_score
            }
            
            doc_pii["entities"].append(entity_data)
            
            print(f"    {entity.text} - {entity.category}")
            if entity.subcategory:
                print(f"      Subcategory: {entity.subcategory}")
        
        results.append(doc_pii)
    
    return results

def recognize_linked_entities(documents: list) -> list:
    """
    Recognize entities and link them to Wikipedia knowledge base
    """
    
    response = client.recognize_linked_entities(
        documents=documents,
        language="en"
    )
    
    results = []
    
    for idx, doc in enumerate(response):
        if doc.is_error:
            print(f"Document {idx} error: {doc.error}")
            continue
        
        doc_entities = []
        
        print(f"\nDocument {idx + 1}:")
        for entity in doc.entities:
            entity_data = {
                "name": entity.name,
                "url": entity.url,
                "data_source": entity.data_source,
                "matches": []
            }
            
            print(f"  Entity: {entity.name}")
            print(f"    Wikipedia URL: {entity.url}")
            print(f"    Data Source: {entity.data_source}")
            
            for match in entity.matches:
                match_data = {
                    "text": match.text,
                    "confidence_score": match.confidence_score
                }
                entity_data["matches"].append(match_data)
                print(f"    Match: {match.text} (confidence: {match.confidence_score:.2f})")
            
            doc_entities.append(entity_data)
        
        results.append(doc_entities)
    
    return results

# Example usage
texts = [
    "John Smith works at Microsoft in Seattle. His SSN is 123-45-6789.",
    "Contact me at john.doe@example.com or call +1-555-0123."
]

entities = recognize_entities(texts)
pii_entities = recognize_pii_entities(texts)
linked = recognize_linked_entities(["Microsoft was founded by Bill Gates in Seattle."])
```

#### 4.1.3 Key Phrase Extraction

```python
def extract_key_phrases(documents: list) -> list:
    """
    Extract key phrases from documents
    """
    
    response = client.extract_key_phrases(
        documents=documents,
        language="en"
    )
    
    results = []
    
    for idx, doc in enumerate(response):
        if doc.is_error:
            print(f"Document {idx} error: {doc.error}")
            continue
        
        print(f"\nDocument {idx + 1}:")
        print(f"  Key Phrases: {', '.join(doc.key_phrases)}")
        
        results.append(doc.key_phrases)
    
    return results

def detect_language(documents: list) -> list:
    """
    Detect language of documents
    Supports 120+ languages
    """
    
    response = client.detect_language(documents=documents)
    
    results = []
    
    for idx, doc in enumerate(response):
        if doc.is_error:
            print(f"Document {idx} error: {doc.error}")
            continue
        
        doc_result = {
            "primary_language": {
                "name": doc.primary_language.name,
                "iso6391_name": doc.primary_language.iso6391_name,
                "confidence_score": doc.primary_language.confidence_score
            }
        }
        
        print(f"\nDocument {idx + 1}:")
        print(f"  Language: {doc.primary_language.name}")
        print(f"  ISO Code: {doc.primary_language.iso6391_name}")
        print(f"  Confidence: {doc.primary_language.confidence_score:.2f}")
        
        results.append(doc_result)
    
    return results

# Example
docs = [
    "Microsoft is a technology company based in Redmond, Washington.",
    "The company develops operating systems, productivity software, and cloud services."
]

key_phrases = extract_key_phrases(docs)
languages = detect_language([
    "Hello world",
    "Bonjour le monde",
    "Hola mundo",
    "こんにちは世界"
])
```

### 4.2 Document Summarization

```python
def summarize_documents_extractive(documents: list, sentence_count: int = 3) -> list:
    """
    Extractive summarization - selects key sentences from document
    """
    from azure.ai.textanalytics import ExtractiveSummaryAction
    
    poller = client.begin_analyze_actions(
        documents=documents,
        actions=[
            ExtractiveSummaryAction(
                max_sentence_count=sentence_count
            )
        ],
        language="en"
    )
    
    results = []
    
    for result in poller.result():
        extractive_summary_results = result[0]
        
        if not extractive_summary_results.is_error:
            for doc in extractive_summary_results.results:
                if not doc.is_error:
                    summary_sentences = []
                    print(f"\nExtractive Summary:")
                    for sentence in doc.sentences:
                        summary_sentences.append({
                            "text": sentence.text,
                            "rank_score": sentence.rank_score,
                            "offset": sentence.offset
                        })
                        print(f"  {sentence.text}")
                        print(f"    Rank Score: {sentence.rank_score:.2f}")
                    
                    results.append(summary_sentences)
    
    return results

def summarize_documents_abstractive(documents: list, sentence_count: int = 3) -> list:
    """
    Abstractive summarization - generates new sentences (NEW feature)
    More natural but may introduce information not in original
    """
    from azure.ai.textanalytics import AbstractiveSummaryAction
    
    poller = client.begin_analyze_actions(
        documents=documents,
        actions=[
            AbstractiveSummaryAction(
                max_sentence_count=sentence_count
            )
        ],
        language="en"
    )
    
    results = []
    
    for result in poller.result():
        abstractive_summary_results = result[0]
        
        if not abstractive_summary_results.is_error:
            for doc in abstractive_summary_results.results:
                if not doc.is_error:
                    summary_data = {
                        "summaries": []
                    }
                    
                    print(f"\nAbstractive Summary:")
                    for summary in doc.summaries:
                        summary_data["summaries"].append({
                            "text": summary.text,
                            "contexts": [c.offset for c in summary.contexts] if summary.contexts else []
                        })
                        print(f"  {summary.text}")
                    
                    results.append(summary_data)
    
    return results

# Example
long_article = [
    """
    Artificial Intelligence has revolutionized many industries in recent years.
    Machine learning algorithms can now process vast amounts of data and identify
    patterns that humans might miss. In healthcare, AI systems assist doctors in
    diagnosing diseases more accurately. In finance, AI-powered systems detect
    fraudulent transactions in real-time. The automotive industry uses AI for
    developing self-driving cars. However, there are concerns about AI ethics,
    including bias in algorithms and the impact on employment. Researchers and
    policymakers are working to ensure AI is developed responsibly.
    """
]

extractive_summary = summarize_documents_extractive(long_article, sentence_count=3)
abstractive_summary = summarize_documents_abstractive(long_article, sentence_count=2)
```

### 4.3 Question Answering (successor to QnA Maker)

```python
from azure.ai.language.questionanswering import QuestionAnsweringClient
from azure.core.credentials import AzureKeyCredential
import os

# Initialize QA client
qa_endpoint = os.environ["LANGUAGE_ENDPOINT"]
qa_key = os.environ["LANGUAGE_KEY"]

qa_client = QuestionAnsweringClient(endpoint=qa_endpoint, credential=AzureKeyCredential(qa_key))

def query_knowledge_base(project_name: str, deployment_name: str, question: str, 
                        top: int = 3, confidence_threshold: float = 0.5):
    """
    Query a Question Answering knowledge base
    
    Parameters:
    - project_name: Name of QA project
    - deployment_name: Deployment name (e.g., 'production')
    - question: User's question
    - top: Number of answers to return
    - confidence_threshold: Minimum confidence score
    """
    
    output = qa_client.get_answers(
        question=question,
        project_name=project_name,
        deployment_name=deployment_name,
        top=top,
        confidence_threshold=confidence_threshold
    )
    
    answers = []
    
    print(f"Question: {question}\n")
    
    for answer in output.answers:
        answer_data = {
            "answer": answer.answer,
            "confidence": answer.confidence,
            "id": answer.id,
            "source": answer.source,
            "metadata": answer.metadata,
            "dialog": answer.dialog
        }
        
        answers.append(answer_data)
        
        print(f"Answer: {answer.answer}")
        print(f"  Confidence: {answer.confidence:.2f}")
        print(f"  Source: {answer.source}")
        if answer.metadata:
            print(f"  Metadata: {answer.metadata}")
        print()
    
    return answers

def query_with_metadata_filtering(project_name: str, deployment_name: str, 
                                 question: str, metadata_filters: dict):
    """
    Query with metadata filtering for more precise results
    
    Example metadata_filters:
    {
        "metadata": [
            {"key": "category", "value": "billing"},
            {"key": "product", "value": "azure"}
        ]
    }
    """
    from azure.ai.language.questionanswering.models import AnswersOptions, MetadataFilter
    
    # Build metadata filters
    filters = [
        MetadataFilter(key=f["key"], value=f["value"]) 
        for f in metadata_filters.get("metadata", [])
    ]
    
    options = AnswersOptions(
        question=question,
        top=5,
        confidence_threshold=0.3,
        filters=filters
    )
    
    output = qa_client.get_answers(
        project_name=project_name,
        deployment_name=deployment_name,
        options=options
    )
    
    return output.answers

def query_text_directly(question: str, text_documents: list):
    """
    Query text directly without a knowledge base
    Useful for ad-hoc question answering
    """
    from azure.ai.language.questionanswering.models import AnswersFromTextOptions, TextDocument
    
    # Convert text documents
    documents = [
        TextDocument(id=str(i), text=text) 
        for i, text in enumerate(text_documents)
    ]
    
    options = AnswersFromTextOptions(
        question=question,
        text_documents=documents
    )
    
    output = qa_client.get_answers_from_text(options)
    
    print(f"Question: {question}\n")
    for answer in output.answers:
        print(f"Answer: {answer.answer}")
        print(f"  Confidence: {answer.confidence:.2f}")
        print(f"  Short Answer: {answer.short_answer}\n")
    
    return output.answers

# Example usage
# Query knowledge base
answers = query_knowledge_base(
    project_name="company-faq",
    deployment_name="production",
    question="What are your business hours?"
)

# Query with filtering
filtered_answers = query_with_metadata_filtering(
    project_name="company-faq",
    deployment_name="production",
    question="How do I reset my password?",
    metadata_filters={
        "metadata": [
            {"key": "category", "value": "account"},
            {"key": "difficulty", "value": "beginner"}
        ]
    }
)

# Query text directly
context_texts = [
    "Our office is located at 123 Main Street, Seattle, WA 98101.",
    "We are open Monday through Friday from 9 AM to 5 PM Pacific Time.",
    "Customer support is available 24/7 via phone and email."
]

direct_answers = query_text_directly(
    question="What are the office hours?",
    text_documents=context_texts
)
```

### 4.4 Conversational Language Understanding (CLU)

```python
from azure.ai.language.conversations import ConversationAnalysisClient
from azure.core.credentials import AzureKeyCredential
import os

# Initialize CLU client
clu_endpoint = os.environ["LANGUAGE_ENDPOINT"]
clu_key = os.environ["LANGUAGE_KEY"]

clu_client = ConversationAnalysisClient(endpoint=clu_endpoint, credential=AzureKeyCredential(clu_key))

def analyze_conversation(project_name: str, deployment_name: str, query: str):
    """
    Analyze user utterance with CLU
    Returns intents and entities
    """
    
    result = clu_client.analyze_conversation(
        task={
            "kind": "Conversation",
            "analysisInput": {
                "conversationItem": {
                    "participantId": "user1",
                    "id": "1",
                    "modality": "text",
                    "language": "en",
                    "text": query
                }
            },
            "parameters": {
                "projectName": project_name,
                "deploymentName": deployment_name,
                "verbose": True
            }
        }
    )
    
    # Extract predictions
    prediction = result["result"]["prediction"]
    
    # Top intent
    top_intent = prediction["topIntent"]
    intents = prediction["intents"]
    entities = prediction["entities"]
    
    analysis_result = {
        "query": query,
        "top_intent": {
            "intent": top_intent,
            "confidence": intents[0]["confidenceScore"]
        },
        "all_intents": [],
        "entities": []
    }
    
    print(f"Query: {query}\n")
    print(f"Top Intent: {top_intent}")
    print(f"Confidence: {intents[0]['confidenceScore']:.2f}\n")
    
    # All intents
    print("All Intents:")
    for intent in intents:
        intent_data = {
            "intent": intent["category"],
            "confidence": intent["confidenceScore"]
        }
        analysis_result["all_intents"].append(intent_data)
        print(f"  {intent['category']}: {intent['confidenceScore']:.2f}")
    
    # Entities
    if entities:
        print("\nEntities:")
        for entity in entities:
            entity_data = {
                "category": entity["category"],
                "text": entity["text"],
                "confidence": entity["confidenceScore"],
                "offset": entity["offset"],
                "length": entity["length"]
            }
            
            # Handle resolutions (for learned components)
            if "resolutions" in entity:
                entity_data["resolutions"] = entity["resolutions"]
            
            # Handle list key (for list components)
            if "listKey" in entity:
                entity_data["list_key"] = entity["listKey"]
            
            analysis_result["entities"].append(entity_data)
            
            print(f"  {entity['category']}: {entity['text']}")
            print(f"    Confidence: {entity['confidenceScore']:.2f}")
            if "resolutions" in entity:
                print(f"    Resolutions: {entity['resolutions']}")
    
    return analysis_result

def analyze_with_orchestration(project_name: str, deployment_name: str, query: str):
    """
    Use orchestration workflow to route to appropriate app
    (CLU, Question Answering, LUIS, Custom)
    """
    
    result = clu_client.analyze_conversation(
        task={
            "kind": "Conversation",
            "analysisInput": {
                "conversationItem": {
                    "participantId": "user1",
                    "id": "1",
                    "modality": "text",
                    "language": "en",
                    "text": query
                }
            },
            "parameters": {
                "projectName": project_name,
                "deploymentName": deployment_name,
                "verbose": True
            }
        }
    )
    
    prediction = result["result"]["prediction"]
    
    # Check orchestration result
    if prediction.get("projectKind") == "Orchestration":
        top_intent = prediction["topIntent"]
        target_project_kind = prediction["intents"][0]["targetProjectKind"]
        
        print(f"Orchestration routed to: {top_intent}")
        print(f"Target project kind: {target_project_kind}")
        
        # Get the result from the target project
        if target_project_kind == "QuestionAnswering":
            # QA result
            result_data = prediction["intents"][0]["result"]
            answers = result_data["answers"]
            print(f"\nQA Answer: {answers[0]['answer']}")
        elif target_project_kind == "Conversation":
            # CLU result
            result_data = prediction["intents"][0]["result"]["prediction"]
            print(f"\nCLU Intent: {result_data['topIntent']}")
    
    return prediction

# Example usage
# Simple intent/entity detection
result = analyze_conversation(
    project_name="booking-assistant",
    deployment_name="production",
    query="Book a flight to Seattle on December 15th"
)

# Orchestration workflow
orchestration_result = analyze_with_orchestration(
    project_name="customer-service-orchestrator",
    deployment_name="production",
    query="What are your refund policies?"
)
```

### 4.5 Custom Text Classification

```python
def classify_custom_text(project_name: str, deployment_name: str, documents: list):
    """
    Classify text using custom classification model
    
    Supports:
    - Single-label classification (one category per document)
    - Multi-label classification (multiple categories per document)
    """
    from azure.ai.textanalytics import SingleCategoryClassifyAction, MultiCategoryClassifyAction
    
    # For single-label classification
    poller = client.begin_analyze_actions(
        documents=documents,
        actions=[
            SingleCategoryClassifyAction(
                project_name=project_name,
                deployment_name=deployment_name
            )
        ],
        language="en"
    )
    
    results = []
    
    for result in poller.result():
        classification_results = result[0]
        
        if not classification_results.is_error:
            for doc in classification_results.results:
                if not doc.is_error:
                    classification = doc.classifications[0]
                    
                    doc_result = {
                        "category": classification.category,
                        "confidence_score": classification.confidence_score
                    }
                    
                    print(f"Category: {classification.category}")
                    print(f"  Confidence: {classification.confidence_score:.2f}")
                    
                    results.append(doc_result)
    
    return results

def classify_multi_label(project_name: str, deployment_name: str, documents: list):
    """
    Multi-label classification - document can have multiple categories
    """
    from azure.ai.textanalytics import MultiCategoryClassifyAction
    
    poller = client.begin_analyze_actions(
        documents=documents,
        actions=[
            MultiCategoryClassifyAction(
                project_name=project_name,
                deployment_name=deployment_name
            )
        ],
        language="en"
    )
    
    results = []
    
    for result in poller.result():
        classification_results = result[0]
        
        if not classification_results.is_error:
            for doc in classification_results.results:
                if not doc.is_error:
                    doc_categories = []
                    
                    print(f"\nDocument classifications:")
                    for classification in doc.classifications:
                        category_data = {
                            "category": classification.category,
                            "confidence_score": classification.confidence_score
                        }
                        doc_categories.append(category_data)
                        
                        print(f"  {classification.category}: {classification.confidence_score:.2f}")
                    
                    results.append(doc_categories)
    
    return results

# Example
emails = [
    "I need help resetting my password for my account.",
    "When will my order #12345 be delivered?",
    "I'd like to request a refund for my recent purchase."
]

classifications = classify_custom_text(
    project_name="support-ticket-classifier",
    deployment_name="production",
    documents=emails
)
```

### 4.6 Custom Named Entity Recognition

```python
def recognize_custom_entities(project_name: str, deployment_name: str, documents: list):
    """
    Recognize entities using custom trained NER model
    """
    from azure.ai.textanalytics import RecognizeCustomEntitiesAction
    
    poller = client.begin_analyze_actions(
        documents=documents,
        actions=[
            RecognizeCustomEntitiesAction(
                project_name=project_name,
                deployment_name=deployment_name
            )
        ],
        language="en"
    )
    
    results = []
    
    for result in poller.result():
        entity_results = result[0]
        
        if not entity_results.is_error:
            for doc in entity_results.results:
                if not doc.is_error:
                    doc_entities = []
                    
                    print(f"\nCustom Entities:")
                    for entity in doc.entities:
                        entity_data = {
                            "text": entity.text,
                            "category": entity.category,
                            "confidence_score": entity.confidence_score,
                            "offset": entity.offset,
                            "length": entity.length
                        }
                        
                        doc_entities.append(entity_data)
                        
                        print(f"  {entity.text} - {entity.category}")
                        print(f"    Confidence: {entity.confidence_score:.2f}")
                    
                    results.append(doc_entities)
    
    return results

# Example
technical_docs = [
    "The API endpoint returns a 200 status code with JSON payload.",
    "Configure the TCP/IP settings on port 8080 for the web server."
]

custom_entities = recognize_custom_entities(
    project_name="technical-ner",
    deployment_name="v1",
    documents=technical_docs
)
```

### 4.7 Conversation Summarization

```python
def summarize_conversation(conversation_items: list):
    """
    Summarize conversation (chat, meeting transcript, call center)
    
    conversation_items format:
    [
        {"id": "1", "text": "...", "role": "agent", "participantId": "agent1"},
        {"id": "2", "text": "...", "role": "customer", "participantId": "customer1"}
    ]
    """
    from azure.ai.language.conversations import ConversationAnalysisClient
    from azure.ai.language.conversations.models import (
        ConversationSummarizationTask,
        ConversationItemInput,
        SummaryAspect
    )
    
    # Convert conversation items
    items = [
        ConversationItemInput(
            id=item["id"],
            text=item["text"],
            participant_id=item["participantId"],
            modality="text",
            role=item.get("role")
        )
        for item in conversation_items
    ]
    
    # Analyze with summarization
    poller = clu_client.begin_conversation_analysis(
        task=ConversationSummarizationTask(
            conversation_items=items,
            summary_aspects=[SummaryAspect.ISSUE, SummaryAspect.RESOLUTION]
        )
    )
    
    result = poller.result()
    
    # Extract summaries
    summaries = {}
    for conversation in result["conversations"]:
        for summary in conversation["summaries"]:
            aspect = summary["aspect"]
            text = summary["text"]
            summaries[aspect] = text
            print(f"{aspect}: {text}")
    
    return summaries

# Example
call_transcript = [
    {"id": "1", "text": "Hello, I'm having trouble logging into my account.", "role": "customer", "participantId": "customer1"},
    {"id": "2", "text": "I'm sorry to hear that. Let me help you reset your password.", "role": "agent", "participantId": "agent1"},
    {"id": "3", "text": "Thank you. I've sent the reset link to your email.", "role": "agent", "participantId": "agent1"},
    {"id": "4", "text": "Great, I received it and can log in now. Thank you!", "role": "customer", "participantId": "customer1"}
]

summary = summarize_conversation(call_transcript)
```

### 4.8 Batch Processing with Analytics Actions

```python
def batch_analyze_multiple_operations(documents: list):
    """
    Batch process multiple operations in single request
    More efficient than individual calls
    """
    from azure.ai.textanalytics import (
        RecognizeEntitiesAction,
        ExtractKeyPhrasesAction,
        AnalyzeSentimentAction,
        RecognizePiiEntitiesAction
    )
    
    # Run multiple actions simultaneously
    poller = client.begin_analyze_actions(
        documents=documents,
        actions=[
            RecognizeEntitiesAction(),
            ExtractKeyPhrasesAction(),
            AnalyzeSentimentAction(),
            RecognizePiiEntitiesAction()
        ],
        language="en",
        show_stats=True
    )
    
    results = {
        "entities": [],
        "key_phrases": [],
        "sentiment": [],
        "pii": []
    }
    
    for result in poller.result():
        # Process each action result
        entities_result = result[0]
        key_phrases_result = result[1]
        sentiment_result = result[2]
        pii_result = result[3]
        
        # Entities
        if not entities_result.is_error:
            for doc in entities_result.results:
                if not doc.is_error:
                    results["entities"].append([e.text for e in doc.entities])
        
        # Key phrases
        if not key_phrases_result.is_error:
            for doc in key_phrases_result.results:
                if not doc.is_error:
                    results["key_phrases"].append(doc.key_phrases)
        
        # Sentiment
        if not sentiment_result.is_error:
            for doc in sentiment_result.results:
                if not doc.is_error:
                    results["sentiment"].append(doc.sentiment)
        
        # PII
        if not pii_result.is_error:
            for doc in pii_result.results:
                if not doc.is_error:
                    results["pii"].append(doc.redacted_text)
    
    return results

# Example
documents = [
    "John Smith works at Microsoft. His email is john@example.com.",
    "The meeting is scheduled for tomorrow at 2 PM.",
    "Please call me at 555-0123 to discuss the project."
]

batch_results = batch_analyze_multiple_operations(documents)
print(f"Entities: {batch_results['entities']}")
print(f"Key Phrases: {batch_results['key_phrases']}")
print(f"Sentiment: {batch_results['sentiment']}")
print(f"Redacted: {batch_results['pii']}")
```

---

## 5. AZURE AI SPEECH

### 5.1 Speech-to-Text (STT)

```python
import azure.cognitiveservices.speech as speechsdk
import os
import time

# Initialize speech config
speech_key = os.environ["SPEECH_KEY"]
speech_region = os.environ["SPEECH_REGION"]

speech_config = speechsdk.SpeechConfig(
    subscription=speech_key,
    region=speech_region
)

def recognize_speech_from_microphone():
    """
    Real-time speech recognition from microphone
    """
    
    # Configure audio input
    audio_config = speechsdk.AudioConfig(use_default_microphone=True)
    
    # Create speech recognizer
    speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    print("Speak into your microphone...")
    
    # Single utterance recognition
    result = speech_recognizer.recognize_once()
    
    if result.reason == speechsdk.ResultReason.RecognizedSpeech:
        print(f"Recognized: {result.text}")
        return result.text
    elif result.reason == speechsdk.ResultReason.NoMatch:
        print("No speech could be recognized")
        return None
    elif result.reason == speechsdk.ResultReason.Canceled:
        cancellation = result.cancellation_details
        print(f"Speech Recognition canceled: {cancellation.reason}")
        if cancellation.reason == speechsdk.CancellationReason.Error:
            print(f"Error: {cancellation.error_details}")
        return None

def recognize_speech_from_file(audio_file_path: str):
    """
    Recognize speech from audio file
    Supported formats: WAV (PCM), MP3, OGG, FLAC
    """
    
    audio_config = speechsdk.AudioConfig(filename=audio_file_path)
    
    speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    result = speech_recognizer.recognize_once()
    
    if result.reason == speechsdk.ResultReason.RecognizedSpeech:
        print(f"Recognized: {result.text}")
        return result.text
    else:
        return None

def continuous_recognition_from_microphone():
    """
    Continuous speech recognition with streaming
    Provides real-time results as user speaks
    """
    
    audio_config = speechsdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    done = False
    all_results = []
    
    def recognizing_callback(evt):
        """Called during recognition (intermediate results)"""
        print(f"Recognizing: {evt.result.text}")
    
    def recognized_callback(evt):
        """Called when phrase is recognized (final result)"""
        if evt.result.reason == speechsdk.ResultReason.RecognizedSpeech:
            print(f"Recognized: {evt.result.text}")
            all_results.append(evt.result.text)
    
    def canceled_callback(evt):
        """Called when recognition is canceled"""
        print(f"Canceled: {evt}")
        nonlocal done
        done = True
    
    def stopped_callback(evt):
        """Called when recognition stops"""
        print("Recognition stopped")
        nonlocal done
        done = True
    
    # Connect callbacks
    speech_recognizer.recognizing.connect(recognizing_callback)
    speech_recognizer.recognized.connect(recognized_callback)
    speech_recognizer.session_stopped.connect(stopped_callback)
    speech_recognizer.canceled.connect(canceled_callback)
    
    # Start continuous recognition
    speech_recognizer.start_continuous_recognition()
    
    print("Continuous recognition started. Speak into the microphone...")
    print("Press Ctrl+C to stop.")
    
    try:
        while not done:
            time.sleep(0.5)
    except KeyboardInterrupt:
        print("\nStopping...")
    
    speech_recognizer.stop_continuous_recognition()
    
    return all_results

def recognize_with_detailed_results(audio_file_path: str):
    """
    Get detailed recognition results including:
    - Word-level timestamps
    - Confidence scores
    - Lexical/ITN/Masked ITN/Display forms
    """
    
    # Enable detailed results
    speech_config.output_format = speechsdk.OutputFormat.Detailed
    
    audio_config = speechsdk.AudioConfig(filename=audio_file_path)
    speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    result = speech_recognizer.recognize_once()
    
    if result.reason == speechsdk.ResultReason.RecognizedSpeech:
        # Parse JSON details
        import json
        details = json.loads(result.json)
        
        print("Detailed Results:")
        print(f"Display Text: {details['DisplayText']}")
        
        # Word-level details
        if 'NBest' in details:
            best_result = details['NBest'][0]
            print(f"\nConfidence: {best_result['Confidence']}")
            print(f"Lexical: {best_result['Lexical']}")
            print(f"ITN: {best_result['ITN']}")
            print(f"Display: {best_result['Display']}")
            
            # Word timestamps
            if 'Words' in best_result:
                print("\nWord Timestamps:")
                for word in best_result['Words']:
                    print(f"  {word['Word']}: {word['Offset']/10000000:.2f}s - {(word['Offset']+word['Duration'])/10000000:.2f}s")
                    print(f"    Confidence: {word.get('Confidence', 'N/A')}")
        
        return details
    
    return None

# Example usage
# From microphone
# text = recognize_speech_from_microphone()

# From file
text = recognize_speech_from_file("audio.wav")

# Continuous recognition
# results = continuous_recognition_from_microphone()

# Detailed results
details = recognize_with_detailed_results("audio.wav")
```

### 5.2 Batch Transcription

```python
import requests
import time
import json

def batch_transcribe(audio_url: str, locale: str = "en-US"):
    """
    Batch transcription for large audio files
    
    Use cases:
    - Long recordings (hours)
    - Multiple channel audio
    - Large volume processing
    
    Audio can be in Azure Blob Storage
    """
    
    speech_key = os.environ["SPEECH_KEY"]
    speech_region = os.environ["SPEECH_REGION"]
    
    # Batch transcription endpoint
    endpoint = f"https://{speech_region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions"
    
    headers = {
        "Ocp-Apim-Subscription-Key": speech_key,
        "Content-Type": "application/json"
    }
    
    # Create transcription job
    transcription_definition = {
        "contentUrls": [audio_url],
        "locale": locale,
        "displayName": "My Transcription",
        "properties": {
            "wordLevelTimestampsEnabled": True,
            "diarizationEnabled": True,  # Speaker separation
            "profanityFilterMode": "Masked",  # None, Masked, Removed, Tags
            "punctuationMode": "DictatedAndAutomatic"
        }
    }
    
    # Submit transcription
    response = requests.post(endpoint, headers=headers, json=transcription_definition)
    response.raise_for_status()
    
    transcription = response.json()
    transcription_id = transcription["self"].split("/")[-1]
    
    print(f"Transcription ID: {transcription_id}")
    print("Transcription started...")
    
    # Poll for completion
    status_url = transcription["self"]
    
    while True:
        status_response = requests.get(status_url, headers=headers)
        status = status_response.json()
        
        current_status = status["status"]
        print(f"Status: {current_status}")
        
        if current_status == "Succeeded":
            # Get results
            files_url = status["links"]["files"]
            files_response = requests.get(files_url, headers=headers)
            files = files_response.json()
            
            # Download transcription result
            for file_data in files["values"]:
                if file_data["kind"] == "Transcription":
                    result_url = file_data["links"]["contentUrl"]
                    result_response = requests.get(result_url)
                    result = result_response.json()
                    
                    return result
        
        elif current_status == "Failed":
            print(f"Transcription failed: {status}")
            return None
        
        time.sleep(10)

def process_batch_transcription_results(result: dict):
    """
    Process batch transcription results
    Extract text, timestamps, speaker diarization
    """
    
    processed_results = {
        "full_text": "",
        "phrases": [],
        "speakers": {}
    }
    
    # Combine all phrases
    combined_phrases = result["combinedRecognizedPhrases"]
    processed_results["full_text"] = combined_phrases[0]["display"]
    
    # Individual phrases with details
    for phrase in result["recognizedPhrases"]:
        phrase_data = {
            "text": phrase["nBest"][0]["display"],
            "confidence": phrase["nBest"][0]["confidence"],
            "speaker": phrase.get("speaker"),
            "offset": phrase["offset"],
            "duration": phrase["duration"],
            "words": []
        }
        
        # Word-level details
        if "words" in phrase["nBest"][0]:
            for word in phrase["nBest"][0]["words"]:
                phrase_data["words"].append({
                    "word": word["word"],
                    "offset": word["offset"],
                    "duration": word["duration"],
                    "confidence": word.get("confidence")
                })
        
        processed_results["phrases"].append(phrase_data)
        
        # Group by speaker
        if phrase.get("speaker"):
            speaker_id = phrase["speaker"]
            if speaker_id not in processed_results["speakers"]:
                processed_results["speakers"][speaker_id] = []
            processed_results["speakers"][speaker_id].append(phrase_data["text"])
    
    return processed_results

# Example usage
audio_url = "https://mystorageaccount.blob.core.windows.net/audio/recording.wav?sas_token"
result = batch_transcribe(audio_url)

if result:
    processed = process_batch_transcription_results(result)
    print(f"\nFull Transcription:\n{processed['full_text']}")
    
    print("\nSpeaker Diarization:")
    for speaker, phrases in processed['speakers'].items():
        print(f"\n{speaker}:")
        for phrase in phrases:
            print(f"  - {phrase}")
```

### 5.3 Text-to-Speech (TTS)

```python
def synthesize_speech_to_file(text: str, output_file: str, voice: str = "en-US-JennyNeural"):
    """
    Synthesize speech and save to file
    
    Voice examples:
    - en-US-JennyNeural (Female, friendly)
    - en-US-GuyNeural (Male, casual)
    - en-US-AriaNeural (Female, professional)
    - en-GB-SoniaNeural (British Female)
    - fr-FR-DeniseNeural (French Female)
    """
    
    speech_config.speech_synthesis_voice_name = voice
    
    # Configure audio output
    audio_config = speechsdk.audio.AudioOutputConfig(filename=output_file)
    
    # Create synthesizer
    speech_synthesizer = speechsdk.SpeechSynthesizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    # Synthesize
    result = speech_synthesizer.speak_text_async(text).get()
    
    if result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
        print(f"Speech synthesized and saved to: {output_file}")
        return True
    elif result.reason == speechsdk.ResultReason.Canceled:
        cancellation = result.cancellation_details
        print(f"Synthesis canceled: {cancellation.reason}")
        if cancellation.reason == speechsdk.CancellationReason.Error:
            print(f"Error: {cancellation.error_details}")
        return False

def synthesize_with_ssml(ssml: str, output_file: str):
    """
    Synthesize speech using SSML for fine-grained control
    
    SSML features:
    - Voice selection
    - Prosody (rate, pitch, volume)
    - Emphasis
    - Breaks/pauses
    - Audio mixing
    - Pronunciation
    """
    
    audio_config = speechsdk.audio.AudioOutputConfig(filename=output_file)
    speech_synthesizer = speechsdk.SpeechSynthesizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    result = speech_synthesizer.speak_ssml_async(ssml).get()
    
    if result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
        print(f"SSML speech synthesized")
        return True
    return False

# Comprehensive SSML example
ssml_advanced = """
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" 
       xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
    
    <!-- Primary voice -->
    <voice name="en-US-JennyNeural">
        <mstts:express-as style="cheerful" styledegree="2">
            Hello! Welcome to our service.
        </mstts:express-as>
        
        <!-- Add pause -->
        <break time="500ms"/>
        
        <!-- Adjust prosody -->
        <prosody rate="0.9" pitch="+5%">
            Let me help you today.
        </prosody>
        
        <!-- Emphasis -->
        Today's special offer is <emphasis level="strong">amazing</emphasis>!
        
        <!-- Say as -->
        Please call us at <say-as interpret-as="telephone">555-123-4567</say-as>.
        
        The meeting is on <say-as interpret-as="date" format="mdy">12/15/2025</say-as>.
        
        <!-- Phoneme for pronunciation -->
        <phoneme alphabet="ipa" ph="təˈmeɪtoʊ">tomato</phoneme>
        
        <!-- Substitute -->
        <sub alias="World Wide Web Consortium">W3C</sub>
        
        <!-- Different voice for quote -->
        <voice name="en-US-GuyNeural">
            <prosody rate="0.8" pitch="-2st">
                "I think, therefore I am," said the philosopher.
            </prosody>
        </voice>
        
        <!-- Audio mixing -->
        <audio src="https://example.com/chime.wav">
            Could not play sound
        </audio>
        
        Thank you for your attention!
    </voice>
</speak>
"""

def get_available_voices(locale: str = "en-US"):
    """
    List all available voices for a locale
    """
    speech_synthesizer = speechsdk.SpeechSynthesizer(
        speech_config=speech_config,
        audio_config=None
    )
    
    result = speech_synthesizer.get_voices_async(locale).get()
    
    if result.reason == speechsdk.ResultReason.VoicesListRetrieved:
        print(f"Available voices for {locale}:")
        for voice in result.voices:
            print(f"  {voice.short_name}")
            print(f"    Name: {voice.local_name}")
            print(f"    Gender: {voice.gender}")
            print(f"    Locale: {voice.locale}")
            if hasattr(voice, 'style_list') and voice.style_list:
                print(f"    Styles: {', '.join(voice.style_list)}")
            print()
        
        return result.voices
    
    return []

def synthesize_with_viseme(text: str):
    """
    Get viseme (mouth shape) data for lip-sync animation
    """
    
    speech_synthesizer = speechsdk.SpeechSynthesizer(
        speech_config=speech_config,
        audio_config=None
    )
    
    viseme_data = []
    
    def viseme_received(evt):
        print(f"Viseme ID: {evt.viseme_id}, Audio offset: {evt.audio_offset / 10000}ms")
        viseme_data.append({
            "viseme_id": evt.viseme_id,
            "audio_offset": evt.audio_offset
        })
    
    speech_synthesizer.viseme_received.connect(viseme_received)
    
    result = speech_synthesizer.speak_text_async(text).get()
    
    return viseme_data

# Example usage
text = "Hello! This is a speech synthesis example."
synthesize_speech_to_file(text, "output.wav", voice="en-US-JennyNeural")

# SSML synthesis
synthesize_with_ssml(ssml_advanced, "output_ssml.wav")

# List voices
voices = get_available_voices("en-US")

# Viseme for animation
visemes = synthesize_with_viseme("Hello world")
```

### 5.4 Speech Translation

```python
def translate_speech_from_microphone(target_languages: list):
    """
    Real-time speech translation
    
    Supports translation to multiple target languages simultaneously
    """
    
    # Create translation config
    translation_config = speechsdk.translation.SpeechTranslationConfig(
        subscription=os.environ["SPEECH_KEY"],
        region=os.environ["SPEECH_REGION"]
    )
    
    # Set source language
    translation_config.speech_recognition_language = "en-US"
    
    # Add target languages
    for lang in target_languages:
        translation_config.add_target_language(lang)
    
    # Configure audio input
    audio_config = speechsdk.AudioConfig(use_default_microphone=True)
    
    # Create translator
    translation_recognizer = speechsdk.translation.TranslationRecognizer(
        translation_config=translation_config,
        audio_config=audio_config
    )
    
    print(f"Speak into your microphone (will translate to: {', '.join(target_languages)})...")
    
    result = translation_recognizer.recognize_once()
    
    if result.reason == speechsdk.ResultReason.TranslatedSpeech:
        print(f"\nRecognized (English): {result.text}")
        print("\nTranslations:")
        
        translations = {}
        for lang in target_languages:
            translation = result.translations[lang]
            translations[lang] = translation
            print(f"  {lang}: {translation}")
        
        return translations
    
    elif result.reason == speechsdk.ResultReason.NoMatch:
        print("No speech could be recognized")
    elif result.reason == speechsdk.ResultReason.Canceled:
        cancellation = result.cancellation_details
        print(f"Translation canceled: {cancellation.reason}")
        if cancellation.reason == speechsdk.CancellationReason.Error:
            print(f"Error: {cancellation.error_details}")
    
    return None

def translate_with_synthesis(source_text: str, target_language: str, voice_name: str):
    """
    Translate speech and synthesize in target language
    """
    
    translation_config = speechsdk.translation.SpeechTranslationConfig(
        subscription=os.environ["SPEECH_KEY"],
        region=os.environ["SPEECH_REGION"]
    )
    
    translation_config.speech_recognition_language = "en-US"
    translation_config.add_target_language(target_language)
    
    # Set voice for synthesis
    translation_config.voice_name = voice_name
    
    audio_config = speechsdk.AudioConfig(use_default_microphone=True)
    translation_recognizer = speechsdk.translation.TranslationRecognizer(
        translation_config=translation_config,
        audio_config=audio_config
    )
    
    # Handle synthesis result
    def synthesis_callback(evt):
        print(f"Synthesis audio received: {len(evt.result.audio)} bytes")
        # Save or play audio
        with open(f"translation_{target_language}.wav", "wb") as f:
            f.write(evt.result.audio)
    
    translation_recognizer.synthesizing.connect(synthesis_callback)
    
    result = translation_recognizer.recognize_once()
    
    if result.reason == speechsdk.ResultReason.TranslatedSpeech:
        translated_text = result.translations[target_language]
        print(f"Translated: {translated_text}")
        return translated_text
    
    return None

# Example usage
# Translate to multiple languages
target_langs = ["es", "fr", "de", "ja"]  # Spanish, French, German, Japanese
translations = translate_speech_from_microphone(target_langs)

# Translate with synthesis
translate_with_synthesis(
    "Hello, how are you?",
    target_language="es",
    voice_name="es-ES-ElviraNeural"
)
```

### 5.5 Pronunciation Assessment

```python
def assess_pronunciation(reference_text: str, audio_file: str):
    """
    Assess pronunciation quality
    
    Metrics:
    - Accuracy: How correctly phonemes are pronounced
    - Fluency: How natural and smooth the speech is
    - Completeness: How much of the text is spoken
    - Prosody: Stress, intonation, rhythm
    """
    
    # Configure pronunciation assessment
    pronunciation_config = speechsdk.PronunciationAssessmentConfig(
        reference_text=reference_text,
        grading_system=speechsdk.PronunciationAssessmentGradingSystem.HundredMark,
        granularity=speechsdk.PronunciationAssessmentGranularity.Phoneme,
        enable_miscue=True
    )
    
    # Enable prosody evaluation
    pronunciation_config.enable_prosody_assessment()
    
    audio_config = speechsdk.AudioConfig(filename=audio_file)
    speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config
    )
    
    # Apply pronunciation assessment
    pronunciation_config.apply_to(speech_recognizer)
    
    result = speech_recognizer.recognize_once()
    
    if result.reason == speechsdk.ResultReason.RecognizedSpeech:
        pronunciation_result = speechsdk.PronunciationAssessmentResult(result)
        
        assessment = {
            "accuracy_score": pronunciation_result.accuracy_score,
            "fluency_score": pronunciation_result.fluency_score,
            "completeness_score": pronunciation_result.completeness_score,
            "pronunciation_score": pronunciation_result.pronunciation_score,
            "prosody_score": getattr(pronunciation_result, 'prosody_score', None),
            "words": []
        }
        
        print(f"\nPronunciation Assessment Results:")
        print(f"  Overall Score: {assessment['pronunciation_score']:.1f}/100")
        print(f"  Accuracy: {assessment['accuracy_score']:.1f}/100")
        print(f"  Fluency: {assessment['fluency_score']:.1f}/100")
        print(f"  Completeness: {assessment['completeness_score']:.1f}/100")
        if assessment['prosody_score']:
            print(f"  Prosody: {assessment['prosody_score']:.1f}/100")
        
        # Word-level assessment
        import json
        result_json = json.loads(result.json)
        
        if 'NBest' in result_json and len(result_json['NBest']) > 0:
            words = result_json['NBest'][0].get('Words', [])
            
            print("\nWord-level Assessment:")
            for word in words:
                word_assessment = {
                    "word": word['Word'],
                    "accuracy_score": word.get('PronunciationAssessment', {}).get('AccuracyScore', 0),
                    "error_type": word.get('PronunciationAssessment', {}).get('ErrorType', 'None')
                }
                assessment['words'].append(word_assessment)
                
                print(f"  {word['Word']}: {word_assessment['accuracy_score']:.1f} ({word_assessment['error_type']})")
        
        return assessment
    
    return None

# Example usage
reference = "Hello, how are you today?"
assessment = assess_pronunciation(reference, "user_speech.wav")
```

---

## 6. AZURE OPENAI SERVICE

### 6.1 Latest Models (December 2025)

**GPT-4 Models:**
- `gpt-4` (8K context)
- `gpt-4-32k` (32K context)
- `gpt-4-turbo` (128K context) - Latest, most capable
- `gpt-4o` (Omni - multimodal capabilities)

**GPT-3.5 Models:**
- `gpt-35-turbo` (4K context)
- `gpt-35-turbo-16k` (16K context)

**Embedding Models:**
- `text-embedding-3-small` (1,536 dimensions)
- `text-embedding-3-large` (3,072 dimensions)
- `text-embedding-ada-002` (1,536 dimensions - legacy)

**Image Models:**
- `dall-e-3` (Latest image generation)
- `dall-e-2` (Previous generation)

**Audio Models:**
- `whisper` (Speech-to-text)

```python
from openai import AzureOpenAI
import os
import tiktoken

# Initialize Azure OpenAI client
client = AzureOpenAI(
    api_key=os.environ["AZURE_OPENAI_KEY"],
    api_version="2024-02-01",  # Latest API version
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"]
)

def create_deployment():
    """
    Create Azure OpenAI deployment via Azure CLI
    """
    # Note: Deployments are typically created via Azure Portal or CLI
    pass

# List deployments
deployments_cmd = """
az cognitiveservices account deployment list \
  --name "my-openai-resource" \
  --resource-group "ai-resources-rg" \
  -o table
"""
```

### 6.2 Chat Completions API

```python
def chat_completion_basic(messages: list, deployment_name: str = "gpt-4"):
    """
    Basic chat completion
    """
    
    response = client.chat.completions.create(
        model=deployment_name,
        messages=messages,
        temperature=0.7,
        max_tokens=800,
        top_p=0.95,
        frequency_penalty=0,
        presence_penalty=0
    )
    
    return response.choices[0].message.content

def chat_with_system_message(user_query: str, system_prompt: str, deployment_name: str = "gpt-4"):
    """
    Chat with system message for behavior control
    
    System message best practices:
    - Define the assistant's role
    - Set constraints and guidelines
    - Specify output format
    - Define personality/tone
    """
    
    messages = [
        {
            "role": "system",
            "content": system_prompt
        },
        {
            "role": "user",
            "content": user_query
        }
    ]
    
    response = client.chat.completions.create(
        model=deployment_name,
        messages=messages,
        temperature=0.7,
        max_tokens=1000
    )
    
    # Extract response
    assistant_message = response.choices[0].message.content
    
    # Token usage
    print(f"\nToken Usage:")
    print(f"  Prompt: {response.usage.prompt_tokens}")
    print(f"  Completion: {response.usage.completion_tokens}")
    print(f"  Total: {response.usage.total_tokens}")
    
    return assistant_message

def streaming_chat(messages: list, deployment_name: str = "gpt-4"):
    """
    Stream chat responses for real-time user experience
    """
    
    response = client.chat.completions.create(
        model=deployment_name,
        messages=messages,
        stream=True,
        temperature=0.7
    )
    
    print("Assistant: ", end="", flush=True)
    full_response = ""
    
    for chunk in response:
        if chunk.choices and chunk.choices[0].delta.content:
            content = chunk.choices[0].delta.content
            print(content, end="", flush=True)
            full_response += content
    
    print()  # New line
    return full_response

def chat_with_json_mode(user_query: str, deployment_name: str = "gpt-4"):
    """
    Force JSON output format
    Model will always return valid JSON
    """
    
    messages = [
        {
            "role": "system",
            "content": "You are a helpful assistant that outputs JSON."
        },
        {
            "role": "user",
            "content": user_query
        }
    ]
    
    response = client.chat.completions.create(
        model=deployment_name,
        messages=messages,
        response_format={"type": "json_object"},
        temperature=0.7
    )
    
    import json
    result = json.loads(response.choices[0].message.content)
    return result

def reproducible_outputs(messages: list, deployment_name: str = "gpt-4", seed: int = 42):
    """
    Get reproducible outputs using seed parameter
    Useful for testing and deterministic behavior
    """
    
    response = client.chat.completions.create(
        model=deployment_name,
        messages=messages,
        seed=seed,  # Ensures reproducibility
        temperature=0  # Lower temperature for consistency
    )
    
    return response.choices[0].message.content

# Example usage
system_prompt = """
You are a helpful customer service assistant for an e-commerce company.
You should be polite, professional, and concise.
If you don't know something, admit it rather than making up information.
"""

response = chat_with_system_message(
    "How do I return a product?",
    system_prompt
)
print(response)

# Streaming example
messages = [
    {"role": "system", "content": "You are a helpful coding assistant."},
    {"role": "user", "content": "Explain Python list comprehensions."}
]
streaming_chat(messages)

# JSON mode
json_result = chat_with_json_mode(
    "Extract the name, age, and city from: John Smith is 30 years old and lives in Seattle."
)
print(json_result)
```

### 6.3 Function Calling

```python
def function_calling_example():
    """
    Function calling allows the model to call external functions
    """
    
    # Define functions
    functions = [
        {
            "type": "function",
            "function": {
                "name": "get_current_weather",
                "description": "Get the current weather in a given location",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "The city and state, e.g. San Francisco, CA"
                        },
                        "unit": {
                            "type": "string",
                            "enum": ["celsius", "fahrenheit"],
                            "description": "The temperature unit"
                        }
                    },
                    "required": ["location"]
                }
            }
        },
        {
            "type": "function",
            "function": {
                "name": "get_stock_price",
                "description": "Get the current stock price for a given symbol",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "symbol": {
                            "type": "string",
                            "description": "The stock symbol, e.g. MSFT"
                        }
                    },
                    "required": ["symbol"]
                }
            }
        }
    ]
    
    messages = [
        {"role": "user", "content": "What's the weather like in Seattle?"}
    ]
    
    # First API call
    response = client.chat.completions.create(
        model="gpt-4",
        messages=messages,
        tools=functions,
        tool_choice="auto"  # Let model decide
    )
    
    response_message = response.choices[0].message
    
    # Check if model wants to call a function
    if response_message.tool_calls:
        # Execute function calls
        tool_call = response_message.tool_calls[0]
        function_name = tool_call.function.name
        function_args = json.loads(tool_call.function.arguments)
        
        print(f"Model wants to call: {function_name}")
        print(f"With arguments: {function_args}")
        
        # Execute the actual function
        if function_name == "get_current_weather":
            function_response = get_current_weather(**function_args)
        elif function_name == "get_stock_price":
            function_response = get_stock_price(**function_args)
        
        # Add function response to messages
        messages.append(response_message)
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": json.dumps(function_response)
        })
        
        # Second API call with function result
        final_response = client.chat.completions.create(
            model="gpt-4",
            messages=messages
        )
        
        return final_response.choices[0].message.content
    
    return response_message.content

def get_current_weather(location: str, unit: str = "fahrenheit"):
    """Mock weather function"""
    return {
        "location": location,
        "temperature": "72",
        "unit": unit,
        "forecast": ["sunny", "cloudy"]
    }

def get_stock_price(symbol: str):
    """Mock stock price function"""
    return {
        "symbol": symbol,
        "price": "350.25",
        "change": "+2.5%"
    }

def parallel_function_calling():
    """
    Model can call multiple functions in parallel
    """
    
    functions = [
        # ... same function definitions as above
    ]
    
    messages = [
        {
            "role": "user",
            "content": "What's the weather in Seattle and the stock price of Microsoft?"
        }
    ]
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=messages,
        tools=functions,
        tool_choice="auto"
    )
    
    response_message = response.choices[0].message
    
    # Handle multiple tool calls
    if response_message.tool_calls:
        messages.append(response_message)
        
        for tool_call in response_message.tool_calls:
            function_name = tool_call.function.name
            function_args = json.loads(tool_call.function.arguments)
            
            # Execute function
            if function_name == "get_current_weather":
                result = get_current_weather(**function_args)
            elif function_name == "get_stock_price":
                result = get_stock_price(**function_args)
            
            # Add result
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(result)
            })
        
        # Get final response
        final_response = client.chat.completions.create(
            model="gpt-4",
            messages=messages
        )
        
        return final_response.choices[0].message.content

# Example
result = function_calling_example()
print(result)
```

### 6.4 Embeddings

```python
def create_embeddings(texts: list, model: str = "text-embedding-3-small"):
    """
    Create embeddings for semantic search
    """
    
    response = client.embeddings.create(
        input=texts,
        model=model
    )
    
    embeddings = [item.embedding for item in response.data]
    
    print(f"Created {len(embeddings)} embeddings")
    print(f"Embedding dimension: {len(embeddings[0])}")
    print(f"Total tokens used: {response.usage.total_tokens}")
    
    return embeddings

def cosine_similarity(vec1: list, vec2: list) -> float:
    """
    Calculate cosine similarity between two vectors
    """
    import numpy as np
    
    vec1 = np.array(vec1)
    vec2 = np.array(vec2)
    
    dot_product = np.dot(vec1, vec2)
    norm1 = np.linalg.norm(vec1)
    norm2 = np.linalg.norm(vec2)
    
    return dot_product / (norm1 * norm2)

def semantic_search(query: str, documents: list, top_k: int = 3):
    """
    Semantic search using embeddings
    """
    
    # Create embeddings for all texts
    all_texts = [query] + documents
    embeddings = create_embeddings(all_texts)
    
    query_embedding = embeddings[0]
    doc_embeddings = embeddings[1:]
    
    # Calculate similarities
    similarities = [
        (i, cosine_similarity(query_embedding, doc_emb))
        for i, doc_emb in enumerate(doc_embeddings)
    ]
    
    # Sort by similarity
    similarities.sort(key=lambda x: x[1], reverse=True)
    
    # Return top k results
    results = []
    print(f"\nTop {top_k} results for: '{query}'\n")
    
    for i, (doc_idx, score) in enumerate(similarities[:top_k]):
        results.append({
            "document": documents[doc_idx],
            "score": score,
            "rank": i + 1
        })
        print(f"{i+1}. (Score: {score:.4f}) {documents[doc_idx][:100]}...")
    
    return results

def embeddings_with_reduced_dimensions(texts: list):
    """
    Use text-embedding-3-* with custom dimensions for optimization
    """
    
    response = client.embeddings.create(
        input=texts,
        model="text-embedding-3-large",
        dimensions=1024  # Reduce from 3072 to 1024
    )
    
    # Smaller embeddings = lower storage costs, faster retrieval
    return [item.embedding for item in response.data]

# Example usage
documents = [
    "Python is a high-level programming language.",
    "Machine learning is a subset of artificial intelligence.",
    "Azure OpenAI provides access to GPT models.",
    "Neural networks are inspired by biological neurons.",
    "Cloud computing enables scalable infrastructure."
]

query = "What is AI and machine learning?"
results = semantic_search(query, documents, top_k=3)
```

### 6.5 Retrieval Augmented Generation (RAG)

```python
def rag_pipeline(question: str, knowledge_base: list):
    """
    Complete RAG pipeline:
    1. Retrieve relevant documents
    2. Augment prompt with context
    3. Generate answer
    """
    
    # Step 1: Retrieve relevant documents
    relevant_docs = semantic_search(question, knowledge_base, top_k=3)
    
    # Step 2: Create augmented prompt
    context = "\n\n".join([doc["document"] for doc in relevant_docs])
    
    augmented_prompt = f"""
Use the following context to answer the question. If the answer is not in the context, say "I don't have enough information to answer this question."

Context:
{context}

Question: {question}

Answer:"""
    
    # Step 3: Generate answer
    messages = [
        {"role": "system", "content": "You are a helpful assistant that answers questions based on provided context."},
        {"role": "user", "content": augmented_prompt}
    ]
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=messages,
        temperature=0.3  # Lower temperature for factual answers
    )
    
    answer = response.choices[0].message.content
    
    return {
        "answer": answer,
        "sources": relevant_docs
    }

def rag_with_citations(question: str, knowledge_base: list):
    """
    RAG with source citations
    """
    
    relevant_docs = semantic_search(question, knowledge_base, top_k=3)
    
    # Number each document
    numbered_context = "\n\n".join([
        f"[{i+1}] {doc['document']}" 
        for i, doc in enumerate(relevant_docs)
    ])
    
    augmented_prompt = f"""
Use the following numbered sources to answer the question. 
Cite the sources using [1], [2], etc. in your answer.

Sources:
{numbered_context}

Question: {question}

Answer (with citations):"""
    
    messages = [
        {"role": "system", "content": "You are a helpful assistant that provides cited answers."},
        {"role": "user", "content": augmented_prompt}
    ]
    
    response = client.chat.completions.create(
        model="gpt-4",
        messages=messages,
        temperature=0.3
    )
    
    return response.choices[0].message.content

# Example usage
knowledge_base = [
    "Azure OpenAI Service provides REST API access to OpenAI's powerful language models including GPT-4, GPT-3.5-Turbo, and Embeddings.",
    "The service offers enterprise-grade security with role-based access control and private networking.",
    "You can deploy models in your own Azure subscription with data residency in your region.",
    "Azure OpenAI supports both synchronous and streaming responses.",
    "Token limits vary by model: GPT-4 supports up to 8K tokens, GPT-4-turbo supports 128K tokens."
]

result = rag_pipeline("What models does Azure OpenAI support?", knowledge_base)
print(f"Answer: {result['answer']}\n")
print("Sources used:")
for source in result['sources']:
    print(f"  - {source['document'][:80]}...")
```

### 6.6 Token Counting and Cost Estimation

```python
import tiktoken

def count_tokens(text: str, model: str = "gpt-4") -> int:
    """
    Count tokens in text for a specific model
    """
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

def count_messages_tokens(messages: list, model: str = "gpt-4") -> int:
    """
    Count tokens for chat messages including overhead
    """
    encoding = tiktoken.encoding_for_model(model)
    
    # Different models have different token overheads
    if model in ["gpt-3.5-turbo", "gpt-35-turbo", "gpt-4"]:
        tokens_per_message = 3
        tokens_per_name = 1
    else:
        tokens_per_message = 3
        tokens_per_name = 1
    
    num_tokens = 0
    for message in messages:
        num_tokens += tokens_per_message
        for key, value in message.items():
            num_tokens += len(encoding.encode(value))
            if key == "name":
                num_tokens += tokens_per_name
    
    num_tokens += 3  # Every reply is primed with assistant
    return num_tokens

def estimate_cost(prompt_tokens: int, completion_tokens: int, model: str = "gpt-4") -> dict:
    """
    Estimate cost based on token usage
    Prices as of December 2025 (example rates)
    """
    
    pricing = {
        "gpt-4": {
            "input": 0.03 / 1000,
            "output": 0.06 / 1000
        },
        "gpt-4-turbo": {
            "input": 0.01 / 1000,
            "output": 0.03 / 1000
        },
        "gpt-35-turbo": {
            "input": 0.0015 / 1000,
            "output": 0.002 / 1000
        },
        "text-embedding-3-small": {
            "input": 0.00002 / 1000,
            "output": 0
        },
        "text-embedding-3-large": {
            "input": 0.00013 / 1000,
            "output": 0
        }
    }
    
    if model not in pricing:
        model = "gpt-4"  # Default
    
    input_cost = prompt_tokens * pricing[model]["input"]
    output_cost = completion_tokens * pricing[model]["output"]
    total_cost = input_cost + output_cost
    
    return {
        "prompt_tokens": prompt_tokens,
        "completion_tokens": completion_tokens,
        "total_tokens": prompt_tokens + completion_tokens,
        "input_cost": input_cost,
        "output_cost": output_cost,
        "total_cost": total_cost
    }

def optimize_prompt_tokens(text: str, max_tokens: int = 3000) -> str:
    """
    Truncate text to fit within token limit
    """
    encoding = tiktoken.encoding_for_model("gpt-4")
    tokens = encoding.encode(text)
    
    if len(tokens) <= max_tokens:
        return text
    
    # Truncate
    truncated_tokens = tokens[:max_tokens]
    return encoding.decode(truncated_tokens)

# Example usage
text = "This is a sample text for token counting."
tokens = count_tokens(text)
print(f"Token count: {tokens}")

messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello, how are you?"}
]

total_tokens = count_messages_tokens(messages)
print(f"Message tokens: {total_tokens}")

# Estimate cost
cost_estimate = estimate_cost(prompt_tokens=1000, completion_tokens=500, model="gpt-4")
print(f"Estimated cost: ${cost_estimate['total_cost']:.4f}")
```

### 6.7 Content Filtering

```python
def handle_content_filtering():
    """
    Azure OpenAI includes content filtering for:
    - Hate speech
    - Sexual content
    - Violence
    - Self-harm
    
    Severity levels: safe, low, medium, high
    """
    
    try:
        response = client.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "user", "content": "Tell me a story."}
            ]
        )
        
        # Check content filter results
        if hasattr(response.choices[0], 'content_filter_results'):
            filter_results = response.choices[0].content_filter_results
            
            print("Content Filter Results:")
            print(f"  Hate: {filter_results.hate.severity if filter_results.hate else 'N/A'}")
            print(f"  Sexual: {filter_results.sexual.severity if filter_results.sexual else 'N/A'}")
            print(f"  Violence: {filter_results.violence.severity if filter_results.violence else 'N/A'}")
            print(f"  Self-harm: {filter_results.self_harm.severity if filter_results.self_harm else 'N/A'}")
        
        return response.choices[0].message.content
        
    except Exception as e:
        if "content_filter" in str(e).lower():
            print("Content was filtered due to policy violation.")
            print(f"Error: {e}")
            return None
        raise

def check_prompt_shields():
    """
    Prompt Shields protect against jailbreak attempts
    """
    # This is automatically applied by Azure OpenAI
    # No additional code needed
    pass
```

### 6.8 Fine-tuning

```python
def prepare_fine_tuning_data(training_examples: list, output_file: str):
    """
    Prepare data for fine-tuning
    Format: JSONL with messages
    """
    import json
    
    with open(output_file, 'w') as f:
        for example in training_examples:
            json_line = json.dumps({
                "messages": example["messages"]
            })
            f.write(json_line + '\n')
    
    print(f"Training data saved to {output_file}")

def create_fine_tuning_job():
    """
    Create fine-tuning job via REST API
    """
    # Note: Fine-tuning is typically done via Azure OpenAI Studio or REST API
    # Python SDK support may vary
    
    fine_tuning_data = [
        {
            "messages": [
                {"role": "system", "content": "You are a helpful assistant."},
                {"role": "user", "content": "What is the capital of France?"},
                {"role": "assistant", "content": "The capital of France is Paris."}
            ]
        },
        # Add more examples...
    ]
    
    prepare_fine_tuning_data(fine_tuning_data, "training_data.jsonl")
    
    print("""
To create a fine-tuning job:
1. Upload training file to Azure OpenAI
2. Use Azure OpenAI Studio or REST API
3. Monitor training progress
4. Deploy fine-tuned model
    """)

# Training data format example
training_example = {
    "messages": [
        {"role": "system", "content": "Assistant helps users with product information."},
        {"role": "user", "content": "What's the warranty on product X?"},
        {"role": "assistant", "content": "Product X comes with a 2-year warranty covering manufacturing defects."}
    ]
}
```

---

## 7. AZURE AI SEARCH

Azure AI Search (formerly Azure Cognitive Search) provides AI-powered cloud search with semantic search, vector search, and hybrid retrieval.

### 7.1 Index Design

```python
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex,
    SimpleField,
    SearchableField,
    ComplexField,
    SearchField,
    SearchFieldDataType,
    VectorSearch,
    VectorSearchProfile,
    HnswAlgorithmConfiguration,
    SemanticConfiguration,
    SemanticSearch,
    SemanticField,
    SemanticPrioritizedFields
)
from azure.core.credentials import AzureKeyCredential
import os

# Initialize clients
search_endpoint = os.environ["SEARCH_ENDPOINT"]
search_key = os.environ["SEARCH_KEY"]

index_client = SearchIndexClient(
    endpoint=search_endpoint,
    credential=AzureKeyCredential(search_key)
)

def create_search_index_basic(index_name: str):
    """
    Create a basic search index
    """
    
    fields = [
        SimpleField(
            name="id",
            type=SearchFieldDataType.String,
            key=True,
            filterable=True
        ),
        SearchableField(
            name="title",
            type=SearchFieldDataType.String,
            searchable=True,
            filterable=True,
            sortable=True,
            facetable=True
        ),
        SearchableField(
            name="content",
            type=SearchFieldDataType.String,
            searchable=True,
            analyzer_name="en.microsoft"  # English analyzer
        ),
        SimpleField(
            name="category",
            type=SearchFieldDataType.String,
            filterable=True,
            facetable=True
        ),
        SimpleField(
            name="metadata",
            type=SearchFieldDataType.String,
            filterable=True
        ),
        SimpleField(
            name="created_date",
            type=SearchFieldDataType.DateTimeOffset,
            filterable=True,
            sortable=True
        )
    ]
    
    index = SearchIndex(name=index_name, fields=fields)
    
    result = index_client.create_or_update_index(index)
    print(f"Index '{result.name}' created")
    return result

def create_index_with_vector_search(index_name: str):
    """
    Create index with vector search support (NEW)
    """
    
    fields = [
        SimpleField(name="id", type=SearchFieldDataType.String, key=True),
        SearchableField(name="title", type=SearchFieldDataType.String),
        SearchableField(name="content", type=SearchFieldDataType.String),
        SearchField(
            name="content_vector",
            type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
            searchable=True,
            vector_search_dimensions=1536,  # For text-embedding-ada-002
            vector_search_profile_name="my-vector-profile"
        )
    ]
    
    # Configure vector search
    vector_search = VectorSearch(
        profiles=[
            VectorSearchProfile(
                name="my-vector-profile",
                algorithm_configuration_name="my-hnsw-config"
            )
        ],
        algorithms=[
            HnswAlgorithmConfiguration(
                name="my-hnsw-config",
                parameters={
                    "m": 4,  # Number of bi-directional links
                    "efConstruction": 400,  # Size of dynamic candidate list
                    "efSearch": 500,  # Size of the candidate list
                    "metric": "cosine"  # Distance metric
                }
            )
        ]
    )
    
    index = SearchIndex(
        name=index_name,
        fields=fields,
        vector_search=vector_search
    )
    
    result = index_client.create_or_update_index(index)
    print(f"Vector search index '{result.name}' created")
    return result

def create_index_with_semantic_search(index_name: str):
    """
    Create index with semantic search (powered by Bing)
    """
    
    fields = [
        SimpleField(name="id", type=SearchFieldDataType.String, key=True),
        SearchableField(name="title", type=SearchFieldDataType.String),
        SearchableField(name="content", type=SearchFieldDataType.String),
        SearchableField(name="description", type=SearchFieldDataType.String)
    ]
    
    # Semantic configuration
    semantic_config = SemanticConfiguration(
        name="my-semantic-config",
        prioritized_fields=SemanticPrioritizedFields(
            title_field=SemanticField(field_name="title"),
            content_fields=[SemanticField(field_name="content")],
            keywords_fields=[SemanticField(field_name="description")]
        )
    )
    
    semantic_search = SemanticSearch(
        configurations=[semantic_config]
    )
    
    index = SearchIndex(
        name=index_name,
        fields=fields,
        semantic_search=semantic_search
    )
    
    result = index_client.create_or_update_index(index)
    print(f"Semantic search index '{result.name}' created")
    return result

# Create indexes
create_search_index_basic("products-index")
create_index_with_vector_search("documents-vector-index")
create_index_with_semantic_search("articles-semantic-index")
```

### 7.2 Indexing Documents

```python
from azure.search.documents import SearchClient
from datetime import datetime

search_client = SearchClient(
    endpoint=search_endpoint,
    index_name="products-index",
    credential=AzureKeyCredential(search_key)
)

def upload_documents(documents: list):
    """
    Upload documents to search index
    """
    
    result = search_client.upload_documents(documents=documents)
    
    print(f"Uploaded {len(documents)} documents")
    for item in result:
        print(f"  {item.key}: {item.succeeded}")
    
    return result

def merge_or_upload_documents(documents: list):
    """
    Merge or upload - updates existing docs or creates new ones
    """
    
    result = search_client.merge_or_upload_documents(documents=documents)
    return result

def delete_documents(document_keys: list):
    """
    Delete documents by key
    """
    
    documents_to_delete = [{"id": key} for key in document_keys]
    result = search_client.delete_documents(documents=documents_to_delete)
    return result

def upload_with_vectors(documents: list, openai_client):
    """
    Upload documents with vector embeddings
    """
    
    # Generate embeddings for content
    for doc in documents:
        content = doc["content"]
        
        # Create embedding
        embedding_response = openai_client.embeddings.create(
            input=content,
            model="text-embedding-3-small"
        )
        
        doc["content_vector"] = embedding_response.data[0].embedding
    
    # Upload to search index
    result = search_client.upload_documents(documents=documents)
    return result

# Example documents
documents = [
    {
        "id": "1",
        "title": "Azure AI Search Overview",
        "content": "Azure AI Search provides powerful search capabilities...",
        "category": "documentation",
        "metadata": "azure,search",
        "created_date": datetime.utcnow().isoformat()
    },
    {
        "id": "2",
        "title": "Vector Search Guide",
        "content": "Vector search enables semantic similarity matching...",
        "category": "tutorial",
        "metadata": "vector,embedding",
        "created_date": datetime.utcnow().isoformat()
    }
]

upload_documents(documents)
```

### 7.3 Searching

```python
def simple_search(query: str, top: int = 10):
    """
    Simple full-text search
    """
    
    results = search_client.search(
        search_text=query,
        top=top,
        include_total_count=True
    )
    
    print(f"Found {results.get_count()} results for: '{query}'\n")
    
    for result in results:
        print(f"Title: {result['title']}")
        print(f"  Score: {result['@search.score']:.2f}")
        print(f"  Content: {result['content'][:100]}...\n")
    
    return list(results)

def filtered_search(query: str, filter_expression: str):
    """
    Search with OData filter
    
    Filter examples:
    - "category eq 'documentation'"
    - "created_date gt 2025-01-01T00:00:00Z"
    - "category eq 'tutorial' and metadata/any(t: t eq 'azure')"
    """
    
    results = search_client.search(
        search_text=query,
        filter=filter_expression,
        top=10
    )
    
    return list(results)

def faceted_search(query: str, facets: list):
    """
    Search with facets for navigation
    """
    
    results = search_client.search(
        search_text=query,
        facets=facets,
        top=10
    )
    
    print(f"Search: '{query}'\n")
    
    # Display facets
    if results.get_facets():
        print("Facets:")
        for facet_name, facet_results in results.get_facets().items():
            print(f"  {facet_name}:")
            for item in facet_results:
                print(f"    {item['value']}: {item['count']}")
        print()
    
    # Display results
    for result in results:
        print(f"- {result['title']}")
    
    return results

def semantic_search(query: str):
    """
    Semantic search with semantic ranking
    """
    
    results = search_client.search(
        search_text=query,
        query_type="semantic",
        semantic_configuration_name="my-semantic-config",
        query_caption="extractive",  # or "abstractive"
        query_answer="extractive",
        top=5
    )
    
    print(f"Semantic search: '{query}'\n")
    
    # Display semantic answers (if any)
    if results.get_answers():
        print("Semantic Answers:")
        for answer in results.get_answers():
            print(f"  Answer: {answer.text}")
            print(f"  Score: {answer.score:.2f}\n")
    
    # Display results with captions
    for result in results:
        print(f"Title: {result['title']}")
        print(f"  Reranker Score: {result['@search.reranker_score']:.2f}")
        
        # Display semantic captions
        if '@search.captions' in result:
            captions = result['@search.captions']
            if captions:
                print(f"  Caption: {captions[0].text}")
        print()
    
    return list(results)

def vector_search(query_vector: list, top: int = 5):
    """
    Pure vector search
    """
    from azure.search.documents.models import VectorizedQuery
    
    vector_query = VectorizedQuery(
        vector=query_vector,
        k_nearest_neighbors=top,
        fields="content_vector"
    )
    
    results = search_client.search(
        search_text=None,
        vector_queries=[vector_query],
        top=top
    )
    
    return list(results)

def hybrid_search(query_text: str, query_vector: list):
    """
    Hybrid search: combines keyword and vector search
    """
    from azure.search.documents.models import VectorizedQuery
    
    vector_query = VectorizedQuery(
        vector=query_vector,
        k_nearest_neighbors=50,
        fields="content_vector"
    )
    
    results = search_client.search(
        search_text=query_text,
        vector_queries=[vector_query],
        top=10
    )
    
    print(f"Hybrid search: '{query_text}'\n")
    
    for result in results:
        print(f"- {result['title']} (Score: {result['@search.score']:.2f})")
    
    return list(results)

def autocomplete_suggestions(prefix: str, suggester_name: str = "sg"):
    """
    Get autocomplete suggestions
    """
    
    results = search_client.autocomplete(
        search_text=prefix,
        suggester_name=suggester_name,
        mode="twoTerms",  # or "oneTerm", "oneTermWithContext"
        top=5
    )
    
    suggestions = [result['text'] for result in results]
    print(f"Autocomplete for '{prefix}': {suggestions}")
    
    return suggestions

# Example searches
simple_search("azure search")
filtered_search("vector", "category eq 'tutorial'")
faceted_search("azure", ["category", "metadata"])
semantic_search("What is vector search?")
```

## 8. AZURE BOT SERVICE & BOT FRAMEWORK

### 8.1 Bot Framework SDK v4 (Python)

```python
from botbuilder.core import ActivityHandler, TurnContext, MessageFactory
from botbuilder.schema import ChannelAccount, Activity, ActivityTypes
from botbuilder.dialogs import (
    ComponentDialog,
    WaterfallDialog,
    DialogTurnStatus,
    WaterfallStepContext,
    TextPrompt,
    ChoicePrompt,
    ConfirmPrompt,
    NumberPrompt,
    DateTimePrompt
)
from botbuilder.dialogs.prompts import PromptOptions
from botbuilder.dialogs.choices import Choice
import os

class EchoBot(ActivityHandler):
    """
    Basic echo bot that responds to user messages
    """
    
    async def on_message_activity(self, turn_context: TurnContext):
        # Get user message
        user_message = turn_context.activity.text
        
        # Create response
        response = f"You said: {user_message}"
        
        # Send response
        await turn_context.send_activity(MessageFactory.text(response))
    
    async def on_members_added_activity(
        self,
        members_added: list[ChannelAccount],
        turn_context: TurnContext
    ):
        """
        Greet new users when they join the conversation
        """
        for member in members_added:
            if member.id != turn_context.activity.recipient.id:
                await turn_context.send_activity(
                    "Hello! I'm your friendly bot. How can I help you today?"
                )

class AdaptiveCardsBot(ActivityHandler):
    """
    Bot that sends Adaptive Cards
    """
    
    async def on_message_activity(self, turn_context: TurnContext):
        # Create Adaptive Card
        card = {
            "type": "AdaptiveCard",
            "version": "1.4",
            "body": [
                {
                    "type": "TextBlock",
                    "text": "Welcome!",
                    "size": "Large",
                    "weight": "Bolder"
                },
                {
                    "type": "TextBlock",
                    "text": "This is an adaptive card",
                    "wrap": True
                },
                {
                    "type": "Image",
                    "url": "https://example.com/image.png",
                    "size": "Medium"
                }
            ],
            "actions": [
                {
                    "type": "Action.Submit",
                    "title": "Submit",
                    "data": {"action": "submit"}
                }
            ]
        }
        
        # Create attachment
        from botbuilder.core import CardFactory
        
        card_attachment = CardFactory.adaptive_card(card)
        message = MessageFactory.attachment(card_attachment)
        
        await turn_context.send_activity(message)
```

### 8.2 Dialogs and Conversation Flow

```python
class BookingDialog(ComponentDialog):
    """
    Multi-turn dialog for booking flow
    """
    
    def __init__(self, dialog_id: str):
        super().__init__(dialog_id)
        
        # Add prompts
        self.add_dialog(TextPrompt("TextPrompt"))
        self.add_dialog(DateTimePrompt("DateTimePrompt"))
        self.add_dialog(NumberPrompt("NumberPrompt"))
        self.add_dialog(ConfirmPrompt("ConfirmPrompt"))
        
        # Add waterfall dialog
        self.add_dialog(
            WaterfallDialog(
                "BookingWaterfall",
                [
                    self.destination_step,
                    self.date_step,
                    self.guests_step,
                    self.confirm_step,
                    self.final_step
                ]
            )
        )
        
        self.initial_dialog_id = "BookingWaterfall"
    
    async def destination_step(self, step_context: WaterfallStepContext):
        """Ask for destination"""
        return await step_context.prompt(
            "TextPrompt",
            PromptOptions(prompt=MessageFactory.text("Where would you like to go?"))
        )
    
    async def date_step(self, step_context: WaterfallStepContext):
        """Ask for date"""
        # Store destination
        step_context.values["destination"] = step_context.result
        
        return await step_context.prompt(
            "DateTimePrompt",
            PromptOptions(prompt=MessageFactory.text("When would you like to travel?"))
        )
    
    async def guests_step(self, step_context: WaterfallStepContext):
        """Ask for number of guests"""
        # Store date
        step_context.values["date"] = step_context.result[0].value
        
        return await step_context.prompt(
            "NumberPrompt",
            PromptOptions(prompt=MessageFactory.text("How many guests?"))
        )
    
    async def confirm_step(self, step_context: WaterfallStepContext):
        """Confirm booking details"""
        # Store guests
        step_context.values["guests"] = step_context.result
        
        # Create confirmation message
        destination = step_context.values["destination"]
        date = step_context.values["date"]
        guests = step_context.values["guests"]
        
        msg = f"Please confirm:\nDestination: {destination}\nDate: {date}\nGuests: {guests}"
        
        return await step_context.prompt(
            "ConfirmPrompt",
            PromptOptions(prompt=MessageFactory.text(msg))
        )
    
    async def final_step(self, step_context: WaterfallStepContext):
        """Process booking"""
        if step_context.result:
            # User confirmed
            await step_context.context.send_activity(
                "Great! Your booking has been confirmed."
            )
        else:
            await step_context.context.send_activity(
                "Booking cancelled."
            )
        
        return await step_context.end_dialog()
```

### 8.3 State Management

```python
from botbuilder.core import (
    BotFrameworkAdapter,
    ConversationState,
    UserState,
    MemoryStorage,
    TurnContext
)
from botbuilder.schema import Activity

# Initialize storage
storage = MemoryStorage()

# Create state management objects
user_state = UserState(storage)
conversation_state = ConversationState(storage)

class StatefulBot(ActivityHandler):
    """
    Bot with state management
    """
    
    def __init__(self, conversation_state: ConversationState, user_state: UserState):
        self.conversation_state = conversation_state
        self.user_state = user_state
        
        # Create state property accessors
        self.conversation_data_accessor = (
            conversation_state.create_property("ConversationData")
        )
        self.user_profile_accessor = user_state.create_property("UserProfile")
    
    async def on_message_activity(self, turn_context: TurnContext):
        # Get conversation data
        conversation_data = await self.conversation_data_accessor.get(
            turn_context, lambda: {"message_count": 0}
        )
        
        # Get user profile
        user_profile = await self.user_profile_accessor.get(
            turn_context, lambda: {"name": None}
        )
        
        # Update message count
        conversation_data["message_count"] += 1
        
        # Check if we have user's name
        if not user_profile["name"]:
            if turn_context.activity.text.lower().startswith("my name is"):
                user_profile["name"] = turn_context.activity.text[11:].strip()
                await turn_context.send_activity(
                    f"Nice to meet you, {user_profile['name']}!"
                )
            else:
                await turn_context.send_activity(
                    "What's your name? (Say 'My name is...')"
                )
        else:
            await turn_context.send_activity(
                f"Hello {user_profile['name']}! "
                f"This is message #{conversation_data['message_count']} in this conversation."
            )
    
    async def on_turn(self, turn_context: TurnContext):
        await super().on_turn(turn_context)
        
        # Save state changes
        await self.conversation_state.save_changes(turn_context)
        await self.user_state.save_changes(turn_context)
```

### 8.4 Integration with Azure AI Services

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

class IntelligentBot(ActivityHandler):
    """
    Bot integrated with Azure AI Language
    """
    
    def __init__(self):
        # Initialize Language service client
        self.text_analytics_client = TextAnalyticsClient(
            endpoint=os.environ["LANGUAGE_ENDPOINT"],
            credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
        )
    
    async def on_message_activity(self, turn_context: TurnContext):
        user_message = turn_context.activity.text
        
        # Analyze sentiment
        sentiment_result = self.text_analytics_client.analyze_sentiment(
            documents=[user_message]
        )[0]
        
        sentiment = sentiment_result.sentiment
        confidence = sentiment_result.confidence_scores
        
        # Respond based on sentiment
        if sentiment == "negative":
            response = "I sense you're frustrated. How can I help?"
        elif sentiment == "positive":
            response = "Great! I'm glad you're happy. What can I do for you?"
        else:
            response = "How can I assist you today?"
        
        await turn_context.send_activity(response)
        
        # Extract key phrases
        key_phrases = self.text_analytics_client.extract_key_phrases(
            documents=[user_message]
        )[0].key_phrases
        
        if key_phrases:
            await turn_context.send_activity(
                f"Key topics: {', '.join(key_phrases)}"
            )
```

### 8.5 QnA Maker Integration

```python
from azure.ai.language.questionanswering import QuestionAnsweringClient
from azure.core.credentials import AzureKeyCredential

class QnABot(ActivityHandler):
    """
    Bot with QnA Maker / Question Answering
    """
    
    def __init__(self):
        self.qna_client = QuestionAnsweringClient(
            endpoint=os.environ["QNA_ENDPOINT"],
            credential=AzureKeyCredential(os.environ["QNA_KEY"])
        )
        self.project_name = os.environ["QNA_PROJECT_NAME"]
        self.deployment_name = "production"
    
    async def on_message_activity(self, turn_context: TurnContext):
        question = turn_context.activity.text
        
        # Get answer from QnA
        from azure.ai.language.questionanswering.models import AnswersOptions
        
        output = self.qna_client.get_answers(
            question=question,
            project_name=self.project_name,
            deployment_name=self.deployment_name,
            options=AnswersOptions(
                confidence_threshold=0.5,
                top=3
            )
        )
        
        if output.answers:
            best_answer = output.answers[0]
            
            if best_answer.confidence > 0.7:
                await turn_context.send_activity(best_answer.answer)
            else:
                await turn_context.send_activity(
                    f"I'm not entirely sure, but here's what I found:\n{best_answer.answer}"
                )
        else:
            await turn_context.send_activity(
                "I don't have an answer for that. Can you rephrase?"
            )
```

### 8.6 Bot Deployment (Azure CLI)

```bash
# Create App Service Plan
az appservice plan create \
  --name "bot-app-plan" \
  --resource-group "ai-resources-rg" \
  --sku B1 \
  --is-linux

# Create Web App
az webapp create \
  --name "my-intelligent-bot" \
  --resource-group "ai-resources-rg" \
  --plan "bot-app-plan" \
  --runtime "PYTHON:3.9"

# Create Bot Service
az bot create \
  --name "my-intelligent-bot" \
  --resource-group "ai-resources-rg" \
  --kind "webapp" \
  --location "eastus" \
  --sku "F0" \
  --appid "<app-id>" \
  --password "<app-password>"

# Configure channels
az bot directline create \
  --name "my-intelligent-bot" \
  --resource-group "ai-resources-rg"

az bot msteams create \
  --name "my-intelligent-bot" \
  --resource-group "ai-resources-rg" \
  --enable-calling false
```

---

## 9. AZURE AI CONTENT SAFETY

Azure AI Content Safety detects harmful content in text and images.

### 9.1 Text Moderation

```python
from azure.ai.contentsafety import ContentSafetyClient
from azure.core.credentials import AzureKeyCredential
from azure.ai.contentsafety.models import AnalyzeTextOptions, TextCategory
import os

# Initialize client
endpoint = os.environ["CONTENT_SAFETY_ENDPOINT"]
key = os.environ["CONTENT_SAFETY_KEY"]

client = ContentSafetyClient(endpoint, AzureKeyCredential(key))

def analyze_text_content(text: str):
    """
    Analyze text for harmful content
    
    Categories:
    - Hate: Hate speech and discrimination
    - Sexual: Sexual content
    - Violence: Violent content
    - SelfHarm: Self-harm content
    
    Severity levels: 0 (Safe), 2 (Low), 4 (Medium), 6 (High)
    """
    
    request = AnalyzeTextOptions(text=text)
    
    try:
        response = client.analyze_text(request)
        
        print(f"\nAnalyzing: '{text[:50]}...'\n")
        print("Content Safety Results:")
        print(f"  Hate: Severity {response.hate_result.severity}")
        print(f"  Sexual: Severity {response.sexual_result.severity}")
        print(f"  Violence: Severity {response.violence_result.severity}")
        print(f"  Self-harm: Severity {response.self_harm_result.severity}")
        
        # Check if content should be blocked
        threshold = 4  # Block medium and high severity
        
        should_block = any([
            response.hate_result.severity >= threshold,
            response.sexual_result.severity >= threshold,
            response.violence_result.severity >= threshold,
            response.self_harm_result.severity >= threshold
        ])
        
        return {
            "hate": response.hate_result.severity,
            "sexual": response.sexual_result.severity,
            "violence": response.violence_result.severity,
            "self_harm": response.self_harm_result.severity,
            "should_block": should_block
        }
        
    except Exception as e:
        print(f"Error: {e}")
        return None

def analyze_text_with_categories(text: str, categories: list[TextCategory]):
    """
    Analyze specific categories only
    """
    request = AnalyzeTextOptions(
        text=text,
        categories=categories  # e.g., [TextCategory.HATE, TextCategory.VIOLENCE]
    )
    
    response = client.analyze_text(request)
    return response

# Example usage
result = analyze_text_content("This is a sample text to analyze for safety.")

if result and result["should_block"]:
    print("\n⚠️  Content blocked due to policy violation")
else:
    print("\n✅ Content is safe")
```

### 9.2 Image Moderation

```python
from azure.ai.contentsafety.models import AnalyzeImageOptions, ImageData
import base64

def analyze_image_content(image_path: str):
    """
    Analyze image for harmful content
    """
    
    # Read image and convert to base64
    with open(image_path, "rb") as f:
        image_bytes = f.read()
    
    image_data = ImageData(content=image_bytes)
    request = AnalyzeImageOptions(image=image_data)
    
    try:
        response = client.analyze_image(request)
        
        print(f"\nAnalyzing image: {image_path}\n")
        print("Content Safety Results:")
        print(f"  Hate: Severity {response.hate_result.severity}")
        print(f"  Sexual: Severity {response.sexual_result.severity}")
        print(f"  Violence: Severity {response.violence_result.severity}")
        print(f"  Self-harm: Severity {response.self_harm_result.severity}")
        
        return {
            "hate": response.hate_result.severity,
            "sexual": response.sexual_result.severity,
            "violence": response.violence_result.severity,
            "self_harm": response.self_harm_result.severity
        }
        
    except Exception as e:
        print(f"Error: {e}")
        return None

def analyze_image_from_url(image_url: str):
    """
    Analyze image from URL
    """
    import requests
    
    # Download image
    response = requests.get(image_url)
    image_bytes = response.content
    
    image_data = ImageData(content=image_bytes)
    request = AnalyzeImageOptions(image=image_data)
    
    result = client.analyze_image(request)
    return result

# Example
image_result = analyze_image_content("test_image.jpg")
```

### 9.3 Blocklists

```python
from azure.ai.contentsafety.models import (
    TextBlocklist,
    AddOrUpdateTextBlocklistItemsOptions,
    TextBlocklistItem,
    RemoveTextBlocklistItemsOptions
)

def create_blocklist(blocklist_name: str, description: str):
    """
    Create a custom blocklist
    """
    blocklist = TextBlocklist(
        blocklist_name=blocklist_name,
        description=description
    )
    
    result = client.create_or_update_text_blocklist(blocklist)
    print(f"Created blocklist: {result.blocklist_name}")
    return result

def add_blocklist_items(blocklist_name: str, items: list[str]):
    """
    Add items to blocklist
    """
    blocklist_items = [
        TextBlocklistItem(text=item) for item in items
    ]
    
    options = AddOrUpdateTextBlocklistItemsOptions(
        blocklist_items=blocklist_items
    )
    
    result = client.add_or_update_blocklist_items(
        blocklist_name=blocklist_name,
        options=options
    )
    
    print(f"Added {len(result.blocklist_items)} items to {blocklist_name}")
    return result

def analyze_text_with_blocklist(text: str, blocklist_names: list[str]):
    """
    Analyze text using custom blocklists
    """
    request = AnalyzeTextOptions(
        text=text,
        blocklist_names=blocklist_names
    )
    
    response = client.analyze_text(request)
    
    # Check for blocklist matches
    if response.blocklists_match:
        print("\n❌ Blocklist matches found:")
        for match in response.blocklists_match:
            print(f"  - Blocklist: {match.blocklist_name}")
            print(f"    Matched text: {match.blocklist_item_text}")
    
    return response

# Example usage
create_blocklist("profanity-list", "Custom profanity blocklist")
add_blocklist_items("profanity-list", ["badword1", "badword2", "badword3"])

analysis = analyze_text_with_blocklist(
    "This text contains badword1",
    blocklist_names=["profanity-list"]
)
```

### 9.4 Content Safety with Azure OpenAI

```python
def safe_chat_completion(user_message: str):
    """
    Combine Content Safety with Azure OpenAI
    """
    
    # Step 1: Check input with Content Safety
    safety_check = analyze_text_content(user_message)
    
    if safety_check["should_block"]:
        return {
            "success": False,
            "message": "Your message contains inappropriate content and cannot be processed."
        }
    
    # Step 2: Call Azure OpenAI
    from openai import AzureOpenAI
    
    openai_client = AzureOpenAI(
        api_key=os.environ["AZURE_OPENAI_KEY"],
        api_version="2024-02-01",
        azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"]
    )
    
    response = openai_client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "user", "content": user_message}
        ]
    )
    
    ai_response = response.choices[0].message.content
    
    # Step 3: Check output with Content Safety
    output_safety = analyze_text_content(ai_response)
    
    if output_safety["should_block"]:
        return {
            "success": False,
            "message": "The AI response was flagged and cannot be shown."
        }
    
    return {
        "success": True,
        "message": ai_response
    }

# Example
result = safe_chat_completion("Tell me about artificial intelligence.")
print(result["message"])
```

---

## 10. APPLIED AI SERVICES

### 10.1 Azure AI Video Indexer

```python
from azure.mgmt.videoindexer import VideoIndexerManagementClient
from azure.identity import DefaultAzureCredential
import requests
import os

def get_access_token(account_id: str, location: str):
    """
    Get Video Indexer access token
    """
    # Use Azure AD authentication
    credential = DefaultAzureCredential()
    token = credential.get_token("https://management.azure.com/.default")
    
    url = f"https://api.videoindexer.ai/auth/{location}/Accounts/{account_id}/AccessToken"
    
    headers = {
        "Authorization": f"Bearer {token.token}"
    }
    
    params = {
        "allowEdit": True
    }
    
    response = requests.get(url, headers=headers, params=params)
    return response.json()

def upload_video(video_path: str, video_name: str, account_id: str, location: str, access_token: str):
    """
    Upload video for indexing
    """
    url = f"https://api.videoindexer.ai/{location}/Accounts/{account_id}/Videos"
    
    params = {
        "accessToken": access_token,
        "name": video_name,
        "privacy": "Private",
        "language": "en-US"
    }
    
    with open(video_path, "rb") as f:
        files = {"file": f}
        response = requests.post(url, params=params, files=files)
    
    if response.status_code == 200:
        video_id = response.json()["id"]
        print(f"Video uploaded successfully. ID: {video_id}")
        return video_id
    else:
        print(f"Error: {response.status_code}")
        return None

def get_video_insights(video_id: str, account_id: str, location: str, access_token: str):
    """
    Get video insights (transcription, faces, topics, etc.)
    """
    url = f"https://api.videoindexer.ai/{location}/Accounts/{account_id}/Videos/{video_id}/Index"
    
    params = {
        "accessToken": access_token,
        "language": "en-US"
    }
    
    response = requests.get(url, params=params)
    
    if response.status_code == 200:
        insights = response.json()
        
        # Extract key information
        print(f"\nVideo: {insights['name']}")
        print(f"Duration: {insights['durationInSeconds']}s\n")
        
        # Transcription
        if 'videos' in insights and insights['videos']:
            video_insights = insights['videos'][0]['insights']
            
            if 'transcript' in video_insights:
                print("Transcript:")
                for line in video_insights['transcript'][:5]:  # First 5 lines
                    print(f"  [{line['instances'][0]['start']}] {line['text']}")
            
            # Detected faces
            if 'faces' in video_insights:
                print(f"\nDetected {len(video_insights['faces'])} faces:")
                for face in video_insights['faces'][:5]:
                    print(f"  - {face.get('name', 'Unknown')} (Confidence: {face.get('confidence', 0):.2f})")
            
            # Topics
            if 'topics' in video_insights:
                print(f"\nTopics:")
                for topic in video_insights['topics'][:5]:
                    print(f"  - {topic['name']} (Confidence: {topic.get('confidence', 0):.2f})")
            
            # Keywords
            if 'keywords' in video_insights:
                print(f"\nKeywords:")
                for keyword in video_insights['keywords'][:10]:
                    print(f"  - {keyword['text']}")
        
        return insights
    else:
        print(f"Error: {response.status_code}")
        return None

def search_video(query: str, account_id: str, location: str, access_token: str):
    """
    Search within video content
    """
    url = f"https://api.videoindexer.ai/{location}/Accounts/{account_id}/Videos/Search"
    
    params = {
        "accessToken": access_token,
        "query": query
    }
    
    response = requests.get(url, params=params)
    return response.json()
```

### 10.2 Azure AI Immersive Reader

```python
def get_immersive_reader_token():
    """
    Get token for Immersive Reader
    """
    from azure.identity import ClientSecretCredential
    
    credential = ClientSecretCredential(
        tenant_id=os.environ["AZURE_TENANT_ID"],
        client_id=os.environ["AZURE_CLIENT_ID"],
        client_secret=os.environ["AZURE_CLIENT_SECRET"]
    )
    
    token = credential.get_token("https://cognitiveservices.azure.com/.default")
    return token.token

# HTML integration (JavaScript)
immersive_reader_html = """
<!DOCTYPE html>
<html>
<head>
    <script type="text/javascript" src="https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.1.2.0.js"></script>
</head>
<body>
    <div id="content">
        <h1>Sample Content</h1>
        <p>This text will be displayed in Immersive Reader for enhanced readability.</p>
    </div>
    
    <button onclick="launchImmersiveReader()">Launch Immersive Reader</button>
    
    <script>
        async function launchImmersiveReader() {
            const content = {
                title: "Sample Content",
                chunks: [{
                    content: document.getElementById("content").innerText,
                    mimeType: "text/plain"
                }]
            };
            
            const options = {
                onExit: () => console.log("Immersive Reader closed"),
                uiZIndex: 1000
            };
            
            // Get token from backend
            const response = await fetch("/get-token");
            const data = await response.json();
            
            ImmersiveReader.launchAsync(
                data.token,
                data.subdomain,
                content,
                options
            );
        }
    </script>
</body>
</html>
"""
```

### 10.3 Azure Metrics Advisor

```python
from azure.ai.metricsadvisor import MetricsAdvisorClient, MetricsAdvisorAdministrationClient
from azure.ai.metricsadvisor.models import (
    DataFeed,
    DataFeedSchema,
    DataFeedMetric,
    DataFeedDimension,
    DataFeedGranularity,
    DataFeedIngestionSettings,
    AnomalyAlertConfiguration,
    MetricAlertConfiguration
)
from azure.core.credentials import AzureKeyCredential
from datetime import datetime

# Initialize clients
endpoint = os.environ["METRICS_ADVISOR_ENDPOINT"]
subscription_key = os.environ["METRICS_ADVISOR_KEY"]
api_key = os.environ["METRICS_ADVISOR_API_KEY"]

client = MetricsAdvisorClient(
    endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key)
)

admin_client = MetricsAdvisorAdministrationClient(
    endpoint,
    MetricsAdvisorKeyCredential(subscription_key, api_key)
)

def create_data_feed(data_feed_name: str):
    """
    Create a data feed for anomaly detection
    """
    data_feed = DataFeed(
        name=data_feed_name,
        source=SqlServerDataFeedSource(
            connection_string=os.environ["SQL_CONNECTION_STRING"],
            query="SELECT timestamp, revenue, cost FROM metrics"
        ),
        granularity=DataFeedGranularity(
            granularity_type="Daily"
        ),
        schema=DataFeedSchema(
            metrics=[
                DataFeedMetric(name="revenue"),
                DataFeedMetric(name="cost")
            ],
            dimensions=[
                DataFeedDimension(name="category"),
                DataFeedDimension(name="region")
            ],
            timestamp_column="timestamp"
        ),
        ingestion_settings=DataFeedIngestionSettings(
            ingestion_start_time=datetime(2025, 1, 1)
        )
    )
    
    created_data_feed = admin_client.create_data_feed(data_feed)
    print(f"Created data feed: {created_data_feed.id}")
    return created_data_feed

def detect_anomalies(data_feed_id: str, metric_id: str):
    """
    Detect anomalies in metrics
    """
    from datetime import datetime, timedelta
    
    start_time = datetime(2025, 1, 1)
    end_time = datetime.now()
    
    anomalies = client.list_anomalies(
        alert_configuration_id="<config-id>",
        start_time=start_time,
        end_time=end_time
    )
    
    print("\nDetected Anomalies:")
    for anomaly in anomalies:
        print(f"  Timestamp: {anomaly.timestamp}")
        print(f"  Severity: {anomaly.severity}")
        print(f"  Status: {anomaly.status}")
        print(f"  Dimensions: {anomaly.dimension}")
        print()
    
    return list(anomalies)
```

## 11. RESPONSIBLE AI & GOVERNANCE

### 11.1 Responsible AI Principles

**Microsoft's 6 Principles:**
1. **Fairness**: AI systems should treat all people fairly
2. **Reliability & Safety**: AI systems should perform reliably and safely
3. **Privacy & Security**: AI systems should be secure and respect privacy
4. **Inclusiveness**: AI systems should empower everyone and engage people
5. **Transparency**: AI systems should be understandable
6. **Accountability**: People should be accountable for AI systems

### 11.2 Fairness Assessment

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

def test_fairness_across_demographics(texts_by_group: dict):
    """
    Test sentiment analysis fairness across demographic groups
    
    Args:
        texts_by_group: Dict with group names as keys and lists of texts as values
    """
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
    )
    
    results_by_group = {}
    
    for group_name, texts in texts_by_group.items():
        sentiment_results = client.analyze_sentiment(documents=texts)
        
        # Calculate average scores
        positive_scores = []
        negative_scores = []
        
        for result in sentiment_results:
            if not result.is_error:
                positive_scores.append(result.confidence_scores.positive)
                negative_scores.append(result.confidence_scores.negative)
        
        results_by_group[group_name] = {
            "avg_positive": sum(positive_scores) / len(positive_scores) if positive_scores else 0,
            "avg_negative": sum(negative_scores) / len(negative_scores) if negative_scores else 0
        }
    
    # Display results
    print("\nFairness Analysis Across Groups:")
    print("=" * 50)
    for group, scores in results_by_group.items():
        print(f"\n{group}:")
        print(f"  Average Positive: {scores['avg_positive']:.3f}")
        print(f"  Average Negative: {scores['avg_negative']:.3f}")
    
    # Check for disparities
    positive_scores = [s['avg_positive'] for s in results_by_group.values()]
    disparity = max(positive_scores) - min(positive_scores)
    
    if disparity > 0.1:  # 10% threshold
        print(f"\n⚠️  Warning: Disparity of {disparity:.3f} detected between groups")
    else:
        print(f"\n✅ Results are relatively fair (disparity: {disparity:.3f})")
    
    return results_by_group

# Example usage
texts_by_group = {
    "Group A": ["This is a great product", "I love this service", "Excellent quality"],
    "Group B": ["This is wonderful", "Amazing experience", "Very satisfied"]
}

fairness_results = test_fairness_across_demographics(texts_by_group)
```

### 11.3 Transparency and Explainability

```python
def explain_vision_results(image_analysis_result):
    """
    Provide transparent explanations for vision API results
    """
    explanation = {
        "model_version": "Image Analysis 4.0",
        "confidence_threshold": 0.5,
        "features_analyzed": []
    }
    
    if hasattr(image_analysis_result, 'caption'):
        explanation["features_analyzed"].append({
            "feature": "Caption Generation",
            "description": "AI-generated description of image content",
            "confidence": image_analysis_result.caption.confidence,
            "value": image_analysis_result.caption.text
        })
    
    if hasattr(image_analysis_result, 'objects'):
        explanation["features_analyzed"].append({
            "feature": "Object Detection",
            "description": "Identified objects with bounding boxes",
            "count": len(image_analysis_result.objects.list),
            "objects": [
                {
                    "name": obj.tags[0].name if obj.tags else "Unknown",
                    "confidence": obj.tags[0].confidence if obj.tags else 0
                }
                for obj in image_analysis_result.objects.list
            ]
        })
    
    print("\n🔍 Model Explanation:")
    print("=" * 60)
    print(f"Model: {explanation['model_version']}")
    print(f"Confidence Threshold: {explanation['confidence_threshold']}")
    print(f"\nFeatures Analyzed:")
    
    for feature in explanation["features_analyzed"]:
        print(f"\n  {feature['feature']}:")
        print(f"    Description: {feature['description']}")
        if 'confidence' in feature:
            print(f"    Confidence: {feature['confidence']:.2%}")
    
    return explanation

def log_ai_decision(decision: dict, reason: str, confidence: float):
    """
    Log AI decisions for accountability
    """
    import json
    from datetime import datetime
    
    log_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "decision": decision,
        "reason": reason,
        "confidence": confidence,
        "model_version": "1.0",
        "user_id": "<user-id>"
    }
    
    # Log to file or monitoring system
    with open("ai_decisions.log", "a") as f:
        f.write(json.dumps(log_entry) + "\n")
    
    print(f"Decision logged: {decision['action']}")
```

### 11.4 Privacy Preservation

```python
from azure.ai.textanalytics import TextAnalyticsClient
from azure.ai.textanalytics.models import PiiEntityCategory

def redact_pii(text: str):
    """
    Detect and redact PII before processing
    """
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
    )
    
    # Detect PII
    result = client.recognize_pii_entities(
        documents=[text],
        categories=[
            PiiEntityCategory.EMAIL,
            PiiEntityCategory.PHONE_NUMBER,
            PiiEntityCategory.US_SOCIAL_SECURITY_NUMBER,
            PiiEntityCategory.CREDIT_CARD_NUMBER,
            PiiEntityCategory.IP_ADDRESS
        ]
    )[0]
    
    if result.is_error:
        return text
    
    # Redact PII
    redacted_text = text
    for entity in sorted(result.entities, key=lambda e: e.offset, reverse=True):
        start = entity.offset
        end = entity.offset + entity.length
        redacted_text = redacted_text[:start] + "[REDACTED]" + redacted_text[end:]
    
    print(f"\nOriginal: {text}")
    print(f"Redacted: {redacted_text}")
    print(f"\nPII Found: {len(result.entities)} items")
    for entity in result.entities:
        print(f"  - {entity.category}: {entity.text}")
    
    return redacted_text

# Example
text = "Contact John at john@email.com or call 555-1234. SSN: 123-45-6789"
redacted = redact_pii(text)
```

---

## 12. SECURITY, NETWORKING & IDENTITY

### 12.1 Authentication Methods

```python
from azure.identity import (
    DefaultAzureCredential,
    ClientSecretCredential,
    ManagedIdentityCredential,
    AzureCliCredential
)
from azure.ai.textanalytics import TextAnalyticsClient
import os

# Method 1: API Key (least secure)
def auth_with_api_key():
    from azure.core.credentials import AzureKeyCredential
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
    )
    return client

# Method 2: Azure AD - Service Principal
def auth_with_service_principal():
    credential = ClientSecretCredential(
        tenant_id=os.environ["AZURE_TENANT_ID"],
        client_id=os.environ["AZURE_CLIENT_ID"],
        client_secret=os.environ["AZURE_CLIENT_SECRET"]
    )
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=credential
    )
    return client

# Method 3: Managed Identity (most secure for Azure resources)
def auth_with_managed_identity():
    """
    Best practice for applications running in Azure
    No secrets in code!
    """
    credential = ManagedIdentityCredential()
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=credential
    )
    return client

# Method 4: Default Azure Credential (automatic fallback)
def auth_with_default_credential():
    """
    Tries multiple authentication methods in order:
    1. Environment variables
    2. Managed Identity
    3. Azure CLI
    4. Azure PowerShell
    """
    credential = DefaultAzureCredential()
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=credential
    )
    return client

# Recommended approach
client = auth_with_default_credential()
```

### 12.2 Virtual Networks and Private Endpoints

```bash
# Create Virtual Network
az network vnet create \
  --name "ai-vnet" \
  --resource-group "ai-resources-rg" \
  --address-prefix "10.0.0.0/16" \
  --subnet-name "ai-subnet" \
  --subnet-prefix "10.0.1.0/24"

# Create Private Endpoint for AI Service
az network private-endpoint create \
  --name "ai-language-pe" \
  --resource-group "ai-resources-rg" \
  --vnet-name "ai-vnet" \
  --subnet "ai-subnet" \
  --private-connection-resource-id "/subscriptions/<sub-id>/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-language-resource" \
  --group-id "account" \
  --connection-name "ai-language-connection"

# Create Private DNS Zone
az network private-dns zone create \
  --resource-group "ai-resources-rg" \
  --name "privatelink.cognitiveservices.azure.com"

# Link DNS Zone to VNet
az network private-dns link vnet create \
  --resource-group "ai-resources-rg" \
  --zone-name "privatelink.cognitiveservices.azure.com" \
  --name "ai-dns-link" \
  --virtual-network "ai-vnet" \
  --registration-enabled false

# Configure Network Rules
az cognitiveservices account network-rule add \
  --name "my-language-resource" \
  --resource-group "ai-resources-rg" \
  --vnet-name "ai-vnet" \
  --subnet "ai-subnet"

# Deny public access
az cognitiveservices account update \
  --name "my-language-resource" \
  --resource-group "ai-resources-rg" \
  --set properties.publicNetworkAccess=Disabled
```

### 12.3 Role-Based Access Control (RBAC)

```bash
# Built-in roles for AI services

# 1. Cognitive Services User (read-only)
az role assignment create \
  --assignee "user@example.com" \
  --role "Cognitive Services User" \
  --scope "/subscriptions/<sub-id>/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-language-resource"

# 2. Cognitive Services Contributor (read/write)
az role assignment create \
  --assignee "<service-principal-id>" \
  --role "Cognitive Services Contributor" \
  --scope "/subscriptions/<sub-id>/resourceGroups/ai-resources-rg"

# 3. Cognitive Services OpenAI User
az role assignment create \
  --assignee "<managed-identity-id>" \
  --role "Cognitive Services OpenAI User" \
  --scope "/subscriptions/<sub-id>/resourceGroups/ai-resources-rg/providers/Microsoft.CognitiveServices/accounts/my-openai-resource"

# List role assignments
az role assignment list \
  --scope "/subscriptions/<sub-id>/resourceGroups/ai-resources-rg" \
  --output table
```

### 12.4 Key Rotation and Secret Management

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential
import os

def store_key_in_keyvault(key_name: str, key_value: str):
    """
    Store AI service key in Azure Key Vault
    """
    credential = DefaultAzureCredential()
    vault_url = f"https://{os.environ['KEYVAULT_NAME']}.vault.azure.net"
    
    client = SecretClient(vault_url=vault_url, credential=credential)
    client.set_secret(key_name, key_value)
    
    print(f"Key '{key_name}' stored in Key Vault")

def get_key_from_keyvault(key_name: str) -> str:
    """
    Retrieve AI service key from Key Vault
    """
    credential = DefaultAzureCredential()
    vault_url = f"https://{os.environ['KEYVAULT_NAME']}.vault.azure.net"
    
    client = SecretClient(vault_url=vault_url, credential=credential)
    secret = client.get_secret(key_name)
    
    return secret.value

def rotate_cognitive_services_key(resource_name: str, resource_group: str):
    """
    Rotate AI service keys
    """
    import subprocess
    
    # Regenerate key2
    result = subprocess.run([
        "az", "cognitiveservices", "account", "keys", "regenerate",
        "--name", resource_name,
        "--resource-group", resource_group,
        "--key-name", "Key2"
    ], capture_output=True, text=True)
    
    # Update Key Vault with new key
    import json
    keys = json.loads(result.stdout)
    new_key = keys['key2']
    
    store_key_in_keyvault(f"{resource_name}-key2", new_key)
    
    print(f"Key rotated for {resource_name}")

# Example: Use Key Vault for authentication
def create_client_with_keyvault():
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    
    # Get key from Key Vault
    api_key = get_key_from_keyvault("language-api-key")
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=AzureKeyCredential(api_key)
    )
    
    return client
```

---

## 13. MONITORING, DIAGNOSTICS & OPTIMIZATION

### 13.1 Azure Monitor Integration

```python
from azure.monitor.query import LogsQueryClient, MetricsQueryClient
from azure.identity import DefaultAzureCredential
from datetime import datetime, timedelta
import os

def query_ai_service_logs(workspace_id: str):
    """
    Query Azure Monitor logs for AI service
    """
    credential = DefaultAzureCredential()
    client = LogsQueryClient(credential)
    
    # Query for last 24 hours
    query = """
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.COGNITIVESERVICES"
    | where TimeGenerated > ago(24h)
    | summarize 
        TotalCalls = count(),
        AvgDuration = avg(DurationMs),
        ErrorCount = countif(ResultType == "Error")
        by bin(TimeGenerated, 1h)
    | order by TimeGenerated desc
    """
    
    response = client.query_workspace(
        workspace_id=workspace_id,
        query=query,
        timespan=timedelta(days=1)
    )
    
    for table in response.tables:
        for row in table.rows:
            print(f"Time: {row[0]}, Calls: {row[1]}, Avg Duration: {row[2]}ms, Errors: {row[3]}")
    
    return response

def get_ai_service_metrics(resource_id: str):
    """
    Get metrics for AI service
    """
    credential = DefaultAzureCredential()
    client = MetricsQueryClient(credential)
    
    # Query metrics
    metrics = client.query_resource(
        resource_uri=resource_id,
        metric_names=["TotalCalls", "SuccessRate", "TotalErrors", "TotalTokens"],
        timespan=timedelta(hours=24),
        granularity=timedelta(hours=1),
        aggregations=["Average", "Total"]
    )
    
    for metric in metrics.metrics:
        print(f"\nMetric: {metric.name}")
        for timeseries in metric.timeseries:
            for data_point in timeseries.data:
                print(f"  {data_point.time_stamp}: {data_point.total or data_point.average}")
    
    return metrics
```

### 13.2 Application Insights

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
from opencensus.ext.azure import metrics_exporter
from opencensus.stats import aggregation as aggregation_module
from opencensus.stats import measure as measure_module
from opencensus.stats import stats as stats_module
from opencensus.stats import view as view_module
from opencensus.tags import tag_map as tag_map_module
import logging
import os

# Configure Application Insights
connection_string = os.environ["APPLICATIONINSIGHTS_CONNECTION_STRING"]

# Setup logging
logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler(connection_string=connection_string))
logger.setLevel(logging.INFO)

def log_ai_request(operation: str, duration_ms: float, success: bool, **kwargs):
    """
    Log AI service request to Application Insights
    """
    properties = {
        "operation": operation,
        "duration_ms": duration_ms,
        "success": success,
        **kwargs
    }
    
    if success:
        logger.info(f"AI operation succeeded: {operation}", extra={"custom_dimensions": properties})
    else:
        logger.error(f"AI operation failed: {operation}", extra={"custom_dimensions": properties})

def track_custom_metric(metric_name: str, value: float):
    """
    Track custom metrics
    """
    exporter = metrics_exporter.new_metrics_exporter(
        connection_string=connection_string
    )
    
    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder
    
    # Create measure
    measure = measure_module.MeasureFloat(
        metric_name,
        f"Custom metric: {metric_name}",
        "units"
    )
    
    # Create view
    view = view_module.View(
        metric_name,
        f"View for {metric_name}",
        [],
        measure,
        aggregation_module.LastValueAggregation()
    )
    
    view_manager.register_view(view)
    view_manager.register_exporter(exporter)
    
    # Record metric
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()
    
    mmap.measure_float_put(measure, value)
    mmap.record(tmap)

# Example usage
import time

start_time = time.time()

try:
    # Perform AI operation
    # ...
    
    duration = (time.time() - start_time) * 1000
    log_ai_request(
        operation="sentiment_analysis",
        duration_ms=duration,
        success=True,
        model="default",
        language="en"
    )
    
    track_custom_metric("ai_request_duration", duration)
    
except Exception as e:
    duration = (time.time() - start_time) * 1000
    log_ai_request(
        operation="sentiment_analysis",
        duration_ms=duration,
        success=False,
        error=str(e)
    )
```

### 13.3 Cost Optimization

```python
def estimate_monthly_cost(calls_per_day: int, service_type: str, tier: str) -> dict:
    """
    Estimate monthly costs for AI services
    """
    
    # Pricing (example rates - verify current pricing)
    pricing = {
        "language": {
            "free": {"limit": 5000, "cost_per_1k": 0},
            "standard": {"limit": float('inf'), "cost_per_1k": 1.00}
        },
        "vision": {
            "free": {"limit": 5000, "cost_per_1k": 0},
            "standard": {"limit": float('inf'), "cost_per_1k": 1.50}
        },
        "openai_gpt4": {
            "standard": {"input_per_1k_tokens": 0.03, "output_per_1k_tokens": 0.06}
        }
    }
    
    monthly_calls = calls_per_day * 30
    
    if service_type in ["language", "vision"]:
        tier_pricing = pricing[service_type][tier]
        
        if monthly_calls <= tier_pricing["limit"]:
            monthly_cost = 0
        else:
            billable_calls = monthly_calls - tier_pricing["limit"]
            monthly_cost = (billable_calls / 1000) * tier_pricing["cost_per_1k"]
    
    elif service_type == "openai_gpt4":
        # Assume 1000 tokens per call (500 input + 500 output)
        monthly_input_tokens = monthly_calls * 500
        monthly_output_tokens = monthly_calls * 500
        
        input_cost = (monthly_input_tokens / 1000) * pricing[service_type]["standard"]["input_per_1k_tokens"]
        output_cost = (monthly_output_tokens / 1000) * pricing[service_type]["standard"]["output_per_1k_tokens"]
        monthly_cost = input_cost + output_cost
    
    return {
        "service": service_type,
        "tier": tier,
        "calls_per_month": monthly_calls,
        "estimated_monthly_cost": monthly_cost,
        "annual_estimate": monthly_cost * 12
    }

# Example
cost = estimate_monthly_cost(calls_per_day=10000, service_type="language", tier="standard")
print(f"Estimated monthly cost: ${cost['estimated_monthly_cost']:.2f}")
print(f"Annual estimate: ${cost['annual_estimate']:.2f}")

def optimize_batch_requests(texts: list, batch_size: int = 10):
    """
    Batch requests to reduce costs and improve throughput
    """
    from azure.ai.textanalytics import TextAnalyticsClient
    from azure.core.credentials import AzureKeyCredential
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
    )
    
    results = []
    
    # Process in batches
    for i in range(0, len(texts), batch_size):
        batch = texts[i:i+batch_size]
        batch_results = client.analyze_sentiment(documents=batch)
        results.extend(batch_results)
    
    print(f"Processed {len(texts)} texts in {len(range(0, len(texts), batch_size))} API calls")
    print(f"Cost savings: ~{((len(texts) - len(range(0, len(texts), batch_size))) / len(texts)) * 100:.0f}%")
    
    return results
```

---

## 14. DEPLOYMENT & INFRASTRUCTURE AS CODE

### 14.1 Bicep Templates

```bicep
// cognitive-services.bicep

@description('Name of the Cognitive Services account')
param accountName string

@description('Location for all resources')
param location string = resourceGroup().location

@description('SKU tier')
@allowed([
  'F0'  // Free
  'S0'  // Standard
])
param sku string = 'S0'

@description('Kind of Cognitive Service')
@allowed([
  'TextAnalytics'
  'ComputerVision'
  'Face'
  'SpeechServices'
  'OpenAI'
  'CognitiveServices'  // Multi-service
])
param kind string = 'CognitiveServices'

// Cognitive Services Account
resource cognitiveService 'Microsoft.CognitiveServices/accounts@2023-05-01' = {
  name: accountName
  location: location
  sku: {
    name: sku
  }
  kind: kind
  properties: {
    apiProperties: {}
    customSubDomainName: accountName
    networkAcls: {
      defaultAction: 'Allow'
      virtualNetworkRules: []
      ipRules: []
    }
    publicNetworkAccess: 'Enabled'
    disableLocalAuth: false
  }
}

// Diagnostic Settings
resource diagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: '${accountName}-diagnostics'
  scope: cognitiveService
  properties: {
    workspaceId: logAnalyticsWorkspace.id
    logs: [
      {
        category: 'Audit'
        enabled: true
        retentionPolicy: {
          enabled: true
          days: 30
        }
      }
      {
        category: 'RequestResponse'
        enabled: true
        retentionPolicy: {
          enabled: true
          days: 30
        }
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
        retentionPolicy: {
          enabled: true
          days: 30
        }
      }
    ]
  }
}

// Log Analytics Workspace
resource logAnalyticsWorkspace 'Microsoft.OperationalInsights/workspaces@2022-10-01' = {
  name: '${accountName}-logs'
  location: location
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 30
  }
}

// Outputs
output endpoint string = cognitiveService.properties.endpoint
output resourceId string = cognitiveService.id
output name string = cognitiveService.name
```

```bicep
// openai-deployment.bicep

@description('OpenAI account name')
param openAiAccountName string

@description('Model deployment name')
param deploymentName string = 'gpt-4'

@description('Model name')
@allowed([
  'gpt-4'
  'gpt-4-32k'
  'gpt-35-turbo'
  'gpt-35-turbo-16k'
  'text-embedding-ada-002'
])
param modelName string = 'gpt-4'

@description('Model version')
param modelVersion string = '0613'

@description('Deployment capacity (TPM in thousands)')
param capacity int = 10

// Reference existing OpenAI account
resource openAiAccount 'Microsoft.CognitiveServices/accounts@2023-05-01' existing = {
  name: openAiAccountName
}

// Model Deployment
resource deployment 'Microsoft.CognitiveServices/accounts/deployments@2023-05-01' = {
  parent: openAiAccount
  name: deploymentName
  properties: {
    model: {
      format: 'OpenAI'
      name: modelName
      version: modelVersion
    }
    raiPolicyName: 'Microsoft.Default'
  }
  sku: {
    name: 'Standard'
    capacity: capacity
  }
}

output deploymentId string = deployment.id
output deploymentName string = deployment.name
```

### 14.2 Azure Resource Manager (ARM) Templates

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "accountName": {
      "type": "string",
      "metadata": {
        "description": "Name for the Cognitive Services account"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources"
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "S0",
      "allowedValues": ["F0", "S0"],
      "metadata": {
        "description": "SKU tier"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.CognitiveServices/accounts",
      "apiVersion": "2023-05-01",
      "name": "[parameters('accountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('sku')]"
      },
      "kind": "CognitiveServices",
      "properties": {
        "apiProperties": {},
        "customSubDomainName": "[parameters('accountName')]",
        "networkAcls": {
          "defaultAction": "Allow"
        },
        "publicNetworkAccess": "Enabled"
      }
    }
  ],
  "outputs": {
    "endpoint": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.CognitiveServices/accounts', parameters('accountName'))).endpoint]"
    }
  }
}
```

### 14.3 Terraform

```hcl
# cognitive-services.tf

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Resource Group
resource "azurerm_resource_group" "ai_rg" {
  name     = "ai-resources-rg"
  location = "East US"
}

# Cognitive Services - Language
resource "azurerm_cognitive_account" "language" {
  name                = "my-language-service"
  location            = azurerm_resource_group.ai_rg.location
  resource_group_name = azurerm_resource_group.ai_rg.name
  kind                = "TextAnalytics"
  sku_name            = "S0"

  identity {
    type = "SystemAssigned"
  }

  custom_subdomain_name = "my-language-service"
  public_network_access_enabled = true

  tags = {
    environment = "production"
    project     = "ai-102"
  }
}

# Cognitive Services - Vision
resource "azurerm_cognitive_account" "vision" {
  name                = "my-vision-service"
  location            = azurerm_resource_group.ai_rg.location
  resource_group_name = azurerm_resource_group.ai_rg.name
  kind                = "ComputerVision"
  sku_name            = "S1"

  custom_subdomain_name = "my-vision-service"
  public_network_access_enabled = true
}

# Azure OpenAI
resource "azurerm_cognitive_account" "openai" {
  name                = "my-openai-service"
  location            = "East US"
  resource_group_name = azurerm_resource_group.ai_rg.name
  kind                = "OpenAI"
  sku_name            = "S0"

  custom_subdomain_name = "my-openai-service"
  public_network_access_enabled = true
}

# OpenAI Model Deployment
resource "azurerm_cognitive_deployment" "gpt4" {
  name                 = "gpt-4-deployment"
  cognitive_account_id = azurerm_cognitive_account.openai.id

  model {
    format  = "OpenAI"
    name    = "gpt-4"
    version = "0613"
  }

  scale {
    type     = "Standard"
    capacity = 10
  }
}

# Outputs
output "language_endpoint" {
  value     = azurerm_cognitive_account.language.endpoint
  sensitive = false
}

output "language_key" {
  value     = azurerm_cognitive_account.language.primary_access_key
  sensitive = true
}

output "openai_endpoint" {
  value = azurerm_cognitive_account.openai.endpoint
}
```

### 14.4 Container Deployment

```yaml
# docker-compose.yml for AI services

version: '3.8'

services:
  language-container:
    image: mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest
    container_name: language-service
    ports:
      - "5000:5000"
    environment:
      - Eula=accept
      - Billing=${LANGUAGE_ENDPOINT}
      - ApiKey=${LANGUAGE_KEY}
      - Logging:Console:LogLevel:Default=Information
    restart: unless-stopped

  vision-container:
    image: mcr.microsoft.com/azure-cognitive-services/vision/read:3.2
    container_name: vision-service
    ports:
      - "5001:5000"
    environment:
      - Eula=accept
      - Billing=${VISION_ENDPOINT}
      - ApiKey=${VISION_KEY}
    restart: unless-stopped
```

```bash
# Deploy to Azure Container Instances

az container create \
  --resource-group ai-resources-rg \
  --name language-container \
  --image mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest \
  --cpu 2 \
  --memory 4 \
  --port 5000 \
  --environment-variables \
    'Eula=accept' \
    'Billing='$LANGUAGE_ENDPOINT \
    'ApiKey='$LANGUAGE_KEY

# Deploy to Azure Kubernetes Service (AKS)
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: language-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: language-service
  template:
    metadata:
      labels:
        app: language-service
    spec:
      containers:
      - name: language
        image: mcr.microsoft.com/azure-cognitive-services/textanalytics/language:latest
        ports:
        - containerPort: 5000
        env:
        - name: Eula
          value: "accept"
        - name: Billing
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: language-endpoint
        - name: ApiKey
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: language-key
        resources:
          requests:
            memory: "4Gi"
            cpu: "2000m"
          limits:
            memory: "8Gi"
            cpu: "4000m"
---
apiVersion: v1
kind: Service
metadata:
  name: language-service
spec:
  type: LoadBalancer
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app: language-service
EOF
```

## 15. PYTHON CODE EXAMPLES & PATTERNS

### 15.1 Error Handling and Retry Logic

```python
from azure.core.exceptions import (
    HttpResponseError,
    ServiceRequestError,
    ResourceNotFoundError,
    ClientAuthenticationError
)
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import logging

logger = logging.getLogger(__name__)

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type((HttpResponseError, ServiceRequestError)),
    reraise=True
)
def call_ai_service_with_retry(client, text: str):
    """
    Call AI service with automatic retry on transient failures
    """
    try:
        result = client.analyze_sentiment(documents=[text])[0]
        return result
    except ClientAuthenticationError as e:
        logger.error(f"Authentication failed: {e}")
        raise
    except ResourceNotFoundError as e:
        logger.error(f"Resource not found: {e}")
        raise
    except HttpResponseError as e:
        if e.status_code == 429:  # Too many requests
            logger.warning("Rate limit exceeded, retrying...")
            raise
        elif e.status_code >= 500:  # Server error
            logger.warning(f"Server error {e.status_code}, retrying...")
            raise
        else:
            logger.error(f"HTTP error: {e}")
            raise

def handle_rate_limiting(client, texts: list, rate_limit_per_second: int = 10):
    """
    Handle rate limiting with throttling
    """
    import time
    
    results = []
    delay = 1.0 / rate_limit_per_second
    
    for text in texts:
        try:
            result = client.analyze_sentiment(documents=[text])[0]
            results.append(result)
            time.sleep(delay)
        except HttpResponseError as e:
            if e.status_code == 429:
                # Extract retry-after header
                retry_after = int(e.response.headers.get('Retry-After', 60))
                logger.warning(f"Rate limited. Waiting {retry_after} seconds...")
                time.sleep(retry_after)
                # Retry
                result = client.analyze_sentiment(documents=[text])[0]
                results.append(result)
    
    return results
```

### 15.2 Async/Await Patterns

```python
import asyncio
from azure.ai.textanalytics.aio import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential
import os

async def analyze_sentiment_async(texts: list):
    """
    Async sentiment analysis for better performance
    """
    endpoint = os.environ["LANGUAGE_ENDPOINT"]
    key = os.environ["LANGUAGE_KEY"]
    
    async with TextAnalyticsClient(endpoint, AzureKeyCredential(key)) as client:
        results = await client.analyze_sentiment(documents=texts)
        
        for idx, result in enumerate(results):
            if not result.is_error:
                print(f"Text {idx}: {result.sentiment} (confidence: {result.confidence_scores.positive:.2f})")
            else:
                print(f"Text {idx}: Error - {result.error.message}")
        
        return results

async def parallel_ai_operations(texts: list):
    """
    Run multiple AI operations in parallel
    """
    endpoint = os.environ["LANGUAGE_ENDPOINT"]
    key = os.environ["LANGUAGE_KEY"]
    
    async with TextAnalyticsClient(endpoint, AzureKeyCredential(key)) as client:
        # Create tasks
        sentiment_task = client.analyze_sentiment(documents=texts)
        key_phrases_task = client.extract_key_phrases(documents=texts)
        entities_task = client.recognize_entities(documents=texts)
        
        # Run in parallel
        sentiment_results, key_phrases_results, entity_results = await asyncio.gather(
            sentiment_task,
            key_phrases_task,
            entities_task
        )
        
        return {
            "sentiment": sentiment_results,
            "key_phrases": key_phrases_results,
            "entities": entity_results
        }

# Run async functions
texts = ["I love this product!", "This is terrible.", "It's okay."]
results = asyncio.run(analyze_sentiment_async(texts))
parallel_results = asyncio.run(parallel_ai_operations(texts))
```

### 15.3 Production-Ready Patterns

```python
from dataclasses import dataclass
from typing import Optional, List, Dict, Any
from datetime import datetime
import json

@dataclass
class AIServiceConfig:
    """Configuration for AI services"""
    endpoint: str
    key: str
    retry_attempts: int = 3
    timeout: int = 30
    max_batch_size: int = 10

@dataclass
class AIResult:
    """Standardized result format"""
    success: bool
    data: Optional[Any]
    error: Optional[str]
    timestamp: datetime
    duration_ms: float
    metadata: Dict[str, Any]

class AIServiceClient:
    """
    Production-ready AI service client with:
    - Configuration management
    - Error handling
    - Logging
    - Metrics
    """
    
    def __init__(self, config: AIServiceConfig):
        from azure.ai.textanalytics import TextAnalyticsClient
        from azure.core.credentials import AzureKeyCredential
        
        self.config = config
        self.client = TextAnalyticsClient(
            endpoint=config.endpoint,
            credential=AzureKeyCredential(config.key)
        )
        self._setup_logging()
    
    def _setup_logging(self):
        import logging
        self.logger = logging.getLogger(self.__class__.__name__)
        self.logger.setLevel(logging.INFO)
    
    def analyze_sentiment(self, text: str) -> AIResult:
        """
        Analyze sentiment with full error handling and metrics
        """
        import time
        start_time = time.time()
        
        try:
            result = self.client.analyze_sentiment(documents=[text])[0]
            duration = (time.time() - start_time) * 1000
            
            self.logger.info(f"Sentiment analysis completed in {duration:.2f}ms")
            
            return AIResult(
                success=True,
                data={
                    "sentiment": result.sentiment,
                    "confidence": {
                        "positive": result.confidence_scores.positive,
                        "neutral": result.confidence_scores.neutral,
                        "negative": result.confidence_scores.negative
                    }
                },
                error=None,
                timestamp=datetime.utcnow(),
                duration_ms=duration,
                metadata={"text_length": len(text)}
            )
        
        except Exception as e:
            duration = (time.time() - start_time) * 1000
            self.logger.error(f"Sentiment analysis failed: {e}")
            
            return AIResult(
                success=False,
                data=None,
                error=str(e),
                timestamp=datetime.utcnow(),
                duration_ms=duration,
                metadata={"text_length": len(text)}
            )

# Usage
config = AIServiceConfig(
    endpoint=os.environ["LANGUAGE_ENDPOINT"],
    key=os.environ["LANGUAGE_KEY"]
)

client = AIServiceClient(config)
result = client.analyze_sentiment("This is a great product!")

if result.success:
    print(f"Sentiment: {result.data['sentiment']}")
else:
    print(f"Error: {result.error}")
```

### 15.4 Caching Strategy

```python
from functools import lru_cache
import hashlib
import json
import redis
from typing import Optional

class AIResponseCache:
    """
    Cache AI service responses to reduce costs and latency
    """
    
    def __init__(self, redis_host: str = "localhost", redis_port: int = 6379):
        self.redis_client = redis.Redis(
            host=redis_host,
            port=redis_port,
            decode_responses=True
        )
        self.ttl = 3600  # 1 hour
    
    def _generate_cache_key(self, operation: str, params: dict) -> str:
        """Generate cache key from operation and parameters"""
        param_str = json.dumps(params, sort_keys=True)
        hash_str = hashlib.md5(param_str.encode()).hexdigest()
        return f"ai_cache:{operation}:{hash_str}"
    
    def get(self, operation: str, params: dict) -> Optional[dict]:
        """Get cached result"""
        key = self._generate_cache_key(operation, params)
        cached = self.redis_client.get(key)
        
        if cached:
            return json.loads(cached)
        return None
    
    def set(self, operation: str, params: dict, result: dict):
        """Cache result"""
        key = self._generate_cache_key(operation, params)
        self.redis_client.setex(
            key,
            self.ttl,
            json.dumps(result)
        )

def cached_sentiment_analysis(text: str, cache: AIResponseCache, client) -> dict:
    """
    Sentiment analysis with caching
    """
    # Check cache
    cached_result = cache.get("sentiment", {"text": text})
    if cached_result:
        print("Cache hit!")
        return cached_result
    
    # Call API
    print("Cache miss, calling API...")
    result = client.analyze_sentiment(documents=[text])[0]
    
    # Cache result
    result_dict = {
        "sentiment": result.sentiment,
        "confidence": {
            "positive": result.confidence_scores.positive,
            "neutral": result.confidence_scores.neutral,
            "negative": result.confidence_scores.negative
        }
    }
    cache.set("sentiment", {"text": text}, result_dict)
    
    return result_dict

# Simple in-memory caching with LRU
@lru_cache(maxsize=1000)
def cached_analysis_simple(text: str) -> str:
    """Simple LRU cache for sentiment analysis"""
    from azure.ai.textanalytics import TextAnalyticsClient
    from azure.core.credentials import AzureKeyCredential
    
    client = TextAnalyticsClient(
        endpoint=os.environ["LANGUAGE_ENDPOINT"],
        credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
    )
    
    result = client.analyze_sentiment(documents=[text])[0]
    return result.sentiment
```

### 15.5 Stream Processing Pattern

```python
from queue import Queue
from threading import Thread
import time

class AIStreamProcessor:
    """
    Process AI requests in a streaming fashion
    """
    
    def __init__(self, client, batch_size: int = 10, batch_timeout: float = 1.0):
        self.client = client
        self.batch_size = batch_size
        self.batch_timeout = batch_timeout
        self.queue = Queue()
        self.results = []
        self.running = False
    
    def start(self):
        """Start the processor"""
        self.running = True
        self.worker_thread = Thread(target=self._process_batch)
        self.worker_thread.start()
    
    def stop(self):
        """Stop the processor"""
        self.running = False
        self.worker_thread.join()
    
    def submit(self, text: str) -> None:
        """Submit text for processing"""
        self.queue.put(text)
    
    def _process_batch(self):
        """Process texts in batches"""
        batch = []
        last_process_time = time.time()
        
        while self.running or not self.queue.empty():
            try:
                # Try to get item with timeout
                text = self.queue.get(timeout=0.1)
                batch.append(text)
                
                # Process batch if full or timeout reached
                if len(batch) >= self.batch_size or \
                   (time.time() - last_process_time) >= self.batch_timeout:
                    self._process_and_store(batch)
                    batch = []
                    last_process_time = time.time()
            
            except:
                # Queue timeout, check if we should process accumulated batch
                if batch and (time.time() - last_process_time) >= self.batch_timeout:
                    self._process_and_store(batch)
                    batch = []
                    last_process_time = time.time()
        
        # Process remaining items
        if batch:
            self._process_and_store(batch)
    
    def _process_and_store(self, batch: list):
        """Process a batch and store results"""
        try:
            results = self.client.analyze_sentiment(documents=batch)
            self.results.extend(results)
            print(f"Processed batch of {len(batch)} items")
        except Exception as e:
            print(f"Error processing batch: {e}")

# Usage
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

client = TextAnalyticsClient(
    endpoint=os.environ["LANGUAGE_ENDPOINT"],
    credential=AzureKeyCredential(os.environ["LANGUAGE_KEY"])
)

processor = AIStreamProcessor(client, batch_size=5)
processor.start()

# Submit items
for i in range(15):
    processor.submit(f"This is test message {i}")

time.sleep(3)  # Wait for processing
processor.stop()

print(f"Processed {len(processor.results)} items")
```

---

## 16. REST API REFERENCE

### 16.1 Text Analytics REST API

```bash
# Sentiment Analysis
curl -X POST "https://<endpoint>/language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "SentimentAnalysis",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "This is a great product!"
        }
      ]
    }
  }'

# Named Entity Recognition
curl -X POST "https://<endpoint>/language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "EntityRecognition",
    "parameters": {
      "modelVersion": "latest"
    },
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Microsoft was founded by Bill Gates in Seattle."
        }
      ]
    }
  }'

# Key Phrase Extraction
curl -X POST "https://<endpoint>/language/:analyze-text?api-version=2023-04-01" \
  -H "Ocp-Apim-Subscription-Key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "KeyPhraseExtraction",
    "parameters": {},
    "analysisInput": {
      "documents": [
        {
          "id": "1",
          "language": "en",
          "text": "Azure AI services provide powerful AI capabilities."
        }
      ]
    }
  }'
```

### 16.2 Computer Vision REST API

```bash
# Analyze Image
curl -X POST "https://<endpoint>/vision/v4.0/analyze?features=caption,read,tags,objects&language=en&gender-neutral-caption=true" \
  -H "Ocp-Apim-Subscription-Key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/image.jpg"
  }'

# OCR (Read)
curl -X POST "https://<endpoint>/vision/v4.0/read/analyze" \
  -H "Ocp-Apim-Subscription-Key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/document.jpg"
  }'

# Get Read Results
curl -X GET "https://<endpoint>/vision/v4.0/read/analyzeResults/<operation-id>" \
  -H "Ocp-Apim-Subscription-Key: <key>"
```

### 16.3 Azure OpenAI REST API

```bash
# Chat Completions
curl -X POST "https://<resource>.openai.azure.com/openai/deployments/<deployment-name>/chat/completions?api-version=2024-02-01" \
  -H "api-key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is Azure AI?"}
    ],
    "max_tokens": 800,
    "temperature": 0.7
  }'

# Embeddings
curl -X POST "https://<resource>.openai.azure.com/openai/deployments/<embedding-deployment>/embeddings?api-version=2024-02-01" \
  -H "api-key: <key>" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "This is sample text for embedding."
  }'

# List Deployments
curl -X GET "https://<resource>.openai.azure.com/openai/deployments?api-version=2024-02-01" \
  -H "api-key: <key>"
```

---

## 17. LIMITS, QUOTAS & PRICING

### 17.1 Service Limits (as of December 2025)

**Language Service:**
- Free Tier (F0): 5,000 text records per month
- Standard Tier (S): Unlimited (pay per use)
- Max document size: 5,120 characters
- Max batch size: 10 documents per request
- Rate limits: S tier - 1,000 requests per minute

**Computer Vision:**
- Free Tier (F0): 5,000 transactions per month, 20 per minute
- Standard (S1): 10 per second
- Max image size: 50 MB
- Image dimensions: 50 x 50 to 16,000 x 16,000 pixels

**Azure OpenAI:**
- Token limits vary by model:
  - GPT-4: 8K, 32K, 128K context windows
  - GPT-3.5-turbo: 4K, 16K context windows
- Rate limits (TPM - Tokens Per Minute): Configured per deployment
- Default: 10K-240K TPM depending on model and region

**Speech Service:**
- Free Tier (F0):
  - Speech-to-text: 5 hours per month
  - Text-to-speech: 0.5 million characters per month
- Standard (S0): Pay per use
- Concurrent request limits: 20-100 depending on tier

### 17.2 Pricing Examples (USD)

**Text Analytics:**
- 0-500K text records: $2 per 1,000 records
- 0.5M-2.5M: $1 per 1,000
- 2.5M-10M: $0.50 per 1,000

**Computer Vision:**
- Image Analysis 4.0: $1.50 per 1,000 transactions
- OCR Read: $1.50 per 1,000 transactions
- Face API: $1.00 per 1,000 transactions

**Azure OpenAI (approximate):**
- GPT-4: $0.03/1K input tokens, $0.06/1K output tokens
- GPT-4-turbo: $0.01/1K input, $0.03/1K output
- GPT-3.5-turbo: $0.0015/1K input, $0.002/1K output
- Embeddings (text-embedding-3-small): $0.00002/1K tokens

**Document Intelligence:**
- Prebuilt models: $0.01 per page
- Custom models: $0.05 per page (training), $0.01 per page (analysis)

### 17.3 Quota Management

```bash
# View current usage
az cognitiveservices account list-usage \
  --name "my-language-resource" \
  --resource-group "ai-resources-rg"

# View quotas
az cognitiveservices account show \
  --name "my-openai-resource" \
  --resource-group "ai-resources-rg" \
  --query "properties.quotaLimit"
```

---

## 18. TROUBLESHOOTING GUIDE

### 18.1 Common Error Codes

**401 Unauthorized:**
- Cause: Invalid API key or expired token
- Solution: Verify key, check Azure AD token expiration
```python
# Debug authentication
try:
    result = client.analyze_sentiment(documents=["test"])
except ClientAuthenticationError as e:
    print(f"Auth error: {e}")
    print("Check: API key, endpoint, Azure AD credentials")
```

**429 Too Many Requests:**
- Cause: Rate limit exceeded
- Solution: Implement exponential backoff, reduce request rate
```python
import time

def handle_rate_limit(client, text):
    max_retries = 3
    for attempt in range(max_retries):
        try:
            return client.analyze_sentiment(documents=[text])
        except HttpResponseError as e:
            if e.status_code == 429:
                retry_after = int(e.response.headers.get('Retry-After', 60))
                print(f"Rate limited. Waiting {retry_after}s...")
                time.sleep(retry_after)
            else:
                raise
    raise Exception("Max retries exceeded")
```

**403 Forbidden:**
- Cause: Network restrictions, missing RBAC permissions
- Solution: Check virtual network rules, RBAC assignments

**413 Request Entity Too Large:**
- Cause: Document exceeds size limits
- Solution: Split document, reduce batch size

**500 Internal Server Error:**
- Cause: Service-side issue
- Solution: Retry with exponential backoff

### 18.2 Debugging Tips

```python
import logging

# Enable SDK logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('azure')
logger.setLevel(logging.DEBUG)

# Log HTTP requests/responses
from azure.core.pipeline.policies import HttpLoggingPolicy

client = TextAnalyticsClient(
    endpoint=endpoint,
    credential=credential,
    logging_enable=True,
    logging_policy=HttpLoggingPolicy()
)

# Check service health
def check_service_health(endpoint: str, key: str) -> bool:
    """Test if service is accessible"""
    try:
        client = TextAnalyticsClient(
            endpoint=endpoint,
            credential=AzureKeyCredential(key)
        )
        # Simple test
        client.analyze_sentiment(documents=["test"])
        print("✅ Service is healthy")
        return True
    except Exception as e:
        print(f"❌ Service health check failed: {e}")
        return False
```

### 18.3 Performance Optimization

**Issue: High Latency**
- Use batch processing
- Enable async/await
- Implement caching
- Use regional endpoints (closer to your users)
- Consider container deployment for on-premises scenarios

**Issue: High Costs**
- Implement caching
- Batch similar requests
- Use free tier for development/testing
- Monitor usage with Azure Monitor
- Set up cost alerts

---

## 19. MIGRATION GUIDES

### 19.1 Migrating from v3.0 to v3.1 (Language)

```python
# OLD (v3.0)
from azure.ai.textanalytics import TextAnalyticsClient

result = client.recognize_entities(documents=["text"])[0]
for entity in result.entities:
    print(f"{entity.text}: {entity.category}")

# NEW (v3.1)
# Same API, but with additional features
result = client.recognize_entities(documents=["text"])[0]
for entity in result.entities:
    print(f"{entity.text}: {entity.category}")
    if entity.subcategory:  # New in v3.1
        print(f"  Subcategory: {entity.subcategory}")
```

### 19.2 Migrating QnA Maker to Question Answering

```python
# OLD: QnA Maker (deprecated)
from azure.cognitiveservices.knowledge.qnamaker import QnAMakerClient

qna_client = QnAMakerClient(endpoint, credential)
result = qna_client.runtime.generate_answer(
    kb_id=kb_id,
    question="What is Azure?"
)

# NEW: Question Answering (Language Service)
from azure.ai.language.questionanswering import QuestionAnsweringClient

qna_client = QuestionAnsweringClient(endpoint, credential)
output = qna_client.get_answers(
    question="What is Azure?",
    project_name=project_name,
    deployment_name="production"
)
```

### 19.3 Migrating from LUIS to CLU

```python
# OLD: LUIS
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient

luis_client = LUISRuntimeClient(endpoint, credentials)
prediction = luis_client.prediction.get_slot_prediction(
    app_id=app_id,
    slot_name="production",
    prediction_request={"query": "Book a flight to Seattle"}
)

# NEW: Conversational Language Understanding (CLU)
from azure.ai.language.conversations import ConversationAnalysisClient

clu_client = ConversationAnalysisClient(endpoint, credential)
result = clu_client.analyze_conversation(
    task={
        "kind": "Conversation",
        "analysisInput": {
            "conversationItem": {
                "participantId": "1",
                "id": "1",
                "text": "Book a flight to Seattle"
            }
        },
        "parameters": {
            "projectName": project_name,
            "deploymentName": "production",
            "verbose": True
        }
    }
)
```

---

## 20. EXAM PREPARATION ADDENDUM

### 20.1 Key Concepts Summary

**Authentication:**
- API Keys (least secure, easiest)
- Azure AD tokens (recommended)
- Managed Identity (best for Azure resources)

**Networking:**
- Virtual networks for security
- Private endpoints for isolation
- Network security groups for access control

**Monitoring:**
- Azure Monitor for metrics
- Application Insights for telemetry
- Diagnostic logs for troubleshooting

**Best Practices:**
- Always use HTTPS
- Implement retry logic with exponential backoff
- Batch requests when possible
- Cache responses to reduce costs
- Handle PII appropriately
- Monitor and set up alerts

### 20.2 Quick CLI Reference

```bash
# Create multi-service resource
az cognitiveservices account create \
  --name "my-ai-services" \
  --resource-group "rg" \
  --kind "CognitiveServices" \
  --sku "S0" \
  --location "eastus"

# Get keys
az cognitiveservices account keys list \
  --name "my-ai-services" \
  --resource-group "rg"

# Get endpoint
az cognitiveservices account show \
  --name "my-ai-services" \
  --resource-group "rg" \
  --query "properties.endpoint" \
  --output tsv

# Enable diagnostic logging
az monitor diagnostic-settings create \
  --name "diag-settings" \
  --resource "<resource-id>" \
  --workspace "<workspace-id>" \
  --logs '[{"category":"Audit","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

### 20.3 Exam Tips

1. **Understand the difference between services:**
   - Language vs. Translator
   - Face API vs. Custom Vision
   - Azure OpenAI vs. Azure AI Services

2. **Know pricing tiers:**
   - F0 (Free) - limited transactions
   - S0 (Standard) - pay per use
   - Features available per tier

3. **Security knowledge:**
   - When to use Azure AD vs. API keys
   - RBAC roles: User, Contributor, Owner
   - VNet integration and private endpoints

4. **SDK patterns:**
   - Batch processing (max 10 documents for most services)
   - Async operations for long-running tasks
   - Error handling (401, 403, 429, 500)

5. **Container scenarios:**
   - When containers are appropriate
   - Disconnected/air-gapped environments
   - On-premises compliance requirements

6. **Responsible AI:**
   - Six principles (Fairness, Reliability, Privacy, Inclusiveness, Transparency, Accountability)
   - PII detection and redaction
   - Content filtering in Azure OpenAI

### 20.4 Practice Scenarios

**Scenario 1: Build a document processing pipeline**
1. Use Document Intelligence to extract text
2. Use Language service to analyze sentiment
3. Store results in Cosmos DB
4. Monitor with Application Insights

**Scenario 2: Create a customer service bot**
1. Bot Framework for conversation flow
2. CLU for intent recognition
3. Question Answering for FAQ
4. Language service for sentiment analysis
5. Deploy to Azure Bot Service

**Scenario 3: Image analysis application**
1. Computer Vision for image analysis
2. Custom Vision for specific object detection
3. Azure Search for indexing
4. Store images in Blob Storage
5. Secure with managed identity

**Scenario 4: Multi-language content moderation**
1. Translator for language detection
2. Content Safety for moderation
3. Language service for additional analysis
4. Log decisions for accountability

---

## APPENDIX: SDK Version Reference

**Latest SDK Versions (December 2025):**

```bash
# Python packages
pip install azure-ai-textanalytics==5.3.0
pip install azure-ai-vision==0.15.0
pip install azure-ai-formrecognizer==3.3.0
pip install azure-cognitiveservices-speech==1.34.0
pip install azure-ai-contentsafety==1.0.0
pip install azure-search-documents==11.4.0
pip install openai==1.5.0
pip install botbuilder-core==4.15.0

# Azure CLI
az --version  # Should be >= 2.55.0
az extension add --name cognitiveservices
```

**API Versions:**
- Language: 2023-04-01 (stable)
- Vision: v4.0
- Document Intelligence: 2023-07-31
- Azure OpenAI: 2024-02-01
- Speech: v3.1

---

**END OF COMPREHENSIVE AI-102 EXAM CHEATSHEET**

*Total Sections: 20*
*Approximate Line Count: 10,000+*
*Last Updated: December 2025*
*Coverage: Complete AI-102 Exam Syllabus*

---
