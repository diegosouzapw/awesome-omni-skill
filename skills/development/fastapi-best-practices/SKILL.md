---
name: fastapi-best-practices
description: FastAPI best practices e convenções baseadas em produção real. Aplicar em todos os projetos FastAPI.
triggers: [fastapi, api, endpoint, router, pydantic, async, dependency injection, alembic, migration]
---

# FastAPI Best Practices

**Convenções e melhores práticas para projetos FastAPI baseadas em experiência de produção.**

**Última Atualização:** 2026-01-20  
**Fonte:** Baseado em experiência de produção de startups

---

## Quando Usar

Aplicar esta skill quando:
- Criando novos projetos FastAPI
- Refatorando projetos existentes
- Implementando novos endpoints
- Configurando estrutura de projeto
- Trabalhando com Pydantic, Dependencies, Async Routes
- Configurando banco de dados e migrations

---

## Estrutura de Projeto

### Organização por Domínio (Não por Tipo de Arquivo)

```
fastapi-project/
├── alembic/
├── src/
│   ├── {domain}/              # e.g., auth/, posts/, payments/
│   │   ├── router.py          # API endpoints
│   │   ├── schemas.py         # Pydantic models
│   │   ├── models.py          # Database models
│   │   ├── service.py         # Business logic
│   │   ├── dependencies.py   # Route dependencies
│   │   ├── config.py          # Environment variables (domain-specific)
│   │   ├── constants.py       # Constants and error codes
│   │   ├── exceptions.py      # Domain-specific exceptions
│   │   └── utils.py           # Helper functions
│   ├── config.py              # Global configuration
│   ├── models.py              # Global models
│   ├── exceptions.py          # Global exceptions
│   ├── pagination.py          # Global modules
│   ├── database.py            # Database connection
│   └── main.py                # FastAPI app initialization
├── tests/
│   ├── {domain}/              # Tests organized by domain
│   └── conftest.py
├── requirements/
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── .env
├── .env.example
├── .gitignore
├── alembic.ini
└── logging.ini
```

### Regra de Múltiplas Classes

**IMPORTANTE:** Quando um módulo tem mais de uma classe, cada classe deve estar em um arquivo separado:

```
# ❌ ERRADO: Múltiplas classes no mesmo arquivo
src/auth/models.py:
  - User
  - Role
  - Permission

# ✅ CORRETO: Uma classe por arquivo
src/auth/models/
  ├── __init__.py      # Exporta todas as classes
  ├── user.py          # class User
  ├── role.py          # class Role
  └── permission.py    # class Permission
```

**Padrão obrigatório:**
1. Se há mais de 1 classe → criar módulo (pasta)
2. Cada classe em arquivo separado
3. `__init__.py` importa e exporta todas as classes
4. Importação externa: `from src.auth.models import User, Role, Permission`

---

## Async Routes

### Regras Fundamentais

1. **`async def` routes**: Use APENAS para operações não-bloqueantes (I/O com `await`)
2. **`def` routes**: Use para operações bloqueantes (CPU-bound, cálculos pesados)
3. **Nunca misturar**: Não fazer `await` em função `def`

### Exemplos

```python
# ✅ CORRETO: I/O não-bloqueante
@router.get("/posts")
async def get_posts():
    posts = await database.fetch_all("SELECT * FROM posts")
    return posts

# ✅ CORRETO: CPU-bound (bloqueante)
@router.post("/calculate")
def calculate(data: CalculationRequest):
    result = heavy_computation(data)  # Sem await
    return result

# ❌ ERRADO: await em função def
@router.get("/posts")
def get_posts():
    posts = await database.fetch_all("SELECT * FROM posts")  # ERRO!
    return posts

# ✅ CORRETO: Wrapper para função bloqueante
from fastapi.concurrency import run_in_threadpool

@router.post("/process")
async def process_data(data: ProcessRequest):
    my_data = await service.get_my_data()
    # Executar função bloqueante em thread pool
    result = await run_in_threadpool(sync_client.make_request, data=my_data)
    return result
```

---

## Pydantic

### Custom Base Model

```python
from pydantic import BaseModel, ConfigDict

class BaseSchema(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,  # Permite ORM models
        str_strip_whitespace=True,
        validate_assignment=True,
    )
```

