# Docker y Contenedores para Despliegue Ágil

Práctica de la Semana 14 — Construcción de Software, Universidad Continental.

Aplicación Python que expone un servicio de procesamiento de texto mediante una API Flask, empaquetada en una imagen Docker y orquestada con Docker Compose.

## Arquitectura

```
Cliente (curl / navegador)
        |
        | POST /analizar  o  GET /health
        v
+--------------------------- contenedor "web" ---------------------------+
|  Python 3.11-slim                                                     |
|  Flask (puerto 5000)  --->  procesador.py                             |
|                              - contar_palabras()                      |
|                              - frecuencia_palabras()                  |
|                              - resumen()                              |
+-----------------------------------------------------------------------+
        ^
        | volumen .:/app (en Docker Compose)
        +-- código fuente del host
```

- **`procesador.py`**: módulo con 3 funciones puras de análisis de texto.
- **`app.py`**: API Flask con los endpoints `/analizar` (POST) y `/health` (GET).
- **`test_procesador.py`**: 4 pruebas unitarias con `unittest` (se ejecutan también durante el `docker build`).
- **`requirements.txt`**: dependencia `flask==3.0.0`.
- **`Dockerfile`**: imagen basada en `python:3.11-slim`, con cache de capas y ejecución de tests en el build.
- **`.dockerignore`**: excluye `__pycache__`, `.git`, `.env`, entornos virtuales y archivos innecesarios del contexto de build.
- **`docker-compose.yml`**: servicio `web`, mapeo de puertos `5000:5000`, volumen y reinicio automático.

## Requisitos

- Docker Desktop con WSL 2 (Windows) o Docker Engine (Linux/Mac)

```bash
docker --version
docker compose version
```

## Ejecución local (sin Docker)

```bash
pip install -r requirements.txt
python -m unittest test_procesador -v
python app.py
```

## Construcción y ejecución con Docker

```bash
# Construir la imagen (los tests corren durante el build)
docker build -t procesador-texto .

# Ejecutar el contenedor
docker run -p 5000:5000 procesador-texto
```

## Ejecución con Docker Compose (recomendado)

```bash
docker compose up --build

# En segundo plano
docker compose up -d --build
```

Para detener y limpiar:

```bash
docker compose down
```

## Prueba de la API

En otra terminal:

```bash
curl http://localhost:5000/health
```

```json
{"status": "ok"}
```

```bash
curl -X POST http://localhost:5000/analizar ^
  -H "Content-Type: application/json" ^
  -d "{\"texto\": \"Docker es una herramienta increible para el despliegue de aplicaciones\"}"
```

> En Linux/Mac usar `\` en vez de `^` para continuar la línea.

Respuesta esperada:

```json
{
  "frecuencia": {"docker": 1, "es": 1, "una": 1, "herramienta": 1, "increible": 1},
  "palabras": 9,
  "resumen": "Docker es una herramienta increible para el despliegue de aplicaciones"
}
```

Si se envía un JSON sin texto, la API responde `400` con `{"error": "Texto vacio"}`.

## Comandos útiles

```bash
docker ps                 # contenedores activos
docker images             # imágenes disponibles
docker compose ps         # servicios de Compose
docker compose logs -f    # logs en tiempo real
```

## Evidencia

Captura pendiente: `docker ps` + respuesta del endpoint `/analizar` (se adjunta en la entrega).
