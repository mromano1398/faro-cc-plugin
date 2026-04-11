# FastAPI Patterns — Faro Python

Pattern completi per API FastAPI con SQLAlchemy, Alembic, pytest e Docker.

## SETUP

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install --upgrade pip
```

## PYPROJECT.TOML

```toml
[project]
name = "faro-api"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi[standard]>=0.115.0",
    "uvicorn[standard]>=0.30.0",
    "pydantic>=2.9.0",
    "pydantic-settings>=2.5.0",
    "sqlalchemy>=2.0.35",
    "asyncpg>=0.29.0",
    "alembic>=1.13.3",
    "python-jose[cryptography]>=3.3.0",
    "passlib[bcrypt]>=1.7.4",
    "python-multipart>=0.0.10",
    "httpx>=0.27.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.3.0",
    "pytest-asyncio>=0.24.0",
    "pytest-cov>=5.0.0",
    "ruff>=0.6.0",
    "mypy>=1.11.0",
]

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N", "UP", "B", "A", "C4", "T20"]

[tool.mypy]
strict = true
python_version = "3.11"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

## STRUTTURA PROGETTO

```
faro-api/
├── pyproject.toml
├── alembic.ini
├── .env
├── Dockerfile
├── docker-compose.yml
├── src/
│   └── faro_api/
│       ├── __init__.py
│       ├── main.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── deps.py
│       │   ├── v1/
│       │   │   ├── __init__.py
│       │   │   ├── auth.py
│       │   │   ├── users.py
│       │   │   └── progetti.py
│       ├── core/
│       │   ├── config.py
│       │   ├── security.py
│       │   └── logging.py
│       ├── db/
│       │   ├── base.py
│       │   └── session.py
│       ├── models/
│       │   ├── user.py
│       │   └── progetto.py
│       ├── schemas/
│       │   ├── user.py
│       │   └── progetto.py
│       └── services/
│           ├── user_service.py
│           └── progetto_service.py
├── migrations/
│   ├── env.py
│   └── versions/
└── tests/
    ├── conftest.py
    ├── test_auth.py
    └── test_progetti.py
```

## CONFIG — pydantic-settings

`src/faro_api/core/config.py`:
```python
from functools import lru_cache
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", case_sensitive=True)

    app_name: str = "Faro API"
    debug: bool = False

    database_url: str
    database_url_test: str | None = None

    jwt_secret: str
    jwt_algorithm: str = "HS256"
    jwt_expire_minutes: int = 60 * 24  # 24h

    cors_origins: list[str] = ["http://localhost:3000"]

    sentry_dsn: str | None = None


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

## MAIN.PY

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from faro_api.api.v1 import auth, users, progetti
from faro_api.core.config import get_settings

settings = get_settings()


@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    yield
    # Shutdown
    pass


app = FastAPI(
    title=settings.app_name,
    debug=settings.debug,
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.cors_origins,
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=["Authorization", "Content-Type"],
)

app.include_router(auth.router, prefix="/api/v1/auth", tags=["auth"])
app.include_router(users.router, prefix="/api/v1/users", tags=["users"])
app.include_router(progetti.router, prefix="/api/v1/progetti", tags=["progetti"])


@app.get("/health")
async def health():
    return {"status": "ok"}
```

## SQLALCHEMY 2.0 ASYNC

`src/faro_api/db/base.py`:
```python
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import MetaData

naming_convention = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=naming_convention)
```

`src/faro_api/db/session.py`:
```python
from sqlalchemy.ext.asyncio import (
    AsyncSession,
    async_sessionmaker,
    create_async_engine,
)
from faro_api.core.config import get_settings

settings = get_settings()

engine = create_async_engine(
    settings.database_url,
    echo=settings.debug,
    pool_pre_ping=True,
    pool_size=10,
    max_overflow=20,
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,  # OBBLIGATORIO per async
    autoflush=False,
)


async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

## MODELS

`src/faro_api/models/user.py`:
```python
from datetime import datetime
from uuid import UUID, uuid4
from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column
from faro_api.db.base import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[UUID] = mapped_column(primary_key=True, default=uuid4)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True)
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
    )
```

## SCHEMAS

`src/faro_api/schemas/user.py`:
```python
from datetime import datetime
from uuid import UUID
from pydantic import BaseModel, EmailStr, ConfigDict, Field


class UserBase(BaseModel):
    email: EmailStr


class UserCreate(UserBase):
    password: str = Field(min_length=12)


class UserRead(UserBase):
    model_config = ConfigDict(from_attributes=True)
    id: UUID
    is_active: bool
    created_at: datetime


class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

## AUTH JWT

`src/faro_api/core/security.py`:
```python
from datetime import datetime, timedelta, timezone
from jose import jwt, JWTError
from passlib.context import CryptContext
from faro_api.core.config import get_settings

settings = get_settings()
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def hash_password(password: str) -> str:
    return pwd_context.hash(password)


def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)


def create_access_token(sub: str) -> str:
    expire = datetime.now(timezone.utc) + timedelta(minutes=settings.jwt_expire_minutes)
    payload = {"sub": sub, "exp": expire}
    return jwt.encode(payload, settings.jwt_secret, algorithm=settings.jwt_algorithm)


def decode_token(token: str) -> str:
    try:
        payload = jwt.decode(token, settings.jwt_secret, algorithms=[settings.jwt_algorithm])
        return payload["sub"]
    except JWTError as e:
        raise ValueError("Invalid token") from e
```

## DEPENDENCIES

