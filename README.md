# Proyecto CI/CD: Docker, GitHub Actions y AWS EC2

[![Estado del Deploy](https://github.com/[TU_USUARIO]/[TU_REPO]/actions/workflows/deploy.yml/badge.svg)](https://github.com/[TU_USUARIO]/[TU_REPO]/actions)

Este repositorio contiene el código para una aplicación web simple (HTML/Nginx) y un pipeline de CI/CD completamente automatizado.

El objetivo es automatizar el despliegue de un contenedor Docker en una instancia de AWS EC2 cada vez que se realiza un `push` a la rama `main`.

Una característica particular de este proyecto es que la instancia EC2 se utiliza tanto como entorno de desarrollo (donde se edita el código) como servidor de despliegue.

## 🚀 Tecnologías Utilizadas

* **GitHub Actions**: Para la automatización del CI/CD (construcción y despliegue).
* **Docker & Docker Hub**: Para la contenedorización de la aplicación Nginx y el registro de imágenes.
* **AWS EC2 (Amazon Linux 2023)**: Como servidor host para ejecutar el contenedor de producción.
* **Nginx**: Como servidor web base dentro del contenedor.
* **SSH**: Para la conexión segura de GitHub Actions al servidor EC2.

---

## ⚙️ Arquitectura del Pipeline

El workflow está definido en `.github/workflows/deploy.yml` y consta de dos trabajos principales que se ejecutan secuencialmente:

### 1. Job: `build-and-push`

Este trabajo se ejecuta en un *runner* de GitHub (Ubuntu) y es responsable de construir y publicar la imagen de Docker.

1.  **Checkout**: Clona el código del repositorio.
2.  **Login to Docker Hub**: Se autentica en Docker Hub usando los secretos `DOCKERHUB_USERNAME` y `DOCKERHUB_TOKEN`.
3.  **Build and Push**: Construye la imagen de Docker usando el `Dockerfile` local y la sube a Docker Hub con la etiqueta `latest`.

### 2. Job: `deploy`

Este trabajo depende de que `build-and-push` termine con éxito. También se ejecuta en un *runner* de GitHub y es responsable de desplegar la nueva imagen en el servidor EC2.

1.  **Deploy to EC2**:
2.  Usa la acción `appleboy/ssh-action` para conectarse de forma segura a la instancia EC2 usando los secretos `EC2_HOST`, `EC2_USER` y `EC2_SSH_KEY`.
3.  Una vez conectado, ejecuta el siguiente script en el servidor:
    * `docker pull ...`: Descarga la imagen más reciente (`:latest`) desde Docker Hub.
    * `docker stop ...`: Detiene el contenedor que se esté ejecutando (si existe).
    * `docker rm ...`: Elimina el contenedor antiguo.
    * `docker run ...`: Inicia un nuevo contenedor con la imagen recién descargada, mapeando el puerto 80 del host al puerto 80 del contenedor.

---

## 🔧 Configuración Requerida

Para que este pipeline funcione, se deben configurar los siguientes **Secretos de Acciones** en el repositorio de GitHub (`Settings > Secrets and variables > Actions`):

* `DOCKERHUB_USERNAME`: Tu nombre de usuario de Docker Hub.
* `DOCKERHUB_TOKEN`: Un token de acceso generado en Docker Hub con permisos de lectura/escritura.
* `EC2_HOST`: La dirección IP pública o DNS de tu instancia EC2.
* `EC2_USER`: El usuario para la conexión SSH (ej: `ec2-user`).
* `EC2_SSH_KEY`: La clave SSH privada (en formato `.pem`) necesaria para acceder a la instancia.

---

## 📁 Archivos Clave del Proyecto
