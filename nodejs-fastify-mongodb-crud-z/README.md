# Customers API

API REST de clientes criada com Node.js, Fastify e MongoDB. O foco do projeto e demonstrar autenticacao, autorizacao por roles, service token, rate limiting e testes automatizados.

## Funcionalidades

- Health check publico.
- Login com JWT.
- Service token para integracoes.
- RBAC com roles `admin` e `member`.
- CRUD de clientes.
- Rate limit por bearer token ou IP.
- Persistencia em MongoDB.
- Testes com o test runner nativo do Node.js.

## Usuarios

| Role | Username | Password | Permissoes |
|---|---|---|---|
| `admin` | `marceloaraujo` | `123123` | listar, buscar, criar, atualizar e deletar |
| `member` | `ananeri` | `1234` | listar e buscar |

## Endpoints

| Metodo | Rota | Autenticacao | Descricao |
|---|---|---|---|
| GET | `/v1/health` | publica | Verifica se a API esta rodando |
| POST | `/v1/auth/login` | publica | Retorna um JWT |
| POST | `/v1/auth/service-token` | publica | Retorna um service token |
| GET | `/v1/customers` | `admin` ou `member` | Lista clientes |
| GET | `/v1/customers/:id` | `admin` ou `member` | Busca cliente por id |
| POST | `/v1/customers` | `admin` | Cria cliente |
| PUT | `/v1/customers/:id` | `admin` | Atualiza cliente |
| DELETE | `/v1/customers/:id` | `admin` | Remove cliente |

## Como rodar

Instale as dependencias:

```powershell
npm.cmd ci
```

Suba o MongoDB:

```powershell
npm.cmd run infra:up:db
```

Inicie a API:

```powershell
npm.cmd start
```

A API fica disponivel em:

```text
http://localhost:9999
```

## Como verificar

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

## Login

Admin:

```powershell
$body = @{
  username = "marceloaraujo"
  password = "123123"
} | ConvertTo-Json

$login = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/auth/login `
  -ContentType "application/json" `
  -Body $body

$token = $login.token
```

Use o token nas rotas protegidas:

```powershell
Invoke-RestMethod `
  -Uri http://localhost:9999/v1/customers `
  -Headers @{ Authorization = "Bearer $token" }
```

## Service token

```powershell
$body = @{
  username = "marceloaraujo"
  password = "123123"
  adminSuperSecret = "AM I THE BOSS?"
} | ConvertTo-Json

$response = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:9999/v1/auth/service-token `
  -ContentType "application/json" `
  -Body $body

$serviceToken = $response.serviceToken
```

## Testes

```powershell
npm.cmd test
```

A suite valida login, service token, roles, CRUD, health check e rate limit.

## Scripts

| Script | Descricao |
|---|---|
| `npm.cmd start` | Sobe a API |
| `npm.cmd run dev` | Sobe a API em watch mode |
| `npm.cmd test` | Executa os testes |
| `npm.cmd run infra:up` | Sobe API e MongoDB via Docker Compose |
| `npm.cmd run infra:up:db` | Sobe apenas MongoDB |
| `npm.cmd run infra:down` | Para containers e remove volumes |
