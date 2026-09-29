## 🐳 Docker

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Conceptos Fundamentales
1. [¿Qué es Docker y por qué usarlo?](#1-qué-es-docker-y-por-qué-usarlo)
2. [Contenedores vs Máquinas Virtuales](#2-diferencia-entre-contenedores-y-máquinas-virtuales)
3. [Imágenes vs Contenedores](#3-qué-es-una-imagen-y-un-contenedor)
4. [Docker Registry y Docker Hub](#4-qué-es-docker-registry-y-docker-hub)

### 🔹 Dockerfile
5. [Crear un Dockerfile](#5-cómo-crear-un-dockerfile)
6. [Mejores prácticas Dockerfile](#6-mejores-prácticas-para-dockerfile)
7. [Multi-stage builds](#7-qué-son-los-multi-stage-builds)
8. [.dockerignore](#8-qué-es-dockerignore)

### 🔹 Docker Compose
9. [¿Qué es Docker Compose?](#9-qué-es-docker-compose)
10. [docker-compose.yml](#10-cómo-crear-un-docker-composeyml)
11. [Redes en Docker](#11-redes-en-docker)
12. [Volúmenes](#12-volúmenes-en-docker)

### 🔹 Comandos Esenciales
13. [Comandos Docker más usados](#13-comandos-docker-más-usados)
14. [Debugging de contenedores](#14-cómo-debuggear-contenedores)

---

### 🔹 Conceptos Fundamentales

#### 1. ¿Qué es Docker y por qué usarlo?

**Respuesta:**

**Docker** es una plataforma de **contenedorización** que permite empaquetar aplicaciones con todas sus dependencias en **contenedores** portables y ligeros.

**Problema que resuelve:**
```
Desarrollador: "En mi máquina funciona" ¯\_(ツ)_/¯
Producción: Versiones diferentes de Java, librerías, SO
```

**Solución con Docker:**
```
Desarrollador: Crea contenedor con todo incluido
Producción: Ejecuta el mismo contenedor → funciona igual
```

**Beneficios:**

1. **Portabilidad**: Ejecuta la misma imagen en dev, test y prod
2. **Aislamiento**: Cada contenedor tiene su propio filesystem, red, procesos
3. **Consistencia**: Elimina "en mi máquina funciona"
4. **Eficiencia**: Más ligero que VMs (segundos para iniciar)
5. **Escalabilidad**: Fácil crear/destruir instancias
6. **Versionado**: Las imágenes son versionadas (tags)

**Ejemplo práctico:**
```bash
# Sin Docker - Configuración manual
apt-get install java-17
apt-get install maven
git clone repo
mvn package
java -jar app.jar

# Con Docker - Un comando
docker run mi-app:1.0
```

**Casos de uso:**
- Microservicios
- CI/CD pipelines
- Ambientes de desarrollo reproducibles
- Aplicaciones cloud-native
- Testing con diferentes versiones

---

#### 2. Diferencia entre Contenedores y Máquinas Virtuales

**Respuesta:**

**Máquinas Virtuales (VMs):**
```
┌─────────────────────┐
│   App 1   │  App 2  │
├───────────┼─────────┤
│  Guest OS │ Guest OS│  ← Cada VM tiene SO completo
├───────────┴─────────┤
│    Hypervisor       │
├─────────────────────┤
│     Host OS         │
├─────────────────────┤
│     Hardware        │
└─────────────────────┘
```

**Contenedores:**
```
┌─────────────────────┐
│   App 1   │  App 2  │
├───────────┼─────────┤
│ Container │Container│  ← Comparten kernel del host
├───────────┴─────────┤
│   Docker Engine     │
├─────────────────────┤
│     Host OS         │
├─────────────────────┤
│     Hardware        │
└─────────────────────┘
```

**Comparación:**

| Aspecto | VM | Contenedor |
|---------|-----|-----------|
| **Tamaño** | GBs | MBs |
| **Inicio** | Minutos | Segundos |
| **Aislamiento** | Completo (SO) | Proceso (namespace) |
| **Performance** | Overhead de virtualización | Casi nativo |
| **Portabilidad** | Baja | Alta |
| **Densidad** | 10-20 VMs por host | 100+ contenedores |

**¿Cuándo usar cada uno?**

**VMs:**
- Necesitas aislar SOs diferentes (Windows en Linux)
- Seguridad máxima (multi-tenancy)
- Aplicaciones legacy que requieren SO específico

**Contenedores:**
- Microservicios modernos
- Aplicaciones cloud-native
- CI/CD
- Desarrollo local

**Hybrid:** Contenedores dentro de VMs (común en cloud)

---

#### 3. ¿Qué es una imagen y un contenedor?

**Respuesta:**

**Imagen:**
- Plantilla **inmutable** y **read-only**
- Contiene: código, runtime, librerías, variables de entorno
- Se construye a partir de un `Dockerfile`
- Se almacena en un registry (Docker Hub, ECR, etc.)

**Contenedor:**
- **Instancia en ejecución** de una imagen
- Es un proceso aislado del host
- Tiene una capa **writable** sobre la imagen
- Se puede crear, iniciar, detener, eliminar

**Analogía:**
```
Imagen = Clase (plantilla)
Contenedor = Objeto (instancia)
```

**Ejemplo:**
```bash
# Descargar imagen de Java
docker pull openjdk:17

# Crear y ejecutar contenedor desde imagen
docker run -d --name mi-app openjdk:17

# Misma imagen, múltiples contenedores
docker run -d --name app1 openjdk:17
docker run -d --name app2 openjdk:17
docker run -d --name app3 openjdk:17
# 3 contenedores independientes de la misma imagen
```

**Arquitectura de capas:**
```
┌─────────────────────┐
│  Container Layer   │  ← Writable (se pierde al borrar contenedor)
├─────────────────────┤
│   Image Layer 3    │  ← Read-only
├─────────────────────┤
│   Image Layer 2    │  ← Read-only
├─────────────────────┤
│   Image Layer 1    │  ← Read-only (base image)
└─────────────────────┘
```

**Comandos básicos:**
```bash
# Listar imágenes
docker images

# Listar contenedores en ejecución
docker ps

# Listar todos los contenedores
docker ps -a

# Eliminar contenedor
docker rm contenedor_id

# Eliminar imagen
docker rmi imagen_id
```

---

#### 4. ¿Qué es Docker Registry y Docker Hub?

**Respuesta:**

**Docker Registry:**
- Servidor que almacena y distribuye **imágenes Docker**
- Puede ser público o privado

**Docker Hub:**
- Registry público **oficial** de Docker
- Millones de imágenes disponibles
- Gratuito para imágenes públicas

**Alternativas:**
- **Amazon ECR** (Elastic Container Registry)
- **Google GCR** (Google Container Registry)
- **Azure ACR** (Azure Container Registry)
- **Harbor** (self-hosted)
- **GitHub Container Registry**

**Trabajar con registries:**

```bash
# Pull (descargar) desde Docker Hub
docker pull nginx:latest
docker pull postgres:15
docker pull openjdk:17

# Pull desde registry privado
docker pull registry.empresa.com/mi-app:1.0

# Tag imagen para registry privado
docker tag mi-app:latest registry.empresa.com/mi-app:1.0

# Push (subir) a registry
docker push registry.empresa.com/mi-app:1.0

# Login a registry privado
docker login registry.empresa.com
# Username: usuario
# Password: ****
```

**Tags:**
```bash
# Misma imagen, diferentes tags
openjdk:17
openjdk:17-jdk
openjdk:17-jdk-alpine
openjdk:latest

# Convención de tags
mi-app:1.0.0        # Versión específica
mi-app:1.0          # Minor version
mi-app:latest       # Última versión (default)
mi-app:dev          # Branch/environment
```

**Best Practices:**
- Usa tags específicos en producción (no `latest`)
- Escanea imágenes por vulnerabilidades
- Usa registries privados para código propietario
- Implementa lifecycle policies para limpiar imágenes viejas

---

### 🔹 Dockerfile

#### 5. ¿Cómo crear un Dockerfile?

**Respuesta:**

Un **Dockerfile** es un archivo de texto con instrucciones para construir una imagen Docker.

**Estructura básica:**
```dockerfile
# Imagen base
FROM openjdk:17-jdk-slim

# Metadatos
LABEL maintainer="tu@email.com"
LABEL version="1.0"

# Variables de entorno
ENV APP_HOME=/app
ENV PORT=8080

# Directorio de trabajo
WORKDIR ${APP_HOME}

# Copiar archivos
COPY target/mi-app.jar app.jar

# Puerto que expone
EXPOSE ${PORT}

# Usuario no-root (seguridad)
USER nobody

# Comando de inicio
CMD ["java", "-jar", "app.jar"]
```

**Instrucciones principales:**

```dockerfile
# FROM - Imagen base (obligatorio, debe ser primero)
FROM openjdk:17-jdk-slim

# RUN - Ejecuta comando durante el build
RUN apt-get update && apt-get install -y curl

# COPY - Copia archivos del host a la imagen
COPY src/ /app/src/

# ADD - Como COPY pero puede descomprimir y descargar URLs
ADD https://example.com/file.tar.gz /tmp/

# WORKDIR - Establece directorio de trabajo
WORKDIR /app

# ENV - Variables de entorno
ENV JAVA_OPTS="-Xmx512m"

# EXPOSE - Documenta puertos (no los publica)
EXPOSE 8080 8443

# VOLUME - Punto de montaje para datos persistentes
VOLUME /data

# USER - Usuario que ejecuta el contenedor
USER appuser

# CMD - Comando por defecto (se puede sobrescribir)
CMD ["java", "-jar", "app.jar"]

# ENTRYPOINT - Comando principal (no se sobrescribe fácil)
ENTRYPOINT ["java"]
CMD ["-jar", "app.jar"]  # Args por defecto para ENTRYPOINT
```

**Ejemplo completo - Spring Boot app:**

```dockerfile
# === Stage 1: Build ===
FROM maven:3.9-openjdk-17 AS build

WORKDIR /app

# Copiar pom.xml primero (mejor caching)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copiar código y compilar
COPY src ./src
RUN mvn package -DskipTests

# === Stage 2: Runtime ===
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copiar JAR del stage anterior
COPY --from=build /app/target/*.jar app.jar

# Usuario no-root
RUN addgroup --system appgroup && adduser --system appuser --ingroup appgroup
USER appuser

EXPOSE 8080

ENTRYPOINT ["java"]
CMD ["-jar", "app.jar"]
```

**Construir imagen:**
```bash
# Build básico
docker build -t mi-app:1.0 .

# Build con argumentos
docker build --build-arg VERSION=1.0 -t mi-app:1.0 .

# Build sin usar cache
docker build --no-cache -t mi-app:1.0 .
```

---

#### 6. Mejores prácticas para Dockerfile

**Respuesta:**

**1. Usa imágenes base oficiales y slim:**
```dockerfile
# ❌ Imagen pesada
FROM openjdk:17

# ✅ Imagen slim (menor tamaño, menos vulnerabilidades)
FROM openjdk:17-jdk-slim

# ✅ Alpine (más pequeña)
FROM openjdk:17-alpine
```

**2. Ordena comandos para mejor caching:**
```dockerfile
# ❌ Cambios en código invalidan cache de dependencias
COPY . .
RUN mvn package

# ✅ Dependencias se cachean si pom.xml no cambia
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package
```

**3. Minimiza capas combinando RUN:**
```dockerfile
# ❌ Múltiples capas
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
RUN rm -rf /var/lib/apt/lists/*

# ✅ Una sola capa
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*
```

**4. Usa .dockerignore:**
```
# .dockerignore
target/
.git/
.idea/
*.md
*.log
node_modules/
```

**5. No ejecutes como root:**
```dockerfile
# ❌ Por defecto es root
USER root

# ✅ Crea y usa usuario sin privilegios
RUN addgroup --system appgroup && \
    adduser --system appuser --ingroup appgroup
USER appuser
```

**6. Usa multi-stage builds:**
```dockerfile
# Stage build con herramientas pesadas
FROM maven:3.9-openjdk-17 AS build
# ... compilar

# Stage runtime solo con lo necesario
FROM openjdk:17-jdk-slim
COPY --from=build /app/target/app.jar .
```

**7. Especifica versiones exactas:**
```dockerfile
# ❌ Latest puede cambiar
FROM openjdk:latest

# ✅ Versión específica
FROM openjdk:17.0.9-jdk-slim
```

**8. Usa HEALTHCHECK:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

**9. Metadata con LABEL:**
```dockerfile
LABEL org.opencontainers.image.title="Mi App"
LABEL org.opencontainers.image.version="1.0.0"
LABEL org.opencontainers.image.authors="equipo@empresa.com"
```

**10. Variables de entorno para configuración:**
```dockerfile
ENV SPRING_PROFILES_ACTIVE=prod
ENV JAVA_OPTS="-Xmx512m -Xms256m"

CMD java $JAVA_OPTS -jar app.jar
```

---

#### 7. ¿Qué son los multi-stage builds?

**Respuesta:**

**Multi-stage builds** permiten usar múltiples imágenes base en un Dockerfile, copiando solo lo necesario de una stage a otra. Esto reduce el tamaño final de la imagen.

**Problema sin multi-stage:**
```dockerfile
# Imagen final incluye Maven, código fuente, .class, etc.
FROM maven:3.9-openjdk-17

COPY . .
RUN mvn package

CMD ["java", "-jar", "target/app.jar"]

# Resultado: 800 MB (incluye Maven y dependencias de build)
```

**Solución con multi-stage:**
```dockerfile
# === Stage 1: Build (con Maven) ===
FROM maven:3.9-openjdk-17 AS builder

WORKDIR /build

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

# === Stage 2: Runtime (solo JDK slim) ===
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copiar SOLO el JAR del stage anterior
COPY --from=builder /build/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]

# Resultado: 350 MB (solo JDK + JAR)
```

**Ventajas:**
1. **Menor tamaño**: Solo el runtime final, sin herramientas de build
2. **Más seguro**: Menos software = menos vulnerabilidades
3. **Más rápido**: Menos datos que transferir/descargar

**Ejemplo con Node.js:**
```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /build/dist ./dist
COPY --from=builder /build/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

**Copiar desde stages específicos:**
```dockerfile
FROM alpine AS resources
RUN wget https://example.com/file.txt

FROM ubuntu AS config
COPY config.json /tmp/

FROM openjdk:17-slim
COPY --from=resources /file.txt /app/
COPY --from=config /tmp/config.json /app/
```

---

### 🔹 Docker Compose

#### 9. ¿Qué es Docker Compose?

**Respuesta:**

**Docker Compose** es una herramienta para definir y ejecutar aplicaciones **multi-contenedor** usando un archivo YAML.

**Problema sin Compose:**
```bash
# Iniciar BD
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:15

# Iniciar Redis
docker run -d --name redis \
  -p 6379:6379 \
  redis:7

# Crear red
docker network create mi-red

# Conectar contenedores
docker network connect mi-red postgres
docker network connect mi-red redis

# Iniciar app
docker run -d --name app \
  --network mi-red \
  -e DB_HOST=postgres \
  -e REDIS_HOST=redis \
  -p 8080:8080 \
  mi-app:1.0
```

**Solución con Compose:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: postgres
      REDIS_HOST: redis
    depends_on:
      - postgres
      - redis

volumes:
  postgres-data:
```

```bash
# Un solo comando para todo
docker-compose up -d
```

**Comandos principales:**
```bash
# Iniciar todos los servicios
docker-compose up -d

# Ver logs
docker-compose logs -f

# Detener servicios
docker-compose down

# Detener y eliminar volúmenes
docker-compose down -v

# Reiniciar servicio específico
docker-compose restart app

# Ver estado
docker-compose ps

# Ejecutar comando en servicio
docker-compose exec app bash

# Reconstruir imágenes
docker-compose build

# Escalar servicios
docker-compose up -d --scale app=3
```

**Beneficios:**
- Configuración declarativa (YAML)
- Fácil de versionar y compartir
- Redes automáticas entre servicios
- Orquestación simple de múltiples contenedores

---

#### 10. ¿Cómo crear un docker-compose.yml?

**Respuesta:**

**Ejemplo completo - Stack Spring Boot:**

```yaml
version: '3.8'

services:
  # Base de datos PostgreSQL
  postgres:
    image: postgres:15-alpine
    container_name: postgres-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis para cache
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # Aplicación Spring Boot
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        VERSION: ${APP_VERSION:-latest}
    container_name: spring-app
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/myapp
      SPRING_DATASOURCE_USERNAME: admin
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD:-secret}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379
      JAVA_OPTS: -Xmx512m -Xms256m
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
    volumes:
      - app-logs:/app/logs

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  postgres-data:
    driver: local
  app-logs:
    driver: local
```

**Variables de entorno (.env):**
```env
# .env file
APP_VERSION=1.0.0
DB_PASSWORD=supersecret
REDIS_PASSWORD=redissecret
```

**Uso:**
```bash
# Usar variables del .env
docker-compose up -d

# Sobrescribir con variables de entorno
DB_PASSWORD=otra docker-compose up -d
```

**Override para desarrollo:**
```yaml
# docker-compose.override.yml (se aplica automáticamente)
version: '3.8'

services:
  app:
    build:
      target: development
    volumes:
      - ./src:/app/src  # Hot reload
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DEVTOOLS_RESTART_ENABLED: "true"
    ports:
      - "5005:5005"  # Debug port
    command: java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 -jar app.jar
```

---

#### 13. Comandos Docker más usados

**Respuesta:**

**Gestión de contenedores:**
```bash
# Ejecutar contenedor
docker run -d --name mi-app -p 8080:8080 mi-imagen:latest

# Listar contenedores activos
docker ps

# Listar todos (incluidos detenidos)
docker ps -a

# Ver logs
docker logs mi-app
docker logs -f mi-app  # Follow
docker logs --tail 100 mi-app  # Últimas 100 líneas

# Detener contenedor
docker stop mi-app

# Iniciar contenedor detenido
docker start mi-app

# Reiniciar
docker restart mi-app

# Eliminar contenedor
docker rm mi-app
docker rm -f mi-app  # Forzar si está corriendo

# Ejecutar comando en contenedor
docker exec -it mi-app bash
docker exec mi-app ls /app
```

**Gestión de imágenes:**
```bash
# Listar imágenes
docker images

# Descargar imagen
docker pull nginx:latest

# Construir imagen
docker build -t mi-app:1.0 .

# Tag imagen
docker tag mi-app:1.0 usuario/mi-app:1.0

# Subir a registry
docker push usuario/mi-app:1.0

# Eliminar imagen
docker rmi imagen:tag

# Limpiar imágenes sin usar
docker image prune -a
```

**Inspección y debugging:**
```bash
# Inspeccionar contenedor (JSON)
docker inspect mi-app

# Stats en vivo
docker stats
docker stats mi-app

# Puertos mapeados
docker port mi-app

# Procesos en contenedor
docker top mi-app

# Eventos
docker events

# Información del sistema
docker info

# Versión
docker version
```

**Limpieza:**
```bash
# Eliminar contenedores detenidos
docker container prune

# Eliminar imágenes sin usar
docker image prune -a

# Eliminar volúmenes sin usar
docker volume prune

# Eliminar redes sin usar
docker network prune

# Limpiar TODO
docker system prune -a --volumes
```

**Volúmenes:**
```bash
# Crear volumen
docker volume create mi-volumen

# Listar volúmenes
docker volume ls

# Inspeccionar volumen
docker volume inspect mi-volumen

# Eliminar volumen
docker volume rm mi-volumen
```

**Redes:**
```bash
# Crear red
docker network create mi-red

# Listar redes
docker network ls

# Conectar contenedor a red
docker network connect mi-red contenedor

# Desconectar
docker network disconnect mi-red contenedor
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Arquitectura y DevOps](./05-architecture-ops.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: Kubernetes ➡️](./08-kubernetes.md)
