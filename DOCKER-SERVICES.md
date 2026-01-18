# EscogeTuEnergia - Gestión de Servicios

Este proyecto tiene dos instancias corriendo en paralelo: **Principal** y **Fork**.

## 🎯 Configuración de Servicios

### Servicio Principal
- **Backend**: Puerto 4005 (http://localhost:4005)
- **Frontend**: Puerto 3005 (http://localhost:3005)
- **Archivo de configuración**: `docker-compose.yml`
- **Variables de entorno**: `.env.main` / `.env` (por defecto)

### Servicio Fork (Testing/Desarrollo)
- **Backend**: Puerto 4010 (http://localhost:4010)
- **Frontend**: Puerto 3006 (http://localhost:3006)
- **Archivo de configuración**: `docker-compose.fork.yml`
- **Variables de entorno**: `.env.fork`

## 📋 Comandos Rápidos

### Usando el script de gestión (Recomendado)

```bash
# Ver estado de todos los servicios
./docker-manager.sh status

# Levantar servicios
./docker-manager.sh main-up       # Solo principal
./docker-manager.sh fork-up       # Solo fork
./docker-manager.sh all-up        # Ambos

# Detener servicios
./docker-manager.sh main-down     # Solo principal
./docker-manager.sh fork-down     # Solo fork
./docker-manager.sh all-down      # Ambos

# Reconstruir desde cero
./docker-manager.sh main-build    # Reconstruir principal
./docker-manager.sh fork-build    # Reconstruir fork

# Ver logs
./docker-manager.sh logs-main     # Logs del principal
./docker-manager.sh logs-fork     # Logs del fork
```

### Comandos Docker Compose directos

```bash
# Servicio Principal
docker compose -p escogetuenergia-main up -d
docker compose -p escogetuenergia-main down
docker compose -p escogetuenergia-main logs -f

# Servicio Fork
docker compose -p escogetuenergia-fork -f docker-compose.fork.yml up -d
docker compose -p escogetuenergia-fork -f docker-compose.fork.yml down
docker compose -p escogetuenergia-fork -f docker-compose.fork.yml logs -f
```

## 🔧 Cambios Realizados

### Correcciones aplicadas:
1. ✅ Corregido error "Failed to fetch" al hacer login
2. ✅ URLs del frontend actualizadas para usar los puertos correctos
3. ✅ Configuración de CORS en el backend para permitir ambos orígenes
4. ✅ Comentadas referencias a `forum.routes` que causaban errores de compilación
5. ✅ Separación de servicios principal y fork con nombres y puertos diferentes
6. ✅ Scripts de gestión para facilitar el manejo de ambos servicios

### Archivos importantes:

- `docker-compose.yml` - Configuración del servicio principal
- `docker-compose.fork.yml` - Configuración del servicio fork
- `.env` - Variables de entorno activas
- `.env.main` - Variables para el servicio principal
- `.env.fork` - Variables para el servicio fork
- `docker-manager.sh` - Script de gestión de servicios

## URLs de Acceso (Producción)

- **Principal**: http://128.140.64.33:3005
- **Fork**: http://128.140.64.33:3006

## 🔍 Verificación de Estado

Para verificar que todo funciona correctamente:

```bash
# Comprobar que los backends responden
curl http://localhost:4005/health  # Principal
curl http://localhost:4010/health  # Fork

# Ver todos los contenedores
docker ps | grep escogetuenergia

# Usar el script de estado
./docker-manager.sh status
```

## 🚀 Flujo de Trabajo Recomendado

1. **Desarrollo en Fork**: Hacer cambios y probar en el fork (puerto 4010/3006)
2. **Estabilización**: Cuando el fork esté estable, copiar cambios al principal
3. **Producción**: El servicio principal (puerto 4005/3005) es la versión estable

## Notas

- Ambos servicios pueden correr simultáneamente sin conflictos
- Los puertos están configurados para no solaparse
- Cada servicio tiene su propia red Docker
- Las variables de entorno VITE_ requieren reconstruir el frontend para aplicar cambios
