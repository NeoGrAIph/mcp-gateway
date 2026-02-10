# Capabilities

| ID | Capability | Description | Verification |
|---|---|---|---|
| CAP-management-api-openapi | Management API contract | Management API is described by the OpenAPI spec and can be used to manage adapters/tools. | Inspect `openapi/mcp-gateway.openapi.json` and validate it renders in an OpenAPI viewer. |
| CAP-mcp-proxy-endpoint | MCP proxy endpoint | Gateway exposes MCP proxying endpoint (`POST /mcp` and adapter MCP routes). | Deploy locally and verify `POST /mcp` is routed to toolgateway (see runbooks). |
