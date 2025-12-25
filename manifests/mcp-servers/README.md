# RemoteMCPServer Definitions

This directory contains RemoteMCPServer resource definitions that provide tools to agents.

## Creating a New MCP Server

Use the template in `templates/mcp-server-template.yaml` as a starting point.

## MCP Server Protocols

- **STREAMABLE_HTTP**: HTTP-based protocol with streaming support
- **SSE**: Server-Sent Events protocol

## Registering MCP Servers

1. Deploy your MCP server implementation to the cluster
2. Create a Service to expose the MCP endpoint
3. Create a RemoteMCPServer resource pointing to the service
4. Reference the MCP server in agent tool configurations

## Example Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-mcp-server
  namespace: kagent
spec:
  selector:
    app: my-mcp-server
  ports:
  - port: 8080
    targetPort: 8080
    name: mcp
```
