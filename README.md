# AI Knowledge Assistant - Bootstrap API

**Curso:** Ingeniería de Sistemas de IA | **Módulo 0** - Preparación del entorno de Ingeniería de IA
**Proyecto:** Mini-proyecto evaluable - Primer incremento funcional
**Modalidad:** Individual | **Autor:** _(escribe aquí tu nombre)_

> Este incremento **no integra ningún LLM**. La respuesta del asistente es generada por una implementación local de bootstrap (`provider = "bootstrap-local"`). El objetivo es demostrar que la aplicación tiene contratos claros, separación de responsabilidades, procesamiento asíncrono, pruebas automatizadas y un flujo profesional de Git.

## Tabla de contenido

1. [Contexto y objetivo](#1-contexto-y-objetivo)
2. [Tecnologías y restricciones](#2-tecnologías-y-restricciones)
3. [Requisitos](#3-requisitos)
4. [Instalación](#4-instalación)
5. [Activación del entorno virtual](#5-activación-del-entorno-virtual)
6. [Ejecución de la API](#6-ejecución-de-la-api)
7. [Requerimientos funcionales y contratos HTTP](#7-requerimientos-funcionales-y-contratos-http)
8. [Arquitectura y estructura del proyecto](#8-arquitectura-y-estructura-del-proyecto)
9. [Ejecución de pruebas](#9-ejecución-de-pruebas)
10. [Demo de AsyncIO y HTTPX](#10-demo-de-asyncio-y-httpx)
11. [Flujo de Git y GitHub](#11-flujo-de-git-y-github)
12. [Paso a paso realizado](#12-paso-a-paso-realizado)
13. [Entregables y evidencias](#13-entregables-y-evidencias)
14. [Preguntas de sustentación](#14-preguntas-de-sustentación)
15. [Solución de problemas](#15-solución-de-problemas)
16. [Checklist final](#16-checklist-final)

---

## 1. Contexto y objetivo

Una organización desea iniciar el desarrollo de un asistente empresarial de conocimiento. El equipo aún no ha seleccionado un proveedor de modelos de lenguaje, pero necesita una base de software confiable antes de integrar IA generativa.

Este proyecto es el **primer incremento funcional** de AI Knowledge Assistant: una API asíncrona, validada, probada y versionada correctamente, que podrá mantenerse cuando en módulos posteriores se incorporen LLMs, RAG, agentes y observabilidad.

**Resultado de aprendizaje demostrado:**

- Configurar un proyecto Python con entorno virtual y dependencias.
- Construir una API REST con FastAPI.
- Modelar y validar datos de entrada y salida con Pydantic.
- Utilizar `async/await` en los puntos apropiados.
- Consumir la API con HTTPX de forma asíncrona.
- Crear pruebas unitarias y de integración con pytest.
- Gestionar el código con Git mediante commits y ramas coherentes.
- Publicar el repositorio en GitHub y documentar su ejecución.

## 2. Tecnologías y restricciones

**Tecnologías utilizadas:**

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

**Tecnologías NO utilizadas (excluidas en este módulo):** OpenAI, Anthropic, Gemini, Ollama u otro LLM; LangChain o LangGraph; PostgreSQL, Redis o bases vectoriales; Docker; RAG, agentes o memoria conversacional.

## 3. Requisitos

- **Python 3.12 o superior**
- **Git**
- Cuenta de **GitHub**
- Una terminal (PowerShell en Windows, o bash/zsh en Linux/macOS)

```bash
python --version   # debe ser 3.12 o superior
git --version
```

## 4. Instalación

**1. Clonar el repositorio**

```bash
git clone <URL_DEL_REPO>
cd ai-knowledge-assistant
```

**2. Crear el entorno virtual**

```bash
python -m venv .venv
```

**3. Activar el entorno virtual** (ver sección 5). Cuando esté activo se ve `(.venv)` al inicio del prompt.

**4. Instalar las dependencias** (declaradas en `pyproject.toml`), incluyendo las de desarrollo:

```bash
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
```

**5. Verificar la instalación**

```bash
python -m pip list
```

Deben aparecer: `fastapi`, `uvicorn`, `pydantic`, `httpx`, `pytest` y `pytest-asyncio`.

## 5. Activación del entorno virtual

**Windows (PowerShell):**

```powershell
.venv\Scripts\Activate.ps1
```

Si PowerShell muestra un error de permisos:

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

Para desactivarlo: `deactivate`.

## 6. Ejecución de la API

Con el entorno virtual activo, desde la raíz del proyecto:

```bash
uvicorn app.main:app --reload
```

Debe aparecer en la terminal:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Application startup complete.
```

- API: http://127.0.0.1:8000
- Swagger (OpenAPI): http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc

Para detener el servidor: `Ctrl+C`. Si el puerto 8000 está ocupado: `uvicorn app.main:app --reload --port 8001`.

### Pruebas manuales

**Desde Swagger:** abrir `/docs`, desplegar un endpoint, pulsar **Try it out** y **Execute**.

**Desde PowerShell:**

```powershell
Invoke-RestMethod http://127.0.0.1:8000/health

Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/api/v1/chat `
  -ContentType "application/json" `
  -Body '{"question": "¿Qué es FastAPI?"}'

Invoke-RestMethod http://127.0.0.1:8000/api/v1/info
```

**Desde curl (Linux/macOS/Git Bash):**

```bash
curl http://127.0.0.1:8000/health

curl -X POST http://127.0.0.1:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{"question": "¿Qué es FastAPI?"}'

# Entrada inválida: debe responder 422
curl -i -X POST http://127.0.0.1:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -d '{"question": "ab"}'

curl http://127.0.0.1:8000/api/v1/info
```

## 7. Requerimientos funcionales y contratos HTTP

### Requerimientos funcionales

| ID | Requerimiento | Estado |
|---|---|---|
| RF-01 | Endpoint `GET /health` | Cumplido |
| RF-02 | Endpoint `POST /api/v1/chat` | Cumplido |
| RF-03 | El chat recibe una pregunta en formato JSON | Cumplido |
| RF-04 | La pregunta tiene mínimo 3 y máximo 2000 caracteres | Cumplido |
| RF-05 | Entradas inválidas rechazadas mediante validación de Pydantic | Cumplido |
| RF-06 | El chat retorna los campos `answer` y `provider` | Cumplido |
| RF-07 | `provider` identifica una implementación local de bootstrap (`bootstrap-local`) | Cumplido |
| RF-08 | La respuesta la genera una capa de servicio independiente del router | Cumplido |
| RF-09 | Documentación OpenAPI accesible desde `/docs` | Cumplido |
| RF-10 | Endpoint `GET /api/v1/info` con información básica del proyecto | Cumplido |

### GET /health

Respuesta (HTTP 200):

```json
{ "status": "ok" }
```

### POST /api/v1/chat

Request:

```json
{ "question": "¿Qué es FastAPI?" }
```

Respuesta (HTTP 200):

```json
{
  "answer": "FastAPI es un framework web moderno de Python para construir APIs rápidas, tipadas y con documentación OpenAPI automática.",
  "provider": "bootstrap-local"
}
```

**Validación (Pydantic):** `question` es obligatoria, con mínimo **3** y máximo **2000** caracteres (se eliminan los espacios al inicio y al final). Si no se cumple, la API responde **HTTP 422 (Unprocessable Entity)** antes de ejecutar la lógica del servicio.

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

`environment` toma el valor de la variable `APP_ENV` (por defecto `development`).

## 8. Arquitectura y estructura del proyecto

Flujo de una petición:

```
Cliente → FastAPI → Pydantic (valida) → Router → Servicio → Respuesta
```

| Capa | Carpeta | Responsabilidad |
|---|---|---|
| HTTP | `app/api/routes/` | Recibe la petición, delega y responde. No contiene lógica. |
| Datos | `app/schemas/` | Modelos Pydantic de entrada y salida, con sus restricciones. |
| Lógica | `app/services/` | Lógica del asistente, independiente del router. |

Estructura del repositorio:

```
ai-knowledge-assistant/
├── app/
│   ├── __init__.py
│   ├── main.py                  # Crea la app y registra los routers
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes/
│   │       ├── __init__.py
│   │       ├── health.py        # GET /health
│   │       ├── chat.py          # POST /api/v1/chat
│   │       └── info.py          # GET /api/v1/info
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── chat.py              # ChatRequest, ChatResponse
│   │   └── info.py              # InfoResponse
│   └── services/
│       ├── __init__.py
│       └── assistant.py         # AssistantService (lógica local de bootstrap)
├── scripts/
│   ├── __init__.py
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

**Requerimientos técnicos cumplidos:**

- Entorno virtual `.venv` (no versionado).
- Dependencias declaradas en `pyproject.toml`.
- Endpoints con `async def`.
- La lógica del asistente no está en el endpoint: vive en `AssistantService`, que el router recibe mediante `Depends`.
- Demostración de AsyncIO y HTTPX asíncrono en `scripts/`.
- Pruebas de integración que invocan la app FastAPI sin servidor externo (`httpx.ASGITransport`).
- `.gitignore` excluye `.venv`, cachés, archivos `.env` y artefactos locales.
- El repositorio no contiene secretos, tokens ni credenciales.

## 9. Ejecución de pruebas

Con el entorno virtual activo:

```bash
python -m pytest -q
```

**Gate de calidad:** el comando debe finalizar sin pruebas fallidas.

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
| Extra | Unitaria | Pregunta desconocida responde con `provider = bootstrap-local` | `tests/unit/test_assistant_service.py` |

**Diferencia entre ambos tipos:** la prueba unitaria verifica una pieza aislada (el servicio); la de integración verifica varias capas juntas (HTTP, Pydantic, router y servicio).

## 10. Demo de AsyncIO y HTTPX

El script `scripts/async_demo.py` envía varias preguntas al endpoint de chat de forma **concurrente** con `httpx.AsyncClient` y `asyncio.gather`, y mide el tiempo total.

Por defecto llama a la aplicación en memoria (no necesita servidor):

```bash
python -m scripts.async_demo
```

Contra un servidor real (primero enciende la API en otra terminal):

```bash
python -m scripts.async_demo --url http://127.0.0.1:8000
```

## 11. Flujo de Git y GitHub

- Repositorio inicializado con la rama principal `main`.
- Rama de funcionalidad: `feat/bootstrap-api`.
- Commits pequeños y coherentes con la convención `tipo: descripción`:

```
chore: initialize AI knowledge assistant project
feat: add health endpoint
feat: add bootstrap chat endpoint
feat: add application info endpoint
feat: add async HTTPX demo script
test: add bootstrap API test suite
docs: complete README with setup and usage
```

- Rama publicada en GitHub y Pull Request de `feat/bootstrap-api` hacia `main`.

```bash
git log --oneline --graph
```

**Enlaces:**

- Repositorio: `<URL_DEL_REPOSITORIO>`
- Pull Request: `<URL_DEL_PULL_REQUEST>`

## 12. Paso a paso realizado

### Paso 1: Proyecto y entorno virtual

```bash
mkdir ai-knowledge-assistant
cd ai-knowledge-assistant
git init -b main
python -m venv .venv
```

Activación del entorno (ver sección 5) y creación de la estructura en PowerShell:

```powershell
New-Item -ItemType Directory -Force app/api/routes, app/schemas, app/services, scripts, tests/unit, tests/integration
New-Item -ItemType File -Force app/__init__.py, app/api/__init__.py, app/api/routes/__init__.py, app/schemas/__init__.py, app/services/__init__.py, scripts/__init__.py
```

### Paso 2: `pyproject.toml`

Archivo de configuración del proyecto: nombre, versión, versión mínima de Python, dependencias (`fastapi`, `uvicorn[standard]`, `pydantic`, `httpx`), dependencias de desarrollo (`pytest`, `pytest-asyncio`) y configuración de pytest (`asyncio_mode = "auto"`, `testpaths`, `pythonpath`).

```bash
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
```

### Paso 3: `.gitignore`

Excluye `.venv/`, `venv/`, `__pycache__/`, `*.py[cod]`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`, `*.egg-info/`, `build/`, `dist/`, `.env`, `.env.*`, `.vscode/`, `.idea/`, `.DS_Store`, `htmlcov/` y `.coverage`.

### Paso 4: Primer commit y rama de trabajo

```bash
git add .
git commit -m "chore: initialize AI knowledge assistant project"
git switch -c feat/bootstrap-api
```

### Paso 5: Endpoint `/health`

- `app/api/routes/health.py`: router con `GET /health` asíncrono.
- `app/main.py`: crea la aplicación FastAPI y registra el router.

```bash
git add .
git commit -m "feat: add health endpoint"
```

### Paso 6: Endpoint de chat (schemas + servicio + router)

- `app/schemas/chat.py`: `ChatRequest` (con `min_length=3`, `max_length=2000`) y `ChatResponse`.
- `app/services/assistant.py`: `AssistantService` con respuestas locales y `provider = "bootstrap-local"`.
- `app/api/routes/chat.py`: router que valida con Pydantic, delega al servicio con `Depends` y responde.
- `app/main.py`: registro del router con el prefijo `/api/v1`.

```bash
git add .
git commit -m "feat: add bootstrap chat endpoint"
```

### Paso 7: Endpoint `/api/v1/info`

- `app/schemas/info.py`: `InfoResponse`.
- `app/api/routes/info.py`: `GET /info` con `llm_enabled = false`.
- `app/main.py`: registro del router con el prefijo `/api/v1`.

```bash
git add .
git commit -m "feat: add application info endpoint"
```

### Paso 8: Script AsyncIO + HTTPX

- `scripts/async_demo.py`: peticiones concurrentes con `asyncio.gather` y `httpx.AsyncClient`.

```bash
python -m scripts.async_demo
git add .
git commit -m "feat: add async HTTPX demo script"
```

### Paso 9: Pruebas con pytest

- `tests/unit/test_assistant_service.py` (T-01).
- `tests/integration/test_api.py` (T-02 a T-05 y pruebas extra).

```bash
python -m pytest -q
git add .
git commit -m "test: add bootstrap API test suite"
```

### Paso 10: README

Este documento, con requisitos, instalación, activación del entorno, ejecución de la API, ejecución de pruebas y evidencias.

```bash
git add .
git commit -m "docs: complete README with setup and usage"
```

### Paso 11: Publicación en GitHub y Pull Request

```bash
git remote add origin https://github.com/TU_USUARIO/ai-knowledge-assistant.git
git push -u origin main
git push -u origin feat/bootstrap-api
```

En GitHub se creó el Pull Request de `feat/bootstrap-api` hacia `main`.

## 13. Entregables y evidencias

| Entregable | Estado |
|---|---|
| URL del repositorio de GitHub accesible al instructor | Entregado |
| Código fuente completo del proyecto | Entregado |
| `README.md` con requisitos, instalación, activación del entorno, ejecución de la API y de pruebas | Entregado |
| Evidencia de la documentación Swagger en `/docs` | Entregado |
| Evidencia del resultado de `python -m pytest -q` | Entregado |
| Historia de commits y rama de trabajo utilizada | Entregado |
| Pull Request creado para integrar el incremento funcional | Entregado |

**Evidencias a presentar en la sustentación (ejecutar y explicar):**

- Creación/activación del entorno virtual y dependencias instaladas.
- Ejecución de FastAPI y acceso a `/docs`.
- Prueba manual de `GET /health`.
- Prueba manual de `POST /api/v1/chat` con una entrada válida.
- Prueba manual de una entrada inválida y explicación del HTTP 422.
- Ejecución completa de pytest.
- Explicación del flujo request → Pydantic → router → service → response.
- Explicación de la diferencia entre Git y GitHub, y de la rama y commits utilizados.

**Capturas:** _(agrega aquí tus imágenes)_

- Swagger `/docs` con los tres endpoints.
- Resultado de `python -m pytest -q` en verde.
- Salida de `git log --oneline`.

## 14. Preguntas de sustentación

**1. ¿Qué responsabilidad tiene FastAPI en esta solución?**
Recibe las peticiones HTTP, las enruta al endpoint correspondiente, valida los datos con Pydantic y genera automáticamente la documentación OpenAPI en `/docs`.

**2. ¿Qué problema resuelve Pydantic y por qué un request inválido retorna 422?**
Pydantic define y valida la forma y las restricciones de los datos (aquí, que `question` exista y tenga entre 3 y 2000 caracteres). Si el request no cumple el esquema, FastAPI responde 422 (Unprocessable Entity) antes de ejecutar la lógica del servicio: el JSON está bien formado, pero su contenido no es válido.

**3. ¿Qué diferencia existe entre `def` y `async def` en este contexto?**
`async def` define una corrutina que puede ceder el control con `await` mientras espera operaciones de entrada/salida, permitiendo que el servidor atienda otras peticiones. Una `def` normal se ejecuta de forma bloqueante.

**4. ¿Qué hace `await` y por qué es importante en aplicaciones que consumen servicios externos?**
Suspende la corrutina hasta que termine una operación sin bloquear el event loop. Es clave al llamar servicios externos (como un LLM o una API), que tardan en responder: mientras se espera, la aplicación puede seguir atendiendo otras peticiones.

**5. ¿Qué diferencia existe entre una prueba unitaria y una de integración?**
La unitaria prueba una pieza aislada (el servicio). La de integración prueba varias capas trabajando juntas (HTTP, Pydantic, router y servicio) invocando la aplicación completa.

**6. ¿Por qué el servicio del asistente está separado del router?**
Para poder cambiar la lógica (por ejemplo, reemplazarla por un LLM) sin tocar la capa HTTP, y para poder probar la lógica de forma independiente. El router solo recibe, delega y responde.

**7. ¿Qué diferencia existe entre Git y GitHub?**
Git es el sistema de control de versiones que funciona de forma local. GitHub es una plataforma en la nube que aloja repositorios Git y ofrece colaboración, ramas remotas y Pull Requests.

**8. ¿Qué cambiará en el próximo módulo cuando se conecte un LLM real y qué debería permanecer estable?**
Cambiará la implementación interna de `AssistantService` y el valor de `provider`. Deben permanecer estables los contratos HTTP (`/api/v1/chat`, request y response), los schemas, la separación de capas y las pruebas.

## 15. Solución de problemas

| Problema | Causa probable | Solución |
|---|---|---|
| `touch` o `mkdir -p` no funcionan en PowerShell | Son comandos de Linux/macOS | Usar `New-Item` como se muestra en el Paso 1 |
| `feat/bootstrap-api` no se reconoce como cmdlet | Se escribió solo el nombre de la rama | Usar `git switch -c feat/bootstrap-api` |
| `{"detail":"Not Found"}` en una ruta | Otro servidor viejo ocupa el mismo puerto, o el servidor tiene código antiguo | Apagar los servidores (`Ctrl+C`), liberar el puerto o usar `--port 8001`, y reiniciar |
| `/docs` muestra una descripción distinta a la del proyecto | Se está consultando otro servidor | Verificar el puerto y reiniciar |
| `Error: Got unexpected extra arguments` | Se pegó la salida de la terminal como si fuera un comando | Escribir solo `uvicorn app.main:app --reload` |
| `ModuleNotFoundError: No module named 'app'` | Entorno inactivo o faltan `__init__.py` | Activar `.venv`, revisar los `__init__.py` y ejecutar desde la raíz |
| No se puede activar `.venv` en PowerShell | Política de ejecución restringida | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| Pytest falla con pruebas `async` | Dependencias de desarrollo sin instalar | `python -m pip install -e ".[dev]"` |

## 16. Checklist final

**Configuración**

- [x] Python 3.12+ configurado.
- [x] `.venv` creado y no versionado.
- [x] `pyproject.toml` contiene las dependencias.
- [x] `.gitignore` excluye `.venv`, cachés, `.env` y artefactos locales.
- [x] No hay secretos en el repositorio.

**API y validación**

- [x] `GET /health` funciona.
- [x] `POST /api/v1/chat` funciona.
- [x] `GET /api/v1/info` funciona.
- [x] Pydantic valida pregunta mínima y máxima.
- [x] `/docs` funciona.

**Arquitectura y asincronía**

- [x] El servicio está separado del router.
- [x] Existe evidencia de `async/await`.
- [x] HTTPX se utiliza de manera asíncrona.

**Pruebas**

- [x] Existen unit tests.
- [x] Existen integration tests.
- [x] `python -m pytest -q` pasa completamente.

**Git y GitHub**

- [x] Existe rama de funcionalidad (`feat/bootstrap-api`).
- [x] Los commits son claros y coherentes.
- [x] El repositorio fue publicado en GitHub.
- [x] Existe Pull Request o evidencia equivalente.

**Documentación**

- [x] README permite reproducir la instalación.

**Definición de terminado:** la aplicación ejecuta, `/docs` está disponible, los endpoints cumplen sus contratos y la suite completa de pruebas está en verde.
