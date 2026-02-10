# API (OpenAPI)

Status: active | Audience: developer | Last reviewed: 2026-02-09

## Goal

Provide a single, canonical entrypoint for the MCP Gateway Management API contract.

## Source of truth

- OpenAPI: [`openapi/mcp-gateway.openapi.json`](../../openapi/mcp-gateway.openapi.json)

## How to view

- Open the OpenAPI file in an OpenAPI viewer (Swagger UI / Redoc) or your IDE plugin.

## Auth / Permission model (note)

The OpenAPI spec currently may not fully describe security schemes and role/permission checks.

References:
- Entra app roles guidance: [`docs/entra-app-roles.md`](../entra-app-roles.md)
- Source code: `dotnet/Microsoft.McpGateway.*` (auth and permission checks)
