# WhisperX Web Service para RTX 5080

Servicio web de transcripción de audio usando WhisperX optimizado para NVIDIA RTX 5080 (16GB VRAM) con soporte para API REST y interfaz web.

## 🚀 Características

- ✅ **Optimizado para RTX 5080** con CUDA 12.8
- ✅ **WhisperX**: Motor de transcripción de alta precisión
- ✅ **Diarización**: Identificación de múltiples hablantes
- ✅ **Múltiples modelos**: Desde tiny hasta large-v3
- ✅ **API REST**: Integración fácil con tus aplicaciones
- ✅ **Interfaz Web**: Prueba y usa directamente desde el navegador
- ✅ **Docker**: Despliegue simplificado con contenedores

## 📋 Requisitos Previos

- **Docker** y **Docker Compose** instalados
- **NVIDIA Docker Runtime** (nvidia-docker2)
- **GPU**: RTX 5080 o similar con 16GB VRAM
- **Drivers NVIDIA**: Versión compatible con CUDA 12.8+
- **Token de Hugging Face**: Para usar diarización con WhisperX

### Verificar instalación de Docker y NVIDIA

```bash
# Verificar Docker
docker --version
docker compose version

# Verificar soporte GPU
docker run --rm --gpus all nvidia/cuda:12.8.0-base-ubuntu22.04 nvidia-smi
```

## 🔧 Configuración Rápida

### Paso 1: Configurar Variables de Entorno

1. **Obtén tu token de Hugging Face**:
   - Ve a https://huggingface.co/settings/tokens
   - Crea un token de lectura (Read)
   - Copia el token

2. **Crea el archivo `.env`**:

```bash
# Copiar el archivo de ejemplo
cp .env.example .env

# Editar y agregar tu token
nano .env  # o usa tu editor favorito
```

3. **Configura tu token en `.env`**:

```bash
HF_TOKEN=tu_token_de_huggingface_aqui
```

### Paso 2: Construir la Imagen Docker

```bash
docker build -t whisperx-rtx5080:latest -f Dockerfile.gpu .
```

Este proceso puede tardar 10-15 minutos la primera vez.

### Paso 3: Iniciar el Servicio

```bash
docker compose -f docker-compose.gpu.yml up -d
```

### Paso 4: Verificar que Funciona

```bash
# Ver logs
docker compose -f docker-compose.gpu.yml logs -f

# Verificar contenedor activo
docker ps
```

Deberías ver el contenedor `whisperx5080-whisper-asr-webservice-gpu-1` corriendo.

### Paso 5: Acceder al Servicio

- **Interfaz Web**: http://localhost:8000
- **Documentación API**: http://localhost:8000/docs
- **API Alternativa**: http://localhost:8000/redoc

## 📊 Configuración de Modelos

### Modelos Disponibles y Uso de VRAM

| Modelo | VRAM (float16) | Velocidad | Calidad | Recomendado para |
|--------|----------------|-----------|---------|------------------|
| tiny | ~1GB | Muy rápida | Básica | Pruebas rápidas |
| base | ~1.5GB | Rápida | Buena | Transcripciones simples |
| small | ~2.5GB | Media | Buena | Uso general |
| medium | ~5GB | Media-Lenta | Muy buena | **Balance óptimo** |
| large-v2 | ~8GB | Lenta | Excelente | Alta precisión |
| large-v3 | ~10GB | Lenta | Excelente | **Máxima calidad** |

### Cambiar el Modelo

Edita `docker-compose.gpu.yml` y cambia:

```yaml
environment:
  - ASR_MODEL=medium  # Cambia aquí: tiny, base, small, medium, large-v3
```

Luego reinicia:

```bash
docker compose -f docker-compose.gpu.yml restart
```

## 🎯 Uso de la API

### Transcribir un Audio

```bash
curl -X POST "http://localhost:8000/asr" \
  -H "accept: text/plain" \
  -H "Content-Type: multipart/form-data" \
  -F "audio_file=@tu_audio.mp3" \
  -F "task=transcribe" \
  -F "language=es" \
  -F "output=json"
```

### Detectar Idioma

```bash
curl -X POST "http://localhost:8000/detect-language" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "audio_file=@tu_audio.mp3"
```

### Transcribir con Diarización (Múltiples Hablantes)

