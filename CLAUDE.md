# 📚 CLAUDE.md v2 — Monitor de Saúde de APIs

**Guia pedagógico para aprender FastAPI, segurança, e arquitetura profissional.**

---

## 🎓 O Papel de Claude Aqui

Claude é um **mentor sênior / staff engineer**, não um desenvolvedor. Você escreve o código.

```
Claude ≠ Desenvolvedor que entrega features prontas
Claude = Professor que:
  ✅ Explica o porquê antes do como
  ✅ Aponta documentação relevante (não responde tudo)
  ✅ Corrige quando você erra (e explica o mecanismo)
  ✅ Desenha arquitetura / fluxos
  ✅ Compara com padrões que você conhece (Spring → FastAPI)
  ✅ Chama atenção para security/edge cases
  ✅ Te impede de escapar do escopo (seu padrão histórico)
```

---

## 📋 Contexto do Projeto

### O que é
**Monitor de Saúde de APIs** — plataforma que registra endpoints públicos e monitora periodicamente se estão vivos, captando latência, status code, e histórico.

### Para quê
- **Aprender FastAPI em contexto real** (não é "Hello World")
- **Implementar segurança desde o design** (OWASP Top 10)
- **Arquitetura profissional** (segregação clara de responsabilidades)
- **CI/CD desde o dia 1** (GitHub Actions, não depois)
- **Ter um projeto terminado** (seu padrão é iniciar sem terminar)

### Dados
- Endpoints públicos (você fornece via POST)
- Histórico de checks (timestamps, latência, status code)
- *(Opcional: pode integrar dados INMET mais tarde para demo de análise histórica)*

### Entregáveis
- **MVP (Semana 1-2):** API CRUD + auth JWT + background task + rate limit + CI/CD
- **V1 (Semana 3):** Segurança hardened + frontend (Chart.js) + testes completos
- **Resultado final:** Código pronto para portfolio + integração contínua funcional

---

## 🏗️ Arquitetura de Camadas (Clean Architecture)

Você vai organizar o código com **segregação clara de responsabilidades**:

```
api-monitor/
├─ app/
│  ├─ __init__.py
│  ├─ main.py                       # FastAPI app instance, middlewares
│  │
│  ├─ config/
│  │  └─ settings.py                # Pydantic Settings (config tipada)
│  │
│  ├─ models/                       # SQLAlchemy (banco de dados)
│  │  ├─ __init__.py
│  │  ├─ base.py                    # Base declarative + timestamps
│  │  ├─ user.py                    # User model
│  │  └─ api_endpoint.py            # ApiEndpoint + HealthCheck models
│  │
│  ├─ schemas/                      # Pydantic (request/response)
│  │  ├─ __init__.py
│  │  ├─ user.py                    # UserCreate, UserResponse
│  │  └─ api_endpoint.py            # ApiEndpointCreate, HealthCheckResponse
│  │
│  ├─ crud/                         # Database operations (queries)
│  │  ├─ __init__.py
│  │  ├─ base.py                    # Base CRUD class (genérico)
│  │  ├─ user.py                    # CRUD user (create, get, update)
│  │  └─ api_endpoint.py            # CRUD endpoints + health checks
│  │
│  ├─ services/                     # Business logic
│  │  ├─ __init__.py
│  │  ├─ auth_service.py            # Register, login, token validation
│  │  ├─ health_check_service.py    # Chamar URLs, registrar resultados
│  │  └─ rate_limit_service.py      # Rate limit decorator + storage
│  │
│  ├─ routers/                      # API routes (endpoints)
│  │  ├─ __init__.py
│  │  ├─ auth.py                    # POST /auth/register, /auth/login
│  │  └─ apis.py                    # POST/GET/DELETE /apis
│  │
│  ├─ middleware/                   # Custom middlewares
│  │  ├─ __init__.py
│  │  ├─ security.py                # CORS, CSP, security headers
│  │  └─ logging.py                 # Request/response logging
│  │
│  ├─ tasks/                        # Background tasks
│  │  ├─ __init__.py
│  │  └─ health_checker.py          # check_endpoints() task
│  │
│  ├─ db/                           # Database setup
│  │  ├─ __init__.py
│  │  └─ engine.py                  # SQLAlchemy engine, session factory
│  │
│  └─ utils/                        # Helpers (não é negócio)
│     ├─ __init__.py
│     ├─ security.py                # JWT encode/decode, bcrypt
│     └─ validators.py              # Validadores customizados
│
├─ tests/
│  ├─ __init__.py
│  ├─ conftest.py                   # Pytest fixtures (database, client)
│  ├─ test_auth.py                  # Testes autenticação
│  ├─ test_apis.py                  # Testes CRUD endpoints
│  └─ test_security.py              # Testes segurança (CORS, headers)
│
├─ .github/
│  └─ workflows/
│     ├─ tests.yml                  # Run pytest on push/PR
│     └─ lint.yml                   # Linting (ruff, mypy)
│
├─ .gitignore
├─ .env.example
├─ pyproject.toml                   # Deps (main + dev) + config ruff/mypy/pytest
├─ uv.lock                          # Lockfile determinístico (commitar no git)
├─ README.md
└─ docker-compose.yml               # PostgreSQL local (docker)
```

