# kagent-bhg - KAgent Configuration Repository

This repository contains KAgent agent and MCP server definitions with GitOps deployment automation.

## Overview

This repository uses GitOps principles to manage KAgent agents and MCP servers. Changes pushed to the `main` branch are automatically deployed to the cluster via ArgoCD.

## Repository Structure

```
.
├── README.md
├── manifests/
│   ├── agents/              # Agent definitions
│   │   ├── genetics-game-designer.yaml
│   │   ├── genetics-sim-engineer.yaml
│   │   ├── genetics-devops.yaml
│   │   └── ...
│   ├── mcp-servers/         # RemoteMCPServer definitions
│   │   ├── kagent-tool-server.yaml
│   │   ├── github-mcp.yaml
│   │   └── ...
│   └── model-configs/       # ModelConfig definitions
│       ├── openai-gpt-4.yaml
│       └── ...
├── templates/              # Templates for creating new resources
│   ├── agent-template.yaml
│   ├── mcp-server-template.yaml
│   └── model-config-template.yaml
└── .github/
    └── workflows/
        └── deploy.yaml     # CI/CD pipeline
```

## Quick Start

### 1. Connect to ArgoCD

The kind-bhg repository includes an ArgoCD Application that watches this repository:

```bash
cd ../kind-bhg
kubectl apply -f manifests/argocd/kagent-bhg-app.yaml
```

### 2. Make Changes

Edit agent or MCP server definitions in the `manifests/` directory:

```bash
# Edit an agent
vim manifests/agents/my-agent.yaml

# Commit and push
git add manifests/agents/my-agent.yaml
git commit -m "feat: update my-agent configuration"
git push origin main
```

### 3. Automatic Deployment

ArgoCD will automatically:
1. Detect the changes
2. Sync the manifests to the cluster
3. Deploy updated agents/MCP servers

## Creating New Resources

### Creating a New Agent

1. Copy the template:
```bash
cp templates/agent-template.yaml manifests/agents/my-new-agent.yaml
```

2. Edit the agent definition:
```yaml
apiVersion: kagent.dev/v1alpha2
kind: Agent
metadata:
  name: my-new-agent
  namespace: kagent
spec:
  type: Declarative
  description: Description of what this agent does
  declarative:
    modelConfig: openai-gpt-4
    stream: true
    tools:
      - type: McpServer
        mcpServer:
          apiGroup: kagent.dev
          kind: RemoteMCPServer
          name: kagent-tool-server
          toolNames:
            - k8s_apply_manifest
            - helm_upgrade
    systemMessage: |-
      You are an AI agent that...
      
      Your responsibilities include:
      - Task 1
      - Task 2
```

3. Commit and push:
```bash
git add manifests/agents/my-new-agent.yaml
git commit -m "feat: add my-new-agent"
git push origin main
```

### Creating a New MCP Server

1. Copy the template:
```bash
cp templates/mcp-server-template.yaml manifests/mcp-servers/my-mcp-server.yaml
```

2. Edit the MCP server definition:
```yaml
apiVersion: kagent.dev/v1alpha2
kind: RemoteMCPServer
metadata:
  name: my-mcp-server
  namespace: kagent
spec:
  description: Description of tools provided
  protocol: STREAMABLE_HTTP
  url: http://my-mcp-server.kagent:8080/mcp
```

3. Commit and push (same as above)

## Agent Guidelines

### Best Practices

1. **Clear Descriptions**: Make agent descriptions specific and actionable
2. **Focused Responsibilities**: Each agent should have a clear, focused purpose
3. **Appropriate Tools**: Only include tools the agent actually needs
4. **System Messages**: Provide clear instructions and constraints
5. **Naming Convention**: Use lowercase with hyphens (e.g., `my-agent-name`)

### System Message Structure

Good system messages include:
- Clear role definition
- Specific responsibilities
- Explicit constraints
- Output format requirements
- Examples (when helpful)

