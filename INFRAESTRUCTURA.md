# INFRAESTRUCTURA - Escoge tu Energía

**Documentación Completa de Infraestructura, Despliegue y Configuración**

---

## Índice

1. [Descripción General](#descripción-general)
2. [Información Crítica: Base de Datos Compartida](#información-crítica-base-de-datos-compartida)
3. [Instalación Local](#instalación-local)
4. [Variables de Entorno](#variables-de-entorno)
5. [Docker & Contenedores](#docker--contenedores)
6. [Despliegue en Producción](#despliegue-en-producción)
7. [CI/CD Pipeline](#cicd-pipeline)
8. [Monitoreo y Logs](#monitoreo-y-logs)
9. [Solución de Problemas](#solución-de-problemas)

---

## Descripción General

**Escoge tu Energía** es una aplicación full-stack con arquitectura containerizada:

- **Frontend**: Aplicación React/Vite servida por Nginx (puerto 3005)
- **Backend**: API Express con Node.js (puerto 4005)
- **Base de Datos**: MySQL 8.0+ (puerto 3306, acceso remoto via túnel)
- **Administración BD**: Adminer web (puerto 8080, desarrollo solo)

### 🏗️ Arquitectura de Despliegue

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENTE NAVEGADOR                         │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTP/HTTPS
        ┌──────────────┴──────────────┐
        │                             │
    ┌─────────────┐           ┌──────────────┐
    │   Nginx     │           │  Nginx       │
    │ Frontend    │           │ (CORS proxy) │
    │ :3005       │           │ :4005        │
    └─────────────┘           └──────────────┘
        │ React SPA                  │
        │                       ┌────┴──────┐
        │                       │            │
    ┌───┴──────────────────────────┬────────┴────────┐
    │     Docker Compose Network   │                 │
    │  ┌────────────────────────┐  │  ┌────────────┐ │
    │  │  Backend Container     │  │  │  Adminer   │ │
    │  │  (Node.js/Express)     │◄─┼─►│  (dev)     │ │
    │  └─────────┬──────────────┘  │  └────────────┘ │
    │            │ SQL             │                 │
    └────────────┼─────────────────┴─────────────────┘
                 │
        ┌────────┴─────────┐
        │                  │
    ┌──────────────────────────┐
    │  MySQL Database (Remoto) │
    │  via SSH Tunnel          │
    │  localhost:3306          │
    └──────────────────────────┘
```

---

## Información Crítica: Base de Datos Compartida

### REGLA FUNDAMENTAL

**Escoge tu Energía utiliza UNA ÚNICA base de datos compartida:**

```
Base de Datos: EscogeTuEnergia
   ├── Host: mysql (servicio Docker interno)
   ├── Puerto: 3306
   ├── Usuario: joseramon
   ├── Contraseña: ${DB_PASSWORD} (desde .env)
   └── NUNCA crear otras BD (No usar mysql_main, fork, etc.)
```

### Configuración Correcta

**Todos los docker-compose.yml deben usar:**

```yaml
environment:
  DATABASE_URL: "mysql://joseramon:${DB_PASSWORD}@mysql:3306/EscogeTuEnergia"
```

### Configuración Incorrecta (NO HACER)

```yaml
# NO usar mysql_main
DATABASE_URL: "mysql://joseramon:${DB_PASSWORD}@mysql_main:3306/EscogeTuEnergia"

# NO crear bases de datos duplicadas
# NO usar nombres diferentes (EscogeTu, Fork, Testing, etc.)
```

### 🛠️ Verificación y Limpieza

```bash
# 1. Ver qué contenedores tienes activos
docker ps -a

# 2. Listar servicios MySQL por nombre de contenedor
docker ps | grep mysql

# 3. IMPORTANTE: Eliminar contenedores duplicados/viejos
# Si ves múltiples mysql, backend, frontend:
docker stop <CONTAINER_ID>
docker rm <CONTAINER_ID>

# 4. Verificar que solo existe UNA red para data-stack
docker network ls | grep data-stack

# 5. Reconstruir limpiamente
docker compose down -v
docker network create data-stack_default
docker compose up -d
```

### 📋 Tabla de Referencia

| Proyecto | BD | Host | Puerto | Estado |
|----------|----|----|-----|----|
| **Escoge tu Energía** | `EscogeTuEnergia` | `mysql` | 3306 | ✅ **ÚNICA** |
| Otros proyectos | [BD del proyecto] | [Su host] | [Su puerto] | ⚠️ Separados |

---


### 📋 Requisitos Previos

```bash
# Sistema operativo: Linux, macOS, Windows (WSL2)
# Node.js 20+ (o usar Docker)
# Docker & Docker Compose (si prefieres containerizar)
# Git

# Verificar versiones instaladas
node --version        # v20.x.x
npm --version        # 10.x.x
docker --version     # 24.x.x
docker-compose --version  # 2.x.x
```

### 🚀 Opción 1: Instalación Manual (Sin Docker)

#### Paso 1: Clonar repositorio y preparar proyecto

```bash
cd /home/joseramon/escogetuenergia/escogetuenergia

# Instalar dependencias del Frontend
npm install

# Instalar dependencias del Backend
cd server
npm install
cd ..
```

#### Paso 2: Configurar variables de entorno

```bash
# En la raíz del proyecto, crear archivo .env
cat > .env << 'EOF'
# ========== FRONTEND ==========
API_URL=http://localhost:4005
FRONTEND_PORT=5173

# ========== BACKEND ==========
NODE_ENV=development
PORT=4005
DATABASE_URL=mysql://joseramon:tu_contraseña@localhost:3306/EscogeTuEnergia

# ========== JWT ==========
JWT_ACCESS_SECRET=tu_secret_super_seguro_aqui_32_caracteres_minimo
JWT_REFRESH_SECRET=tu_refresh_secret_super_seguro_aqui_32_caracteres_minimo

# ========== APIs EXTERNAS ==========
GOOGLE_SOLAR_API_KEY=tu_google_solar_api_key_aqui

# ========== CORS ==========
FRONTEND_APP_URL=http://localhost:5173,http://localhost:3005,http://127.0.0.1:5173
EOF
```

#### Paso 3: Configurar Base de Datos

```bash
# El proyecto usa Prisma ORM
cd server

# Generar cliente de Prisma
npx prisma generate

# (Opcional) Si ejecutas migrations locales:
# npx prisma migrate dev --name init
# Nota: El proyecto usa una BD remota via SSH tunnel

cd ..
```

#### Paso 4: Establecer túnel SSH a MySQL (si es necesario)

```bash
# En terminal separada:
ssh -L 3306:mysql-server:3306 usuario@servidor-remoto

# Mantener esta conexión abierta mientras desarrollas
```

#### Paso 5: Ejecutar en desarrollo

```bash
# En terminal 1: Frontend (React Vite dev server)
npm run dev
# ➜ Local:   http://localhost:5173

# En terminal 2: Backend (Node.js dev server)
cd server
npm run dev
# Server running on http://localhost:4005

# En terminal 3: (Opcional) Ver cambios TypeScript
npm run lint:watch
```

### 🐳 Opción 2: Instalación con Docker (Recomendado)

#### Paso 1: Preparar variables de entorno

```bash
# Crear archivos .env para diferentes configuraciones

# Configuración principal (localhost)
cat > .env.main << 'EOF'
DB_PASSWORD=tu_contraseña_mysql
JWT_ACCESS_SECRET=tu_secret_super_seguro_aqui_32_caracteres_minimo
JWT_REFRESH_SECRET=tu_refresh_secret_super_seguro_aqui_32_caracteres_minimo
GOOGLE_SOLAR_API_KEY=tu_google_solar_api_key_aqui
EOF

# Configuración fork (para testing)
cat > .env.fork << 'EOF'
DB_PASSWORD=tu_contraseña_mysql
JWT_ACCESS_SECRET=tu_secret_super_seguro_aqui_32_caracteres_minimo
JWT_REFRESH_SECRET=tu_refresh_secret_super_seguro_aqui_32_caracteres_minimo
GOOGLE_SOLAR_API_KEY=tu_google_solar_api_key_aqui
EOF

# Crear .env por defecto
cp .env.main .env
```

#### Paso 2: Asegurar que la red Docker existe

```bash
# El proyecto usa una red externa llamada 'data-stack_default'
# para conectar con MySQL remoto via túnel

# Si no existe, crearla:
docker network create data-stack_default

# Verificar que exista:
docker network ls | grep data-stack_default
```

#### Paso 3: Iniciar servicios con Docker Compose

**Opción A: Usando el script de gestión (Recomendado)**

```bash
# Hacer script ejecutable
chmod +x docker-manager.sh

# Levantar servicio principal (Backend 4005, Frontend 3005)
./docker-manager.sh main-up

# Levantar servicio fork (Backend 4010, Frontend 3006) - para testing
./docker-manager.sh fork-up

# Levantar ambos servicios
./docker-manager.sh all-up

# Ver estado de los servicios
./docker-manager.sh status

# Ver logs en tiempo real
./docker-manager.sh logs-main
./docker-manager.sh logs-fork

# Detener servicios
./docker-manager.sh main-down
./docker-manager.sh all-down

# Reconstruir desde cero (useful después de cambios en Dockerfile)
./docker-manager.sh main-build
./docker-manager.sh fork-build
```

**Opción B: Comandos Docker Compose directos**

```bash
# Construir imágenes
docker compose build

# Levantar servicios en background
docker compose up -d

# Ver logs
docker compose logs -f

# Detener servicios
docker compose down

# Reconstruir sin usar caché
docker compose build --no-cache
docker compose up -d --force-recreate
```

#### Paso 4: Verificar que todo funciona

```bash
# Comprobaciones
curl http://localhost:3005      # Frontend (Nginx)
curl http://localhost:4005      # Backend (Express)
curl http://localhost:8080      # Adminer (BD web) - solo en desarrollo

# O usando el script
./docker-manager.sh status
```

---

## Variables de Entorno

### Formato y Descripción Completa

#### Frontend - Variables de Configuración

| Variable | Descripción | Valor Ejemplo | Requerido |
|----------|------------|--------------|-----------|
| `API_URL` | URL del backend para llamadas API | `http://localhost:4005` | ✅ Sí |
| `FRONTEND_PORT` | Puerto donde escucha Nginx | `3005` | ❌ No (default: 80) |

#### Backend - Variables Críticas

| Variable | Descripción | Valor Ejemplo | Requerido |
|----------|------------|--------------|-----------|
| `NODE_ENV` | Entorno de ejecución | `production` o `development` | ✅ Sí |
| `PORT` | Puerto donde escucha Express | `4005` | ✅ Sí |
| `DATABASE_URL` | Cadena de conexión MySQL | `mysql://user:pass@host:3306/db` | ✅ Sí |

#### Base de Datos - Autenticación

| Variable | Descripción | Ejemplo | Requerido |
|----------|------------|---------|-----------|
| `DB_PASSWORD` | Contraseña usuario MySQL | `TuContraseñaSuperSegura123!` | ✅ Sí |
| `DB_HOST` | Host del servidor MySQL | `localhost` o `host.docker.internal` | ✅ Sí |
| `DB_PORT` | Puerto MySQL | `3306` | ✅ Sí |
| `DB_USER` | Usuario MySQL | `joseramon` | ✅ Sí |
| `DB_NAME` | Nombre de base de datos | `EscogeTuEnergia` **(ÚNICA)** | ✅ Sí |

#### Autenticación JWT - Claves Secretas

| Variable | Descripción | Requisitos | Requerido |
|----------|------------|----------|-----------|
| `JWT_ACCESS_SECRET` | Clave para firmar tokens de acceso | Mín. 32 caracteres, aleatorio, seguro | ✅ Sí |
| `JWT_REFRESH_SECRET` | Clave para firmar tokens de refresco | Mín. 32 caracteres, distinto a ACCESS_SECRET | ✅ Sí |

**Generar claves seguras:**

```bash
# Opción 1: Usar OpenSSL
openssl rand -base64 32

# Opción 2: Usar Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Opción 3: Usar Python
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

#### APIs Externas - Integraciones

| Variable | Descripción | Cómo Obtener | Requerido |
|----------|------------|-------------|-----------|
| `GOOGLE_SOLAR_API_KEY` | API Key para Google Solar API | [Google Cloud Console](https://console.cloud.google.com) | ✅ Sí |
| `OPENROUTER_API_KEY` | (Opcional) API Key para OpenRouter | [OpenRouter Dashboard](https://openrouter.ai/keys) | ❌ No |

#### CORS y Seguridad

| Variable | Descripción | Valor Ejemplo | Requerido |
|----------|------------|--------------|-----------|
| `FRONTEND_APP_URL` | Orígenes CORS permitidos (separados por coma) | `http://localhost:3005,http://localhost:5173,http://127.0.0.1:3005` | ✅ Sí |
| `CORS_ORIGIN` | (Alternativa) Origen CORS único | `http://localhost:3005` | ❌ No |

#### Servicios Opcionales

| Variable | Descripción | Valor Ejemplo | Requerido |
|----------|------------|--------------|-----------|
| `REDIS_URL` | (Futuro) Redis para cache y sesiones | `redis://localhost:6379` | ❌ No |
| `LOG_LEVEL` | Nivel de logging | `debug`, `info`, `warn`, `error` | ❌ No (default: info) |

### Archivos de Configuración

#### Estructura de archivos .env

```bash
# Proyecto raíz (.env y .env.main)
.env                    # Variables activas (copia de .env.main o .env.fork)
.env.main              # Variables para servicio principal
.env.fork              # Variables para servicio fork

# Backend server (para desarrollo local)
server/.env             # Variables específicas del backend (opcional)
```

#### Contenido de ejemplo (.env.complete)

```bash
# ============================================================
#         ESCOGE TU ENERGÍA - VARIABLES DE ENTORNO
# ============================================================

# ---------- ENTORNO DE EJECUCIÓN ----------
NODE_ENV=production                    # production, development, staging

# ---------- SERVIDORES ----------
PORT=4005                              # Puerto del backend
FRONTEND_PORT=3005                     # Puerto del frontend

# ---------- BASE DE DATOS ----------
DATABASE_URL="mysql://joseramon:tu_contraseña@host.docker.internal:3306/EscogeTuEnergia"
DB_PASSWORD=tu_contraseña              # Usado en docker-compose
DB_HOST=host.docker.internal           # localhost (local) o host.docker.internal (docker)
DB_PORT=3306
DB_USER=joseramon
DB_NAME=EscogeTuEnergia

# ---------- AUTENTICACIÓN JWT ----------
# ⚠️ IMPORTANTE: Usar valores seguros y únicos en producción
JWT_ACCESS_SECRET=tu_secret_super_seguro_aqui_32_caracteres_minimo
JWT_REFRESH_SECRET=tu_refresh_secret_super_seguro_aqui_32_caracteres_minimo

# Token durations
JWT_ACCESS_EXPIRES_IN=15m              # 15 minutos
JWT_REFRESH_EXPIRES_IN=7d              # 7 días

# ---------- API FRONTEND ----------
API_URL=http://localhost:4005          # URL para llamadas desde cliente

# ---------- APIS EXTERNAS ----------
GOOGLE_SOLAR_API_KEY=AIzaSy...         # Google Solar API
OPENROUTER_API_KEY=sk-or-...          # OpenRouter (opcional)

# ---------- CORS ----------
FRONTEND_APP_URL="http://localhost:3005,http://localhost:5173,http://127.0.0.1:3005"

# ---------- LOGGING ----------
LOG_LEVEL=info                         # debug, info, warn, error
MORGAN_FORMAT=combined                 # Morgan HTTP request logger format

# ---------- RATE LIMITING ----------
RATE_LIMIT_WINDOW_MS=900000            # 15 minutos
RATE_LIMIT_MAX_REQUESTS=100            # Máximo de requests por ventana

# ---------- MULTER (File Upload) ----------
MULTER_MAX_FILE_SIZE=5242880           # 5MB en bytes
MULTER_DEST=uploads                    # Directorio de uploads

# ---------- PRODUCCIÓN AVANZADO ----------
ENABLE_HTTPS=false                     # true para producción con SSL
SESSION_SECRET=tu_session_secret_aqui
REDIS_URL=redis://localhost:6379      # (Futuro)
```

### Validación y Testing de Variables

```bash
# Script para validar que todas las variables requeridas están configuradas

#!/bin/bash
# scripts/validate-env.sh

required_vars=(
  "DATABASE_URL"
  "JWT_ACCESS_SECRET"
  "JWT_REFRESH_SECRET"
  "API_URL"
  "GOOGLE_SOLAR_API_KEY"
)

missing=0
for var in "${required_vars[@]}"; do
  if [ -z "${!var}" ]; then
    echo "❌ Variable faltante: $var"
    missing=$((missing + 1))
  fi
done

if [ $missing -eq 0 ]; then
  echo "✅ Todas las variables requeridas están configuradas"
  exit 0
else
  echo "❌ Faltan $missing variables"
  exit 1
fi
```

---

## Docker & Contenedores

### 🐳 Estructura de Imágenes Docker

#### Frontend Dockerfile (Imagen Multi-stage)

```dockerfile
# Etapa 1: Build
FROM node:20-bullseye-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Etapa 2: Producción (Nginx)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# Configuración Nginx para React Router SPA
COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

EXPOSE 80
CMD ["/docker-entrypoint.sh"]
```

**Optimizaciones:**
- Multi-stage build: Solo la imagen final contiene código de producción
- Node:20-bullseye-slim: Imagen base mínima (base para build)
- Nginx:alpine: Servidor ultra-ligero para producción
- Volumen final: ~50MB (vs ~1GB con Node en producción)

#### Backend Dockerfile (Imagen Multi-stage)

```dockerfile
# Etapa 1: Build
FROM node:20-bullseye-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

# Etapa 2: Producción
FROM node:20-bullseye-slim AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/prisma ./prisma

EXPOSE 4005
CMD ["node", "dist/server.js"]
```

**Optimizaciones:**
- Prisma client generado una sola vez
- Dependencies en imagen final
- Node:20 (no slim) por binarios necesarios

### 📋 Docker Compose - Configuración

#### docker-compose.yml (Servicio Principal)

Configuración con:
- Backend en puerto 4005
- Frontend en puerto 3005
- Red externa para conexión con MySQL
- Adminer para gestión de BD en desarrollo

#### docker-compose.fork.yml (Servicio Fork para Testing)

Similar al anterior pero con puertos 4010 (backend) y 3006 (frontend).

### Ciclo de Vida de Contenedores

```bash
# Construcción
docker compose build                    # Construir imágenes
docker compose build --no-cache         # Reconstruir sin caché

# Levantamiento
docker compose up -d                    # Background
docker compose up                       # Foreground (ver logs)

# Monitoreo
docker compose ps                       # Ver estado
docker compose logs -f backend          # Logs del backend
docker compose logs -f frontend         # Logs del frontend

# Mantenimiento
docker compose exec backend sh           # Shell dentro del contenedor
docker compose exec frontend sh          # Shell dentro del contenedor

# Detención
docker compose stop                     # Parar sin eliminar datos
docker compose down                     # Parar y eliminar contenedores
docker compose down -v                  # Parar y eliminar volúmenes (⚠️ pierde datos)
```

### Seguridad en Contenedores

```yaml
# Recomendaciones en docker-compose.yml
services:
  backend:
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    user: "1000:1000"  # Ejecutar como usuario no-root
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

---

## Despliegue en Producción

### 🚀 Pasos para Poner en Producción

#### Fase 1: Preparación del Servidor

```bash
# 1. Conectarse al servidor de producción
ssh root@production-server.com

# 2. Actualizar el sistema
apt-get update && apt-get upgrade -y

# 3. Instalar dependencias
apt-get install -y \
  curl \
  wget \
  git \
  docker.io \
  docker-compose \
  nginx \
  certbot \
  python3-certbot-nginx

# 4. Iniciar servicios Docker
systemctl start docker
systemctl enable docker

# 5. Agregar usuario al grupo docker (para no usar sudo)
usermod -aG docker $USER
```

#### Fase 2: Clonar y Configurar Proyecto

```bash
# 1. Clonar repositorio
cd /var/www
git clone https://github.com/tu-usuario/escogetuenergia.git
cd escogetuenergia

# 2. Crear red Docker externa para MySQL
docker network create data-stack_default

# 3. Configurar variables de entorno (IMPORTANTE)
cat > .env.production << 'EOF'
# ⚠️ Usar valores seguros y únicos en producción
NODE_ENV=production
PORT=4005
DATABASE_URL=mysql://joseramon:contraseña_super_segura@mysql-host:3306/EscogeTuEnergia
JWT_ACCESS_SECRET=generado_con_openssl_rand_base64_32
JWT_REFRESH_SECRET=generado_con_openssl_rand_base64_32_diferente
GOOGLE_SOLAR_API_KEY=tu_google_solar_api_key
FRONTEND_APP_URL=https://tudominio.com,https://www.tudominio.com
API_URL=https://api.tudominio.com
EOF

# 4. Ajustar permisos
chmod 600 .env.production
cp .env.production .env
```

#### Fase 3: Configuración de HTTPS/SSL

```bash
# 1. Obtener certificado SSL con Let's Encrypt
certbot certonly --standalone \
  -d tudominio.com \
  -d www.tudominio.com \
  -d api.tudominio.com

# 2. Configurar renovación automática
certbot renew --dry-run

# 3. Crear config de Nginx para reverse proxy
cat > /etc/nginx/sites-available/escogetuenergia << 'EOF'
upstream backend {
    server localhost:4005;
}

upstream frontend {
    server localhost:3005;
}

server {
    listen 80;
    server_name tudominio.com www.tudominio.com api.tudominio.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name tudominio.com www.tudominio.com;

    ssl_certificate /etc/letsencrypt/live/tudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tudominio.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 443 ssl http2;
    server_name api.tudominio.com;

    ssl_certificate /etc/letsencrypt/live/tudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/tudominio.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF

# 4. Habilitar config
ln -s /etc/nginx/sites-available/escogetuenergia /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

#### Fase 4: Lanzamiento de Contenedores

```bash
# 1. Levantar servicios en producción
docker compose -f docker-compose.yml up -d

# 2. Verificar que todo está funcionando
docker compose ps
docker compose logs -f backend
docker compose logs -f frontend
```

#### Fase 5: Configuración de Backups y Monitoreo

```bash
# 1. Script de backup de base de datos (cron job)
cat > /usr/local/bin/backup-db.sh << 'EOF'
#!/bin/bash
DB_USER="joseramon"
DB_PASSWORD="contraseña"
DB_HOST="mysql-host"
DB_NAME="EscogeTuEnergia"
BACKUP_DIR="/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR
mysqldump -u$DB_USER -p$DB_PASSWORD -h$DB_HOST $DB_NAME | gzip > $BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz

# Eliminar backups más antiguos de 30 días
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
EOF

chmod +x /usr/local/bin/backup-db.sh

# 2. Configurar cron para ejecutar backup diariamente
echo "0 2 * * * /usr/local/bin/backup-db.sh" | crontab -

# 3. Monitoreo con systemd
cat > /etc/systemd/system/escogetuenergia.service << 'EOF'
[Unit]
Description=Escoge Tu Energía - Docker Services
After=docker.service
Requires=docker.service

[Service]
Type=simple
WorkingDirectory=/var/www/escogetuenergia
ExecStart=/usr/bin/docker-compose up
ExecStop=/usr/bin/docker-compose down
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable escogetuenergia.service
systemctl start escogetuenergia.service
```

### 📊 Checklist de Producción

```bash
# ✅ Pre-deployment
□ Todas las variables .env configuradas con valores seguros
□ Base de datos migrada (prisma migrate deploy)
□ HTTPS/SSL certificado instalado
□ Firewall configurado (abrir solo puertos necesarios)
□ Backups automáticos configurados
□ Monitoreo y alertas configuradas
□ Logs centralizados configurados

# ✅ Deployment
□ Imágenes Docker construidas correctamente
□ Healthchecks configurados en contenedores
□ Contenedores levantados y pasando health checks
□ Nginx proxy inverso funcionando
□ CORS correctamente configurado

# ✅ Post-deployment
□ Probar flujo completo de login
□ Probar cálculo de tarifas
□ Probar upload de facturas
□ Verificar logs sin errores
□ Pruebas de carga (load testing)
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

El proyecto incluye CI/CD automático en `.github/workflows/ci.yml`:

**Flujo de ejecución:**

```
git push to main/master
         ↓
GitHub Actions triggered
         ↓
┌────────────────────┐  ┌──────────────────┐
│ Frontend Job       │  │ Backend Job      │
│ - Checkout code    │  │ - Checkout code  │
│ - Install deps     │  │ - Install deps   │
│ - Lint check       │  │ - Lint check     │
│ - Unit tests       │  │ - Unit tests     │
│ - Build bundle     │  │ - Build dist     │
└────────────────────┘  └──────────────────┘
         ↓                      ↓
    ✅ or ❌              ✅ or ❌
         └────────────────────┘
              Notification
```

### 🚀 CD Manual (Opcional)

Para desplegar manualmente en producción:

```bash
# En servidor de producción
cd /var/www/escogetuenergia

# 1. Actualizar código
git pull origin main

# 2. Actualizar variables (si cambió algo)
# Editar .env.production

# 3. Reconstruir imágenes
docker compose build --no-cache

# 4. Reinciar servicios
docker compose down
docker compose up -d

# 5. Verificar
docker compose ps
docker compose logs -f
```

---

## Monitoreo y Logs

### 📊 Visualizar Logs de Contenedores

```bash
# Ver logs en tiempo real del backend
docker compose logs -f backend

# Ver logs en tiempo real del frontend
docker compose logs -f frontend

# Ver últimas 100 líneas
docker compose logs --tail 100 backend

# Ver logs con timestamps
docker compose logs -t backend

# Exportar logs a archivo
docker compose logs backend > backend-logs.txt
```

### 🔍 Inspeccionar Contenedores

```bash
# Conectarse a la shell del contenedor
docker compose exec backend sh
docker compose exec frontend sh

# Ver procesos dentro del contenedor
docker compose exec backend ps aux

# Ver variables de entorno
docker compose exec backend env

# Ver utilización de recursos
docker stats

# Inspeccionar detalles del contenedor
docker inspect escogetuenergia-backend
```

### 📈 Monitoreo Básico con Docker

```bash
# Script de monitoreo simple
#!/bin/bash
while true; do
    clear
    echo "=== Estado de Contenedores ==="
    docker stats --no-stream
    echo ""
    echo "=== Health Checks ==="
    docker ps --format "table {{.Names}}\t{{.Status}}"
    sleep 5
done
```

### Configurar Alertas

```bash
# Script para alertas básicas
#!/bin/bash
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

# Verificar si backend está UP
if ! curl -f http://localhost:4005/health &> /dev/null; then
    curl -X POST $WEBHOOK_URL \
      -H 'Content-Type: application/json' \
      -d '{"text":"❌ Backend está DOWN"}'
fi

# Verificar si frontend está UP
if ! curl -f http://localhost:3005 &> /dev/null; then
    curl -X POST $WEBHOOK_URL \
      -H 'Content-Type: application/json' \
      -d '{"text":"❌ Frontend está DOWN"}'
fi
```

---

## Solución de Problemas

### ❌ Problemas Comunes y Soluciones

#### 1. "Cannot GET /" en Frontend (CORS)

**Síntoma:** Frontend carga pero no puede conectar con el backend.

**Solución:**

```bash
# 1. Verificar que backend está levantado
curl http://localhost:4005

# 2. Verificar CORS en .env
echo $FRONTEND_APP_URL

# 3. Si está en Docker, verificar red
docker network ls | grep data-stack_default

# 4. Reiniciar contenedores
docker compose down
docker compose up -d --force-recreate
```

#### 2. "Connect ECONNREFUSED 127.0.0.1:3306" o "Base de datos no encontrada"

**Síntoma:** Backend no puede conectar con MySQL o conecta a una BD vieja.

**CAUSA RAÍZ:** Múltiples contenedores MySQL o DATABASE_URL apuntando a BD equivocada.

**Solución:**

```bash
# 1. PRIMERO: Limpiar contenedores duplicados
docker ps -a | grep -E "mysql|backend"
# Si ves múltiples, eliminarlos:
docker stop backend escogetuenergia-backend yoahorro_backend
docker rm backend escogetuenergia-backend yoahorro_backend

# 2. SEGUNDO: Verificar DATABASE_URL en .env
cat .env | grep DATABASE_URL
# Debe ser EXACTAMENTE:
# DATABASE_URL="mysql://joseramon:password@mysql:3306/EscogeTuEnergia"

# 3. TERCERO: Recrear red Docker
docker network rm data-stack_default 2>/dev/null
docker network create data-stack_default

# 4. CUARTO: Levantar limpiamente
docker compose down -v
docker compose build --no-cache
docker compose up -d

# 5. VERIFICAR: Entrar en el contenedor y probar
docker compose exec backend bash -c "echo $DATABASE_URL"
```

**⚠️ IMPORTANTE:** Si sigue conectando a BD vieja, es porque hay un contenedor antiguo corriendo:

```bash
# Buscar todos los contenedores, incluso detenidos
docker ps -a

# Eliminar TODOS los relacionados a proyectos anteriores
docker rm -f <CONTAINER_ID>

# Verificar que la BD "EscogeTuEnergia" existe en MySQL
mysql -h localhost -u joseramon -p -e "SHOW DATABASES;"
```


#### 3. "Port 8000/4005/4010 already in use" - Conflicto de Puertos

**Síntoma:** El puerto está ocupado por otro contenedor.

**CAUSA RAÍZ:** Múltiples versiones del mismo contenedor (backend) levantadas.

**Solución:**

```bash
# 1. PRIMERO: Ver todos los contenedores
docker ps -a

# 2. Identificar conflictos
docker ps | grep backend
docker ps | grep frontend

# Si ves MÚLTIPLES backend, frontend, mysql con nombres diferentes:
# - backend + escogetuenergia-backend
# - escogetuenergia-backend-fork + backend-fork
# ➜ PROBLEMA: Contenedores duplicados

# 3. SOLUCIÓN: Eliminar todos los contenedores de este proyecto
docker compose down -v
docker-compose -f docker-compose.fork.yml down -v
docker-compose -f docker-compose.main.yml down -v

# 4. Eliminar contenedores FANTASMA (detenidos)
docker ps -a | grep -E "backend|frontend|escoge" | awk '{print $1}' | xargs docker rm -f

# 5. Verificar puertos libres
netstat -tuln | grep -E "3005|4005|4010"
# Si hay algo, matarlo:
sudo lsof -i :4005 | grep -v COMMAND | awk '{print $2}' | xargs kill -9

# 6. Levantar SOLO el compose principal
docker compose up -d

# 7. Verificar que funciona
docker ps
docker compose logs -f
```

**Mapa de puertos esperados:**

| Servicio | Puerto | Contenedor | Estado |
|----------|--------|-----------|--------|
| Frontend | 3005 | escogetuenergia-frontend | ✅ UP |
| Backend | 4005 | escogetuenergia-backend | ✅ UP |
| Adminer | 8080 | escogetuenergia-adminer | ✅ UP |
| (Fork) Backend | 4010 | escogetuenergia-backend-fork | ❌ Detenido |
| (Fork) Frontend | 3006 | escogetuenergia-frontend-fork | ❌ Detenido |


#### 4. "ENOMEM: Out of memory"

**Síntoma:** Proceso se para por falta de memoria.

**Solución:**

```bash
# 1. Verificar uso de memoria
docker stats

# 2. Aumentar límites en docker-compose.yml
deploy:
  resources:
    limits:
      memory: 1024M

# 3. Limpiar volúmenes no usados
docker system prune -a --volumes

# 4. Reconstruir
docker compose down -v && docker compose up -d
```

#### 5. "Prisma Client not found"

**Síntoma:** Error de Prisma al iniciar backend.

**Solución:**

```bash
# 1. En el contenedor, regenerar cliente
docker compose exec backend npx prisma generate

# 2. O reconstruir la imagen
docker compose build --no-cache backend
docker compose up -d

# 3. Verificar archivo .prismarc
ls -la .prismarc
```

#### 6. "Failed to fetch" en login

**Síntoma:** Botón login no funciona.

**Solución:**

```bash
# 1. Verificar API_URL en frontend
# Debería ser http://localhost:4005 en desarrollo
# o https://api.tudominio.com en producción

# 2. Verificar FRONTEND_APP_URL en backend (CORS)
# Debe incluir el origen del frontend

# 3. Limpiar caché del navegador
# Ctrl+Shift+Delete (Chrome, Firefox)
# Cmd+Shift+Delete (Safari)

# 4. Verificar logs del backend
docker compose logs -f backend | grep -i cors

# 5. Reinicar servicios
./docker-manager.sh all-down && ./docker-manager.sh all-up
```

### 🔧 Comandos de Debugging Útiles

```bash
# Ver estado completo
./docker-manager.sh status

# Ver logs de ambos servicios
docker compose logs -f

# Acceder a base de datos (web)
# Ir a http://localhost:8080
# Server: mysql
# Usuario: joseramon
# Contraseña: (desde .env)

# Probar endpoints manualmente
curl -X POST http://localhost:4005/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}'

# Ver variables de entorno cargadas
docker compose exec backend env | grep -E "DATABASE|JWT|NODE"

# Validar JSON en .env
cat .env | grep -v "^#" | grep "="
```

### 📚 Recursos Útiles

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [Express.js Guide](https://expressjs.com/)
- [React Documentation](https://react.dev/)
- [GitHub Actions](https://docs.github.com/en/actions)

---

## 📞 Soporte y Contacto

Para problemas específicos:

1. Revisar logs: `docker compose logs -f`
2. Consultar esta documentación
3. Verificar variables de entorno
4. Contactar al equipo de devops

Última actualización: Enero 2026
- `JWT_ACCESS_SECRET`: firma de JWT de acceso (backend).
- `JWT_REFRESH_SECRET`: firma de JWT de refresco (backend).
- `GOOGLE_SOLAR_API_KEY`: clave para Google Solar API (opcional si no se usan endpoints solares).
- `VITE_API_URL`: base URL de la API para el frontend (dev/fallback).
- `VITE_GOOGLE_CLIENT_ID`: client id OAuth Google para el frontend (opcional).

### Backend local `server/.env`

- `DATABASE_URL`: URL de conexion MySQL (requerida).
- `FRONTEND_APP_URL`: lista de origins permitidos para CORS (requerida, separada por comas).
- `JWT_ACCESS_SECRET`: secreto JWT de acceso (requerida).
- `JWT_REFRESH_SECRET`: secreto JWT de refresco (requerida).
- `PORT`: puerto del backend (default 4005).
- `NODE_ENV`: entorno de ejecucion (default development).
- `ACCESS_TOKEN_EXPIRES_IN`: duracion del access token (default 15m).
- `REFRESH_TOKEN_EXPIRES_IN`: duracion del refresh token (default 30d).
- `BCRYPT_SALT_ROUNDS`: rondas de hash bcrypt (default 12).
- `ADMIN_EMAILS`: emails admin separados por coma (opcional).
- `UPLOAD_DIR`: directorio de uploads (default `uploads`).
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_CALLBACK_URL`: OAuth Google (opcional).
- `FACEBOOK_CLIENT_ID`, `FACEBOOK_CLIENT_SECRET`, `FACEBOOK_CALLBACK_URL`: OAuth Facebook (opcional).
- `GOOGLE_SOLAR_API_KEY`: clave para Google Solar API (opcional).

### Frontend en Docker (runtime)

- `API_URL`: URL de la API que se inyecta en `/config/app.json` en el contenedor Nginx.
- `FRONTEND_PORT`: puerto usado en `app.json` (default 3005).
