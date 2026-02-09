# Проксирование локальных и удаленных серверов MCP

## Что нового

### Проксирование локального сервера Stdio MCP
- Раскрутите локальные серверы MCP за шлюзом, указав команду (`npx`, `uvx` и т. д.).
- Предоставляйте им доступ удаленно через HTTP/Streamable, чтобы облачные агенты могли подключиться.
— Поддержка удостоверений рабочей нагрузки для безопасной аутентификации с помощью ресурсов Azure.

Благодаря этому вы можете превратить **локальные серверы MCP** в **доступные из облака сервисы**, которые напрямую подключаются к вашим рабочим процессам ИИ.

### Проксирование удаленного HTTP MCP-сервера
- Пересылать запросы от шлюза на существующий сервер MCP через потоковый HTTP.


## Инструкции

### Подготовка

- Убедитесь, что развертывание облака выполнено.
- Создайте образ прокси-сервера MCP в ACR.
  ```sh
  az acr build -r "mgreg$resourceLabel" -f sample-servers/mcp-proxy/Dockerfile sample-servers/mcp-proxy -t "mgreg$resourceLabel.azurecr.io/mcp-proxy:1.0.0"
  ```

- Настройка разрешений для субъекта идентификации рабочей нагрузки (при настройке локального сервера mcp).
`mg-identity-<identifier>-workload`.
Это удостоверение создается путем развертывания. Сервер MCP будет использовать идентификатор рабочей нагрузки для доступа к восходящим ресурсам.

### Проксирование локальных серверов
Для запуска локального сервера MCP в stdio и проксирования трафика через шлюз к нему.
Задайте команду запуска сервера и аргументы в переменных среды:
- `MCP_COMMAND`
- `MCP_ARGS`

Установите для `useWorkloadIdentity` значение true, если серверу необходимо использовать удостоверение рабочей нагрузки.

> **Примечание.** При использовании локального сервера с мостовым соединением некоторые системные пакеты могут отсутствовать по умолчанию. Чтобы решить эту проблему, вы можете установить необходимые пакеты в собственный файл Dockerfile и создать собственный образ `mcp-proxy`.

### Проксирование удаленных серверов
Для проксирования другого внутреннего сервера mcp, размещенного в потоковом режиме HTTP. Установите целевую конечную точку в переменной среды
- `MCP_PROXY_URL`


## Примеры

Пример полезных данных для отправки на `mcp-gateway` с использованием конечной точки `POST /adapters` для удаленного запуска сервера mcp.

#### Пример 1: мостовой [сервер Azure MCP](https://github.com/microsoft/mcp/tree/main/servers/Azure.Mcp.Server)
```json
{
  "name": "azure-remote",
  "imageName": "mcp-proxy",
  "imageVersion": "1.0.0",
  "environmentVariables": {
    "MCP_COMMAND": "npx",
    "MCP_ARGS": "-y @azure/mcp@latest server start",
    "AZURE_MCP_INCLUDE_PRODUCTION_CREDENTIALS": "true",
    "DOTNET_SYSTEM_GLOBALIZATION_INVARIANT": "1"
  },
  "description": "Bridged Azure local MCP server"
}
```

#### Пример 2: Мостовое соединение [Azure AI Foundry MCP Server](https://github.com/azure-ai-foundry/mcp-foundry)
```json
{
  "name": "foundry-remote",
  "imageName": "mcp-proxy",
  "imageVersion": "1.0.0",
  "environmentVariables": {
      "MCP_COMMAND": "uvx",
      "MCP_ARGS": "--prerelease=allow --from git+https://github.com/azure-ai-foundry/mcp-foundry.git run-azure-ai-foundry-mcp"
  },
  "useWorkloadIdentity": true,
  "description": "Bridged Azure AI Foundry Local MCP Server"
}
```

#### Пример 3: Мостовой [сервер Azure DevOps MCP](https://github.com/microsoft/azure-devops-mcp)
```json
{
    "name": "ado-remote",
    "imageName": "mcp-proxy",
    "imageVersion": "1.0.0",
    "environmentVariables": {
      "MCP_COMMAND": "npx",
      "MCP_ARGS": "-y @azure-devops/mcp contoso",
      "ADO_MCP_AZURE_TOKEN_CREDENTIALS": "WorkloadIdentityCredential",
      "AZURE_TOKEN_CREDENTIALS": "WorkloadIdentityCredential"
    },
    "useWorkloadIdentity": true,
    "description": "Bridged ADO MCP Local Server"
}
```

> **Примечание.** Разные серверы MCP используют разные правила чтения учетных данных из среды для настройки `TokenCredential` и подключения к вышестоящим ресурсам. Возможно, вам придется изменить имена/значения переменных среды для каждого сервера.<br>
Примеры:
Некоторые серверы ожидают общего переключения, например `AZURE_TOKEN_CREDENTIALS=WorkloadIdentityCredential`.
Другие используют переменные, специфичные для службы (например, `ADO_MCP_AZURE_TOKEN_CREDENTIALS`).

#### Пример 4: Внутренний прокси-сервер MCP (потоковый HTTP)
```json
{
    "name": "internal-mcp",
    "imageName": "mcp-proxy",
    "imageVersion": "1.0.0",
    "environmentVariables": {
      "MCP_PROXY_URL": "https://internal-mcp-server/mcp"
    },
    "description": "Proxied Internal MCP Server"
}
```

## Вопросы безопасности

Перед запуском в производство
- Внедрите соответствующие средства управления доступом на уровне шлюза, чтобы пользователи не могли использовать доступ к удостоверениям рабочей нагрузки через него.
- Всегда регистрируйте только доверенные серверы MCP, включайте политики доступа к сети на модулях серверов.
