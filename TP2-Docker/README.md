# 🐳 Proyecto Docker - Sistema de Cursos

Este proyecto contiene una aplicación completa con frontend (React), backend (Go) y base de datos (MySQL) desplegada usando Docker Compose con entornos separados de QA y Producción.

## 📋 Tabla de Contenidos

- [Construir las Imágenes](#-construir-las-imágenes)
- [Ejecutar los Contenedores](#-ejecutar-los-contenedores)
- [Acceder a la Aplicación](#-acceder-a-la-aplicación)
- [Conectarse a la Base de Datos](#-conectarse-a-la-base-de-datos)
- [Verificar que Todo Funciona](#-verificar-que-todo-funciona)

## 🔨 Construir las Imágenes

Las imágenes ya están disponibles en Docker Hub. Docker Compose las descargará automáticamente:

```bash
# Las imágenes se descargan automáticamente desde Docker Hub:
# - sofiioliveto/backend-app:dev
# - sofiioliveto/frontend-app:dev
# - mysql:8.4
```

**No necesitas construir las imágenes localmente.**

## 🚀 Ejecutar los Contenedores

### Paso 1: Levantar todos los servicios
```bash
docker-compose up -d
```

### Paso 2: Verificar que los contenedores están corriendo
```bash
docker-compose ps
```

Deberías ver todos los servicios con status "Up":
```
NAME           STATUS
backend-prod   Up
backend-qa     Up  
frontend       Up
mysql-prod     Up (healthy)
mysql-qa       Up (healthy)
```

## 🌐 Acceder a la Aplicación

### Frontend
```
🖥️ URL: http://localhost:5173
```

### Backend APIs

#### Entorno QA
```
⚙️ URL Base: http://localhost:8081
```

#### Entorno PROD  
```
🚀 URL Base: http://localhost:8082
```

### Endpoints principales:
- `GET /search` - Obtener todos los cursos
- `POST /login` - Iniciar sesión
- `POST /CreateUser` - Crear usuario
- `GET /user/:id` - Obtener usuario por ID
- `POST /createCourse` - Crear curso

## 🗄️ Conectarse a la Base de Datos

### Credenciales QA
```
Host: localhost
Puerto: 3307
Usuario: root
Contraseña: 44898366
Base de datos: dbarquisoft1
```

### Credenciales PROD
```
Host: localhost
Puerto: 3308
Usuario: root
Contraseña: 44898366
Base de datos: dbarquisoft1
```

### Conectar usando MySQL CLI:
```bash
# QA
mysql -h localhost -P 3307 -u root -p44898366 dbarquisoft1

# PROD
mysql -h localhost -P 3308 -u root -p44898366 dbarquisoft1
```

## ✅ Verificar que Todo Funciona

### 1. Verificar contenedores
```bash
docker-compose ps
```
Todos deben mostrar status "Up".

### 2. Probar frontend
Abrir en navegador: `http://localhost:5173`

### 3. Probar backend QA
```bash
curl http://localhost:8081/search
```

### 4. Probar backend PROD
```bash
curl http://localhost:8082/search
```

### 5. Verificar bases de datos
```bash
# QA
mysql -h localhost -P 3307 -u root -p44898366 -e "SHOW DATABASES;"

# PROD  
mysql -h localhost -P 3308 -u root -p44898366 -e "SHOW DATABASES;"
```

### 6. Ver logs si hay problemas
```bash
docker-compose logs
```