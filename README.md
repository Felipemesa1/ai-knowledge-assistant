# AI Knowledge Assistant - Bootstrap API

Primer incremento funcional: API asíncrona con FastAPI, validada con Pydantic
y probada con pytest. **No integra ningún LLM todavía.**

## Requisitos
- Python 3.12+
- Git

## Instalación
```bash
git clone <URL_DEL_REPO>
cd ai-knowledge-assistant
python -m venv .venv
# Windows: .venv\Scripts\Activate.ps1   |   Linux/macOS: source .venv/bin/activate
python -m pip install -e ".[dev]"
```

## Ejecutar la API
```bash
uvicorn app.main:app --reload
```
- Documentación Swagger: http://127.0.0.1:8000/docs
- Health: GET /health
- Chat: POST /api/v1/chat   `{"question": "¿Qué es FastAPI?"}`
- Info: GET /api/v1/info

## Ejecutar pruebas
```bash
python -m pytest -q
```

## Demo AsyncIO + HTTPX
```bash
python -m scripts.async_demo
```

## Estructura
- `app/api/routes/`: capa HTTP (routers)
- `app/schemas/`: modelos Pydantic
- `app/services/`: lógica del asistente
- `tests/unit` y `tests/integration`