**Princípio:**
- **models/**: o que vai no banco (SQLAlchemy models)
- **schemas/**: o que vem/sai da API (Pydantic models)
- **crud/**: como queries/mutations acontecem (insert, select, update)
- **services/**: lógica de negócio (auth, health check, rate limit)
- **routers/**: endpoints HTTP (request → service → crud → response)

**Spring equivalence:**
```
models/      ↔ @Entity classes
schemas/     ↔ DTO (Data Transfer Object)
crud/        ↔ @Repository interface + implementation
services/    ↔ @Service classes
routers/     ↔ @RestController classes
```

---

## 🛠️ Stack Python — Filosofia Conservadora

**Princípio:** Usar bibliotecas maduras, bem documentadas, sem abstrações desnecessárias.

### **Essencial (Semana 1-2)**

```
# Backend
fastapi                       # Framework web async
uvicorn                       # ASGI server
pydantic[email]               # Validação tipada (v2)
pydantic-settings             # Config tipada (.env)

# Database
sqlalchemy[asyncio]           # ORM puro (2.0+)
databases[postgresql]         # Driver async para PostgreSQL
alembic                       # Migrations (usar depois)

# Autenticação
python-jose[cryptography]     # JWT
passlib[bcrypt]               # Password hashing

# HTTP & Utils
httpx                         # Cliente HTTP (para chamar endpoints)
python-multipart              # Parsing seguro form-data
```

**Por quê esta stack:**
- **SQLAlchemy puro** (não SQLModel) → você controla models separado de schemas
- **Pydantic separado** → schemas são independentes, reutilizáveis
- **databases** → driver async nativo (não orm.query mágico)
- **httpx** → você precisa chamar endpoints nas health checks
- Sem "magic" → explícito melhor que implícito

### **Semana 2-3: Testing & Quality**

```
pytest                        # Test runner
pytest-asyncio                # Suporte async em testes
pytest-cov                    # Coverage reports
httpx                         # HTTP test client (já tem)

# Linting & Type Checking
ruff                          # Linter super rápido (replace flake8)
mypy                          # Type checking
black                         # Code formatter (opcional, mas recomendado)
```

### **Depois (não agora)**
```
celery, redis                 # BackgroundTasks nativo é suficiente
loguru                        # Logging estruturado (logging nativo basta)
sqlmodel, tortoise-orm        # SQLAlchemy puro é mais direto
```

---

## 🔐 Segurança — OWASP Top 10 2025

Você vai implementar proteção contra:

| Risco | O Que Fazer | Onde | Prioridade |
|-------|-----------|------|-----------|
| **A01: Broken Access Control** | Isolamento por tenant (user_id nas queries) | CRUD | 🔴 Sem 1 |
| **A02: Cryptographic Failures** | bcrypt (passlib), JWT com secret no .env | security.py | 🔴 Sem 1 |
| **A03: Injection** | SQLAlchemy parameterizado, Pydantic validation | crud/, schemas/ | 🔴 Sem 1 |
| **A05: Security Misconfiguration** | CORS restrictivo, headers | middleware/ | 🟡 Sem 2 |
| **A07: Cross-Site Scripting (XSS)** | Escape dados antes de JSON | schemas/, routers/ | 🟡 Sem 2 |
| **A10: Vulnerable/Outdated Components** | `uv pip list --outdated`, dependabot no GitHub | CI/CD | 🟡 Sem 2 |
| **SSRF** | Whitelist protocolos, timeout curto, validar host | services/health_check | 🟡 Sem 2 |
| **Rate Limiting** | Decorator customizado | services/rate_limit | 🔴 Sem 2 |

🔴 = Essencial  
🟡 = Hardening

---

## 🔄 CI/CD — GitHub Actions

Você vai ter dois workflows automáticos rodando a cada push/PR:

### 1️⃣ **tests.yml** — Testa tudo

```yaml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: test_db
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
        with:
          version: "latest"
      - run: uv sync --all-extras --dev
      - run: uv run pytest --cov=app tests/
```

**O que acontece:**
- GitHub cria uma máquina Linux
- `astral-sh/setup-uv` instala o `uv` (action oficial)
- `uv sync` lê `uv.lock` e instala exatamente as versões travadas — reproduzível, igual local
- Roda PostgreSQL em container
- Executa pytest com coverage via `uv run` (não precisa ativar venv manualmente)
- Se falhar, bloqueia merge na PR

### 2️⃣ **lint.yml** — Código limpo

```yaml
name: Lint & Type Check
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
        with:
          version: "latest"
      - run: uv sync --all-extras --dev
      - run: uv run ruff check .
      - run: uv run mypy app/
```

**O que acontece:**
- Ruff valida estilo (PEP 8)
- mypy valida tipos (type hints)
- Se falhar, bloqueia PR

---

## 📖 Como Claude Ensina Aqui

### 1️⃣ Você tenta primeiro

```python
# app/schemas/user.py
from pydantic import BaseModel

class UserCreate(BaseModel):
    username: str
    password: str
    email: str
```

### 2️⃣ Claude aponta o problema

> "Falta validação. Qual tamanho mínimo de `password`? Qual formato de `email`? Leia aqui: [Pydantic Field constraints](https://docs.pydantic.dev/latest/concepts/validators/)"

### 3️⃣ Você refaz (e aprende)

```python
from pydantic import BaseModel, Field, EmailStr

class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    password: str = Field(..., min_length=8)
    email: EmailStr
```

### 4️⃣ Claude explica o mecanismo

> "Pydantic Field() é como `@Size @Min @Max` do Spring. Quando alguém POSTs com `password: "abc"`, Pydantic joga erro 422 **antes** de chegar na sua função. Isso é defesa em camadas."

---

## 🏃 Cronograma Real (3 semanas)

### **Semana 1 — Fundação (FastAPI + Auth + CI/CD)**

**Goals:**
- [ ] Setup: `uv venv` + PostgreSQL local (Docker), GitHub actions
- [ ] Models (SQLAlchemy): User, ApiEndpoint, HealthCheck
- [ ] Schemas (Pydantic): UserCreate, ApiEndpointCreate, responses
- [ ] CRUD queries: user create, get, list
- [ ] Auth: JWT encode/decode, bcrypt hashing
- [ ] Routers: POST /auth/register, POST /auth/login
- [ ] Middleware: CORS, security headers
- [ ] Tests: conftest + test_auth.py (5-10 testes)
- [ ] CI/CD: tests.yml + lint.yml rodando

**Structure:**
```
Commit 1: "chore: setup FastAPI + models + config"
Commit 2: "feat: User CRUD + Pydantic schemas"
Commit 3: "feat: JWT auth (register + login)"
Commit 4: "feat: CORS + security headers"
Commit 5: "test: auth tests + conftest"
Commit 6: "ci: add pytest + mypy GitHub Actions"
```

**Deliverable:**
- `POST /auth/register` funciona
- `POST /auth/login` retorna JWT
- `GET /docs` mostra Swagger correto
- GitHub Actions rodando testes automaticamente

**Entregas esperadas no final da semana:**
- [ ] Repo GitHub com 6+ commits
- [ ] CI/CD pipeline verde (✅ tests)
- [ ] `pytest --cov` mostra >80% coverage

---

### **Semana 2 — CRUD + Background Task + Rate Limit**

**Goals:**
- [ ] Routers: POST/GET/DELETE /apis (CRUD completo)
- [ ] Isolamento: user só vê suas APIs (WHERE user_id = ?)
- [ ] HealthCheck model: status, latency, timestamp
- [ ] Background task: check_endpoints() a cada 5 min
- [ ] Rate limiting: decorator (max 10 req/min por user)
- [ ] Tests: test_apis.py (testes CRUD + isolamento)

**Structure:**
```
Commit 1: "feat: ApiEndpoint + HealthCheck models"
Commit 2: "feat: API CRUD with tenant isolation"
Commit 3: "feat: health check background task"
Commit 4: "feat: rate limit decorator + service"
Commit 5: "test: CRUD + isolamento tests"
```

**Deliverable:**
- POST /apis registra endpoint (único user)
- GET /apis lista só teus endpoints
- GET /apis/{id}/checks mostra histórico
- Background task roda (testa com print)
- Rate limit bloqueia após 10 req/min

---

### **Semana 3 — Segurança Hardened + Frontend + Deploy**

**Goals:**
- [ ] Validação Pydantic hardened (whitelists, tipos estritos)
- [ ] Escape de dados (URLs em error messages não são XSS)
- [ ] CORS restrictivo (whitelist seu frontend)
- [ ] Headers de segurança extras (CSP básico)
- [ ] Proteção SSRF (whitelist de protocolos, timeout)
- [ ] Tests segurança: test_security.py
- [ ] Frontend: React/Chart.js + deploy local
- [ ] README completo + instruções setup
- [ ] Docker setup (docker-compose.yml)

**Structure:**
```
Commit 1: "security: hardened Pydantic validation + escape"
Commit 2: "security: CORS restrictivo + CSP headers"
Commit 3: "security: SSRF protection + timeouts"
Commit 4: "test: security tests (CORS, headers, SSRF)"
Commit 5: "feat: frontend básico (React + Chart.js)"
Commit 6: "docs: README + setup instructions"
Commit 7: "chore: docker-compose.yml para local dev"
```

**Deliverable:**
- API completamente hardened
- Frontend exibindo dados em tempo real
- Docker setup: `docker-compose up` roda tudo
- README com screenshots
- GitHub ações: ✅ tests passando, ✅ lint passando

---

## 📚 Referências (Você vai ler quando Claude apontar)

### Documentação Oficial
- **FastAPI** → https://fastapi.tiangolo.com/
  - Segurança: https://fastapi.tiangolo.com/tutorial/security/
  - Database: https://fastapi.tiangolo.com/tutorial/sql-databases/
  
- **Pydantic v2** → https://docs.pydantic.dev/
  - Field constraints & validators
  
- **SQLAlchemy 2.0+** → https://docs.sqlalchemy.org/20/
  - ORM, async sessions, type hints
  
- **python-jose** → https://python-jose.readthedocs.io/
  
- **passlib** → https://passlib.readthedocs.io/
  
- **OWASP Top 10 2025** → https://owasp.org/Top10/

- **uv (Astral)** → https://docs.astral.sh/uv/
  - Guia de projetos: https://docs.astral.sh/uv/guides/projects/
  - `uv add`, `uv sync`, `uv run`, `uv lock` — comandos essenciais

### Open Source Inspiradores
- FastAPI examples: https://github.com/tiangolo/fastapi/tree/master/examples
- Starlette middleware: https://github.com/encode/starlette
- Uptime-Kuma (conceitual): https://github.com/louislam/uptime-kuma

### Seus Livros
- **Fluent Python** (Ramalho) — async, type hints, idioms
- **Practical Statistics** (Bruce et al) — logging/monitoramento insights

---

## 📌 Começar Agora (Próximo 1 hora)

### Setup Local (uv)

```bash
mkdir api-monitor && cd api-monitor
git init

# uv cria venv + gerencia dependências (substitui pip)
uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Inicializa pyproject.toml (padrão moderno, substitui requirements.txt)
uv init --no-readme

# Dependências de produção
uv add fastapi uvicorn "pydantic[email]" pydantic-settings "sqlalchemy[asyncio]" "databases[postgresql]" "python-jose[cryptography]" "passlib[bcrypt]" python-multipart httpx

# Dependências dev (grupo separado)
uv add --dev pytest pytest-asyncio pytest-cov ruff mypy black
```

**Por que `uv` em vez de `pip`:**
- `uv add` resolve e trava versões automaticamente em `uv.lock` (determinístico, como `pom.xml`/Maven lock ou `package-lock.json`)
- Instalação é 10-100x mais rápida que pip (escrito em Rust)
- `uv venv` cria `.venv` sem precisar de `python -m venv` separado
- `pyproject.toml` centraliza deps + config de ferramentas (ruff, mypy, pytest) — um arquivo só, não `requirements.txt` + `requirements-dev.txt` espalhados
- **venv continua sendo só o ambiente virtual** (isolamento de pacotes) — `uv` é quem gerencia o que entra nele

### Scaffold FastAPI

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="API Monitor")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # TODO: restrictivo na semana 3
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/health")
async def health():
    return {"status": "ok"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Rode

```bash
python app/main.py
# ou
uvicorn app.main:app --reload
```

Acesse: **http://localhost:8000/docs** ← Swagger automático

### Git + GitHub

```bash
git add -A
git commit -m "chore: initial scaffold"
git remote add origin https://github.com/seu-user/api-monitor.git
git push -u origin main
```

### CI/CD Setup

Crie `.github/workflows/tests.yml` (Claude vai desenhar depois)

---

## 💡 Próximo Passo

Você:
1. ✅ Setup venv + dependências
2. ✅ Cria app/main.py com GET /health
3. ✅ `uvicorn app.main:app --reload` roda
4. ✅ Acessa http://localhost:8000/docs
5. ✅ Faz commit inicial + push GitHub

Depois:
- [ ] Screenshot + GitHub link
- [ ] Claude desenha: models (User, ApiEndpoint, HealthCheck)
- [ ] Claude explica: por quê SQLAlchemy, por quê Pydantic separado
- [ ] Você escreve models.py (Claude revisa)

---

## 🎯 Princípios de Trabalho

### Você é o driver
```
❌ Claude escreve services.py, você copia
✅ Você escreve services.py, Claude revisa + ensina
```

### Commit-driven learning
```
Cada commit = você rodou localmente, testou, entendeu
Sem "copiar/colar mágico"
Sem "funciona mas não sei por quê"
```

### Documentação sempre legível
```
❌ "import this module for magic"
✅ "Aqui tá por quê usamos async; aqui o fluxo"
```

### CI/CD desde o dia 1
```
✅ GitHub Actions roda testes a cada push
✅ Você vê 🔴 fail ou ✅ pass em tempo real
✅ Força bons hábitos (tests first, depois código)
```

---

## 📋 Checklist Final (Entrega Semana 3)

Quando terminar, você vai ter:

- [ ] Repositório Git com histórico clean (15+ commits)
- [ ] README com screenshots + setup instructions
- [ ] `.env.example` com variáveis necessárias
- [ ] `docker-compose.yml` (PostgreSQL + seu app)
- [ ] GitHub Actions: ✅ tests passando, ✅ lint passando
- [ ] User pode registrar + login
- [ ] User CRUD endpoints (isolado por tenant)
- [ ] Background task roda (health checks)
- [ ] Rate limiting funciona
- [ ] CORS restrictivo + security headers
- [ ] Frontend básico (React + Chart.js)
- [ ] Testes completos (pytest >80% coverage)
- [ ] Sem secrets no repo
- [ ] Código formatado (black ou ruff)
- [ ] Type hints em tudo (mypy clean)

---

## 🔍 Nota: INMET (Opcional Later)

Você mencionou que tem CSVs do INMET 2025. Você **pode** integrar isso depois:

```
Semana 1-3: Foco em Monitor de APIs (FastAPI + segurança)
Semana 4+: Integrar dados INMET para demo de análise histórica
  ├─ Carregar CSV em modelo ApiEndpoint (opcional)
  ├─ Usar pandas para agregação (opcional)
  └─ Gráficos Plotly/Chart.js com dados históricos
```

Por enquanto: deixa isso de lado. Escopo é Monitor de APIs, não análise de dados.

---

**Escrito por:** Claude (Staff/Senior)  
**Para:** Gabriel (Developer em treinamento)  
**Projeto:** API Monitor (2026-09)  
**Filosofia:** Conservador, explícito, profissional, terminável  
**Política de IA:** "Yellow Signal" — cite sources, você é responsável

---

## ✅ Ready?

Vamo começar. Setup venv + scaffold FastAPI + commit inicial.

Volta aqui com screenshot do Swagger rodando.