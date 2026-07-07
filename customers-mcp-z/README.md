# Customers MCP

Servidor MCP que expoe a Customers API como ferramentas para agentes compativeis com Model Context Protocol.

## Funcionalidades

- Lista clientes pela API REST.
- Busca cliente por `_id`, `name` ou `phone`.
- Cria cliente.
- Atualiza cliente.
- Remove cliente.
- Expoe um resource com informacoes da API.
- Expoe um prompt para auxiliar buscas de clientes.

## Como funciona

O MCP roda via stdio e chama a API REST em:

```text
http://localhost:9999/v1
```

Para autenticar essas chamadas, ele exige a variavel de ambiente:

```text
SERVICE_TOKEN
```

Esse token deve ser gerado na API pelo endpoint:

```text
POST /v1/auth/service-token
```

## Tools

| Tool | Descricao |
|---|---|
| `list_customers` | Lista todos os clientes |
| `get_customer` | Busca cliente por `_id`, `name` ou `phone` |
| `create_customer` | Cria cliente |
| `update_customer` | Atualiza cliente por `_id` |
| `delete_customer` | Remove cliente por `_id` |

## Resource

| Resource | Descricao |
|---|---|
| `customers://api-info` | Descreve os endpoints da Customers API |

## Como rodar

Instale as dependencias:

```powershell
npm.cmd ci
```

Defina o service token:

```powershell
$env:SERVICE_TOKEN = "<service-token-gerado-na-api>"
```

Inicie o MCP:

```powershell
npm.cmd start
```

## MCP Inspector

```powershell
npm.cmd run mcp:inspect
```

Use o inspector para testar as tools e ler o resource `customers://api-info`.

## Scripts

| Script | Descricao |
|---|---|
| `npm.cmd start` | Inicia o servidor MCP via stdio |
| `npm.cmd run dev` | Inicia com watch e inspector do Node |
| `npm.cmd test` | Executa os testes |
| `npm.cmd run mcp:inspect` | Abre o MCP Inspector |