### BaseSettings por Domínio

```python
from pydantic_settings import BaseSettings

class DatabaseSettings(BaseSettings):
    host: str
    port: int = 5432
    user: str
    password: str
    
    class Config:
        env_prefix = "DB_"

class APISettings(BaseSettings):
    secret_key: str
    algorithm: str = "HS256"
    
    class Config:
        env_prefix = "API_"
```

### Field Constraints

```python
from pydantic import BaseModel, EmailStr, Field, AnyUrl

class UserCreate(BaseSchema):
    email: EmailStr
    password: str = Field(min_length=8, max_length=100)
    age: int = Field(ge=18, le=120)
    website: AnyUrl | None = None
```

### Serialização

```python
class PostResponse(BaseSchema):
    id: UUID4
    title: str
    content: str
    
    @model_serializer
    def ser_model(self) -> dict[str, Any]:
        """Return a dict which contains only serializable fields."""
        return {
            "id": str(self.id),
            "title": self.title,
            "content": self.content,
        }
```

---

## Dependencies

### Validação Complexa

```python
from fastapi import Depends, HTTPException

async def verify_token(token: str = Header(...)) -> dict:
    if not is_valid(token):
        raise HTTPException(401, "Invalid token")
    return decode_token(token)

@router.get("/protected")
async def protected_route(user: dict = Depends(verify_token)):
    return {"user": user}
```

### Chain Dependencies

```python
async def get_current_user(token: str = Depends(verify_token)) -> dict:
    user = await get_user_by_token(token)
    if not user:
        raise InvalidCredentials()
    return user

async def verify_owner(
    post_id: UUID4,
    current_user: dict = Depends(get_current_user)
) -> dict:
    post = await get_post(post_id)
    if post["owner_id"] != current_user["id"]:
        raise UserNotOwner()
    return post

@router.delete("/posts/{post_id}")
async def delete_post(post: dict = Depends(verify_owner)):
    await delete_post_by_id(post["id"])
    return {"deleted": True}
```

---

## Database

### SQL-First Approach

**Preferir operações no banco de dados:**

```python
# ✅ CORRETO: Agregação no banco
from sqlalchemy import desc, func, select, text
from sqlalchemy.sql.functions import coalesce

async def get_posts(creator_id: UUID4, *, limit: int = 10, offset: int = 0) -> list[dict[str, Any]]:
    select_query = (
        select(
            (
                posts.c.id,
                posts.c.slug,
                posts.c.title,
                func.json_build_object(
                    text("'id', profiles.id"),
                    text("'first_name', profiles.first_name"),
                    text("'last_name', profiles.last_name"),
                    text("'username', profiles.username"),
                ).label("creator"),
            )
        )
        .select_from(posts.join(profiles, posts.c.owner_id == profiles.c.id))
        .where(posts.c.owner_id == creator_id)
        .limit(limit)
        .offset(offset)
        .order_by(desc(coalesce(posts.c.updated_at, posts.c.published_at, posts.c.created_at)))
    )
    return await database.fetch_all(select_query)
```

### SQL Puro para Queries Complexas

**Quando usar SQL puro:**
- Queries muito grandes e complicadas
- Agregações complexas
- Performance crítica
- Lógica SQL específica do banco

**Referências obrigatórias (templates/snippets do cursor-multiagent-system):**
- **FastAPI/SQLAlchemy:** `core/templates/database/fastapi-repository-snippet.py` - Base repository com métodos `_query_one`, `_query_list`, `_query_scalar` para SQL puro seguro
- **Django:** `core/templates/database/django-sql-snippets.py` - Funções genéricas para SQL puro (Django) com proteção contra SQL injection

**Proteção contra SQL Injection:**
- SEMPRE usar parâmetros nomeados
- NUNCA concatenar strings SQL
- Usar `text()` do SQLAlchemy com parâmetros
- Validar inputs antes de executar

