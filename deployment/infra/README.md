# Развертывание шлюза MCP в Azure

Этот каталог содержит шаблоны инфраструктуры как кода и сценарии для развертывания шлюза MCP в Azure.

## Варианты развертывания

Существует два способа развертывания инфраструктуры шлюза MCP:

### Вариант 1: сценарий PowerShell (рекомендуется)

Используйте сценарий развертывания PowerShell для лучшего контроля и разделения задач. Этот подход:
- Развертывает инфраструктуру Azure с помощью Bicep.
— Отдельно настраиваются ресурсы Kubernetes.
- Обеспечивает лучшую обработку ошибок и обратную связь о ходе работы.

**Предварительные требования:**
- Azure CLI установлен и прошел проверку подлинности (`az login`)
- PowerShell 5.1 или выше.
- Соответствующие разрешения Azure.

**Основное использование:**

```powershell
.\Deploy-McpGateway.ps1 -ResourceGroupName "rg-mcpgateway-dev" -ClientId "<your-entra-client-id>"
```

**Расширенное использование:**

```powershell
# Deploy to a specific region with a custom resource label
.\Deploy-McpGateway.ps1 `
    -ResourceGroupName "rg-mcpgateway-prod" `
    -ClientId "<your-entra-client-id>" `
    -ResourceLabel "mcpprod" `
    -Location "westus2"

# Deploy with private endpoints enabled
.\Deploy-McpGateway.ps1 `
    -ResourceGroupName "rg-mcpgateway-secure" `
    -ClientId "<your-entra-client-id>" `
    -EnablePrivateEndpoints
```

**Параметры:**

| Параметр | Требуется | Описание |
|-----------|----------|-------------|
| `ResourceGroupName` | Да | Имя группы ресурсов Azure (создается, если не существует) |
| `ClientId` | Да | Entra ID идентификатор клиента для аутентификации |
| `ResourceLabel` | Нет | Буквенно-цифровой суффикс имени ресурса (3–30 символов). По умолчанию имя группы ресурсов |
| `Location` | Нет | Регион Azure для развертывания. По умолчанию: `westus3` |
| `EnablePrivateEndpoints` | Нет | Включить частные конечные точки для ACR и Cosmos DB |

### Вариант 2: Прямое развертывание бицепса (устаревший вариант)

Выполните развертывание напрямую с помощью Bicep с помощью встроенного сценария развертывания:

```bash
# Create resource group
az group create --name rg-mcpgateway-dev --location eastus

# Deploy using Bicep
az deployment group create \
  --name mcpgateway-deployment \
  --resource-group rg-mcpgateway-dev \
  --template-file azure-deployment.bicep \
  --parameters clientId=<your-entra-client-id>
```

**С дополнительными параметрами:**

```bash
az deployment group create \
  --name mcpgateway-deployment \
  --resource-group rg-mcpgateway-dev \
  --template-file azure-deployment.bicep \
  --parameters \
    clientId=<your-entra-client-id> \
    resourceLabel=mcpdev \
    location=westus2 \
    enablePrivateEndpoints=true
```

**Отключить встроенный скрипт развертывания Kubernetes:**

```bash
az deployment group create \
  --name mcpgateway-deployment \
  --resource-group rg-mcpgateway-dev \
  --template-file azure-deployment.bicep \
  --parameters \
    clientId=<your-entra-client-id> \
    enableKubernetesDeploymentScript=false
