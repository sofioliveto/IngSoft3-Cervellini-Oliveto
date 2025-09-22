# 📝 Decisiones Técnicas TP2 - Dockerización

## 1. Introducción del Proyecto

Inicialmente, consideramos dockerizar una aplicación sencilla en React, pero al no contar con base de datos, no cumplía con todos los requisitos del TP. Por eso decidimos dockerizar una aplicación existente desarrollada en Arquitectura de Software 1, compuesta por:

- **Frontend:** React + Vite + JavaScript
- **Backend:** GoLang
- **Base de datos:** MySQL

**Objetivo:** Dockerizar toda la app y preparar entornos QA y Producción.

---

## 2. Proceso de Dockerización

### Preparación del entorno local

1. Clonamos el repositorio en `tp2-docker`.
2. Instalamos dependencias en el frontend:
   ```bash
   npm install
   ```
3. Ejecutamos el backend:
   ```bash
   go run main.go
   ```
   luego de exportar variables de entorno:
   ```bash
   export DB_USER=root
   export DB_PASSWORD=44898366
   export DB_HOST=localhost
   export DB_PORT=3306
   export DB_NAME=dbarquisoft1
   ```
4. Ejecutamos el frontend:
   ```bash
   npm run dev
   ```
5. Iniciamos sesión en Docker Hub:
   ```bash
   docker login
   ```

### Creación de los Dockerfiles

#### Dockerfile - Backend

- **Build:**
  ```dockerfile
  FROM golang:1.21-alpine AS builder      # Usa imagen oficial de Go (liviana, basada en Alpine) para compilar
  WORKDIR /app                           # Establece el directorio de trabajo dentro del contenedor
  COPY . .                               # Copia todos los archivos del backend al contenedor
  RUN go mod download                    # Descarga las dependencias de Go declaradas en go.mod
  RUN go build -o main .                 # Compila la app Go y genera un binario llamado 'main'
  ```
- **Runtime:**
  ```dockerfile
  FROM alpine:latest                     # Usa una imagen base aún más liviana para ejecutar el binario
  COPY --from=builder /app/main .        # Copia el binario compilado desde la etapa anterior
  EXPOSE 8080                            # Expone el puerto 8080 para que Docker pueda mapearlo
  CMD ["./main"]                        # Comando por defecto: ejecuta el binario
  ```

#### Dockerfile - Frontend

- **Build:**
  ```dockerfile
  FROM node:20-alpine                    # Usa imagen oficial de Node.js (liviana, basada en Alpine) para compilar
  WORKDIR /app                           # Establece el directorio de trabajo dentro del contenedor
  COPY . .                               # Copia todos los archivos del frontend al contenedor
  RUN npm install                        # Instala las dependencias de Node.js declaradas en package.json
  RUN npm run build                      # Compila la app React y genera archivos estáticos en /dist
  ```
- **Runtime:**
  ```dockerfile
  FROM nginx:stable-alpine               # Usa imagen oficial de nginx (liviana) para servir archivos estáticos
  COPY --from=builder /app/dist /usr/share/nginx/html   # Copia los archivos compilados al directorio público de nginx
  EXPOSE 80                              # Expone el puerto 80 para servir la app
  ```

> Cada línea está pensada para optimizar el tamaño, la seguridad y la velocidad de despliegue de los contenedores.

---

## 3. Creación y Despliegue de Imágenes

### Build de imágenes
```bash
docker build -t sofiioliveto/backend-app:dev .
docker build -t sofiioliveto/frontend-app:dev .
```

### Push a Docker Hub
```bash
docker push sofiioliveto/backend-app:dev
docker push sofiioliveto/frontend-app:dev
```

### Versionado
- `:dev` → versión de desarrollo
- `:v1.0` → versión estable para producción

---

## 4. Integración de base de datos

- Se eligió **MySQL 8.4** por experiencia previa y compatibilidad con Go + GORM.
- El archivo `backend/db-clients.go` se conecta a la DB usando variables de entorno:

```go
dbUsername := os.Getenv("DB_USER")
dbPassword := os.Getenv("DB_PASSWORD")
dbHost := os.Getenv("DB_HOST")
dbPort := os.Getenv("DB_PORT")
dbSchema := os.Getenv("DB_NAME")

dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local",
    dbUsername, dbPassword, dbHost, dbPort, dbSchema)
```

---

## 5. QA y PROD con misma imagen

Ambos servicios `backend-qa` y `backend-prod` utilizan la misma imagen `sofiioliveto/backend-app:dev`.

La diferenciación se logra con variables de entorno:

```yaml
environment:
  ENVIRONMENT: qa|prod
  DB_HOST: mysql-qa|mysql-prod
  DB_PORT: "3306"
  DB_NAME: db-qa|db-prod
  DB_USER: root
  DB_PASSWORD: 44898366
```

Esto permite cambiar el comportamiento sin reconstruir imágenes.

---

## 6. Entorno reproducible con Docker Compose

Creamos un `docker-compose.yml` que levanta automáticamente:

- 2 bases de datos (QA/PROD)
- 2 backend (QA/PROD)
- 1 frontend

**Características:**
- Volúmenes persistentes: `mysqlqa_data` y `mysqlprod_data`
- Redes separadas: `qa`, `prod`
- Aislamiento total entre entornos
- Persistencia al reiniciar contenedores

---

## 7. Versionado de imágenes

Una vez validada la imagen `:dev`, se generó y subió una versión etiquetada `v1.0`:

```bash
docker tag sofiioliveto/backend-app:dev sofiioliveto/backend-app:v1.0
docker push sofiioliveto/backend-app:v1.0

docker tag sofiioliveto/frontend-app:dev sofiioliveto/frontend-app:v1.0
docker push sofiioliveto/frontend-app:v1.0
```

Se actualizó `docker-compose.yml` para que use `:v1.0`.

### Estrategia de Versionado

Adoptamos **SemVer**:
- `v1.0.0`: primera versión estable
- `v1.1.0`: nueva funcionalidad backward compatible
- `v2.0.0`: cambios incompatibles o restructuración