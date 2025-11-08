# WhisperX5080 – Servicio Web de Transcripción (GPU RTX 5080)

Servicio de ASR listo para producción con Docker, API REST y Swagger.
Optimizado para GPU NVIDIA (RTX 5080) usando imágenes precompiladas.

## Características
- API REST con Swagger en `http://localhost:8000/docs`.
- Motores: `faster_whisper` (por defecto) y `whisperx` (diarización, requiere `HF_TOKEN`).
- Modelos desde `tiny` hasta `large-v3`.
- Despliegue simple con Docker Compose.

## Requisitos
- Docker y Docker Compose.
- NVIDIA Container Toolkit (soporte `--gpus`).
- Drivers NVIDIA compatibles.

Verifica GPU:
```
docker run --rm --gpus all nvidia/cuda:12.8.0-base-ubuntu22.04 nvidia-smi
```

## Uso rápido
### 1) Clonar y preparar entorno
```
# Clona este repositorio
# (o copia el contenido en tu carpeta local)

# Opcional (solo WhisperX)
copy .env.example .env
# Edita .env y coloca tu HF token
```

### 2) Elegir motor
- `docker-compose.yml` usa `ASR_ENGINE=faster_whisper` (sin token).
- `docker-compose.gpu.yml` usa `ASR_ENGINE=whisperx` y lee `HF_TOKEN`.

### 3) Levantar el servicio
```
# Faster Whisper (recomendado general)
docker compose up -d

# WhisperX (diarización)
docker compose -f docker-compose.gpu.yml up -d
```

### 4) Acceder
- Interfaz Swagger: `http://localhost:8000/docs`
- Redoc: `http://localhost:8000/redoc`

## Ejemplos de API
### Transcribir (español)
```
curl -X POST "http://localhost:8000/asr" \
  -H "accept: text/plain" \
  -H "Content-Type: multipart/form-data" \
  -F "audio_file=@tu_audio.mp3" \
  -F "task=transcribe" \
  -F "language=es" \
  -F "output=json"
```

### Detectar idioma
```
curl -X POST "http://localhost:8000/detect-language" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "audio_file=@tu_audio.mp3"
```

### Diarización (WhisperX)
```
curl -X POST "http://localhost:8000/asr" \
  -H "accept: text/plain" \
  -H "Content-Type: multipart/form-data" \
  -F "audio_file=@tu_audio.mp3" \
  -F "task=transcribe" \
  -F "language=es" \
  -F "diarize=true" \
  -F "min_speakers=2" \
  -F "max_speakers=5" \
  -F "output=json"
```

## n8n – Nota importante
- En el nodo `HTTP Request` usa `multipart/form-data`.
- El nombre del archivo debe ser exactamente `audio_file`.
- Ejemplo de URL: `http://localhost:8000/asr?task=transcribe&language=es`.

## Personalización
- Cambia el modelo editando `ASR_MODEL` en el compose (`tiny`, `base`, `small`, `medium`, `large-v3`).
- Para VRAM limitada, usa modelos más pequeños.

## Comandos útiles
```
docker compose down
docker compose logs -f
nvidia-smi
```

## Licencia
Sin licencia explícita. Añade la que prefieras si corresponde.