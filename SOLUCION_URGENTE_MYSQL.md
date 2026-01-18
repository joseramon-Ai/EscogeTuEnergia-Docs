# 🚨 SOLUCIÓN URGENTE - Error de Autenticación MySQL

## Problema Detectado

El backend NO puede conectarse a la base de datos porque el usuario `joseramon` usa el plugin de autenticación `sha256_password` o `caching_sha2_password`, que no es compatible con el cliente MySQL del contenedor.

**Error exacto:**
```
Error in connector: Error querying the database: Unknown authentication plugin `sha256_password'
```

---

## ✅ Solución (DEBE EJECUTARLO ADRIAN EN EL SERVIDOR MYSQL)

Adrian necesita ejecutar este comando en el servidor MySQL para cambiar el plugin de autenticación del usuario:

```sql
-- Conectar a MySQL como root o usuario con privilegios
mysql -u root -p

-- Cambiar el plugin de autenticación del usuario joseramon
ALTER USER 'joseramon'@'%' IDENTIFIED WITH mysql_native_password BY 'joseramon-Saturdays-2025';
FLUSH PRIVILEGES;

-- Verificar el cambio
SELECT User, Host, plugin FROM mysql.user WHERE User='joseramon';
```

**Resultado esperado:**
```
+------------+------+-----------------------+
| User       | Host | plugin                |
+------------+------+-----------------------+
| joseramon  | %    | mysql_native_password |
+------------+------+-----------------------+
```

---

## 📋 Pasos Completos

### 1. Adrian ejecuta en el servidor MySQL:

```bash
# Opción A: Si tiene acceso directo
mysql -u root -p -e "ALTER USER 'joseramon'@'%' IDENTIFIED WITH mysql_native_password BY 'joseramon-Saturdays-2025'; FLUSH PRIVILEGES;"

# Opción B: Si usa Docker
docker exec -i mysql-container mysql -u root -p -e "ALTER USER 'joseramon'@'%' IDENTIFIED WITH mysql_native_password BY 'joseramon-Saturdays-2025'; FLUSH PRIVILEGES;"

# Opción C: Si usa un contenedor específico
docker exec -i data-stack_mysql_1 mysql -u root -p -e "ALTER USER 'joseramon'@'%' IDENTIFIED WITH mysql_native_password BY 'joseramon-Saturdays-2025'; FLUSH PRIVILEGES;"
```

### 2. Después, tú reinicias el backend:

```bash
cd /home/joseramon/escogetuenergia/escogetuenergia
docker compose restart backend
docker compose logs -f backend
```

---

## 🔍 Diagnóstico Actual

```bash
# Estado de contenedores
docker ps
# ✅ Todos los contenedores están UP

# Error en logs
docker logs escogetuenergia-backend --tail 20
# ❌ Error: Unknown authentication plugin `sha256_password'

# Variable de conexión
docker exec escogetuenergia-backend bash -c 'echo $DATABASE_URL'
# ✅ mysql://joseramon:joseramon-Saturdays-2025@mysql:3306/EscogeTuEnergia
```

---

## 🚨 Alternativa Temporal (Si Adrian no puede cambiar ahora)

Si Adrian no puede ejecutar el comando inmediatamente, puedes modificar temporalmente la `DATABASE_URL` para incluir parámetros SSL:

```bash
# Editar .env
nano .env

# Cambiar:
DATABASE_URL="mysql://joseramon:joseramon-Saturdays-2025@mysql:3306/EscogeTuEnergia"

# Por (con parámetros adicionales):
DATABASE_URL="mysql://joseramon:joseramon-Saturdays-2025@mysql:3306/EscogeTuEnergia?sslaccept=strict"
```

**PERO LA SOLUCIÓN DEFINITIVA ES CAMBIAR EL PLUGIN DE AUTENTICACIÓN**

---

## 📞 Mensaje para Adrian

> Hola Adrian,
> 
> El backend no puede conectarse a MySQL porque el usuario `joseramon` usa `sha256_password`.
> 
> ¿Puedes ejecutar este comando en el servidor MySQL?
> 
> ```sql
> ALTER USER 'joseramon'@'%' IDENTIFIED WITH mysql_native_password BY 'joseramon-Saturdays-2025';
> FLUSH PRIVILEGES;
> ```
> 
> Es necesario para que Prisma pueda conectarse correctamente.
> 
> Gracias!

---

## 🔧 Verificación Post-Solución

Después de que Adrian ejecute el comando, verifica que funciona:

```bash
# 1. Reiniciar backend
docker compose restart backend

# 2. Ver logs (no debe haber errores)
docker compose logs -f backend

# 3. Probar endpoint de salud
curl http://localhost:4005/health
# Esperado: {"status":"ok"}

# 4. Probar login
curl -X POST http://localhost:4005/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"tu_email@example.com","password":"tu_password"}'
# Esperado: JSON con tokens
```

---

## 📚 Contexto Técnico

**¿Por qué pasa esto?**

- MySQL 8.0+ usa `caching_sha2_password` por defecto (más seguro)
- Prisma/Node.js requiere `mysql_native_password` para funcionar correctamente
- La solución es cambiar el plugin del usuario en MySQL

**Alternativas NO recomendadas:**
- ❌ Downgrade de MySQL a 5.7
- ❌ Cambiar cliente MySQL en el contenedor
- ✅ Cambiar plugin del usuario (SOLUCIÓN CORRECTA)

---

## ✅ Estado Final Esperado

```
┌──────────────────────────────────────────┐
│  Frontend (3005) → Backend (4005)        │
│            ↓                             │
│      MySQL (3306)                        │
│  Usuario: joseramon                      │
│  Plugin: mysql_native_password ✅        │
│  BD: EscogeTuEnergia                     │
└──────────────────────────────────────────┘
```

---

Generado: Enero 16, 2026 - 10:30  
**URGENTE:** Requiere acción de Adrian (administrador MySQL)
