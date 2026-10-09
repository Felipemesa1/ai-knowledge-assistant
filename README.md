# AI Knowledge Assistant - Bootstrap API

Primer incremento funcional del proyecto **AI Knowledge Assistant**: una API asíncrona construida con FastAPI, con datos validados con Pydantic, probada con pytest y versionada con Git/GitHub.

> **Importante:** este incremento **no integra ningún LLM**. La respuesta del asistente es generada por una implementación local de bootstrap (`provider = "bootstrap-local"`). El objetivo es dejar una base de software confiable antes de incorporar IA generativa en módulos posteriores.

## Tabla de contenido

1. [Tecnologías](#tecnologías)
2. [Requisitos](#requisitos)
3. [Instalación](#instalación)
4. [Activación del entorno virtual](#activación-del-entorno-virtual)
5. [Ejecución de la API](#ejecución-de-la-api)
6. [Endpoints](#endpoints)
7. [Ejecución de pruebas](#ejecución-de-pruebas)
8. [Demo de AsyncIO y HTTPX](#demo-de-asyncio-y-httpx)
9. [Arquitectura y estructura del proyecto](#arquitectura-y-estructura-del-proyecto)
10. [Flujo de Git](#flujo-de-git)
11. [Solución de problemas](#solución-de-problemas)
12. [Próximos pasos](#próximos-pasos)

## Tecnologías

| Tecnología | Uso |
|---|---|
| Python 3.12+ | Lenguaje |
| FastAPI | Framework web / API REST |
| Uvicorn | Servidor ASGI |
| Pydantic | Modelado y validación de datos |
| AsyncIO | Programación asíncrona |
| HTTPX | Cliente HTTP asíncrono |
| pytest + pytest-asyncio | Pruebas automatizadas |
| Git / GitHub | Control de versiones |

**Fuera de alcance en este módulo:** proveedores LLM (OpenAI, Anthropic, Gemini, Ollama), LangChain/LangGraph, PostgreSQL, Redis, bases vectoriales, Docker, RAG, agentes y memoria conversacional.

## Requisitos

- **Python 3.12 o superior**
- **Git**
- Una terminal (PowerShell en Windows, o bash/zsh en Linux/macOS)

Verifica las versiones instaladas:

```bash
python --version
git --version
```

## Instalación

### 1. Clonar el repositorio

```bash
git clone <URL_DEL_REPO>
cd ai-knowledge-assistant
```

### 2. Crear el entorno virtual

```bash
python -m venv .venv
```

### 3. Activar el entorno virtual

Consulta la sección [Activación del entorno virtual](#activación-del-entorno-virtual). Cuando esté activo verás `(.venv)` al inicio del prompt.

### 4. Instalar las dependencias

Las dependencias están declaradas en `pyproject.toml`. Instálalas junto con las de desarrollo (pruebas):

```bash
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
```

Para verificar la instalación:

```bash
python -m pip list
```

Deben aparecer, entre otros: `fastapi`, `uvicorn`, `pydantic`, `httpx`, `pytest` y `pytest-asyncio`.

## Activación del entorno virtual

**Windows (PowerShell):**

```powershell
.venv\Scripts\Activate.ps1
```

Si PowerShell muestra un error de permisos para ejecutar scripts:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

**Windows (CMD):**

```cmd
.venv\Scripts\activate.bat
```

**Linux / macOS:**

```bash
source .venv/bin/activate
```

Para desactivar el entorno en cualquier momento:

```bash
deactivate
```

## Ejecución de la API

Con el entorno virtual activo, desde la raíz del proyecto:

```bash
uvicorn app.main:app --reload
```

Debes ver en la terminal:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Application startup complete.
```

- API: http://127.0.0.1:8000
- Documentación Swagger (OpenAPI): http://127.0.0.1:8000/docs
- Documentación ReDoc: http://127.0.0.1:8000/redoc

Para detener el servidor presiona `Ctrl+C`.

Si el puerto 8000 está ocupado, usa otro:

```bash
uvicorn app.main:app --reload --port 8001
```

## Endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/health` | Estado de la aplicación |
| POST | `/api/v1/chat` | Envía una pregunta y recibe una respuesta local de bootstrap |
| GET | `/api/v1/info` | Información básica del proyecto |
| GET | `/docs` | Documentación interactiva Swagger |

### GET /health

Respuesta (HTTP 200):

```json
{
  "status": "ok"
}
```

### POST /api/v1/chat

Request:

```json
{
  "question": "¿Qué es FastAPI?"
}
```

Respuesta (HTTP 200):

```json
{
  "answer": "FastAPI es un framework web moderno de Python para construir APIs rápidas, tipadas y con documentación OpenAPI automática.",
  "provider": "bootstrap-local"
}
```

**Reglas de validación (Pydantic):**

- `question` es obligatoria.
- Longitud mínima: **3** caracteres.
- Longitud máxima: **2000** caracteres.
- Los espacios al inicio y al final se eliminan antes de validar.

Si la entrada no cumple, la API responde **HTTP 422 (Unprocessable Entity)** con el detalle del error. Ejemplo:

```json
{ "question": "ab" }
```

### GET /api/v1/info

Respuesta (HTTP 200):

```json
{
  "name": "AI Knowledge Assistant",
  "version": "0.1.0",
  "environment": "development",
  "llm_enabled": false
}
```

El campo `environment` toma el valor de la variable de entorno `APP_ENV` (por defecto `development`).

### Probar los endpoints manualmente

**Desde Swagger:** abre http://127.0.0.1:8000/docs, despliega un endpoint, pulsa **Try it out** y luego **Execute**.

**Desde PowerShell:**

```powershell
# Health
Invoke-RestMethod http://127.0.0.1:8000/health

# Chat con entrada válida
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/api/v1/chat `
  -ContentType "application/json" `
  -Body '{"question": "¿Qué es FastAPI?"}'

# Info
Invoke-RestMethod http://127.0.0.1:8000/api/v1/info
```

**Desde curl (Linux/macOS/Git Bash):**

```bash
curl http://127.0.0.1:8000/health

curl -X POST http://127.0.0.1:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{"question": "¿Qué es FastAPI?"}'

# Entrada inválida (debe responder 422)
curl -i -X POST http://127.0.0.1:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{"question": "ab"}'

curl http://127.0.0.1:8000/api/v1/info
```

## Ejecución de pruebas

Con el entorno virtual activo:

```bash
python -m pytest -q
```

El resultado esperado es que **todas las pruebas pasen** (sin fallos).

### Pruebas incluidas

| ID | Tipo | Descripción | Archivo |
|---|---|---|---|
| T-01 | Unitaria | El servicio responde correctamente a una pregunta conocida | `tests/unit/test_assistant_service.py` |
| T-02 | Integración | `GET /health` retorna HTTP 200 | `tests/integration/test_api.py` |
| T-03 | Integración | `POST /api/v1/chat` con pregunta válida retorna HTTP 200 | `tests/integration/test_api.py` |
| T-04 | Integración | `POST /api/v1/chat` con pregunta demasiado corta retorna HTTP 422 | `tests/integration/test_api.py` |
| T-05 | Integración | `GET /api/v1/info` retorna HTTP 200 y `llm_enabled = false` | `tests/integration/test_api.py` |
| Extra | Integración | Pregunta de más de 2000 caracteres retorna HTTP 422 | `tests/integration/test_api.py` |
| Extra | Integración | Request sin el campo `question` retorna HTTP 422 | `tests/integration/test_api.py` |
| Extra | Integración | `/docs` está disponible | `tests/integration/test_api.py` |

Las pruebas de integración invocan la aplicación FastAPI en memoria con `httpx.ASGITransport`, por lo que **no requieren un servidor en ejecución**.

## Demo de AsyncIO y HTTPX

El script `scripts/async_demo.py` envía varias preguntas al endpoint de chat de forma **concurrente** usando `httpx.AsyncClient` y `asyncio.gather`.

Por defecto llama a la aplicación en memoria (no necesita servidor):

```bash
python -m scripts.async_demo
```

Contra un servidor real (primero enciende la API en otra terminal):

```bash
python -m scripts.async_demo --url http://127.0.0.1:8000
```

## Arquitectura y estructura del proyecto

Flujo de una petición:

```
Cliente → FastAPI → Pydantic (valida) → Router → Servicio → Respuesta
```

Cada capa tiene una única responsabilidad:

| Capa | Carpeta | Responsabilidad |
|---|---|---|
| HTTP | `app/api/routes/` | Recibe la petición, delega y responde. No contiene lógica de negocio. |
| Datos | `app/schemas/` | Modelos Pydantic de entrada y salida, con sus restricciones. |
| Lógica | `app/services/` | Lógica del asistente, independiente del router. |

Estructura del repositorio:

```
ai-knowledge-assistant/
├── app/
│   ├── __init__.py
│   ├── main.py                  # Creación de la app y registro de routers
│   ├── api/
│   │   └── routes/
│   │       ├── health.py        # GET /health
│   │       ├── chat.py          # POST /api/v1/chat
│   │       └── info.py          # GET /api/v1/info
│   ├── schemas/
│   │   ├── chat.py              # ChatRequest, ChatResponse
│   │   └── info.py              # InfoResponse
│   └── services/
│       └── assistant.py         # AssistantService (lógica local de bootstrap)
├── scripts/
│   └── async_demo.py            # Demo de AsyncIO + HTTPX
├── tests/
│   ├── unit/
│   │   └── test_assistant_service.py
│   └── integration/
│       └── test_api.py
├── .gitignore
├── pyproject.toml
└── README.md
```

## Flujo de Git

El trabajo se realizó con control de versiones real:

- Rama principal: `main`
- Rama de funcionalidad: `feat/bootstrap-api`
- Commits pequeños y coherentes (convención `tipo: descripción`):

```
chore: initialize AI knowledge assistant project
feat: add health endpoint
feat: add bootstrap chat endpoint
feat: add application info endpoint
feat: add async HTTPX demo script
test: add bootstrap API test suite
docs: complete README with setup and usage
```

- Publicación en GitHub y Pull Request de `feat/bootstrap-api` hacia `main`.

Para ver la historia:

```bash
git log --oneline --graph
```

## Solución de problemas

| Problema | Causa probable | Solución |
|---|---|---|
| `ModuleNotFoundError: No module named 'app'` | El entorno no está activo o faltan los `__init__.py` | Activa `.venv`, revisa que existan los `__init__.py` y ejecuta los comandos desde la raíz del proyecto |
| `{"detail":"Not Found"}` en una ruta | Hay otro servidor usando el mismo puerto, o el servidor tiene código viejo | Apaga todos los servidores (`Ctrl+C`), libera el puerto o usa `--port 8001`, y reinicia |
| No se puede activar `.venv` en PowerShell | Política de ejecución restringida | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| Pytest no encuentra las pruebas o falla con `async` | Dependencias de desarrollo sin instalar | `python -m pip install -e ".[dev]"` |
| `address already in use` al iniciar uvicorn | Puerto ocupado | Usa otro puerto con `--port` o cierra el proceso que lo ocupa |

## Seguridad

- El repositorio **no contiene** secretos, tokens ni credenciales.
- `.gitignore` excluye `.venv`, cachés, archivos `.env` y artefactos locales.

## Próximos pasos

En los siguientes módulos se conectará un LLM real. Lo que cambiará es la implementación interna de `AssistantService` (y el valor de `provider`). Deben permanecer estables:

- Los contratos HTTP (`/api/v1/chat`, request y response).
- Los schemas de Pydantic.
- La separación entre router, schemas y servicio.
- La suite de pruebas.
