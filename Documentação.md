# Documentação de APIs — Hub Social

## Como acessar a documentação interativa

Com o servidor rodando (`uvicorn app:app --reload`), acesse:

| Interface | URL | Descrição |
|---|---|---|
| **Swagger UI** | http://localhost:8000/docs | Interface visual, testa os endpoints direto no navegador |
| **ReDoc** | http://localhost:8000/redoc | Documentação em formato leitura |
| **OpenAPI JSON** | http://localhost:8000/openapi.json | Schema bruto (importar no Postman/Insomnia) |

### Como importar no Postman ou Insomnia
1. Abra o Postman → **Import** → URL → cole `http://localhost:8000/openapi.json`
2. Todos os endpoints são importados automaticamente com exemplos

---

## Endpoints documentados (13 no total)

### 🔐 Autenticação

| # | Método | Endpoint | Descrição | Auth | Status |
|---|---|---|---|---|---|
| 1 | POST | `/auth/cadastro` | Criar conta com bcrypt | ❌ | 201, 409, 422 |
| 2 | POST | `/auth/login` | Login → retorna JWT | ❌ | 200, 401, 403 |
| 3 | GET  | `/auth/me` | Perfil do usuário logado | ✅ | 200, 401 |

### 💰 Doações

| # | Método | Endpoint | Descrição | Auth | Status |
|---|---|---|---|---|---|
| 4 | GET    | `/doacoes/` | Listar doações da ONG | ✅ | 200, 401 |
| 5 | POST   | `/doacoes/` | Registrar nova doação | ✅ | 201, 401 |
| 6 | PATCH  | `/doacoes/{id}/status` | Atualizar status | ✅ | 200, 400, 404 |
| 7 | DELETE | `/doacoes/{id}` | Remover doação | ✅ | 204, 404 |

### 👥 Voluntários

| # | Método | Endpoint | Descrição | Auth | Status |
|---|---|---|---|---|---|
| 8  | GET   | `/voluntarios/` | Listar voluntários | ✅ | 200, 401 |
| 9  | POST  | `/voluntarios/` | Cadastrar (respeita limite do plano) | ✅ | 201, 403 |
| 10 | PATCH | `/voluntarios/{id}/desativar` | Desativar voluntário | ✅ | 200, 404 |

### 📅 Eventos (Plano Impacto+)

| # | Método | Endpoint | Descrição | Auth | Status |
|---|---|---|---|---|---|
| 11 | GET  | `/eventos/` | Listar eventos por data | ✅ | 200, 403 |
| 12 | POST | `/eventos/` | Criar evento | ✅ | 201, 403 |

### 📊 Relatórios (Plano Impacto+)

| # | Método | Endpoint | Descrição | Auth | Status |
|---|---|---|---|---|---|
| 13 | GET | `/relatorios/visao-geral` | Resumo completo da ONG | ✅ | 200, 403 |
| 14 | GET | `/relatorios/doacoes-por-tipo` | Agrupado por tipo | ✅ | 200, 403 |

---

## Exemplos de Request / Response

### POST /auth/cadastro
**Request:**
```json
{
  "nome":     "Maria Silva",
  "email":    "maria@ong.org.br",
  "senha":    "Senha123",
  "nome_ong": "Lar dos Bichinhos",
  "plano":    "impacto"
}
```
**Response 201:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type":   "bearer",
  "usuario_id":   1,
  "nome":         "Maria Silva",
  "nome_ong":     "Lar dos Bichinhos",
  "plano":        "impacto"
}
```
**Response 409:**
```json
{ "detail": "Este e-mail já está cadastrado. Tente fazer login." }
```

---

### POST /auth/login
**Request (form-data):**
```
username=maria@ong.org.br
password=Senha123
```
**Response 200:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type":   "bearer",
  "usuario_id":   1,
  "nome":         "Maria Silva",
  "nome_ong":     "Lar dos Bichinhos",
  "plano":        "impacto"
}
```

---

### POST /doacoes/
**Header:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "doador":    "João Souza",
  "tipo":      "financeira",
  "descricao": "Doação mensal",
  "valor":     "200.00"
}
```
**Response 201:**
```json
{
  "id":         1,
  "usuario_id": 1,
  "doador":     "João Souza",
  "tipo":       "financeira",
  "descricao":  "Doação mensal",
  "valor":      "200.00",
  "status":     "pendente",
  "criado_em":  "2025-05-01T14:30:00"
}
```

---

### GET /relatorios/visao-geral
**Response 200 (Plano Impacto):**
```json
{
  "ong":   "Lar dos Bichinhos",
  "plano": "impacto",
  "doacoes": {
    "total":            18,
    "confirmadas":      14,
    "valor_arrecadado": 3450.00
  },
  "voluntarios": {
    "ativos":       12,
    "limite_plano": 25
  },
  "eventos": {
    "total": 5
  }
}
```
**Response 403 (Plano Semente):**
```json
{ "detail": "Relatórios disponíveis a partir do Plano Impacto." }
```
