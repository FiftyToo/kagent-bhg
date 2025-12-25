# ModelConfig Definitions

This directory contains ModelConfig resources that define LLM configurations for agents.

## Creating a New ModelConfig

Use the template in `templates/model-config-template.yaml` as a starting point.

## Supported Providers

- **openai**: OpenAI models (GPT-4, GPT-3.5, etc.)
- **anthropic**: Anthropic models (Claude)
- **azure**: Azure OpenAI Service
- **google**: Google AI models (Gemini)

## API Key Management

API keys should be stored as Kubernetes Secrets:

```bash
kubectl create secret generic openai-api-key -n kagent \
  --from-literal=api-key="sk-..."
```

Then reference in the ModelConfig:

```yaml
spec:
  apiKeySecret:
    name: openai-api-key
    key: api-key
```