Example:
```yaml
systemMessage: |-
  You are a Kubernetes deployment specialist.
  
  Responsibilities:
  - Review deployment manifests for best practices
  - Suggest security improvements
  - Validate resource requirements
  
  Constraints:
  - Only suggest changes, don't automatically apply them
  - Always explain your reasoning
  - Follow Kubernetes security best practices
  
  Output format:
  - Start with a summary
  - List specific issues found
  - Provide recommended changes with YAML snippets
```

## MCP Server Guidelines

### Available MCP Servers

Current MCP servers in the cluster:

1. **kagent-tool-server** - Kubernetes and Helm operations
   - Tools: k8s_apply_manifest, k8s_create_resource, k8s_delete_resource, helm_upgrade, helm_list_releases

2. **kagent-grafana-mcp** - Grafana/monitoring queries
   - Tools: Grafana-related operations

3. **kagent-querydoc** - Documentation queries
   - Tools: Documentation search and retrieval

### Adding External MCP Servers

To add a new MCP server:

1. Deploy the MCP server to the cluster
2. Create a Service exposing the MCP endpoint
3. Create a RemoteMCPServer resource in this repository
4. Reference it in agent definitions

## CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/deploy.yaml`) performs:

1. **Validation**: Validates YAML syntax and Kubernetes resource definitions
2. **Testing**: Runs basic tests on agent configurations
3. **Deployment**: Syncs with ArgoCD (automatic via ArgoCD's own sync)

### Manual Sync

To manually trigger a sync:

```bash
# Using ArgoCD CLI
argocd app sync kagent-bhg

# Using kubectl
kubectl patch app kagent-bhg -n argocd --type merge -p '{"operation":{"initiatedBy":{"username":"manual"},"sync":{}}}'
```

## Monitoring

### Check Deployment Status

```bash
# Via ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Visit https://localhost:8080

# Via ArgoCD CLI
argocd app get kagent-bhg

# Check agent status
kubectl get agents -n kagent
```

### View Agent Logs

```bash
# List agents
kubectl get agents -n kagent

# Get agent pod
kubectl get pods -n kagent -l kagent.dev/agent=my-agent-name

# View logs
kubectl logs -n kagent -l kagent.dev/agent=my-agent-name
```

## Troubleshooting

### Agent Not Deploying

```bash
# Check ArgoCD application status
argocd app get kagent-bhg

# Check for sync errors
kubectl describe app kagent-bhg -n argocd

# Check agent resource
kubectl describe agent my-agent-name -n kagent
```

### Agent Not Working

```bash
# Check agent pod
kubectl get pods -n kagent -l kagent.dev/agent=my-agent-name

# Check logs
kubectl logs -n kagent -l kagent.dev/agent=my-agent-name

# Check model config
kubectl get modelconfig -n kagent

# Verify API keys
kubectl get secrets -n kagent | grep api-key
```

### MCP Server Not Available

```bash
# Check MCP server resource
kubectl get remotemcpserver -n kagent

# Describe the MCP server
kubectl describe remotemcpserver my-mcp-server -n kagent

# Check if service exists
kubectl get svc -n kagent | grep mcp

# Test connectivity
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- curl http://my-mcp-server.kagent:8080/health
```

## Security

### Secret Management

- **Never commit secrets to this repository**
- Use Kubernetes Secrets for sensitive data
- Reference secrets in agent configurations
- Consider using Sealed Secrets or External Secrets Operator

### API Keys

API keys should be stored as Kubernetes Secrets:

```bash
# Create OpenAI API key secret
kubectl create secret generic openai-api-key -n kagent \
  --from-literal=api-key="your-api-key"

# Reference in ModelConfig
apiVersion: kagent.dev/v1alpha2
kind: ModelConfig
metadata:
  name: openai-gpt-4
  namespace: kagent
spec:
  provider: openai
  model: gpt-4-turbo
  apiKeySecret:
    name: openai-api-key
    key: api-key
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT
