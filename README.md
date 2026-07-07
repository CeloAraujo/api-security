# API Security Auth Rate Limiting

Este projeto demonstra uma API REST de clientes com seguranca aplicada na pratica:

- autenticacao com JWT;
- autorizacao por roles (RBAC);
- service token para integracoes;
- rate limiting por token/IP;
- persistencia em MongoDB;
- testes automatizados com o test runner nativo do Node.js;
- um servidor MCP que expoe a API como ferramentas para agentes compativeis com Model Context Protocol.

O repositorio tem dois subprojetos principais:

```text
api-security-auth-rate-limiting/
  nodejs-fastify-mongodb-crud-z/  # API REST Fastify + MongoDB
  customers-mcp-z/                # Servidor MCP que consome a API REST
```

## Objetivo

O objetivo e mostrar como proteger uma API Node.js com camadas simples e verificaveis:

1. O usuario faz login com `username` e `password`.
2. A API emite um JWT com a role do usuario.
3. As rotas protegidas exigem `Authorization: Bearer <token>`.
4. Algumas rotas exigem role `admin`.
5. Tambem existe um fluxo de `service-token`, pensado para integracoes como MCP.
6. O rate limit limita chamadas usando o bearer token ou o IP como chave.

## Funcionalidades

### API REST

A API gerencia clientes (`customers`) com os campos:

```json
{
  "_id": "MongoDB ObjectId",
  "name": "Nome do cliente",
  "phone": "Telefone"
}
```

Endpoints principais:

| Metodo | Rota | Publica | Roles | Descricao |
|---|---|---:|---|---|
| GET | `/v1/health` | Sim | nenhuma | Verifica se a API esta de pe |
| POST | `/v1/auth/login` | Sim | nenhuma | Gera JWT para usuario e senha |
| POST | `/v1/auth/service-token` | Sim | nenhuma | Gera service token usando usuario, senha e segredo admin |
| GET | `/v1/customers` | Nao | `admin`, `member` | Lista clientes |
| GET | `/v1/customers/:id` | Nao | `admin`, `member` | Busca cliente por id |
| POST | `/v1/customers` | Nao | `admin` | Cria cliente |
| PUT | `/v1/customers/:id` | Nao | `admin` | Atualiza cliente |
| DELETE | `/v1/customers/:id` | Nao | `admin` | Remove cliente |

### Usuarios e roles

Os usuarios de autenticacao estao definidos em `nodejs-fastify-mongodb-crud-z/src/auth.js`:

| Role | Username | Password | Permissoes |
|---|---|---|---|
| `admin` | `marceloaraujo` | `123123` | listar, buscar, criar, atualizar e deletar |
| `member` | `ananeri` | `1234` | listar e buscar |

O segredo exigido para gerar service token e:

```text
AM I THE BOSS?
```

### Rate limit

O rate limit esta configurado em `nodejs-fastify-mongodb-crud-z/src/config.js`:

```js
export const REQUESTS_PER_MINUTE = 90
```

A chave do rate limit vem de:

1. `Authorization: Bearer <token>`, quando o header existe;
2. IP da requisicao, quando nao existe bearer token.

Ao exceder o limite, a API retorna `429 Too Many Requests`.

### Servidor MCP

O subprojeto `customers-mcp-z` cria um servidor MCP que encapsula a API REST como tools.

Ele usa:

- base URL fixa: `http://localhost:9999/v1`;
- variavel obrigatoria: `SERVICE_TOKEN`;
- transporte stdio, adequado para VS Code, Copilot e outros clientes MCP.

Tools registradas:

| Tool | Descricao |
|---|---|
| `list_customers` | Lista todos os clientes |
| `get_customer` | Busca cliente por `_id`, `name` ou `phone` |
| `create_customer` | Cria cliente |
| `update_customer` | Atualiza cliente por `_id` |
| `delete_customer` | Remove cliente por `_id` |

Resource registrado:

| Resource | Descricao |
|---|---|
| `customers://api-info` | Descreve a API REST usada pelo MCP |

Prompt registrado:

| Prompt | Descricao |
|---|---|
| `findCustomer` | Ajuda o agente a procurar clientes |

## Arquitetura

Fluxo principal da API:

```text
Cliente HTTP
  -> Fastify
    -> hook onRequest valida JWT ou service token
    -> requireRole valida permissao em rotas admin
    -> handlers CRUD
    -> MongoDB
```

Fluxo com MCP:

```text
Agente MCP / Copilot
  -> customers-mcp-z via stdio
    -> CustomerService
      -> CustomerHttpClient
        -> API REST em http://localhost:9999/v1
          -> MongoDB
```

Componentes da API:

| Arquivo | Responsabilidade |
|---|---|
| `src/index.js` | Sobe o Fastify, registra plugins, hooks e rotas |
| `src/auth.js` | Login, service token, JWT, roles e rate limit |
| `src/config.js` | Configuracao de banco e limite por minuto |
| `src/db.js` | Conexao com MongoDB |
| `config/seed.js` | Seed inicial de clientes |
| `config/users.js` | Dados iniciais de clientes usados no seed/testes |
| `test/api.test.js` | Testes de auth, roles, rate limit e CRUD |

Componentes do MCP:

| Arquivo | Responsabilidade |
|---|---|
| `src/index.ts` | Entrada do MCP, valida `SERVICE_TOKEN` e conecta stdio |
| `src/mcp/server.ts` | Cria o servidor MCP e registra tools/resource/prompt |
| `src/application/customer-service.ts` | Regras de busca e operacoes de cliente |
| `src/infrastructure/customer-http-client.ts` | Chamadas HTTP para a API REST |
| `src/mcp/tools/*.ts` | Tools MCP |
| `src/mcp/resources/api-info.ts` | Resource com descricao da API |

## Pre-requisitos

Para a API:

- Node.js 20 ou superior;
- Docker;
- Docker Compose;
- npm.

Para o MCP:

- Node.js compativel com TypeScript nativo. O `package.json` do MCP declara `v24.14.0`.

No PowerShell desta maquina, pode ser necessario usar `npm.cmd` em vez de `npm`, porque o `npm.ps1` pode ser bloqueado pela Execution Policy.

## Como instalar

Entre no projeto da API:

```powershell
cd C:\Users\marcelo.araujo\Desktop\ia-aplicada\api-security-auth-rate-limiting\nodejs-fastify-mongodb-crud-z
```

Instale as dependencias:

```powershell
npm.cmd ci
```

Verifique vulnerabilidades:

```powershell
npm.cmd audit
```

Resultado esperado:

```text
found 0 vulnerabilities
```

## Como rodar a API

Suba o MongoDB:

```powershell
npm.cmd run infra:up:db
```

Rode a API:

```powershell
npm.cmd start
```

A API sobe por padrao em:

```text
http://localhost:9999
```

Se a porta `9999` ja estiver ocupada, rode em outra porta:

```powershell
npx.cmd cross-env DB_NAME=customers PORT=10099 node src/index.js
```

Nesse caso, a URL passa a ser:

```text
http://localhost:10099
```

## Como verificar se esta tudo ok

Health check:

```powershell
Invoke-RestMethod http://localhost:9999/v1/health
```

Resposta esperada:

```json
{
  "app": "customers",
  "version": "v1.0.1"
}
```

Verificar se o MongoDB esta rodando:

```powershell
docker compose ps
```

Verificar quem esta usando a porta `9999`:

```powershell
Get-NetTCPConnection -LocalPort 9999
```

## Como testar a API

Com o MongoDB rodando, execute:

```powershell
npm.cmd test
```

A suite cobre:

- emissao de service token;
- login JWT;
- bloqueio de credenciais invalidas;
- bloqueio de rota protegida sem token;
- permissoes de `admin`;
- restricoes de `member`;
- CRUD de clientes;
- rate limit;
- health check.

Resultado esperado:

```text
# tests 24
# pass 24
# fail 0
```

## Como testar autenticacao JWT

### Login como admin

```powershell
$adminBody = @{
  username = "marceloaraujo"
  password = "123123"
} | ConvertTo-Json

$adminLogin = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/auth/login `
  -ContentType "application/json" `
  -Body $adminBody

$adminToken = $adminLogin.token
$adminToken
```

### Login como member

```powershell
$memberBody = @{
  username = "ananeri"
  password = "1234"
} | ConvertTo-Json

$memberLogin = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/auth/login `
  -ContentType "application/json" `
  -Body $memberBody

$memberToken = $memberLogin.token
$memberToken
```

## Como verificar roles

### Member pode listar clientes

```powershell
Invoke-RestMethod `
  -Uri http://localhost:9999/v1/customers `
  -Headers @{ Authorization = "Bearer $memberToken" }
```

Resultado esperado: HTTP 200 com a lista de clientes.

### Member nao pode criar cliente

```powershell
$customerBody = @{
  name = "Cliente Member"
  phone = "11999999999"
} | ConvertTo-Json

Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/customers `
  -ContentType "application/json" `
  -Headers @{ Authorization = "Bearer $memberToken" } `
  -Body $customerBody
```

Resultado esperado: HTTP 403 com mensagem de permissao insuficiente.

### Admin pode criar cliente

```powershell
$customerBody = @{
  name = "Cliente Admin"
  phone = "11888888888"
} | ConvertTo-Json

Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/customers `
  -ContentType "application/json" `
  -Headers @{ Authorization = "Bearer $adminToken" } `
  -Body $customerBody
```

Resultado esperado: HTTP 201 com `id` do cliente criado.

## Como gerar e usar service token

O service token e util para integracoes. O MCP usa esse tipo de token.

Gerar service token como admin:

```powershell
$serviceBody = @{
  username = "marceloaraujo"
  password = "123123"
  adminSuperSecret = "AM I THE BOSS?"
} | ConvertTo-Json

$serviceResponse = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/auth/service-token `
  -ContentType "application/json" `
  -Body $serviceBody

$serviceToken = $serviceResponse.serviceToken
$serviceResponse
```

Usar o service token:

```powershell
Invoke-RestMethod `
  -Uri http://localhost:9999/v1/customers `
  -Headers @{ Authorization = "Bearer $serviceToken" }
```

## Como testar rate limit

O limite esta configurado em 90 requisicoes por minuto. Para testar com um token:

```powershell
1..95 | ForEach-Object {
  try {
    Invoke-RestMethod `
      -Uri http://localhost:9999/v1/customers `
      -Headers @{ Authorization = "Bearer $serviceToken" } | Out-Null

    "Request $_ OK"
  } catch {
    "Request $_ falhou: $($_.Exception.Message)"
  }
}
```

Depois de ultrapassar o limite, a API deve retornar erro HTTP 429.

## Como rodar o MCP

Primeiro, garanta que a API esta rodando em:

```text
http://localhost:9999/v1
```

Depois gere um service token como mostrado acima.

Entre no projeto MCP:

```powershell
cd C:\Users\marcelo.araujo\Desktop\ia-aplicada\api-security-auth-rate-limiting\customers-mcp-z
```

Instale as dependencias:

```powershell
npm.cmd ci
```

Defina o service token:

```powershell
$env:SERVICE_TOKEN = "<cole-o-service-token-aqui>"
```

Inicie o servidor MCP:

```powershell
npm.cmd start
```

O servidor MCP roda via stdio. Isso significa que ele normalmente e iniciado por um cliente MCP, como VS Code/Copilot, em vez de expor uma porta HTTP propria.

## Como inspecionar o MCP

Use o MCP Inspector:

```powershell
npm.cmd run mcp:inspect
```

No inspector, confira:

- tools: `list_customers`, `get_customer`, `create_customer`, `update_customer`, `delete_customer`;
- resource: `customers://api-info`;
- prompt: `findCustomer`.

## Configuracao MCP no VS Code

Exemplo de `.vscode/mcp.json`:

```json
{
  "servers": {
    "customers-mcp": {
      "command": "node",
      "args": [
        "C:\\Users\\marcelo.araujo\\Desktop\\ia-aplicada\\api-security-auth-rate-limiting\\customers-mcp-z\\src\\index.ts"
      ],
      "env": {
        "SERVICE_TOKEN": "<cole-o-service-token-aqui>"
      }
    }
  }
}
```

Depois de alterar a configuracao, recarregue o VS Code.

## Variaveis de ambiente

API:

| Variavel | Padrao | Descricao |
|---|---|---|
| `DB_USER` | `root` | Usuario do MongoDB |
| `DB_PASSWORD` | `example` | Senha do MongoDB |
| `DB_HOST` | `localhost` | Host do MongoDB |
| `DB_PORT` | `27017` | Porta do MongoDB |
| `DB_NAME` | aleatorio em teste | Nome do banco |
| `PORT` | `9999` | Porta HTTP da API |
| `NODE_ENV` | vazio | Quando `test`, habilita comportamento de teste |

MCP:

| Variavel | Obrigatoria | Descricao |
|---|---:|---|
| `SERVICE_TOKEN` | Sim | Token usado pelo MCP para chamar a API |

## Scripts uteis

API (`nodejs-fastify-mongodb-crud-z`):

| Script | Descricao |
|---|---|
| `npm.cmd start` | Sobe a API em `localhost:9999` |
| `npm.cmd run dev` | Sobe a API em modo watch |
| `npm.cmd test` | Roda os testes automatizados |
| `npm.cmd run infra:up` | Sobe API e MongoDB pelo Docker Compose |
| `npm.cmd run infra:up:db` | Sobe apenas o MongoDB |
| `npm.cmd run infra:down` | Para containers e remove volumes |

MCP (`customers-mcp-z`):

| Script | Descricao |
|---|---|
| `npm.cmd start` | Inicia o servidor MCP via stdio |
| `npm.cmd run dev` | Inicia com watch e inspector do Node |
| `npm.cmd test` | Roda testes do MCP |
| `npm.cmd run mcp:inspect` | Abre o MCP Inspector |

## Troubleshooting

### `npm.ps1 nao pode ser carregado`

Use `npm.cmd`:

```powershell
npm.cmd ci
npm.cmd test
npm.cmd start
```

### Porta 9999 ocupada

Veja quem esta usando:

```powershell
Get-NetTCPConnection -LocalPort 9999 | Select-Object LocalAddress,LocalPort,State,OwningProcess
```

Suba em outra porta:

```powershell
npx.cmd cross-env DB_NAME=customers PORT=10099 node src/index.js
```

### API retorna 401

Possiveis causas:

- header `Authorization` ausente;
- token JWT invalido;
- service token nao foi gerado nesta execucao da API;
- token escrito sem o formato `Bearer <token>`.

### API retorna 403

O usuario esta autenticado, mas nao tem role suficiente. Exemplo: `member` tentando criar, atualizar ou deletar cliente.

### API retorna 429

O rate limit foi excedido. Aguarde a janela de 1 minuto ou use outro token apenas em ambiente de teste.

### MCP encerra ao iniciar

Verifique se `SERVICE_TOKEN` foi definido:

```powershell
$env:SERVICE_TOKEN
```

Tambem confirme que a API REST esta rodando em `http://localhost:9999/v1`.

## Checklist rapido

Para validar tudo do zero:

```powershell
cd C:\Users\marcelo.araujo\Desktop\ia-aplicada\api-security-auth-rate-limiting\nodejs-fastify-mongodb-crud-z
npm.cmd ci
npm.cmd run infra:up:db
npm.cmd test
npm.cmd start
```

Em outro terminal:

```powershell
Invoke-RestMethod http://localhost:9999/v1/health
```

Depois:

1. faca login como admin;
2. liste clientes;
3. crie um cliente;
4. faca login como member;
5. tente criar cliente e confirme o `403`;
6. gere um service token;
7. use o service token no MCP.
