# README TÉCNICO - Escoge tu Energía

**Documento de Arquitectura y Stack Técnico**

---

## 📋 Índice

1. [Tech Stack](#tech-stack)
2. [Arquitectura de Alto Nivel](#arquitectura-de-alto-nivel)
3. [Estructura del Proyecto](#estructura-del-proyecto)
4. [Modelos de Datos](#modelos-de-datos)
5. [Patrones Arquitectónicos](#patrones-arquitectónicos)
6. [Infraestructura y DevOps](#infraestructura-y-devops)

---

## Tech Stack

### 🎯 Frontend

| Categoría | Tecnología | Versión | Propósito |
|-----------|------------|---------|----------|
| **Framework** | React | 18.3.1 | Framework UI principal |
| **Build Tool** | Vite | 5.4.19 | Bundler rápido y optimizado |
| **Lenguaje** | TypeScript | 5.8.3 | Tipado estático |
| **Routing** | React Router | 6.30.1 | Navegación y ruteo SPA |
| **Estado Global** | React Query | 5.83.0 | Gestión de datos asíncronos |
| **Formularios** | React Hook Form | 7.61.1 | Validación y gestión de formularios |
| **Validación** | Zod | 3.25.76 | Validación de esquemas TypeScript |
| **UI Components** | shadcn/ui + Radix UI | 1.x | Componentes accesibles sin estilos |
| **Estilos** | Tailwind CSS | 3.4.17 | Utility-first CSS framework |
| **Iconos** | Lucide React | 0.462.0 | Librería de iconos modular |
| **Gráficos** | Recharts | 2.15.4 | Gráficos React responsivos |
| **Drag & Drop** | dnd-kit | 6.3.1 | Librería drag and drop moderna |
| **Notificaciones** | Sonner | 1.7.4 | Toast notifications |
| **Temas** | next-themes | 0.3.0 | Gestión de temas (light/dark) |
| **Testing** | Vitest + Testing Library | 2.1.8 / 16.2.0 | Unit & integration tests |

### 🔧 Backend

| Categoría | Tecnología | Versión | Propósito |
|-----------|------------|---------|----------|
| **Runtime** | Node.js | 20+ | Entorno de ejecución JavaScript |
| **Framework Web** | Express | 5.1.0 | Framework HTTP minimalista |
| **ORM** | Prisma | 6.19.0 | ORM type-safe para acceso a datos |
| **Base de Datos** | MySQL | 8.0+ | Base de datos relacional |
| **Lenguaje** | TypeScript | 5.9.3 | Tipado estático |
| **Validación** | Zod | 4.1.12 | Validación de esquemas |
| **Autenticación** | JWT (jsonwebtoken) | 9.0.2 | Token-based authentication |
| **Encriptación** | bcrypt | 6.0.0 | Hashing de contraseñas |
| **CORS** | cors | 2.8.5 | Control de acceso cross-origin |
| **Seguridad** | Helmet | 8.1.0 | Headers de seguridad HTTP |
| **Seguridad** | hpp | 0.2.3 | Protección contra HTTP Parameter Pollution |
| **Rate Limiting** | express-rate-limit | 7.5.0 | Limitación de tasa de solicitudes |
| **Compresión** | compression | 1.8.0 | Compresión de respuestas |
| **Logging** | Morgan | 1.10.1 | HTTP request logger |
| **Carga de archivos** | Multer | 2.0.2 | Middleware para subida de archivos |
| **HTTP Client** | Axios | 1.13.2 | Cliente HTTP para integraciones |
| **Parsing CSV** | csv-parse | 5.5.6 | Parseo de archivos CSV |
| **Fechas** | dayjs | 1.11.19 | Manipulación de fechas (alternativa Moment.js) |
| **Testing** | Vitest | 2.1.8 | Test runner compatible con Jest |

### 🏗️ DevOps & Herramientas

| Herramienta | Versión | Propósito |
|-------------|---------|----------|
| **Linting** | ESLint + TypeScript ESLint | 9.32.0 | Análisis estático de código |
| **Pre-commit** | Husky | 9.1.7 | Git hooks automation |
| **Staged Linting** | lint-staged | 15.2.10 | Lint solo archivos staged |
| **CI/CD** | GitHub Actions | - | Integración continua y deployment |
| **PostCSS** | PostCSS + Autoprefixer | 8.5.6 | Procesamiento de CSS |
| **Containers** | Docker + Docker Compose | - | Orquestación de servicios (opcional) |

---

## Arquitectura de Alto Nivel

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (NAVEGADOR)                        │
│                         React 18 + Vite                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │   Pages (Lazy)   │  │   Components     │  │   Services   │  │
│  │                  │  │   (shadcn/ui)    │  │   (API Call) │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
│          │                      │                     │           │
│          └──────────────────────┼─────────────────────┘           │
│                                 │                                 │
│                          ┌──────▼──────┐                         │
│                          │  React Query │ (Data Fetching)        │
│                          └──────┬──────┘                         │
│                                 │                                 │
└─────────────────────────────────┼─────────────────────────────────┘
                                  │
                    ┌─────────────▼──────────────┐
                    │   HTTP Requests (Axios)    │
                    │   REST API Endpoints       │
                    └─────────────┬──────────────┘
                                  │
┌─────────────────────────────────▼─────────────────────────────────┐
│                   API SERVER (Express + Node.js)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    ROUTES (Controllers)                    │ │
│  │  /api/users  /api/bills  /api/tariffs  /api/matching...  │ │
│  └────────────────────┬─────────────────────────────────────┘ │
│                       │                                         │
│  ┌────────────────────▼─────────────────────────────────────┐ │
│  │          MIDDLEWARE (Security & Validation)             │ │
│  │  - Helmet (Security Headers)  - CORS  - Rate Limiting  │ │
│  │  - JWT Auth  - Input Validation (Zod)  - Compression  │ │
│  └────────────────────┬─────────────────────────────────────┘ │
│                       │                                         │
│  ┌────────────────────▼─────────────────────────────────────┐ │
│  │        SERVICES (Business Logic Layer)                   │ │
│  │  - UserService     - BillService    - MatchingService   │ │
│  │  - TariffService   - GamificationService  - etc...      │ │
│  └────────────────────┬─────────────────────────────────────┘ │
│                       │                                         │
│  ┌────────────────────▼─────────────────────────────────────┐ │
│  │   REPOSITORIES (Data Access Layer)                       │ │
│  │   - UserRepository  - BillRepository  - TariffRepo...   │ │
│  │   (Abstracción de Prisma)                              │ │
│  └────────────────────┬─────────────────────────────────────┘ │
│                       │                                         │
└───────────────────────┼─────────────────────────────────────────┘
                        │
        ┌───────────────▼───────────────┐
        │   Prisma Client (ORM)          │
        │   Type-Safe Database Access    │
        └───────────────┬───────────────┘
                        │
        ┌───────────────▼───────────────┐
        │    MySQL Database 8.0+        │
        │  (user, bills, tariffs,       │
        │   gamification, reviews, ...) │
        └───────────────────────────────┘
```

### Flujo de Datos

1. **Usuario Interactúa** con la UI (React Components)
2. **React Query** gestiona las peticiones y caché
3. **Axios** realiza HTTP requests al backend
4. **Express Routes** reciben y direccionan las peticiones
5. **Middlewares** validan seguridad y entrada
6. **Services** procesan la lógica de negocio
7. **Repositories** acceden a los datos vía Prisma
8. **MySQL** almacena y recupera datos
9. **Response** viaja de vuelta al cliente

### Capas de Seguridad

- **Helmet**: Headers de seguridad HTTP
- **CORS**: Control de origen en las peticiones
- **Rate Limiting**: Prevención de abuso
- **JWT**: Autenticación token-based
- **bcrypt**: Hashing de contraseñas
- **Zod**: Validación de esquemas
- **HPP**: Protección contra HTTP Parameter Pollution
- **Compression**: Reducción de tamaño de respuestas

---

## Estructura del Proyecto

### 🎨 Frontend: `src/`

```
src/
├── main.tsx                    # Punto de entrada React
├── App.tsx                     # Componente raíz
├── App.css                     # Estilos globales
├── index.css                   # Reset y utilidades CSS
├── vite-env.d.ts              # Tipos Vite
│
├── app/                        # Configuración de aplicación
│   ├── router.tsx             # Definición de rutas (lazy loading)
│   ├── hooks/                 # Hooks personalizados globales
│   │   ├── use-app-config.ts
│   │   ├── use-mobile.tsx
│   │   └── use-toast.ts
│   ├── layouts/               # Layouts compartidos
│   │   └── PageLayout.tsx
│   └── providers/             # Contexto y providers globales
│       ├── AuthProvider.tsx
│       └── QueryClientProvider.tsx
│
├── components/                 # Componentes reutilizables
│   ├── ui/                    # Componentes base (shadcn/ui)
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── dialog.tsx
│   │   ├── form.tsx
│   │   ├── card.tsx
│   │   └── ... (20+ componentes shadcn)
│   │
│   ├── common/                # Componentes comunes de negocio
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── Navbar.tsx
│   │   └── ...
│   │
│   ├── games/                 # Componentes del módulo gamificación
│   │   ├── RewardCard.tsx
│   │   ├── BadgeDisplay.tsx
│   │   └── LeaderboardCard.tsx
│   │
│   ├── auth-modal.tsx         # Modal de autenticación
│   ├── hero-section.tsx       # Sección hero landing
│   ├── calculator-section.tsx # Sección calculadora
│   ├── market-insights.tsx    # Panel de insights de mercado
│   ├── smart-matching-dashboard.tsx  # Dashboard matching
│   ├── collective-buying.tsx  # Componente compra colectiva
│   ├── educational-chatbot.tsx       # Chatbot educativo
│   ├── gamification-dashboard.tsx    # Dashboard gamificación
│   ├── solar-calculator.tsx   # Calculadora solar
│   ├── consumption-upload.tsx # Upload de consumo
│   ├── bono-social-guide.tsx # Guía de bono social
│   ├── user-profile-drawer.tsx       # Panel perfil usuario
│   ├── user-reviews.tsx       # Reseñas de usuarios
│   ├── datadis-help-modal.tsx # Help modal Datadis
│   └── ...
│
├── pages/                      # Páginas principales (Route Components)
│   ├── Index.tsx              # Página inicio/dashboard
│   ├── Calculator.tsx         # Página calculadora
│   ├── CalculatorUpload.tsx   # Upload de facturas
│   ├── CalculatorManual.tsx   # Calculadora manual
│   ├── Matching.tsx           # Matching de tarifas
│   ├── Bills.tsx              # Gestión de facturas
│   ├── Analytics.tsx          # Analítica avanzada
│   ├── Gamification.tsx       # Dashboard gamificación
│   ├── Education.tsx          # Centro educativo
│   ├── Chatbot.tsx            # Página chatbot
│   ├── Collective.tsx         # Página compra colectiva
│   ├── BonoSocial.tsx         # Bono social
│   ├── Reviews.tsx            # Reseñas de proveedores
│   ├── Resources.tsx          # Recursos y guías
│   ├── HomeEnergyGuides.tsx   # Guías de energía
│   ├── NotFound.tsx           # Página 404
│   └── simulators/            # Simuladores especializados
│       ├── VehicleSimulator.tsx
│       ├── SolarSimulator.tsx
│       └── ...
│
├── hooks/                      # Hooks personalizados reutilizables
│   ├── use-app-config.ts      # Config de aplicación
│   ├── use-mobile.tsx         # Detectar dispositivo móvil
│   └── use-toast.ts           # Sistema de notificaciones
│
├── services/                   # Servicios de API y lógica
│   ├── config.ts              # Configuración de API base
│   ├── bills.ts               # Servicio de facturas
│   ├── billParser.ts          # Parser de facturas PDF/CSV
│   ├── collectiveBuying.ts    # Servicio compra colectiva
│   ├── gamification.ts        # Servicio gamificación
│   ├── intelligentMatching.ts # Servicio matching tarifas
│   ├── marketData.ts          # Servicio datos de mercado
│   ├── solar.ts               # Servicio calculadora solar
│   ├── localStorage.ts        # Persistencia local
│   └── openrouter.ts          # Integración con OpenRouter (IA)
│
├── lib/                        # Utilidades y helpers
│   ├── session.ts             # Gestión de sesión
│   ├── session.test.ts        # Tests de sesión
│   └── utils.ts               # Funciones utilitarias
│
└── types/                      # TypeScript types globales (si existe)
    └── ... (tipos compartidos)
```

**Propósito de cada carpeta:**

- **`app/`**: Configuración global (router, providers, hooks, layouts)
- **`components/`**: Componentes React reutilizables (UI + features)
- **`pages/`**: Componentes de página (conectados con rutas)
- **`services/`**: Lógica de negocio, llamadas a API
- **`hooks/`**: Custom hooks reutilizables
- **`lib/`**: Funciones utilitarias y helpers
- **`types/`**: Definiciones TypeScript globales

---

### 🔧 Backend: `server/src/`

```
server/
├── package.json               # Dependencias backend
├── tsconfig.json             # Configuración TypeScript
├── vitest.config.ts          # Configuración Vitest
│
├── src/
│   ├── server.ts             # Punto de entrada principal
│   ├── app.ts                # Instancia Express (middlewares, setup)
│   │
│   ├── config/               # Configuración centralizada
│   │   ├── env.ts            # Variables de entorno validadas
│   │   ├── database.ts       # Configuración Prisma
│   │   └── cors.ts           # CORS configuration
│   │
│   ├── middleware/           # Middlewares Express
│   │   ├── auth.ts           # JWT authentication
│   │   ├── errorHandler.ts   # Manejo global de errores
│   │   ├── requestLogger.ts  # Logging de requests
│   │   ├── validation.ts     # Validación con Zod
│   │   └── security.ts       # Middlewares de seguridad
│   │
│   ├── routes/               # Controladores y rutas HTTP
│   │   ├── index.ts          # Agregador de rutas
│   │   ├── auth.routes.ts    # Autenticación (login, register)
│   │   ├── users.routes.ts   # Gestión de usuarios
│   │   ├── bills.routes.ts   # Upload y parseo de facturas
│   │   ├── tariffs.routes.ts # CRUD de tarifas
│   │   ├── matching.routes.ts        # Algoritmo matching
│   │   ├── gamification.routes.ts    # Endpoints gamificación
│   │   ├── datadis.routes.ts # Integración Datadis
│   │   ├── collective.routes.ts      # Compra colectiva
│   │   ├── reviews.routes.ts # Reseñas de proveedores
│   │   └── forum.routes.ts   # Endpoints del foro
│   │
│   ├── services/             # Lógica de negocio (Business Logic)
│   │   ├── auth.service.ts   # Autenticación y JWT
│   │   ├── user.service.ts   # Gestión de usuarios
│   │   ├── bill.service.ts   # Lógica de facturas
│   │   ├── tariff.service.ts # Gestión de tarifas
│   │   ├── matching.service.ts       # Algoritmo matching
│   │   ├── gamification.service.ts   # Lógica gamificación
│   │   ├── datadis.service.ts        # Sincronización Datadis
│   │   ├── collective.service.ts     # Lógica compra colectiva
│   │   ├── solar.service.ts  # Cálculos solar
│   │   ├── consumption.service.ts    # Análisis de consumo
│   │   └── ...
│   │
│   ├── repositories/         # Data Access Layer (DAL)
│   │   ├── user.repository.ts        # Acceso User
│   │   ├── bill.repository.ts        # Acceso Bill
│   │   ├── tariff.repository.ts      # Acceso Tariff
│   │   ├── gamification.repository.ts        # Acceso Gamification
│   │   ├── consumption.repository.ts # Acceso Consumo
│   │   ├── forum.repository.ts       # Acceso Forum
│   │   └── ...
│   │
│   ├── schemas/              # Validación con Zod
│   │   ├── auth.schema.ts    # Validación auth
│   │   ├── user.schema.ts    # Validación usuarios
│   │   ├── bill.schema.ts    # Validación facturas
│   │   ├── tariff.schema.ts  # Validación tarifas
│   │   └── ...
│   │
│   ├── types/                # TypeScript interfaces & types
│   │   ├── auth.types.ts     # Tipos auth
│   │   ├── user.types.ts     # Tipos usuario
│   │   ├── api.types.ts      # Tipos de respuesta API
│   │   └── ...
│   │
│   ├── utils/                # Funciones auxiliares
│   │   ├── password.ts       # Hash y validación contraseñas
│   │   ├── jwt.ts            # Generación y validación JWT
│   │   ├── dateUtils.ts      # Utilidades de fechas
│   │   ├── csvParser.ts      # Parseo de CSV
│   │   ├── billParser.ts     # Extracción datos de facturas
│   │   ├── errorHandler.ts   # Manejo de errores
│   │   └── ...
│
├── prisma/
│   ├── schema.prisma         # Definición del modelo de datos
│   └── migrations/           # Historial de migraciones
│       ├── migration_lock.toml
│       ├── 20251113002404_init/
│       ├── 20251113023055_add_social_fields/
│       ├── 20251117140000_add_user_name_fields/
│       └── ...
│
└── uploads/                  # Archivos subidos por usuarios
    ├── bills/               # Facturas guardadas
    └── ...
```

**Propósito de cada carpeta:**

- **`config/`**: Centraliza configuración (env vars, BD, CORS)
- **`middleware/`**: Middlewares Express (auth, validación, seguridad)
- **`routes/`**: Controladores HTTP y definición de endpoints
- **`services/`**: Lógica de negocio (desacoplada de HTTP)
- **`repositories/`**: Acceso a datos (abstrae Prisma)
- **`schemas/`**: Validación de entrada con Zod
- **`types/`**: Interfaces y tipos TypeScript
- **`utils/`**: Funciones auxiliares (password, JWT, etc.)
- **`prisma/`**: Definición BD y migraciones
- **`uploads/`**: Almacenamiento de archivos del usuario

---

### 📊 Base de Datos: Modelos Principales

```
User
├── id (UUID)
├── email (Unique)
├── passwordHash
├── fullName, firstName, lastName
├── phone, avatarUrl
├── createdAt, updatedAt
└── Relaciones:
    ├── bills (1:N)
    ├── energyProfile (1:1)
    ├── sessions (1:N)
    ├── gamification_stats (1:1)
    ├── group_memberships (1:N)
    ├── reviews (1:N)
    ├── user_badges (1:N)
    ├── hourlyConsumptions (1:N)
    ├── monthlyConsumptions (1:N)
    ├── forumTopics (1:N)
    └── forumPosts (1:N)

Bill
├── id (UUID)
├── userId (FK)
├── filePath
├── extractedData (JSON)
├── periodStart, periodEnd
├── totalCost, totalConsumptionKwh
└── uploadedAt

UserEnergyProfile
├── id (UUID)
├── userId (FK, Unique)
├── address, distributor
├── tariffType
├── consumptionKwhMonth
├── contractedPowerKw
├── monthly_bill
├── household_size, home_type, region
├── consumption_pattern (Enum: diurno/nocturno/mixto)
├── has_electric_vehicle
├── has_solar_panels
├── has_heat_pump

Tariff
├── id (UUID)
├── company
├── planName
├── priceKwh, pricePowerKw
├── isGreen
└── createdAt

HourlyConsumption
├── id (UUID)
├── userId (FK)
├── cups
├── date, time
├── consumptionKwh
├── origin (default: datadis_csv)
└── method (default: datadis)

MonthlyConsumption
├── id (UUID)
├── userId (FK)
├── cups, period
├── energiaP1-P6 (consumo por periodo tarifario)
└── method (default: csv)

gamification_stats
├── user_id (PK, FK)
├── total_points
├── current_level
├── total_savings
├── co2_saved_kg
└── referral_code

buying_groups
├── id (UUID)
├── name, description
├── provider
├── target_members, current_members
├── estimated_savings
├── status (Enum: active/full/negotiating/completed)
└── deadline

group_memberships
├── id (UUID)
├── user_id (FK), group_id (FK)
├── joined_at
└── status (Enum: pending/active/completed)

reviews
├── id (UUID)
├── user_id (FK)
├── provider_name
├── rating, title, comment
├── is_verified
└── helpful_count

user_badges
├── id (UUID)
├── user_id (FK)
├── badge_id
└── unlocked_at

ForumCategory
├── id (UUID)
├── title, description
├── slug (Unique)
├── order, icon
└── topics (1:N)

ForumTopic
├── id (UUID)
├── title, content
├── views, isPinned, isLocked
├── type (default: discussion)
├── price (opcional)
├── status (default: active)
├── userId (FK), categoryId (FK)
└── posts (1:N)

ForumPost
├── id (UUID)
├── content
├── userId (FK), topicId (FK)
└── createdAt, updatedAt
```

---

## Patrones Arquitectónicos

### 1. **Layered Architecture (Arquitectura por Capas)**

```
┌─────────────────────────────────────────┐
│     Presentation (Routes/Controllers)    │
├─────────────────────────────────────────┤
│     Business Logic (Services)            │
├─────────────────────────────────────────┤
│     Data Access (Repositories)           │
├─────────────────────────────────────────┤
│     Data Storage (Database)              │
└─────────────────────────────────────────┘
```

**Beneficios:**
- Separación clara de responsabilidades
- Facilita testing (mock en cada capa)
- Reutilización de código
- Fácil de escalar y mantener

---

### 2. **Repository Pattern**

Los **Repositories** abstraen el acceso a datos:

```typescript
// routes/users.routes.ts
app.get("/users/:id", async (req, res) => {
  const user = await userRepository.findById(req.params.id);
  res.json(user);
});

// services/user.service.ts
export class UserService {
  constructor(private userRepository: UserRepository) {}
  
  async getUser(id: string) {
    return this.userRepository.findById(id);
  }
}

// repositories/user.repository.ts
export class UserRepository {
  async findById(id: string) {
    return prisma.user.findUnique({ where: { id } });
  }
}
```

**Ventajas:**
- Desacoplamiento de Prisma
- Fácil de testear (mock repository)
- Cambio de BD sin cambiar la lógica

---

### 3. **Service Pattern**

Los **Services** contienen la lógica de negocio:

```typescript
// services/matching.service.ts
export class MatchingService {
  async findBestTariffs(userProfile: UserEnergyProfile) {
    // 1. Obtener perfil usuario
    // 2. Aplicar algoritmo matching
    // 3. Calcular ahorro
    // 4. Retornar top 5 tarifas
  }
}
```

---

### 4. **Provider Pattern (Frontend)**

Contexto global para estado compartido:

```typescript
// app/providers/AuthProvider.tsx
export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}

// Uso: const { user } = useContext(AuthContext);
```

---

### 5. **Dependency Injection (DI)**

Inyección de dependencias para desacoplamiento:

```typescript
// Constructor injection
const userService = new UserService(userRepository);
const authService = new AuthService(userService);

// Permite testing con mocks:
const mockRepo = { findById: jest.fn() };
const service = new UserService(mockRepo);
```

---

## Infraestructura y DevOps

### 📦 Scripts de Desarrollo

**Frontend:**
```bash
npm run dev              # Vite dev server (localhost:5173)
npm run build            # Build para producción
npm run preview          # Preview build local
npm run test             # Ejecutar tests Vitest
npm run test:watch      # Watch mode tests
npm run lint             # ESLint + Prettier
```

**Backend:**
```bash
npm run dev              # tsx watch (auto-reload)
npm run build            # Compilar TypeScript
npm run start            # Ejecutar desde dist/
npm run test             # Vitest
npm run lint             # tsc --noEmit
npm run prisma:migrate  # Ejecutar migraciones
npm run prisma:generate # Generar Prisma Client
```

---

### 🔐 Variables de Entorno

**Frontend (`.env`):**
```env
VITE_API_URL=http://localhost:4005/api
VITE_GOOGLE_CLIENT_ID=your_client_id
```

**Backend (`server/.env`):**
```env
DATABASE_URL=mysql://root:root@localhost:3306/escogetuenergia
PORT=4005
JWT_ACCESS_SECRET=your_secret_key
JWT_REFRESH_SECRET=your_refresh_secret
FRONTEND_APP_URL=http://localhost:5173
GOOGLE_CLIENT_ID=your_google_id
GOOGLE_CLIENT_SECRET=your_google_secret
FACEBOOK_APP_ID=your_fb_id
FACEBOOK_APP_SECRET=your_fb_secret
BCRYPT_SALT_ROUNDS=10
ADMIN_EMAILS=admin@example.com
UPLOAD_DIR=./uploads
ACCESS_TOKEN_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d
```

---

### 🚀 CI/CD (GitHub Actions)

Ubicado en `.github/workflows/ci.yml`:

```yaml
- Lint (ESLint)
- Tests (Vitest - Frontend & Backend)
- Build (Vite + TypeScript)
- Deploy (si aplica)
```

Se ejecuta en cada **push** y **pull request**.

---

### 🐳 Docker (Opcional)

- `Dockerfile`: Imagen para aplicación web
- `server/Dockerfile`: Imagen para API
- `docker-compose.yml`: Orquestación multi-contenedor
- `docker-entrypoint.sh`: Script de inicialización
- `docker-manager.sh`: Utilidades para gestionar contenedores

---

### 📝 Características Clave de Arquitectura

| Aspecto | Implementación |
|--------|-----------------|
| **Seguridad** | JWT + bcrypt, Helmet, CORS, Rate Limiting, HPP |
| **Validación** | Zod (Frontend & Backend) |
| **Testing** | Vitest + Testing Library |
| **Type Safety** | TypeScript en ambas capas |
| **State Management** | React Query (async) + Context (auth) |
| **Logging** | Morgan (HTTP requests) |
| **Error Handling** | Try-catch + Error boundaries |
| **Code Splitting** | React.lazy + Suspense (Frontend) |
| **Compresión** | gzip en Express |
| **Database** | Prisma ORM + MySQL 8.0+ |

---

## � Guía de Inicio Rápido: Levantar Todos los Servicios

Esta sección describe paso a paso cómo levantar toda la infraestructura del proyecto en tu máquina local.

### ✅ Requisitos Previos

Antes de empezar, asegúrate de tener instalado:

- **Node.js**: 20.0 o superior ([Descargar](https://nodejs.org/))
- **npm**: 10.0 o superior (incluido con Node.js)
- **MySQL**: 8.0 o superior ([Descargar](https://dev.mysql.com/downloads/mysql/))
- **Git**: Para clonar el repositorio

**Verificar instalaciones:**
```bash
node --version    # v20.x.x o superior
npm --version     # 10.x.x o superior
mysql --version   # 8.0 o superior
```

---

### 📋 Paso 1: Preparar el Entorno

#### 1.1 Clonar o descargar el proyecto
```bash
cd /home/joseramon/escogetuenergia/escogetuenergia
```

#### 1.2 Crear archivo `.env` (Frontend)
En la **raíz del proyecto**, crear `.env`:
```bash
cat > .env << 'EOF'
VITE_API_URL=http://localhost:4005/api
VITE_GOOGLE_CLIENT_ID=your_google_client_id_here
VITE_ENVIRONMENT=development
EOF
```

#### 1.3 Crear archivo `server/.env` (Backend)
En la carpeta **`server/`**, crear `.env`:
```bash
cd server
cat > .env << 'EOF'
# Database
DATABASE_URL=mysql://root:root@localhost:3306/escogetuenergia

# Server
PORT=4005
NODE_ENV=development

# JWT
JWT_ACCESS_SECRET=your_access_secret_key_here_min_32_chars
JWT_REFRESH_SECRET=your_refresh_secret_key_here_min_32_chars
ACCESS_TOKEN_EXPIRES_IN=15m
REFRESH_TOKEN_EXPIRES_IN=7d

# Frontend URL
FRONTEND_APP_URL=http://localhost:5173

# Google OAuth (opcional)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# Facebook OAuth (opcional)
FACEBOOK_APP_ID=your_facebook_app_id
FACEBOOK_APP_SECRET=your_facebook_app_secret

# Security
BCRYPT_SALT_ROUNDS=10
HPP_ENABLED=true
RATE_LIMIT_ENABLED=true

# File Upload
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760

# Admin
ADMIN_EMAILS=admin@example.com,admin2@example.com
EOF
cd ..
```

---

### 🗄️ Paso 2: Configurar Base de Datos MySQL

#### 2.1 Iniciar servidor MySQL

**En Linux/macOS (si está instalado localmente):**
```bash
# Iniciar MySQL si está detenido
sudo systemctl start mysql
# o
brew services start mysql@8.0
```

**En Windows:**
```cmd
net start MySQL80
```

**Verificar que MySQL está corriendo:**
```bash
mysql -u root -p -e "SELECT VERSION();"
# Ingresar contraseña cuando se pida
```

#### 2.2 Crear base de datos
```bash
# Conectarse a MySQL
mysql -u root -p

# Ejecutar en MySQL CLI:
CREATE DATABASE IF NOT EXISTS escogetuenergia CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;
```

**O en una línea:**
```bash
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS escogetuenergia CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

---

### 📦 Paso 3: Instalar Dependencias

#### 3.1 Instalar dependencias Frontend
```bash
# Desde la raíz del proyecto
npm ci
# o
npm install
```

#### 3.2 Instalar dependencias Backend
```bash
# Desde la carpeta server/
cd server
npm ci
# o
npm install
cd ..
```

---

### 🔄 Paso 4: Ejecutar Migraciones de Base de Datos

**Muy importante:** Las migraciones crean la estructura de la BD.

```bash
# Desde la carpeta server/
cd server

# Opción 1: Aplicar migraciones existentes
npm run prisma:migrate

# Opción 2: Si deseas crear migraciones nuevas
npm run prisma:generate

cd ..
```

**Verificar que la BD fue creada:**
```bash
mysql -u root -p escogetuenergia -e "SHOW TABLES;"
```

Deberías ver tablas como: `users`, `bills`, `tariffs`, `gamification_stats`, etc.

---

### ▶️ Paso 5: Levantar los Servicios

Abre **3 terminales** (o tabs en tu terminal favorita) y ejecuta cada uno en paralelo:

#### Terminal 1: Base de Datos MySQL (si no está como servicio)
```bash
# Opcional si MySQL no está como servicio del sistema
mysql -u root -p
# Deja la conexión abierta
```

#### Terminal 2: Backend (API Server)
```bash
cd /home/joseramon/escogetuenergia/escogetuenergia/server
npm run dev

# Deberías ver algo como:
# ✓ Servidor escuchando en puerto 4005
# ✓ Conexión a base de datos establecida
```

**El backend debe estar disponible en:** `http://localhost:4005`

#### Terminal 3: Frontend (Vite Dev Server)
```bash
cd /home/joseramon/escogetuenergia/escogetuenergia
npm run dev

# Deberías ver algo como:
# ✓ Local: http://localhost:5173/
# ✓ Press q to quit
```

**El frontend debe estar disponible en:** `http://localhost:5173`

---

### ✨ Paso 6: Verificar que Todo Funciona

1. **Abrir navegador en:** `http://localhost:5173`
2. **Deberías ver** la página de inicio (Home Page)
3. **Verificar logs** en ambas terminales (no deben haber errores en rojo)

**Pruebas básicas:**
- Navega por las páginas principales
- Intenta hacer login/register
- Carga una factura (si tienes una disponible)

---

### 🔧 Parar los Servicios

Para detener todos los servicios:

1. **Terminal Backend:** Presiona `Ctrl+C`
2. **Terminal Frontend:** Presiona `Ctrl+C`
3. **Terminal MySQL:** Presiona `Ctrl+D` (si está abierta)

---

### 🐳 Alternativa: Levantar con Docker (Opcional)

Si prefieres usar Docker Compose:

```bash
# Asegúrate de tener Docker y Docker Compose instalados
docker --version
docker-compose --version

# Levantar todos los servicios
docker-compose up -d

# Ver logs
docker-compose logs -f

# Parar servicios
docker-compose down
```

**Archivos Docker disponibles:**
- `docker-compose.yml`: Configuración principal
- `docker-compose.main.yml`: Configuración alternativa
- `Dockerfile`: Para la aplicación web
- `server/Dockerfile`: Para la API

---

### 🆘 Troubleshooting Común

| Problema | Solución |
|----------|----------|
| **Puerto 4005 ya está en uso** | `lsof -i :4005` (macOS/Linux) o `netstat -ano \| findstr :4005` (Windows). Mata el proceso o usa otro puerto en `.env` |
| **Puerto 5173 ya está en uso** | Similar al anterior. O usa `npm run dev -- --port 5174` |
| **Error: "Can't connect to MySQL"** | Verifica que MySQL está corriendo: `mysql -u root -p -e "SELECT 1;"` |
| **Error: "Unknown database 'escogetuenergia'"** | Ejecuta: `mysql -u root -p -e "CREATE DATABASE escogetuenergia;"` |
| **Error en migraciones Prisma** | Ejecuta `npm run prisma:generate` en `server/` |
| **Dependencies install fallida** | Borra `node_modules` y `package-lock.json`, luego `npm ci` |
| **Frontend no puede conectar con API** | Verifica que `VITE_API_URL=http://localhost:4005/api` en `.env` |
| **Error CORS en navegador** | Backend debe tener `CORS` habilitado en `middleware/` |
| **Contraseña MySQL rechazada** | Asegúrate de usar la contraseña correcta o crea usuario sin contraseña |

---

### 📝 Checklist de Startup

```
[ ] Node.js 20+ instalado
[ ] npm 10+ instalado
[ ] MySQL 8.0+ instalado y corriendo
[ ] Archivo .env creado en raíz
[ ] Archivo server/.env creado
[ ] Base de datos 'escogetuenergia' creada
[ ] npm ci ejecutado en raíz
[ ] npm ci ejecutado en server/
[ ] npm run prisma:migrate ejecutado
[ ] Backend levantado (terminal 1)
[ ] Frontend levantado (terminal 2)
[ ] Acceso a http://localhost:5173 exitoso
[ ] Logs sin errores rojos
```

---

### 🔄 Desarrollo Diario

Una vez que todo está levantado:

**Frontend:**
```bash
npm run dev          # Dev server con hot reload
npm run lint         # Verificar código
npm run test         # Ejecutar tests
```

**Backend:**
```bash
cd server
npm run dev          # Dev server con auto-reload (tsx watch)
npm run lint         # Verificar código
npm run test         # Ejecutar tests
npm run prisma:migrate  # Si hiciste cambios al schema
```

---

## �📚 Referencias Adicionales

- [Prisma Docs](https://www.prisma.io/docs/)
- [Express Docs](https://expressjs.com/)
- [React Docs](https://react.dev/)
- [Vite Docs](https://vitejs.dev/)
- [Tailwind CSS Docs](https://tailwindcss.com/)
- [shadcn/ui Docs](https://ui.shadcn.com/)
- [Zod Docs](https://zod.dev/)
- [React Query Docs](https://tanstack.com/query/latest)

---

**Documento generado:** Enero 2026  
**Versión del proyecto:** 0.0.0  
**Última actualización arquitectónica:** 2026-01-15

