# MCP Gateway

Этот репозиторий содержит решение MCP Gateway (Microsoft MCP Gateway) и связанные артефакты деплоя/интеграции.

## Структура

- `docs/` — документация и runbooks.
  - Entry point: [docs/README.md](docs/README.md)
  - Capabilities: [docs/reference/capabilities.md](docs/reference/capabilities.md)
- `openapi/` — OpenAPI спецификация Management API (`openapi/mcp-gateway.openapi.json`).
- `deployment/` — инфраструктурные материалы и деплой (Azure/Kubernetes).
- `dotnet/` — исходники компонентов gateway/tools/management.
- `sample-servers/` — примеры адаптеров/прокси (source-of-truth для примеров).

## Документация

- Start here: [docs/README.md](docs/README.md)
- Runbooks: [docs/runbooks/README.md](docs/runbooks/README.md)
- API reference: [docs/reference/api.md](docs/reference/api.md)
