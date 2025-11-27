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

**Note for Ollama Users**: Use `backend: openai` with a custom `baseUrl` pointing to your Ollama server's OpenAI-compatible endpoint (e.g., `http://your-ollama-server:11434/v1`). The k8sgpt CRD does not have a dedicated Ollama backend, but Ollama's OpenAI-compatible API works seamlessly with the `openai` backend setting.

### Models

Common model options for OpenAI:
- `gpt-4o-mini` (recommended for cost-effective usage)
- `gpt-3.5-turbo`
- `gpt-4`
- `gpt-4-turbo`

For Ollama, use your installed model names (e.g., `llama2`, `mistral`, `codellama`, or custom models like `gpt-oss:120b`)

### Secret Management

**For Production**: Always use external secret management (e.g., Sealed Secrets, External Secrets Operator, Vault) rather than storing API keys in values.yaml.

**For Local Testing**: You can set `k8sgpt.ai.secret.create: true` and provide the API key in values.yaml, but this is not recommended for production.

### Ollama Configuration

To use Ollama as your AI backend:

1. Set the backend to `openai` (Ollama implements the OpenAI-compatible API)
2. Configure the `baseUrl` to point to your Ollama server (without trailing slash)
3. Specify your Ollama model name

Example configuration in `values.yaml`:
```yaml
k8sgpt:
  ai:
    backend: openai
    model: llama2  # or mistral, codellama, etc.
    baseUrl: "http://192.168.1.100:11434/v1"
```

**Important**: Remove the trailing slash from `baseUrl` to avoid API path issues.

### Auto-Remediation

K8sGPT can automatically attempt to fix detected issues. This feature is **disabled by default** for safety.

**Warning**: Auto-remediation will make changes to your cluster. Always test in a non-production environment first.

#### Configuration

Edit `values.yaml` to enable/disable auto-remediation:

```yaml
k8sgpt:
  ai:
    autoRemediation:
      enabled: true  # Set to false to disable (default: false)
      resources:
        - Pod
        - Deployment
        - Service
        - Ingress
```

After changing the configuration, apply it with:
```bash
helm upgrade k8sgpt . --namespace k8sgpt-operator-system
```

#### Auto-Remediation Resource Types

Control which resource types are eligible for auto-remediation:
- `Pod` - Automatically restart or fix pod issues
- `Deployment` - Fix deployment configuration issues
- `Service` - Correct service misconfigurations
- `Ingress` - Fix ingress routing problems

#### Monitoring Auto-Remediation

Check current auto-remediation status:
```bash
kubectl get k8sgpt -n k8sgpt-operator-system k8sgpt -o jsonpath='{.spec.ai.autoRemediation}'
```

View remediation attempts in Result resources:
```bash
kubectl get results -n k8sgpt-operator-system -o yaml | grep -A 10 autoRemediationStatus
```

### Cleanup Configuration

This chart includes automated cleanup hooks that run when you uninstall the release. Configure cleanup behavior in `values.yaml`:

```yaml
cleanup:
  # Enable/disable automatic cleanup on helm uninstall
  enabled: true

  # Delete all K8sGPT custom resources before uninstalling
  deleteCRs: true

  # Delete CRDs (WARNING: This removes CRDs from the entire cluster)
  # Set to false in production environments with shared CRDs
  deleteCRDs: true
```

**Cleanup Behavior**:
- When `enabled: true`, a pre-delete hook Job automatically cleans up resources before uninstall
- The Job deletes K8sGPT custom resources (K8sGPT instances, Results, Mutations)
- If `deleteCRDs: true`, it also removes all three K8sGPT CRDs from the cluster
- All cleanup resources are automatically removed after successful execution

**Production Recommendation**: Set `cleanup.deleteCRDs: false` in production to preserve CRDs that may be shared across multiple deployments.

**Development Workflow**: The default settings (`cleanup.enabled: true` and `cleanup.deleteCRDs: true`) ensure clean iterations during development without manual cleanup.

## Uninstallation

The chart includes automated cleanup hooks that remove CRDs and custom resources during uninstallation:

```bash
# Standard uninstall (with automatic cleanup)
helm uninstall k8sgpt -n k8sgpt-operator-system
```

The pre-delete hook will automatically:
1. Delete all K8sGPT custom resources (instances, results, mutations)
2. Delete the CRDs (if `cleanup.deleteCRDs: true`)
3. Clean up all associated resources

**To disable automatic cleanup**, set `cleanup.enabled: false` in `values.yaml` before uninstalling, or manually delete the cleanup Job:

```bash
# Disable cleanup before uninstall
kubectl delete job -n k8sgpt-operator-system -l helm.sh/hook=pre-delete
helm uninstall k8sgpt -n k8sgpt-operator-system
```

**Manual CRD cleanup** (if needed):

```bash
kubectl delete crd k8sgpts.core.k8sgpt.ai
kubectl delete crd results.core.k8sgpt.ai
kubectl delete crd mutations.core.k8sgpt.ai
```

## Additional Resources

- [K8sGPT Documentation](https://docs.k8sgpt.ai/)
- [K8sGPT Operator GitHub](https://github.com/k8sgpt-ai/k8sgpt-operator)
- [K8sGPT Helm Charts](https://charts.k8sgpt.ai/)

## Troubleshooting

### Installation Issues

1. **"metadata.managedFields must be nil" error**:
   - This occurs when orphaned CRDs exist from a previous installation
   - **Solution**: Delete the CRDs manually, then reinstall:
     ```bash
     kubectl delete crd k8sgpts.core.k8sgpt.ai mutations.core.k8sgpt.ai results.core.k8sgpt.ai
     helm install k8sgpt . -f values.yaml
     ```

2. **"cannot reuse a name that is still in use" error**:
   - A failed Helm release still exists in the cluster
   - **Solution**: Check releases in all namespaces and uninstall properly:
     ```bash
     helm list --all-namespaces
     helm uninstall k8sgpt -n k8sgpt-operator-system
     ```

### Runtime Issues

3. **Operator not starting**: Check if the namespace exists and has proper RBAC permissions
4. **K8sGPT pod not running**: Verify the secret exists and contains the correct API key
5. **API errors**: Ensure your API key is valid and has sufficient credits/quota

### Cleanup Issues

6. **CRDs not being deleted after uninstall**:
   - Verify cleanup is enabled: `cleanup.enabled: true` in values.yaml
   - Check cleanup job logs:
     ```bash
     kubectl get jobs -n k8sgpt-operator-system
     kubectl logs -n k8sgpt-operator-system job/k8sgpt-cleanup
     ```

7. **Cleanup job fails with permission errors**:
   - The cleanup ServiceAccount may not have proper RBAC permissions
   - Check ClusterRole binding:
     ```bash
     kubectl get clusterrolebinding k8sgpt-cleanup
     ```


