# Guía para Construir y Ejecutar una Imagen Personalizada de WhisperX con GPU

Este documento contiene todos los pasos necesarios para construir una imagen de Docker personalizada para WhisperX que sea compatible con GPUs modernas (CUDA 12.8+) y para ejecutarla como un servicio con una API y una interfaz web.

---

### **Paso 1: Modificar el `Dockerfile.gpu`**

El objetivo de este paso es actualizar la versión de CUDA que utiliza la imagen base para que sea compatible con tu hardware.

1.  **Abre el archivo `Dockerfile.gpu`** que se encuentra en este mismo directorio.

2.  **Busca la siguiente línea** al principio del archivo:

    ```dockerfile
    FROM nvidia/cuda:12.6.3-base-ubuntu22.04
    ```

3.  **Reemplázala** por esta línea, que utiliza una versión de CUDA más reciente:

    ```dockerfile
    FROM nvidia/cuda:12.8.0-runtime-ubuntu22.04
    ```

---

### **Paso 2: Construir la Imagen de Docker**

Este comando compilará tu imagen de Docker personalizada a partir del `Dockerfile` que acabas de modificar.

1.  **Abre tu terminal** en este directorio (`F:\Docker\whisper_custom`).

2.  **Ejecuta el siguiente comando:**

    ```bash
    docker build -t whisper-custom:latest -f Dockerfile.gpu .
    ```

    *   `-t whisper-custom:latest`: Le da un nombre (`whisper-custom`) y una etiqueta (`latest`) a tu imagen.
    *   `-f Dockerfile.gpu`: Especifica que se debe usar el archivo `Dockerfile.gpu`.
    *   `.`: Indica que el contexto de la construcción son los archivos del directorio actual.

    *Nota: Este proceso puede tardar varios minutos.*

---

### **Paso 3: Crear el archivo `docker-compose.yml`**

Este archivo define cómo se ejecutará tu imagen de Docker, habilitando la GPU y configurando WhisperX.

1.  **Crea un archivo** llamado `docker-compose.yml` en este mismo directorio.

2.  **Copia y pega el siguiente contenido** dentro de ese archivo:

    ```yaml
    version: '3.8'
    services:
      whisperx:
        image: whisper-custom:latest
        restart: always
        ports:
          - "8000:9000"
        deploy:
          resources:
            reservations:
              devices:
                - driver: nvidia
                  count: 1
                  capabilities: [gpu]
        environment:
          - ASR_ENGINE=whisperx
          - ASR_MODEL=large-v3
        volumes:
          - whisper_models:/root/.cache/whisper
          - whisper_models:/app/models

    volumes:
      whisper_models:
    ```

---

### **Paso 4: Iniciar el Servicio**

Con el archivo `docker-compose.yml` ya creado, este comando iniciará el servicio.

1.  **Abre tu terminal** en este directorio.

2.  **Ejecuta el comando:**

    ```bash
    docker compose up -d
    ```

---

### **Paso 5: Verificar el Servicio**

Comprueba que el contenedor esté funcionando correctamente.

1.  **Espera un par de minutos** a que el servicio se inicie.

2.  **Ejecuta este comando** para ver los contenedores activos:

    ```bash
    docker ps
    ```

    *Deberías ver un contenedor llamado `whisper_custom-whisperx-1` con el estado `Up`.*

3.  **Accede a la interfaz web** en tu navegador para empezar a transcribir:
    `http://localhost:8000`

4.  **Explora la API** en la siguiente URL:
    `http://localhost:8000/docs`

---

### **Paso 6: Subir tu Imagen a Docker Hub (Opcional)**

Sigue estos pasos si quieres guardar tu imagen personalizada en la nube para usarla en otras máquinas.

1.  **Inicia sesión en Docker Hub**:

    ```bash
    docker login
    ```

2.  **Re-etiqueta tu imagen** (reemplaza `TU_USUARIO_DOCKERHUB` con tu nombre de usuario real):

    ```bash
    docker tag whisper-custom:latest TU_USUARIO_DOCKERHUB/whisper-custom:latest
    ```

3.  **Sube la imagen** a tu cuenta de Docker Hub:

    ```bash
    docker push TU_USUARIO_DOCKERHUB/whisper-custom:latest
    ```