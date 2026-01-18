# 📥 Instrucciones de Descarga - Documentación Completa

## 📦 Archivo Comprimido Disponible

Se ha generado un archivo `documentacion_escogetuenergia.tar.gz` (47 KB) que contiene:

### 📄 Documentos Incluidos

1. ✅ **README_TECNICO.md** (38 KB)
   - Stack tecnológico completo
   - Arquitectura de alto nivel
   - Estructura del proyecto
   - Guía de levantamiento de servicios

2. ✅ **BASE_DE_DATOS.md** (30 KB)
   - Esquema ER en Mermaid
   - Diccionario de datos (15 tablas)
   - Tipos de datos y relaciones
   - Índices y políticas de cascada

3. ✅ **BACKEND_API.md** (33 KB)
   - Lista completa de endpoints
   - Servicios y clases principales
   - Autenticación JWT (10-step diagram)
   - Ejemplos de request/response

4. ✅ **FRONTEND.md** (32 KB)
   - Jerarquía de componentes
   - Gestión de estado (Context, React Query, useState)
   - Rutas del cliente (16 rutas)
   - Catálogo de 30+ componentes UI

5. ✅ **INFRAESTRUCTURA.md** (31 KB) - **ACTUALIZADO**
   - ⚠️ Sección crítica: BD compartida
   - Instalación local (opciones manual y Docker)
   - Variables de entorno documentadas
   - Docker & Contenedores
   - Despliegue en producción
   - CI/CD Pipeline
   - Monitoreo y Logs
   - **Solución de problemas MEJORADA**

6. ✅ **docker-compose.yml** - ACTUALIZADO
   - Corregido para usar BD "EscogeTuEnergia"
   - Comentarios clarificadores

7. ✅ **docker-compose.fork.yml** - ACTUALIZADO
   - Corregido mysql_main → mysql
   - Ahora apunta a BD correcta

8. ✅ **docker-compose.main.yml** - Incluido
   - Referencia con túnel local

---

## 🔴 Cambios Importantes Realizados

### 1. **Base de Datos Compartida**
- Todos los docker-compose ahora usan ÚNICA BD: `EscogeTuEnergia`
- ❌ Eliminadas referencias a `mysql_main` y bases duplicadas
- ✅ Consistencia garantizada en todas las configuraciones

### 2. **Sección Nueva en INFRAESTRUCTURA.md**
- ⚠️ **"Información Crítica: Base de Datos Compartida"**
- Regla fundamental y tabla de referencia
- Comandos de limpieza para eliminar contenedores fantasma

### 3. **Solución de Problemas Mejorada**
- Problema 2: Conflicto de BD (actualizado completamente)
- Problema 3: Conflicto de puertos (expandido)
- Comandos específicos para eliminar duplicados

---

## 📥 Cómo Descargar

### Opción 1: Descargar archivo comprimido (Recomendado)

```bash
# El archivo está en:
# /home/joseramon/escogetuenergia/escogetuenergia/documentacion_escogetuenergia.tar.gz

# Para extraer en tu equipo:
tar -xzf documentacion_escogetuenergia.tar.gz -C "D:\JOSE RAMON GARCIA RUIZ\UOC\07 SEMESTRE (09-2025)\Practicas\PEC 3\Entrega\readme\"
```

### Opción 2: Copiar archivos individuales manualmente

Los archivos están disponibles en:
```
/home/joseramon/escogetuenergia/escogetuenergia/
├── README_TECNICO.md
├── BASE_DE_DATOS.md
├── BACKEND_API.md
├── FRONTEND.md
├── INFRAESTRUCTURA.md
├── docker-compose.yml
├── docker-compose.fork.yml
└── docker-compose.main.yml
```

---

## 📋 Checklist de Implementación

Antes de usar la documentación, sigue estos pasos:

### 1️⃣ **Limpieza Inicial**

```bash
# Eliminar contenedores duplicados
docker ps -a | grep -E "backend|frontend|mysql"
# Ver qué hay y decidir cuáles eliminar

# Limpiar contenedores viejos
docker compose down -v
docker ps -a | grep escoge | awk '{print $1}' | xargs docker rm -f
```

### 2️⃣ **Verificar Variables de Entorno**

```bash
# El archivo .env debe contener:
cat .env | grep -E "DB_PASSWORD|DATABASE_URL|JWT|GOOGLE_SOLAR"
```