```bash
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

### Formatos de Salida Soportados

- `txt`: Texto plano
- `json`: JSON con timestamps
- `srt`: Subtítulos SubRip
- `vtt`: WebVTT subtítulos
- `tsv`: Valores separados por tabulación

## 🔍 Solución de Problemas

### Error: "No se puede conectar al daemon de Docker"

```bash
sudo systemctl start docker
sudo usermod -aG docker $USER
# Cerrar sesión y volver a entrar
```

### Error: "could not select device driver"

```bash
# Instalar NVIDIA Container Toolkit
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
```

### Error: "You must set the HF_TOKEN environment variable"

Asegúrate de tener el token en tu archivo `.env`:

```bash
HF_TOKEN=hf_tu_token_aqui
```

### El contenedor se detiene inmediatamente

```bash
# Ver logs para identificar el error
docker compose -f docker-compose.gpu.yml logs

# Errores comunes:
# - Token HF inválido o faltante
# - GPU no detectada
# - Falta de memoria VRAM
```

### Problemas de memoria VRAM

Si te quedas sin memoria, prueba:

1. **Usar modelo más pequeño**: Cambiar a `medium` o `small`
2. **Habilitar cuantización int8**:
   ```yaml
   environment:
     - ASR_QUANTIZATION=int8
   ```

## 🛠️ Comandos Útiles

```bash
# Detener el servicio
docker compose -f docker-compose.gpu.yml down

# Ver logs en tiempo real
docker compose -f docker-compose.gpu.yml logs -f

# Reconstruir después de cambios
docker compose -f docker-compose.gpu.yml up -d --build

# Limpiar caché de modelos
docker volume rm whisperx5080_cache-whisper

# Entrar al contenedor
docker exec -it whisperx5080-whisper-asr-webservice-gpu-1 bash

# Ver uso de GPU
nvidia-smi

# Monitorear GPU en tiempo real
watch -n 1 nvidia-smi
```

## 📈 Optimizaciones de Rendimiento

### Para RTX 5080 (16GB)

La configuración actual ya está optimizada con:

- ✅ `PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:512`
- ✅ `CUDA_MODULE_LOADING=LAZY`
- ✅ Cuantización `float16` por defecto
- ✅ Límite de memoria del contenedor a 14GB

### Si necesitas más velocidad:

```yaml
environment:
  - ASR_MODEL=small  # Modelo más rápido
  - ASR_QUANTIZATION=int8  # Cuantización más agresiva
```

### Si necesitas más calidad:

```yaml
environment:
  - ASR_MODEL=large-v3  # Mejor modelo
  - ASR_QUANTIZATION=float16  # Mayor precisión
```

## 🌐 Idiomas Soportados

WhisperX soporta 99+ idiomas, incluyendo:

- 🇪🇸 Español (es)
- 🇺🇸 Inglés (en)
- 🇫🇷 Francés (fr)
- 🇩🇪 Alemán (de)
- 🇮🇹 Italiano (it)
- 🇵🇹 Portugués (pt)
- 🇨🇳 Chino (zh)
- 🇯🇵 Japonés (ja)
- 🇰🇷 Coreano (ko)
- Y muchos más...

Ver lista completa en: http://localhost:8000/docs

## 📝 Variables de Entorno Disponibles

| Variable | Valores | Por Defecto | Descripción |
|----------|---------|-------------|-------------|
| `HF_TOKEN` | string | - | Token de Hugging Face (obligatorio) |
| `ASR_ENGINE` | whisperx, faster_whisper, openai_whisper | whisperx | Motor de transcripción |
| `ASR_MODEL` | tiny, base, small, medium, large-v3 | large-v3 | Modelo a usar |
| `ASR_DEVICE` | cuda, cpu | cuda | Dispositivo de cómputo |
| `ASR_QUANTIZATION` | float32, float16, int8 | float16 | Precisión del modelo |
| `MODEL_IDLE_TIMEOUT` | número | 0 | Tiempo antes de descargar modelo (segundos) |

Ver `.env.example` para configuración completa.

## 📚 Documentación Adicional

- [Documentación de WhisperX](https://github.com/m-bain/whisperX)
- [Modelos de Whisper](https://github.com/openai/whisper#available-models-and-languages)
- [NVIDIA Docker](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)

## 📄 Licencia

MIT License - Ver archivo [LICENCE](LICENCE)

## 🤝 Contribuciones

Basado en [whisper-asr-webservice](https://github.com/ahmetoner/whisper-asr-webservice) por Ahmet Öner.

Optimizado para RTX 5080 por la comunidad.

## 💡 Soporte

Si encuentras problemas:

1. Revisa la sección **Solución de Problemas**
2. Verifica los logs: `docker compose -f docker-compose.gpu.yml logs`
3. Verifica que tu GPU sea detectada: `nvidia-smi`
4. Asegúrate de tener el token HF configurado

---

**¡Disfruta transcribiendo con WhisperX en tu RTX 5080! 🎉**