```

## Развернутые ресурсы

При развертывании создаются следующие ресурсы Azure:

### Основная инфраструктура
- **Служба Azure Kubernetes (AKS)**: управляемый кластер Kubernetes.
- 2-узловой кластер с виртуальными машинами D4ds_v5.
- Azure RBAC включен.
- Эмитент OIDC и идентификатор рабочей нагрузки включены.
  
- **Реестр контейнеров Azure (ACR)**: хранилище образов контейнеров.
- Стандартный артикул
- Интегрирован с AKS для извлечения изображений.

- **Azure Cosmos DB**: база данных документов для состояния шлюза.
- Уровень согласованности сеанса
- Три контейнера: AdapterContainer, CacheContainer, ToolContainer.
- Дополнительная поддержка частных конечных точек.

### Сеть
- **Виртуальная сеть (VNet)**: сетевая изоляция.
- Адресное пространство 10.0.0.0/16
- Подсеть AKS (10.0.1.0/24)
- Подсеть шлюза приложений (10.0.2.0/24)
- Подсеть частной конечной точки (10.0.3.0/24)

- **Шлюз приложений**: балансировщик нагрузки уровня 7.
- Стандарт_v2 SKU
- HTTP-интерфейс на порту 80
- Проверка работоспособности для серверного мониторинга.

- **Публичный IP-адрес**: статический общедоступный IP-адрес с меткой DNS.

### Идентификация и доступ
- **Управляемые удостоверения**:
— Идентификатор службы шлюза (с ролью участника данных Cosmos DB).
— Личность администратора (для операций AKS).
- Идентификация рабочей нагрузки (для аутентификации на уровне модуля).
  
- **Федеративные учетные данные**: для интеграции учетной записи службы Kubernetes.

### Мониторинг
- **Application Insights**: мониторинг приложений и телеметрия.

## Параметры сети

### Публичный доступ (по умолчанию)
Ресурсы доступны через Интернет с надлежащей аутентификацией.

### Частные конечные точки
Включить флагом `-EnablePrivateEndpoints`:
— ACR и Cosmos DB доступны только внутри виртуальной сети.
- Частные зоны DNS настраиваются автоматически.
- Идеально подходит для производственных сред, требующих сетевой изоляции.

## После развертывания

После успешного развертывания:

1. **Доступ к шлюзу**: используйте полное доменное имя из выходных данных развертывания.
   ```
   http://<public-ip-dns-label>.<region>.cloudapp.azure.com
   ```

2. **Проверьте модули Kubernetes**:
   ```bash
   kubectl get pods -n adapter
   ```

3. **Проверьте журналы шлюза**:
   ```bash
   kubectl logs -n adapter -l app=mcpgateway
   ```

## Поиск неисправностей

### Проблемы со скриптами PowerShell

**Не выполнены необходимые условия:**
- Убедитесь, что установлен Azure CLI: `az --version`.
- Войдите в Azure: `az login`.

**Ошибки развертывания:**
- Проверьте выходные данные Azure CLI на наличие конкретных ошибок.
- Убедитесь, что у вас есть соответствующие разрешения в подписке.
– Убедитесь, что метка ресурса уникальна и соответствует требованиям к именованию.

**Проблемы с развертыванием Kubernetes:**
- Убедитесь, что кластер AKS работает: `az aks show -g <rg-name> -n <aks-name>`.
- Проверить статус модуля: `az aks command invoke -g <rg-name> -n <aks-name> --command "kubectl get pods -A"`.

### Проблемы с использованием бицепса

**Ошибки проверки шаблона:**
```bash
az deployment group validate \
  --resource-group <rg-name> \
  --template-file azure-deployment.bicep \
  --parameters clientId=<your-client-id>
```

**Ошибки сценария развертывания:**
— Проверьте журналы сценариев развертывания на портале Azure.
- Убедитесь, что управляемое удостоверение имеет соответствующие разрешения.
- Проверка доступности кластера AKS.

## Очистка

Чтобы удалить все развернутые ресурсы:

```powershell
# Delete the entire resource group
az group delete --name rg-mcpgateway-dev --yes --no-wait
```

## Миграция со встроенного скрипта на PowerShell

Если вы ранее выполняли развертывание с помощью встроенного сценария развертывания Bicep:

1. Шаблон Bicep теперь поддерживает оба метода через параметр `enableKubernetesDeploymentScript`.
2. Чтобы обновить существующее развертывание без встроенного сценария:
   ```powershell
   .\Deploy-McpGateway.ps1 -ResourceGroupName <existing-rg> -ClientId <client-id>
   ```
3. Скрипт PowerShell обновит инфраструктуру и перенастроит ресурсы Kubernetes.

## Архитектура

```
┌─────────────────────────────────────────────────────────────┐
│                      Internet                               │
└────────────────────────┬────────────────────────────────────┘
                         │
                    ┌────▼─────┐
                    │ Public IP│
                    └────┬─────┘
                         │
                ┌────────▼──────────┐
                │ Application       │
                │ Gateway           │
                └────────┬──────────┘
                         │
        ┌────────────────┼────────────────┐
        │                                 │
┌───────▼────────┐              ┌────────▼────────┐
│                │              │                 │
│  AKS Cluster   │──────────────│  ACR            │
│                │              │  (Images)       │
└───────┬────────┘              └─────────────────┘
        │
        │ Workload Identity
        │
┌───────▼────────┐              ┌─────────────────┐
│                │              │                 │
│  Cosmos DB     │              │  App Insights   │
│  (State)       │              │  (Monitoring)   │
└────────────────┘              └─────────────────┘
```

## Вопросы безопасности

- **Аутентификация**: для аутентификации используется Entra ID (Azure AD).
- **Авторизация**: Azure RBAC для AKS, Cosmos DB RBAC для доступа к данным.
- **Сетевая изоляция**: дополнительные частные конечные точки для повышения безопасности.
- **Идентификация**: Идентификатор рабочей нагрузки для аутентификации на уровне модуля (секреты не требуются).
- **Секреты**: управляемые удостоверения устраняют необходимость хранения учетных данных.

## Дополнительные ресурсы

- [Документация по службе Azure Kubernetes](https://docs.microsoft.com/azure/aks/)
- [Документация по реестру контейнеров Azure](https://docs.microsoft.com/azure/container-registry/)
- [Документация по Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/)
- [Документация по бицепсу](https://docs.microsoft.com/azure/azure-resource-manager/bicep/)