`src/faro_api/api/deps.py`:
```python
from typing import Annotated
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

from faro_api.db.session import get_db
from faro_api.models.user import User
from faro_api.core.security import decode_token

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")


async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: Annotated[AsyncSession, Depends(get_db)],
) -> User:
    try:
        user_id = decode_token(token)
    except ValueError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token non valido",
            headers={"WWW-Authenticate": "Bearer"},
        )

    result = await db.execute(select(User).where(User.id == user_id))
    user = result.scalar_one_or_none()
    if not user or not user.is_active:
        raise HTTPException(status_code=401, detail="Utente non trovato o disattivato")
    return user


CurrentUser = Annotated[User, Depends(get_current_user)]
DbSession = Annotated[AsyncSession, Depends(get_db)]
```

## ENDPOINT PATTERN

`src/faro_api/api/v1/progetti.py`:
```python
from uuid import UUID
from fastapi import APIRouter, HTTPException, status
from sqlalchemy import select

from faro_api.api.deps import CurrentUser, DbSession
from faro_api.models.progetto import Progetto
from faro_api.schemas.progetto import ProgettoCreate, ProgettoRead

router = APIRouter()


@router.get("/", response_model=list[ProgettoRead])
async def list_progetti(user: CurrentUser, db: DbSession):
    result = await db.execute(
        select(Progetto).where(Progetto.user_id == user.id).order_by(Progetto.created_at.desc())
    )
    return result.scalars().all()


@router.post("/", response_model=ProgettoRead, status_code=status.HTTP_201_CREATED)
async def create_progetto(data: ProgettoCreate, user: CurrentUser, db: DbSession):
    progetto = Progetto(**data.model_dump(), user_id=user.id)
    db.add(progetto)
    await db.flush()
    await db.refresh(progetto)
    return progetto


@router.delete("/{progetto_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_progetto(progetto_id: UUID, user: CurrentUser, db: DbSession):
    result = await db.execute(
        select(Progetto).where(Progetto.id == progetto_id, Progetto.user_id == user.id)
    )
    progetto = result.scalar_one_or_none()
    if not progetto:
        raise HTTPException(status_code=404, detail="Progetto non trovato")
    await db.delete(progetto)
```

## ALEMBIC — Setup

```bash
alembic init -t async migrations
```

`migrations/env.py` (aggiungi):
```python
from faro_api.db.base import Base
from faro_api.core.config import get_settings
from faro_api.models import user, progetto  # noqa

target_metadata = Base.metadata
config.set_main_option("sqlalchemy.url", get_settings().database_url)
```

Comandi:
```bash
alembic revision --autogenerate -m "add progetti"
alembic upgrade head
alembic downgrade -1
```

## TEST — pytest + rollback

`tests/conftest.py`:
```python
import asyncio
import pytest
import pytest_asyncio
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from httpx import AsyncClient, ASGITransport

from faro_api.main import app
from faro_api.db.session import get_db
from faro_api.db.base import Base
from faro_api.core.config import get_settings

settings = get_settings()


@pytest_asyncio.fixture
async def engine():
    eng = create_async_engine(settings.database_url_test, echo=False)
    async with eng.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield eng
    await eng.dispose()


@pytest_asyncio.fixture
async def db_session(engine) -> AsyncSession:
    async with engine.connect() as conn:
        trans = await conn.begin()
        session_maker = async_sessionmaker(bind=conn, expire_on_commit=False)
        async with session_maker() as session:
            yield session
        await trans.rollback()


@pytest_asyncio.fixture
async def client(db_session):
    async def override_get_db():
        yield db_session

    app.dependency_overrides[get_db] = override_get_db
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as c:
        yield c
    app.dependency_overrides.clear()
```

`tests/test_progetti.py`:
```python
import pytest


@pytest.mark.asyncio
async def test_create_progetto(client, auth_headers):
    response = await client.post(
        "/api/v1/progetti/",
        json={"nome": "Test"},
        headers=auth_headers,
    )
    assert response.status_code == 201
    assert response.json()["nome"] == "Test"


@pytest.mark.asyncio
async def test_cross_user_isolation(client, auth_headers_user_a, auth_headers_user_b):
    # User A crea
    r = await client.post("/api/v1/progetti/", json={"nome": "A's"}, headers=auth_headers_user_a)
    pid = r.json()["id"]

    # User B non vede
    r = await client.get("/api/v1/progetti/", headers=auth_headers_user_b)
    assert pid not in [p["id"] for p in r.json()]

    # User B non può cancellare
    r = await client.delete(f"/api/v1/progetti/{pid}", headers=auth_headers_user_b)
    assert r.status_code == 404
```

## DOCKERFILE

```dockerfile
FROM python:3.12-slim AS base

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential libpq-dev curl \
    && rm -rf /var/lib/apt/lists/*

COPY pyproject.toml ./
RUN pip install --upgrade pip && pip install -e .

COPY src/ ./src/
COPY migrations/ ./migrations/
COPY alembic.ini .

# Utente non-root
RUN useradd --create-home --shell /bin/bash faro
USER faro

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "faro_api.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

## DOCKER-COMPOSE

```yaml
version: '3.9'

services:
  api:
    build: .
    ports: ["8000:8000"]
    environment:
      DATABASE_URL: postgresql+asyncpg://faro:faro@db:5432/faro
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: faro
      POSTGRES_PASSWORD: faro
      POSTGRES_DB: faro
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "faro"]
      interval: 5s

volumes:
  pgdata:
```

## COMANDI UTILI

```bash
# Dev server
uvicorn faro_api.main:app --reload

# Lint + type check
ruff check src/ tests/
ruff format src/ tests/
mypy src/

# Test
pytest -v
pytest --cov=src/faro_api --cov-report=html

# Migrations
alembic revision --autogenerate -m "descrizione"
alembic upgrade head
```