Ejemplo correcto:
```env
DB_PASSWORD=tu_contraseña_real
DATABASE_URL="mysql://joseramon:${DB_PASSWORD}@mysql:3306/EscogeTuEnergia"
JWT_ACCESS_SECRET=tu_secret_aqui_32_chars_min
JWT_REFRESH_SECRET=tu_refresh_secret_aqui_32_chars_min
GOOGLE_SOLAR_API_KEY=tu_google_key_aqui
```

### 3️⃣ **Levantar Servicios**

```bash
# PRIMERO: Crear red si no existe
docker network create data-stack_default 2>/dev/null

# SEGUNDO: Levantar
docker compose up -d

# TERCERO: Verificar
docker compose logs -f
docker ps

# CUARTO: Probar
curl http://localhost:4005        # Backend
curl http://localhost:3005        # Frontend
curl http://localhost:8080        # Adminer
```

### 4️⃣ **Validar Conexión BD**

```bash
# Ver si el backend conecta correctamente
docker compose logs backend | grep -i database

# Entrada a Adminer (web):
# http://localhost:8080
# Server: mysql
# Usuario: joseramon
# Contraseña: (desde .env)
```

---

## 🚨 Problemas Conocidos y Soluciones Rápidas

### Problema: "Backend no arranca"
**Solución:** Ver logs
```bash
docker compose logs backend
# Buscar línea con DATABASE_URL
# Si dice mysql_main → CAMBIAR A mysql
```

### Problema: "Frontend conecta a BD vieja"
**Solución:** 
```bash
# 1. Limpiar todo
docker compose down -v
docker ps -a | xargs docker rm -f

# 2. Recrear
docker network create data-stack_default
docker compose up -d --force-recreate
```

### Problema: "Puerto 4005/3005 ocupado"
**Solución:**
```bash
sudo lsof -i :4005
kill -9 <PID>
docker compose up -d
```

---

## 📞 Resumen de Cambios en la Documentación

| Documento | Cambios | Estado |
|-----------|---------|--------|
| **INFRAESTRUCTURA.md** | ✅ Sección BD compartida añadida | Actualizado |
| **docker-compose.yml** | ✅ Comentarios clarificadores | Actualizado |
| **docker-compose.fork.yml** | ✅ Corregido mysql_main → mysql | Actualizado |
| **README_TECNICO.md** | ℹ️ Sin cambios (válido) | Vigente |
| **BASE_DE_DATOS.md** | ℹ️ Sin cambios (válido) | Vigente |
| **BACKEND_API.md** | ℹ️ Sin cambios (válido) | Vigente |
| **FRONTEND.md** | ℹ️ Sin cambios (válido) | Vigente |

---

## 📥 Instrucciones de Copia Manual a tu Equipo

Si no puedes descargar el .tar.gz, copia estos archivos manualmente:

**Desde Linux (servidor):**
```bash
# Listar archivos
ls -lh /home/joseramon/escogetuenergia/escogetuenergia/*.md

# Copiar a tu carpeta local (si tienes acceso):
scp /home/joseramon/escogetuenergia/escogetuenergia/*.md user@tu-equipo:/destino/
```

**O manualmente:**
1. Abre cada archivo .md en el editor
2. Copia su contenido completo
3. Crea el archivo en tu equipo local

---

## ✅ Validación Final

Antes de considera que todo está listo:

```bash
# 1. Verifica que backend conecta a BD correcta
docker compose exec backend npx prisma db push

# 2. Verifica que frontend carga
curl -s http://localhost:3005 | head -20

# 3. Verifica autenticación
curl -X POST http://localhost:4005/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"test"}'

# Esperado: Respuesta JSON con tokens o error 401
```

---

## 📝 Resumen Ejecutivo

✅ **Base de Datos:** UNA ÚNICA BD "EscogeTuEnergia"
✅ **Documentación:** 5 archivos .md + 3 docker-compose.yml
✅ **Limpieza:** Todos los conflictos de puertos identificados
✅ **Soluciones:** Guía completa de troubleshooting actualizada

**Total de líneas documentadas:** 4,500+ líneas
**Documentos generados:** 8 archivos
**Estado:** Listo para producción ✅

---

Última actualización: Enero 16, 2026
Generado por: GitHub Copilot