```python
from sqlalchemy import text
from typing import Dict, Any, Optional

# ✅ CORRETO: SQL puro com parâmetros seguros
async def complex_query(user_id: UUID, filters: Dict[str, Any]) -> List[Dict]:
    sql = text("""
        SELECT 
            p.id,
            p.title,
            COUNT(c.id) as comment_count,
            AVG(r.rating) as avg_rating
        FROM posts p
        LEFT JOIN comments c ON c.post_id = p.id
        LEFT JOIN ratings r ON r.post_id = p.id
        WHERE p.owner_id = :user_id
          AND p.status = :status
          AND p.created_at >= :start_date
        GROUP BY p.id, p.title
        HAVING COUNT(c.id) > :min_comments
        ORDER BY avg_rating DESC
        LIMIT :limit
    """)
    
    result = await database.fetch_all(
        sql,
        {
            "user_id": str(user_id),
            "status": filters.get("status", "published"),
            "start_date": filters.get("start_date"),
            "min_comments": filters.get("min_comments", 0),
            "limit": filters.get("limit", 10)
        }
    )
    return [dict(row) for row in result]
```

### Cache System para Consultas Repetitivas

**Quando usar cache:**
- Consultas muito acessadas que não mudam frequentemente
- Dados de sistemas externos (REST/RPC)
- Resultados de cálculos pesados
- Dados de referência (configurações, constantes)

**Referência obrigatória (template/snippet do cursor-multiagent-system):**
- `core/templates/cache/redis-cache-snippet.py` - Sistema de cache Redis genérico (Django e standalone)

**Padrão de uso:**
```python
from core.templates.cache.redis_cache_snippet import CacheSystem
import os

# Configurar cache (exemplo com variáveis de ambiente)
cache = CacheSystem(
    host=os.getenv("REDIS_HOST", "localhost"),
    port=int(os.getenv("REDIS_PORT", 6379)),
    password=os.getenv("REDIS_PASSWORD"),
    db=0
)

async def get_user_profile(user_id: UUID) -> Dict:
    cache_key = f"user_profile:{user_id}"
    
    # Tentar cache primeiro
    cached = cache.read(cache_key)
    if cached:
        return cached
    
    # Se não estiver em cache, buscar do banco
    profile = await database.fetch_one(
        "SELECT * FROM profiles WHERE id = :id",
        {"id": str(user_id)}
    )
    
    if profile:
        # Salvar no cache (expira em 1 hora)
        cache.save(cache_key, dict(profile), expiration=3600)
        return dict(profile)
    
    return None
```

---

## Migrations (Alembic)

### Regras

1. **Migrations devem ser estáticas e reversíveis**
2. **Nomes descritivos**: `2022-08-24_post_content_idx.py`
3. **Configurar template em alembic.ini**:
   ```ini
   file_template = %%(year)d-%%(month).2d-%%(day).2d_%%(slug)s
   ```

### PROIBIÇÃO CRÍTICA

**TODO E QUALQUER AGENTE ESTÁ PROIBIDO DE MEXER EM MIGRATIONS JÁ APLICADAS:**

- ❌ **NUNCA** alterar migrations já commitadas e aplicadas
- ❌ **NUNCA** editar arquivos de migration existentes
- ❌ **NUNCA** deletar migrations antigas
- ✅ **SEMPRE** criar nova migration para alterações
- ✅ **SEMPRE** testar migrations em ambiente de desenvolvimento primeiro

**Razão:** Alterar migrations anteriores não tem efeito no banco de dados já migrado. Isso vale para Django e FastAPI (Alembic).

**Processo correto:**
1. Se precisa alterar schema → criar NOVA migration
2. Se migration anterior está errada → criar migration de correção
3. Nunca editar arquivo de migration existente

---

## Testing

### Async Test Client

```python
from httpx import AsyncClient
from src.main import app

@pytest.mark.asyncio
async def test_create_post():
    async with AsyncClient(app=app, base_url="http://test") as client:
        resp = await client.post("/posts", json={"title": "Test"})
        assert resp.status_code == 201
```

---

## Checklist

- [ ] Estrutura por domínio implementada
- [ ] Async routes apenas para I/O não-bloqueante
- [ ] Pydantic V2 com custom base model
- [ ] BaseSettings separados por domínio
- [ ] Dependencies para validação complexa
- [ ] SQL-first approach para queries
- [ ] SQL puro com proteção contra injection (quando necessário)
- [ ] Cache system para consultas repetitivas (quando necessário)
- [ ] Migrations nunca alteradas após aplicadas
- [ ] Testes com async client
- [ ] Validações usando Field constraints e validators built-in
