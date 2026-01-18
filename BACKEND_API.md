# BACKEND API - Escoge tu Energía

**Documentación Completa de la API REST**

---

##  Índice

1. [Información General](#información-general)
2. [Autenticación y Seguridad](#autenticación-y-seguridad)
3. [Listado de Endpoints](#listado-de-endpoints)
4. [Servicios Principales](#servicios-principales)
5. [Repositorios](#repositorios)
6. [Middlewares](#middlewares)
7. [Manejo de Errores](#manejo-de-errores)

---

## Información General

###  Configuración del Backend

| Aspecto | Detalle |
|---------|---------|
| **Framework** | Express 5.1.0 |
| **Lenguaje** | TypeScript 5.9.3 |
| **ORM** | Prisma 6.19.0 |
| **Base de Datos** | MySQL 8.0+ |
| **Puerto por defecto** | 4005 |
| **Base Path** | `/api` |
| **Arquitectura** | Layered (Routes → Services → Repositories) |

###  Estructura de Carpetas

```
server/src/
├── routes/          # Controladores HTTP (endpoints)
├── services/        # Lógica de negocio
├── repositories/    # Acceso a datos (Prisma)
├── middleware/      # Middlewares (auth, validación, errores)
├── schemas/         # Validación Zod
├── utils/           # Utilidades (tokens, password, transformers)
├── config/          # Configuración (env, prisma)
└── types/           # TypeScript types
```

---

## Autenticación y Seguridad

###  Sistema de Autenticación

El backend utiliza **JWT (JSON Web Tokens)** con dos tipos de tokens:

#### **1. Access Token**
- **Duración**: 15 minutos (configurable en `.env`)
- **Propósito**: Acceso a recursos protegidos
- **Almacenamiento**: Memoria del cliente (no localStorage)
- **Header**: `Authorization: Bearer <access_token>`

#### **2. Refresh Token**
- **Duración**: 7 días (configurable)
- **Propósito**: Renovar access tokens
- **Almacenamiento**: Base de datos (tabla `user_sessions`)
- **Seguridad**: Hash del token almacenado en BD

###  Estructura del Token (JWT Payload)

```typescript
{
  sub: "user-uuid",           // ID del usuario
  email: "user@example.com",  // Email del usuario
  sessionId: "session-uuid",  // ID de la sesión
  type: "access" | "refresh", // Tipo de token
  iat: 1234567890,           // Issued at (timestamp)
  exp: 1234567890            // Expiration (timestamp)
}
```

### ️ Middleware de Autenticación

#### `authenticate`

Valida el access token en el header `Authorization`.

**Flujo:**
1. Extrae el token del header `Authorization: Bearer <token>`
2. Verifica la firma JWT con `JWT_ACCESS_SECRET`
3. Valida que el usuario existe en la BD
4. Valida que la sesión existe y no ha expirado
5. Adjunta `req.user` con datos del usuario

**Uso en rutas:**
```typescript
router.get('/protected', authenticate, handler);
```

**Error Responses:**
- `401` - Token no proporcionado, inválido o expirado
- `404` - Usuario no encontrado

#### `requireAdmin`

Valida que el usuario autenticado tenga permisos de administrador.

**Criterio**: El email del usuario debe estar en la lista `ADMIN_EMAILS` (variable de entorno).

**Uso:**
```typescript
router.post('/admin-only', authenticate, requireAdmin, handler);
```

**Error Response:**
- `403` - No tiene permisos de administrador

###  Capas de Seguridad

| Capa | Tecnología | Propósito |
|------|-----------|----------|
| **Headers HTTP** | Helmet | Protección XSS, clickjacking, etc. |
| **CORS** | cors | Control de orígenes permitidos |
| **Rate Limiting** | express-rate-limit | Prevención de abuso |
| **Input Validation** | Zod | Validación de esquemas |
| **Password Hashing** | bcrypt | Hash de contraseñas (10 rounds) |
| **Token Hashing** | bcrypt | Hash de refresh tokens en BD |
| **HPP** | hpp | Protección contra parameter pollution |
| **Compression** | compression | Compresión gzip de respuestas |

###  Flujo de Autenticación

```
1. REGISTER/LOGIN
   ↓
2. Server genera Access Token (15m) + Refresh Token (7d)
   ↓
3. Refresh Token hasheado se guarda en user_sessions
   ↓
4. Cliente recibe ambos tokens
   ↓
5. Cliente usa Access Token en cada request
   ↓
6. Cuando Access Token expira (401):
   ↓
7. Cliente envía Refresh Token a /auth/refresh
   ↓
8. Server valida Refresh Token contra hash en BD
   ↓
9. Server genera nuevos tokens
   ↓
10. Cliente recibe nuevos tokens
```

###  Logout

- Cliente envía Refresh Token a `/api/auth/logout`
- Server elimina la sesión de la tabla `user_sessions`
- Tokens quedan invalidados

---

## Listado de Endpoints

###  Base URL

```
http://localhost:4005/api
```

###  Módulo: Autenticación (`/auth`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `POST` | `/auth/register` | ❌ | Registrar nuevo usuario |
| `POST` | `/auth/login` | ❌ | Login con email/password |
| `POST` | `/auth/refresh` | ❌ | Renovar tokens con refresh token |
| `POST` | `/auth/logout` | ❌ | Cerrar sesión (invalidar token) |
| `GET` | `/auth/google` | ❌ | Iniciar OAuth con Google |
| `GET` | `/auth/google/callback` | ❌ | Callback OAuth Google |
| `GET` | `/auth/facebook` | ❌ | Iniciar OAuth con Facebook |
| `GET` | `/auth/facebook/callback` | ❌ | Callback OAuth Facebook |

#### `POST /auth/register`

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "fullName": "Juan Pérez",
  "phone": "+34612345678"
}
```

**Response (201):**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "fullName": "Juan Pérez",
    "firstName": "Juan",
    "lastName": "Pérez",
    "phone": "+34612345678",
    "avatarUrl": null,
    "createdAt": "2026-01-15T10:00:00.000Z"
  },
  "tokens": {
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc..."
  }
}
```

**Errores:**
- `409` - Email ya registrado
- `400` - Datos inválidos

#### `POST /auth/login`

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response (200):**
```json
{
  "user": { /* UserResponse */ },
  "tokens": {
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc..."
  }
}
```

**Errores:**
- `401` - Credenciales inválidas

#### `POST /auth/refresh`

**Request Body:**
```json
{
  "refreshToken": "eyJhbGc..."
}
```

**Response (200):**
```json
{
  "user": { /* UserResponse */ },
  "tokens": {
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc..."
  }
}
```

**Errores:**
- `401` - Token inválido o expirado

#### `POST /auth/logout`

**Request Body:**
```json
{
  "refreshToken": "eyJhbGc..."
}
```

**Response (204):** Sin contenido

---

###  Módulo: Usuarios (`/user`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `GET` | `/user/me` | ✅ | Obtener perfil del usuario autenticado |
| `PATCH` | `/user/me` | ✅ | Actualizar perfil del usuario |
| `POST` | `/user/me/avatar` | ✅ | Subir avatar (imagen) |
| `DELETE` | `/user/me` | ✅ | Eliminar cuenta del usuario |

#### `GET /user/me`

**Headers:**
```
Authorization: Bearer <access_token>
```

**Response (200):**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "fullName": "Juan Pérez",
    "firstName": "Juan",
    "lastName": "Pérez",
    "phone": "+34612345678",
    "avatarUrl": "/uploads/avatars/uuid-123.jpg",
    "createdAt": "2026-01-15T10:00:00.000Z"
  }
}
```

#### `PATCH /user/me`

**Request Body:**
```json
{
  "firstName": "Juan Carlos",
  "lastName": "Pérez García",
  "email": "newemail@example.com",
  "phone": "+34612345679",
  "password": "NewSecurePass123!"
}
```

**Response (200):**
```json
{
  "user": { /* UserResponse actualizado */ }
}
```

#### `POST /user/me/avatar`

**Content-Type:** `multipart/form-data`

**Form Data:**
- `avatar`: Archivo de imagen (JPG, PNG, GIF)

**Límites:**
- Tamaño máximo: 5 MB
- Solo imágenes

**Response (200):**
```json
{
  "user": {
    "avatarUrl": "/uploads/avatars/uuid-1234567890.jpg"
  }
}
```

#### `DELETE /user/me`

**Response (204):** Sin contenido

**Efecto:** Eliminación en cascada de todos los datos del usuario.

---

###  Módulo: Facturas (`/bills`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `POST` | `/bills/upload` | ✅ | Subir factura (PDF o CSV) |
| `GET` | `/bills` | ✅ | Listar todas las facturas del usuario |
| `GET` | `/bills/:id` | ✅ | Obtener una factura específica |
| `DELETE` | `/bills/:id` | ✅ | Eliminar una factura |

#### `POST /bills/upload`

**Content-Type:** `multipart/form-data`

**Form Data:**
- `file`: Archivo PDF o CSV (requerido)
- `periodStart`: Fecha de inicio (opcional, formato ISO)
- `periodEnd`: Fecha de fin (opcional)
- `totalCost`: Coste total en € (opcional)
- `totalConsumptionKwh`: Consumo en kWh (opcional)
- `extractedData`: JSON con datos parseados (opcional)

**Límites:**
- Tamaño máximo: 10 MB
- Formatos: PDF, CSV

**Response (201):**
```json
{
  "bill": {
    "id": "uuid",
    "userId": "uuid",
    "filePath": "uploads/bill-123.pdf",
    "extractedData": { /* datos parseados */ },
    "periodStart": "2025-12-01T00:00:00.000Z",
    "periodEnd": "2025-12-31T23:59:59.000Z",
    "totalCost": "89.45",
    "totalConsumptionKwh": 350.5,
    "uploadedAt": "2026-01-15T10:00:00.000Z"
  }
}
```

#### `GET /bills`

**Response (200):**
```json
{
  "bills": [
    { /* BillResponse 1 */ },
    { /* BillResponse 2 */ }
  ]
}
```

#### `GET /bills/:id`

**Response (200):**
```json
{
  "bill": { /* BillResponse */ }
}
```

**Errores:**
- `404` - Factura no encontrada

#### `DELETE /bills/:id`

**Response (204):** Sin contenido

**Efecto:** Elimina el registro y el archivo físico del disco.

---

###  Módulo: Tarifas (`/tariffs`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `GET` | `/tariffs` | ❌ | Listar todas las tarifas |
| `POST` | `/tariffs` | ✅ Admin | Crear nueva tarifa |
| `PATCH` | `/tariffs/:id` | ✅ Admin | Actualizar tarifa |
| `DELETE` | `/tariffs/:id` | ✅ Admin | Eliminar tarifa |

#### `GET /tariffs`

**Response (200):**
```json
{
  "tariffs": [
    {
      "id": "uuid",
      "company": "Iberdrola",
      "planName": "Tarifa Plana",
      "priceKwh": "0.15000",
      "pricePowerKw": "0.05000",
      "isGreen": true,
      "createdAt": "2026-01-10T00:00:00.000Z"
    }
  ]
}
```

#### `POST /tariffs` (Admin)

**Request Body:**
```json
{
  "company": "Naturgy",
  "planName": "Tarifa Estable 2026",
  "priceKwh": 0.14,
  "pricePowerKw": 0.045,
  "isGreen": false
}
```

**Response (201):**
```json
{
  "tariff": { /* TariffResponse */ }
}
```

---

### ⚡ Módulo: Perfiles Energéticos (`/energy-profile`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `POST` | `/energy-profile` | ✅ | Crear perfil energético |
| `GET` | `/energy-profile` | ✅ | Obtener perfil energético |
| `PATCH` | `/energy-profile` | ✅ | Actualizar/crear perfil (upsert) |

#### `POST /energy-profile`

**Request Body:**
```json
{
  "address": "Calle Mayor 123, Madrid",
  "distributor": "UFD",
  "tariffType": "2.0TD",
  "consumptionKwhMonth": 300,
  "contractedPowerKw": 4.6,
  "monthlyBill": 85.50,
  "householdSize": "3-4 personas",
  "homeType": "piso",
  "region": "Madrid",
  "consumptionPattern": "mixto",
  "hasElectricVehicle": false,
  "hasSolarPanels": false,
  "hasHeatPump": false
}
```

**Response (201):**
```json
{
  "profile": {
    "id": "uuid",
    "userId": "uuid",
    "address": "Calle Mayor 123, Madrid",
    /* ...resto de campos */
  }
}
```

---

###  Módulo: Consumo (`/consumption`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `POST` | `/consumption/upload` | ✅ | Subir CSV de consumo |

#### `POST /consumption/upload`

**Content-Type:** `multipart/form-data`

**Form Data:**
- `file`: Archivo CSV con datos de consumo

**Formato CSV esperado:**
```csv
CUPS,Fecha,Hora,Consumo_kWh
ES0021000000000001AA,2025-12-01,00:00,0.5
ES0021000000000001AA,2025-12-01,01:00,0.3
```

**Response (201):**
```json
{
  "message": "Consumo cargado correctamente",
  "inserted": 720,
  "skipped": 0
}
```

---

###  Módulo: Sincronización Datadis (`/datadis`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `POST` | `/datadis/sync` | ✅ | Sincronizar consumo desde Datadis |

#### `POST /datadis/sync`

**Request Body:**
```json
{
  "username": "12345678A",
  "password": "myDatadisPassword"
}
```

**Proceso:**
1. Login en API de Datadis
2. Obtener CUPS del usuario
3. Descargar consumo horario de últimos 30 días
4. Guardar en tabla `hourly_consumptions`

**Response (201):**
```json
{
  "message": "Consumo sincronizado correctamente",
  "result": {
    "cups": "ES0021000000000001AA",
    "recordsInserted": 720,
    "dateRange": {
      "from": "2025-12-16",
      "to": "2026-01-15"
    }
  }
}
```

---

###  Módulo: Recomendaciones (`/recommendations`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `POST` | `/recommendations/calculate` | ✅ | Calcular mejores tarifas |
| `GET` | `/recommendations/history` | ✅ | Historial de recomendaciones |

#### `POST /recommendations/calculate`

**Request Body:**
```json
{
  "totalConsumptionKwh": 300,
  "contractedPowerKw": 4.6,
  "currentMonthlyCost": 85.50
}
```

**Response (200):**
```json
{
  "recommendations": [
    {
      "tariff": {
        "id": "uuid",
        "company": "Naturgy",
        "planName": "Tarifa Verde"
      },
      "estimatedMonthlyCost": 75.20,
      "estimatedSavings": 10.30,
      "savingsPercentage": 12.05
    }
  ]
}
```

---

### ☀️ Módulo: Solar (`/solar`)

| Método | Endpoint | Auth | Descripción |
|--------|----------|------|-------------|
| `GET` | `/solar/building-insights` | ❌ | Obtener datos solares de ubicación |

#### `GET /solar/building-insights`

**Query Parameters:**
- `lat`: Latitud (requerido)
- `lng`: Longitud (requerido)
- `peakpower`: Potencia pico instalada en kW (opcional, default: 5)
- `loss`: Pérdidas del sistema % (opcional, default: 14)
- `angle`: Ángulo de inclinación en grados (opcional, default: óptimo)
- `aspect`: Orientación en grados (opcional, default: sur)

**Ejemplo:**
```
GET /solar/building-insights?lat=40.4168&lng=-3.7038&peakpower=5
```

**Response (200):**
```json
{
  "insights": {
    "yearlyEnergyDcKwh": 7500,
    "monthlyEnergyKwh": 625,
    "dailyEnergyKwh": 20.5,
    "maxSunshineHoursPerDay": 5.2,
    "maxSunshineHoursPerYear": 1898,
    "estimatedPanelsFor5kW": 13,
    "carbonOffsetKgPerYear": 1875,
    "estimatedInstallationCost": 6500,
    "estimatedAnnualSavings": 975,
    "paybackPeriodYears": 6.7
  }
}
```

---

## Servicios Principales

###  AuthService

**Archivo:** `services/auth.service.ts`

**Responsabilidad:** Gestión completa de autenticación y sesiones.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `registerUser(payload, metadata)` | Registra un nuevo usuario, crea sesión y retorna tokens |
| `loginUser(payload, metadata)` | Autentica usuario, crea sesión y retorna tokens |
| `refreshTokens(refreshToken, metadata)` | Renueva tokens validando refresh token contra BD |
| `logout(refreshToken)` | Invalida sesión eliminando registro de BD |
| `createSessionForUser(userId, email, metadata)` | Crea sesión para OAuth (sin password) |

#### Funciones Auxiliares:

```typescript
// Genera ambos tokens JWT
issueTokens(userId, email, sessionId): { accessToken, refreshToken }

// Persiste sesión con refresh token hasheado
persistSession(sessionId, userId, refreshToken, metadata): Promise<void>

// Split nombre completo en first/last name
splitFullName(fullName): { firstName, lastName }
```

#### Flujo Interno de `loginUser`:

```typescript
1. Buscar usuario por email en BD
2. Comparar password con hash (bcrypt)
3. Generar UUID para sessionId
4. Generar access + refresh tokens (JWT)
5. Hashear refresh token (bcrypt)
6. Guardar sesión en user_sessions con hash
7. Retornar usuario + tokens al cliente
```

---

###  UserService

**Archivo:** `services/user.service.ts`

**Responsabilidad:** Gestión de perfiles de usuario.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `getProfile(userId)` | Obtiene perfil completo del usuario |
| `updateProfile(userId, payload)` | Actualiza datos del perfil |
| `deleteUser(userId)` | Elimina usuario (cascade) |

#### Lógica de `updateProfile`:

- Valida que el usuario existe
- Actualiza campos modificados (solo los enviados)
- Si se actualiza firstName o lastName, regenera fullName
- Si se proporciona password, lo hashea antes de guardar
- Retorna usuario actualizado transformado (sin password)

---

###  BillService

**Archivo:** `services/bill.service.ts`

**Responsabilidad:** Gestión de facturas eléctricas.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `createFromUpload(userId, filePath, payload)` | Crea registro de factura desde archivo subido |
| `listUserBills(userId)` | Lista todas las facturas de un usuario |
| `getUserBill(userId, billId)` | Obtiene una factura específica |
| `deleteUserBill(userId, billId)` | Elimina factura de BD y archivo físico |

#### Parser de Facturas:

- **Fechas:** Convierte strings ISO a objetos Date
- **Costes:** Convierte a `Prisma.Decimal` para precisión
- **JSON:** Parsea `extractedData` desde string JSON
- **Validación:** Lanza `AppError 400` si datos son inválidos

---

###  TariffService

**Archivo:** `services/tariff.service.ts`

**Responsabilidad:** CRUD de tarifas eléctricas.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `list()` | Lista todas las tarifas disponibles |
| `create(payload)` | Crea nueva tarifa (admin) |
| `update(id, payload)` | Actualiza tarifa existente (admin) |
| `remove(id)` | Elimina tarifa (admin) |

#### Campos Decimal:

- `priceKwh`: Precio por kWh (5 decimales de precisión)
- `pricePowerKw`: Precio de potencia por kW/día (5 decimales)

---

### ⚡ EnergyProfileService

**Archivo:** `services/energy-profile.service.ts`

**Responsabilidad:** Gestión de perfiles energéticos de usuarios.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `createProfile(userId, payload)` | Crea perfil energético (relación 1:1) |
| `getProfile(userId)` | Obtiene perfil energético del usuario |
| `upsertProfile(userId, payload)` | Crea o actualiza perfil (idempotente) |

#### Validaciones:

- `consumptionPattern`: Enum ('diurno', 'nocturno', 'mixto')
- `monthlyBill`: DECIMAL(10,2) para precisión monetaria
- Relación 1:1 con User (campo `userId` es UNIQUE)

---

###  ConsumptionService

**Archivo:** `services/consumption.service.ts`

**Responsabilidad:** Parseo e importación de consumos desde CSV.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `parseCsv(buffer, userId)` | Parsea CSV buffer a objetos de consumo |
| `saveConsumption(rows)` | Guarda consumos en BD (batch insert) |

#### Formato CSV:

```csv
CUPS,Fecha,Hora,Consumo_kWh
ES0021000000000001AA,2025-12-01,00:00,0.5
```

#### Lógica de Importación:

- Detecta duplicados por (userId, cups, date, time)
- Hace upsert para evitar duplicados
- Retorna estadísticas: `{ inserted, skipped }`

---

###  DatadisService

**Archivo:** `services/datadis.service.ts`

**Responsabilidad:** Sincronización con API de Datadis.

#### Método Principal:

```typescript
syncConsumption(userId, username, password): Promise<SyncResult>
```

#### Flujo de Sincronización:

```
1. Login en Datadis con username/password
   ↓
2. Obtener token de autorización
   ↓
3. Consultar CUPS del usuario
   ↓
4. Para cada CUPS:
   a. Obtener consumo horario últimos 30 días
   b. Parsear respuesta JSON
   c. Transformar a formato interno
   d. Guardar en hourly_consumptions
   ↓
5. Retornar resumen (CUPS, registros insertados, rango de fechas)
```

#### API Endpoints de Datadis:

- `POST /nikola-auth/tokens/login` - Autenticación
- `GET /api-private/api/get-supplies` - Obtener CUPS
- `GET /api-private/api/get-consumption-data` - Consumo horario

---

###  RecommendationService

**Archivo:** `services/recommendation.service.ts`

**Responsabilidad:** Cálculo de recomendaciones de tarifas.

#### Métodos Públicos:

| Método | Descripción |
|--------|-------------|
| `calculate(userId, input)` | Calcula mejores tarifas según consumo |
| `history(userId)` | Historial de recomendaciones guardadas |

#### Algoritmo de Matching:

```typescript
1. Obtener todas las tarifas disponibles
   ↓
2. Para cada tarifa:
   a. Calcular coste energía: consumptionKwh × priceKwh
   b. Calcular coste potencia: contractedPowerKw × pricePowerKw × 30 días
   c. Sumar costes fijos estimados
   d. Total = coste_energía + coste_potencia + fijos
   ↓
3. Calcular ahorro vs tarifa actual:
   ahorro = currentMonthlyCost - estimatedCost
   ↓
4. Ordenar por mayor ahorro
   ↓
5. Retornar top 5 recomendaciones
```

---

### ☀️ SolarService

**Archivo:** `services/solar.service.ts`

**Responsabilidad:** Integración con API de PVGIS (solar).

#### Método Principal:

```typescript
fetchBuildingInsights(params): Promise<SolarInsights>
```

#### API Externa:

- **Proveedor:** PVGIS (Photovoltaic Geographical Information System)
- **URL Base:** `https://re.jrc.ec.europa.eu/api/v5_2/`
- **Endpoint:** `/PVcalc`

#### Parámetros:

- `lat`, `lng`: Coordenadas geográficas
- `peakpower`: Potencia pico en kW (default: 5)
- `loss`: Pérdidas del sistema % (default: 14)
- `angle`: Ángulo de inclinación (default: óptimo)
- `aspect`: Orientación (default: sur, 0°)

#### Cálculos:

```typescript
// Energía anual producida
yearlyEnergyDcKwh = PVGIS_totals.E_y

// Ahorro estimado (€/año)
annualSavings = yearlyEnergyKwh × precioMedioKwh

// CO2 evitado (kg/año)
co2SavedKg = yearlyEnergyKwh × 0.25 // factor España

// Retorno de inversión
paybackYears = installationCost / annualSavings
```

---

## Repositorios

Los repositorios abstraen el acceso a datos vía Prisma. Todos siguen el patrón:

```typescript
export const entityRepository = {
  create(data): Promise<Entity>
  findById(id): Promise<Entity | null>
  findAll(): Promise<Entity[]>
  update(id, data): Promise<Entity>
  delete(id): Promise<void>
}
```

###  Repositorios Disponibles:

| Archivo | Entidad | Responsabilidad |
|---------|---------|-----------------|
| `user.repository.ts` | User | CRUD usuarios |
| `session.repository.ts` | UserSession | CRUD sesiones JWT |
| `bill.repository.ts` | Bill | CRUD facturas |
| `tariff.repository.ts` | Tariff | CRUD tarifas |
| `energy-profile.repository.ts` | UserEnergyProfile | CRUD perfiles energéticos |
| `hourly-consumption.repository.ts` | HourlyConsumption | CRUD consumo horario |
| `monthly-consumption.repository.ts` | MonthlyConsumption | CRUD consumo mensual |

### Ejemplo: UserRepository

```typescript
export const userRepository = {
  async create(data: Prisma.UserCreateInput) {
    return prisma.user.create({ data });
  },
  
  async findById(id: string) {
    return prisma.user.findUnique({ where: { id } });
  },
  
  async findByEmail(email: string) {
    return prisma.user.findUnique({ where: { email } });
  },
  
  async update(id: string, data: Prisma.UserUpdateInput) {
    return prisma.user.update({ where: { id }, data });
  },
  
  async delete(id: string) {
    return prisma.user.delete({ where: { id } });
  }
}
```

---

## Middlewares

###  `authenticate`

**Archivo:** `middleware/auth.ts`

**Propósito:** Valida JWT access token y adjunta usuario a `req.user`.

**Flujo:**
1. Extrae token del header `Authorization: Bearer <token>`
2. Verifica firma JWT con `JWT_ACCESS_SECRET`
3. Valida usuario existe en BD
4. Valida sesión existe y no expiró
5. Adjunta `req.user = { id, email, sessionId }`

**Uso:**
```typescript
app.get('/protected', authenticate, handler);
```

---

###  `requireAdmin`

**Archivo:** `middleware/auth.ts`

**Propósito:** Valida que usuario tiene permisos de admin.

**Criterio:** Email del usuario en `ADMIN_EMAILS` (env).

**Uso:**
```typescript
app.post('/admin', authenticate, requireAdmin, handler);
```

---

### ✅ `validate(schema)`

**Archivo:** `middleware/validate.ts`

**Propósito:** Valida request body contra esquema Zod.

**Uso:**
```typescript
import { loginSchema } from '../schemas/auth.schema';

app.post('/login', validate(loginSchema), handler);
```

**Errores:**
- `400` - Datos inválidos (incluye detalles de validación Zod)

---

### ❌ `errorHandler`

**Archivo:** `middleware/error-handler.ts`

**Propósito:** Manejo global de errores.

**Tipos de errores:**

```typescript
// Error personalizado de aplicación
class AppError extends Error {
  statusCode: number;
  isOperational: boolean;
}

// Errores de Prisma
PrismaClientKnownRequestError
PrismaClientValidationError

// Errores JWT
JsonWebTokenError
TokenExpiredError

// Errores de validación Zod
ZodError
```

**Formato de respuesta:**
```json
{
  "error": "Mensaje de error",
  "code": "ERROR_CODE",
  "statusCode": 400
}
```

---

###  `asyncHandler`

**Archivo:** `utils/async-handler.ts`

**Propósito:** Wrapper para handlers async (evita try-catch).

**Uso:**
```typescript
app.get('/endpoint', asyncHandler(async (req, res) => {
  const data = await someAsyncOperation();
  res.json(data);
}));
```

---

## Manejo de Errores

###  Clase AppError

```typescript
class AppError extends Error {
  statusCode: number;
  isOperational: boolean;
  
  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}
```

### Códigos de Estado HTTP

| Código | Significado | Uso |
|--------|-------------|-----|
| `200` | OK | Request exitoso |
| `201` | Created | Recurso creado exitosamente |
| `204` | No Content | Operación exitosa sin respuesta |
| `400` | Bad Request | Datos inválidos |
| `401` | Unauthorized | No autenticado o token inválido |
| `403` | Forbidden | No tiene permisos |
| `404` | Not Found | Recurso no encontrado |
| `409` | Conflict | Conflicto (ej: email duplicado) |
| `500` | Internal Server Error | Error del servidor |

### Formato de Error

```json
{
  "error": "Descripción del error",
  "code": "CODIGO_ERROR",
  "statusCode": 400,
  "details": { /* detalles adicionales */ }
}
```

### Ejemplos de Errores Comunes

```json
// 401 - No autorizado
{
  "error": "No autorizado",
  "statusCode": 401
}

// 409 - Email duplicado
{
  "error": "El correo ya esta registrado",
  "statusCode": 409
}

// 400 - Validación Zod
{
  "error": "Datos invalidos",
  "statusCode": 400,
  "details": {
    "email": "Email invalido",
    "password": "La contraseña debe tener al menos 8 caracteres"
  }
}

// 404 - Recurso no encontrado
{
  "error": "Usuario no encontrado",
  "statusCode": 404
}
```

---

##  Utilidades

###  Tokens (JWT)

**Archivo:** `utils/tokens.ts`

```typescript
// Genera access token (15 min)
signAccessToken(payload): string

// Genera refresh token (7 días)
signRefreshToken(payload): string

// Verifica access token
verifyAccessToken(token): TokenPayload

// Verifica refresh token
verifyRefreshToken(token): TokenPayload
```

---

###  Password

**Archivo:** `utils/password.ts`

```typescript
// Hash password con bcrypt (10 rounds)
hashPassword(password): Promise<string>

// Compara password con hash
comparePassword(password, hash): Promise<boolean>

// Hash token (refresh token)
hashToken(token): Promise<string>

// Compara token con hash
compareTokenHash(token, hash): Promise<boolean>
```

---

###  Transformers

**Archivo:** `utils/transformers.ts`

Funciones para transformar entidades Prisma a respuestas API (omitir campos sensibles).

```typescript
// Omite passwordHash, retorna UserResponse
toUserResponse(user): UserResponse

// Formatea factura
toBillResponse(bill): BillResponse

// Formatea tarifa
toTariffResponse(tariff): TariffResponse

// Formatea perfil energético
toEnergyProfileResponse(profile): EnergyProfileResponse
```

---

##  Resumen de Arquitectura

```
┌─────────────────────────────────────────────────┐
│              HTTP REQUEST                        │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│          MIDDLEWARES                             │
│  - authenticate                                  │
│  - requireAdmin                                  │
│  - validate(schema)                             │
│  - errorHandler                                  │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│          ROUTES (Controllers)                    │
│  - authRouter                                    │
│  - userRouter                                    │
│  - billsRouter                                   │
│  - tariffsRouter                                 │
│  - consumptionRouter                            │
│  - datadisRouter                                │
│  - recommendationsRouter                        │
│  - energyProfileRouter                          │
│  - solarRouter                                  │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│          SERVICES (Business Logic)               │
│  - authService                                   │
│  - userService                                   │
│  - billService                                   │
│  - tariffService                                 │
│  - consumptionService                           │
│  - datadisService                               │
│  - recommendationService                        │
│  - energyProfileService                         │
│  - solarService                                 │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│          REPOSITORIES (Data Access)              │
│  - userRepository                                │
│  - sessionRepository                             │
│  - billRepository                                │
│  - tariffRepository                              │
│  - energyProfileRepository                      │
│  - hourlyConsumptionRepository                  │
│  - monthlyConsumptionRepository                 │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│          PRISMA CLIENT (ORM)                     │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│          MySQL DATABASE                          │
└─────────────────────────────────────────────────┘
```

---

##  Referencias

- [Express Documentation](https://expressjs.com/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [JWT RFC 7519](https://tools.ietf.org/html/rfc7519)
- [Zod Documentation](https://zod.dev/)
- [bcrypt NPM](https://www.npmjs.com/package/bcrypt)

---

**Documento generado:** Enero 2026  
**Versión API:** 1.0.0  
**Última actualización:** 2026-01-15
