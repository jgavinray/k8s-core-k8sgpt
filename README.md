# K8sGPT Deployment Helm Chart

This Helm chart deploys K8sGPT in your local Kubernetes cluster using the official [k8sgpt-operator](https://github.com/k8sgpt-ai/k8sgpt-operator) as a dependency.

## Prerequisites

- Kubernetes cluster (local or remote)
- Helm 3.x installed
- kubectl configured to access your cluster

## Quick Start

### 1. Add the K8sGPT Helm Repository

The chart will automatically use the official K8sGPT Helm repository, but you may want to add it manually for reference:

```bash
helm repo add k8sgpt https://charts.k8sgpt.ai/
helm repo update
```

### 2. Install Dependencies

```bash
helm dependency update
```

### 3. Configure Your Deployment

Edit `values.yaml` to configure:
- AI backend (OpenAI, Azure OpenAI, LocalAI, etc.)
- Model selection
- API key configuration
- Namespace settings

### 4. Deploy K8sGPT

**Option A: Using the chart with API key in values.yaml (for testing only)**

```bash
# Edit values.yaml and set:
# k8sgpt.ai.secret.create: true
# k8sgpt.ai.secret.apiKey: "your-api-key-here"

helm install k8sgpt . --namespace k8sgpt-operator-system --create-namespace
```

**Option B: Using external secret (recommended for production)**

```bash
# First, create the secret manually
kubectl create secret generic k8sgpt-secret \
  --from-literal=openai-api-key=YOUR_OPENAI_API_KEY \
  -n k8sgpt-operator-system

# Then deploy (with secret.create: false in values.yaml)
helm install k8sgpt . --namespace k8sgpt-operator-system --create-namespace
```

### 5. Verify Installation

```bash
# Check operator pods
kubectl get pods -n k8sgpt-operator-system

# Check K8sGPT custom resource
kubectl get k8sgpt -n k8sgpt-operator-system

# View K8sGPT logs
kubectl logs -n k8sgpt-operator-system -l app=k8sgpt
```

## Configuration

### AI Backends

K8sGPT supports multiple AI backends. Update `values.yaml` to use different providers:

- **OpenAI**: `backend: openai`
- **Azure OpenAI**: `backend: azureopenai`
- **LocalAI**: `backend: localai`
- **Amazon Bedrock**: `backend: amazonbedrock`
- **Google**: `backend: google`
- **Cohere**: `backend: cohere`
- **Ollama**: `backend: ollama`

### Models

Common model options:
- `gpt-4o-mini` (recommended for cost-effective usage)
- `gpt-3.5-turbo`
- `gpt-4`
- `gpt-4-turbo`

### Secret Management

**For Production**: Always use external secret management (e.g., Sealed Secrets, External Secrets Operator, Vault) rather than storing API keys in values.yaml.

**For Local Testing**: You can set `k8sgpt.ai.secret.create: true` and provide the API key in values.yaml, but this is not recommended for production.

## Uninstallation

```bash
helm uninstall k8sgpt --namespace k8sgpt-operator-system
```

## Additional Resources

- [K8sGPT Documentation](https://docs.k8sgpt.ai/)
- [K8sGPT Operator GitHub](https://github.com/k8sgpt-ai/k8sgpt-operator)
- [K8sGPT Helm Charts](https://charts.k8sgpt.ai/)

## Troubleshooting

1. **Operator not starting**: Check if the namespace exists and has proper RBAC permissions
2. **K8sGPT pod not running**: Verify the secret exists and contains the correct API key
3. **API errors**: Ensure your API key is valid and has sufficient credits/quota


