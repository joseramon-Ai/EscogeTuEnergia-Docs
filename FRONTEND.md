# FRONTEND - Escoge tu Energía

**Documentación Completa del Cliente React**

---

## 📋 Índice

1. [Información General](#información-general)
2. [Arquitectura del Frontend](#arquitectura-del-frontend)
3. [Gestión de Estado](#gestión-de-estado)
4. [Rutas del Cliente](#rutas-del-cliente)
5. [Jerarquía de Componentes](#jerarquía-de-componentes)
6. [Componentes Principales](#componentes-principales)
7. [Hooks Personalizados](#hooks-personalizados)
8. [Servicios](#servicios)
9. [Estilos y UI](#estilos-y-ui)

---

## Información General

### 🎨 Stack Tecnológico

| Tecnología | Versión | Propósito |
|------------|---------|----------|
| **React** | 18.3.1 | Framework UI |
| **TypeScript** | 5.8.3 | Tipado estático |
| **Vite** | 5.4.19 | Build tool y dev server |
| **React Router** | 6.30.1 | Enrutamiento SPA |
| **React Query** | 5.83.0 | Gestión de estado async |
| **React Hook Form** | 7.61.1 | Gestión de formularios |
| **Zod** | 3.25.76 | Validación de esquemas |
| **Tailwind CSS** | 3.4.17 | Estilos utility-first |
| **shadcn/ui** | 1.x | Componentes UI accesibles |
| **Radix UI** | 1.x | Primitivas UI sin estilos |
| **Lucide React** | 0.462.0 | Iconos |
| **Recharts** | 2.15.4 | Gráficos y visualizaciones |

### 📁 Estructura del Proyecto

```
src/
├── main.tsx                    # Punto de entrada
├── App.tsx                     # Componente raíz
├── index.css                   # Estilos globales
├── vite-env.d.ts              # Types de Vite
│
├── app/                        # Configuración global
│   ├── router.tsx             # Definición de rutas
│   ├── providers/             # Providers globales
│   │   ├── index.tsx          # AppProviders (wrapper)
│   │   └── auth-provider.tsx  # Context de autenticación
│   ├── layouts/               # Layouts compartidos
│   │   └── page-layout.tsx    # Layout principal
│   └── hooks/                 # Hooks globales
│       └── use-auth.ts        # Hook de autenticación
│
├── pages/                      # Páginas (Route Components)
│   ├── Index.tsx              # Dashboard principal
│   ├── Calculator.tsx         # Selector de calculadora
│   ├── CalculatorManual.tsx   # Formulario manual
│   ├── CalculatorUpload.tsx   # Upload de archivo
│   ├── Matching.tsx           # Matching de tarifas
│   ├── Bills.tsx              # Gestión de facturas
│   ├── Reviews.tsx            # Reseñas
│   ├── Collective.tsx         # Compra colectiva
│   ├── Gamification.tsx       # Dashboard gamificación
│   ├── Education.tsx          # Centro educativo
│   ├── Resources.tsx          # Recursos
│   ├── HomeEnergyGuides.tsx   # Guías hogar
│   ├── BonoSocial.tsx         # Bono social
│   ├── Chatbot.tsx            # Chatbot educativo
│   ├── NotFound.tsx           # Página 404
│   └── simulators/            # Simuladores
│       └── SolarSimulator.tsx
│
├── components/                 # Componentes reutilizables
│   ├── ui/                    # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   ├── form.tsx
│   │   ├── input.tsx
│   │   ├── tabs.tsx
│   │   └── ... (30+ componentes)
│   │
│   ├── common/                # Componentes comunes
│   │   └── loading-screen.tsx
│   │
│   ├── navigation.tsx         # Barra de navegación
│   ├── hero-section.tsx       # Hero landing
│   ├── calculator-section.tsx # Sección calculadora
│   ├── smart-matching-dashboard.tsx  # Dashboard matching
│   ├── market-insights.tsx    # Insights mercado
│   ├── advanced-market-analytics.tsx # Analítica avanzada
│   ├── gamification-dashboard.tsx    # Dashboard gamificación
│   ├── collective-buying.tsx  # Compra colectiva
│   ├── educational-chatbot.tsx       # Chatbot
│   ├── education-section.tsx  # Sección educación
│   ├── user-reviews.tsx       # Reseñas usuarios
│   ├── auth-modal.tsx         # Modal login/register
│   ├── user-profile-drawer.tsx       # Panel perfil
│   ├── onboarding-flow.tsx    # Onboarding inicial
│   ├── solar-calculator.tsx   # Calculadora solar
│   ├── consumption-upload.tsx # Upload consumo
│   ├── bono-social-guide.tsx  # Guía bono social
│   ├── datadis-help-modal.tsx # Ayuda Datadis
│   ├── calculadora-factura-form.tsx  # Formulario factura
│   └── logo.tsx               # Logo componente
│
├── hooks/                      # Hooks personalizados
│   ├── use-mobile.tsx         # Detectar móvil
│   ├── use-toast.ts           # Sistema toast
│   └── use-app-config.ts      # Config app
│
├── services/                   # Servicios API
│   ├── config.ts              # Config API base
│   ├── bills.ts               # Servicio facturas
│   ├── billParser.ts          # Parser facturas
│   ├── intelligentMatching.ts # Matching tarifas
│   ├── gamification.ts        # Gamificación
│   ├── collectiveBuying.ts    # Compra colectiva
│   ├── marketData.ts          # Datos mercado
│   ├── solar.ts               # Calculadora solar
│   ├── localStorage.ts        # Persistencia local
│   └── openrouter.ts          # Integración OpenRouter (IA)
│
├── lib/                        # Utilidades
│   ├── session.ts             # Gestión sesión
│   ├── session.test.ts        # Tests sesión
│   └── utils.ts               # Funciones utils (cn, etc.)
│
└── types/                      # TypeScript types
    └── auth.ts                # Tipos autenticación
```

---

## Arquitectura del Frontend

### 🏗️ Patrón de Arquitectura

**Arquitectura por Capas (Layered Architecture)**

```
┌─────────────────────────────────────────────┐
│         ENTRY POINT (main.tsx)              │
│              React.StrictMode                │
└────────────────┬────────────────────────────┘
                 │
┌────────────────▼────────────────────────────┐
│           APP COMPONENT (App.tsx)            │
│           - AppProviders                     │
│           - AppRouter                        │
└────────────────┬────────────────────────────┘
                 │
       ┌─────────┴──────────┐
       │                    │
┌──────▼───────┐  ┌────────▼─────────┐
│   PROVIDERS  │  │     ROUTER       │
│              │  │                  │
│ - QueryClient│  │ - React Router   │
│ - AuthContext│  │ - Lazy Routes    │
│ - Toasts     │  │ - Suspense       │
└──────┬───────┘  └────────┬─────────┘
       │                    │
       │         ┌──────────▼──────────┐
       │         │   PAGES (Routes)    │
       │         │                     │
       │         │ - Index.tsx         │
       │         │ - Calculator.tsx    │
       │         │ - Matching.tsx      │
       │         │ - etc...            │
       │         └──────────┬──────────┘
       │                    │
       └──────────┬─────────┘
                  │
       ┌──────────▼──────────┐
       │   LAYOUT WRAPPER    │
       │   (PageLayout)      │
       │                     │
       │ - Navigation        │
       │ - Main Container    │
       │ - Onboarding        │
       └──────────┬──────────┘
                  │
       ┌──────────▼──────────────────┐
       │   FEATURE COMPONENTS        │
       │                             │
       │ - HeroSection               │
       │ - CalculatorSection         │
       │ - SmartMatchingDashboard    │
       │ - GamificationDashboard     │
       │ - etc...                    │
       └──────────┬──────────────────┘
                  │
       ┌──────────▼──────────────────┐
       │   UI COMPONENTS (shadcn)    │
       │                             │
       │ - Button, Card, Dialog      │
       │ - Form, Input, Select       │
       │ - Tabs, Sheet, Drawer       │
       │ - etc... (30+ componentes)  │
       └─────────────────────────────┘
```

### 🔄 Flujo de Datos

```
User Interaction
       ↓
Component Event Handler
       ↓
Service Function (API Call)
       ↓
React Query (useQuery/useMutation)
       ↓
Backend API (Express)
       ↓
Response
       ↓
React Query Cache Update
       ↓
Component Re-render
       ↓
UI Update
```

---

## Gestión de Estado

### 🗄️ Estrategia de Estado Multi-Capa

El frontend utiliza **múltiples estrategias de estado** según el tipo de datos:

#### 1. **Estado Global: React Context API**

**📦 AuthContext (Autenticación)**

- **Archivo**: `src/app/providers/auth-provider.tsx`
- **Propósito**: Gestión de sesión de usuario y tokens JWT
- **Alcance**: Global (toda la aplicación)

**Datos Almacenados:**
```typescript
interface AuthContextValue {
  user: AuthUser | null;              // Datos del usuario autenticado
  tokens: AuthTokens | null;          // Access + Refresh tokens
  status: "idle" | "ready";           // Estado de carga
  isAuthenticated: boolean;           // Flag de autenticación
  setSession: (payload) => void;      // Actualizar sesión
  logout: () => void;                 // Cerrar sesión
  updateUser: (user) => void;         // Actualizar usuario
}
```

**Tipos de Datos:**
```typescript
// Usuario autenticado
interface AuthUser {
  id: string;
  email: string;
  fullName: string | null;
  firstName: string | null;
  lastName: string | null;
  phone: string | null;
  avatarUrl: string | null;
  createdAt: string;
}

// Tokens JWT
interface AuthTokens {
  accessToken: string;   // Token de acceso (15 min)
  refreshToken: string;  // Token de refresco (7 días)
}

// Estado de sesión
interface SessionState {
  user: AuthUser;
  tokens: AuthTokens;
}
```

**Persistencia:**
- Los datos se guardan en `sessionStorage` (no `localStorage`)
- Se sincronizan entre tabs del navegador
- Se limpian automáticamente al cerrar el navegador

**Uso del Hook:**
```typescript
import { useAuth } from "@/app/hooks/use-auth";

function MyComponent() {
  const { user, isAuthenticated, logout } = useAuth();
  
  if (!isAuthenticated) {
    return <LoginPrompt />;
  }
  
  return <div>Hola {user?.firstName}</div>;
}
```

---

#### 2. **Estado Asíncrono: React Query (TanStack Query)**

- **Librería**: `@tanstack/react-query` v5.83.0
- **Propósito**: Gestión de datos del servidor (fetching, caching, sincronización)
- **Archivo Config**: `src/app/providers/index.tsx`

**Configuración:**
```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,              // Datos frescos 1 minuto
      retry: 1,                       // 1 reintento en caso de error
      refetchOnWindowFocus: false,    // No refetch al enfocar ventana
    },
  },
});
```

**Datos Gestionados:**
- ✅ Listado de facturas del usuario
- ✅ Catálogo de tarifas eléctricas
- ✅ Perfil energético del usuario
- ✅ Historial de consumo
- ✅ Recomendaciones de tarifas
- ✅ Datos de gamificación
- ✅ Grupos de compra colectiva
- ✅ Reseñas de proveedores

**Ejemplo de Uso:**
```typescript
import { useQuery, useMutation } from "@tanstack/react-query";
import { billsService } from "@/services/bills";

function BillsList() {
  // Query para obtener facturas
  const { data, isLoading, error } = useQuery({
    queryKey: ["bills"],
    queryFn: () => billsService.list(),
  });
  
  // Mutation para subir factura
  const uploadMutation = useMutation({
    mutationFn: (file: File) => billsService.upload(file),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["bills"] });
    },
  });
  
  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  
  return (
    <div>
      {data.bills.map(bill => <BillCard key={bill.id} bill={bill} />)}
    </div>
  );
}
```

**Ventajas:**
- ⚡ Caché automático de respuestas
- 🔄 Sincronización automática en background
- 📡 Deduplicación de requests
- ♻️ Revalidación automática
- 🎯 Estados de carga/error integrados

---

#### 3. **Estado Local: useState + useReducer**

Para estado específico de componentes:

```typescript
// Estado simple de componente
const [isOpen, setIsOpen] = useState(false);
const [activeTab, setActiveTab] = useState("calculator");
const [formData, setFormData] = useState({ name: "", email: "" });
```

---

#### 4. **Estado de Formularios: React Hook Form**

- **Librería**: `react-hook-form` v7.61.1
- **Propósito**: Gestión de formularios complejos con validación

**Ejemplo:**
```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  email: z.string().email("Email inválido"),
  password: z.string().min(8, "Mínimo 8 caracteres"),
});

function LoginForm() {
  const form = useForm({
    resolver: zodResolver(schema),
    defaultValues: {
      email: "",
      password: "",
    },
  });
  
  const onSubmit = (data) => {
    // Enviar datos al servidor
  };
  
  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input {...form.register("email")} />
      {form.formState.errors.email && <span>{form.formState.errors.email.message}</span>}
    </form>
  );
}
```

---

#### 5. **Persistencia Local: sessionStorage**

**Archivo**: `src/lib/session.ts`

**Funciones:**
```typescript
// Cargar sesión desde sessionStorage
loadSession(): SessionState | null

// Guardar sesión en sessionStorage
persistSession(user: AuthUser, tokens: AuthTokens): void

// Limpiar sesión
clearSession(): void
```

**Keys de Storage:**
- `escoge_user` - Datos del usuario
- `escoge_tokens` - Tokens JWT

**Sincronización:**
- Event listener en `storage` para sincronizar entre tabs
- Auto-limpieza al cerrar navegador (sessionStorage)

---

### 📊 Resumen de Estrategias de Estado

| Tipo de Estado | Librería/Método | Alcance | Persistencia | Uso |
|----------------|-----------------|---------|--------------|-----|
| **Autenticación** | React Context | Global | sessionStorage | Usuario, tokens JWT |
| **Datos Servidor** | React Query | Cache | Memoria | Facturas, tarifas, perfil |
| **UI Local** | useState | Componente | Memoria | Modales, tabs, inputs |
| **Formularios** | React Hook Form | Componente | Memoria | Validación, submit |
| **URL State** | React Router | Global | URL | Parámetros, rutas |
| **Temas** | localStorage | Global | localStorage | Dark mode (futuro) |

---

## Rutas del Cliente

### 🗺️ Configuración de Rutas

**Archivo**: `src/app/router.tsx`

**Características:**
- ✅ **Lazy Loading**: Todas las páginas se cargan bajo demanda
- ✅ **Code Splitting**: Bundles separados por ruta
- ✅ **Suspense**: Loading screen durante carga de chunks
- ✅ **404 Fallback**: Página NotFound para rutas no existentes

### 📍 Mapa Completo de Rutas

| Ruta | Componente | Descripción | Auth |
|------|-----------|-------------|------|
| `/` | `Index.tsx` | Dashboard principal con tabs | ❌ |
| `/calculadora` | `Calculator.tsx` | Selector manual vs upload | ❌ |
| `/calculadora/manual` | `CalculatorManual.tsx` | Formulario manual factura | ❌ |
| `/calculadora/subir-archivo` | `CalculatorUpload.tsx` | Upload PDF/CSV factura | ✅ |
| `/matching` | `Matching.tsx` | Dashboard matching tarifas | ❌ |
| `/reviews` | `Reviews.tsx` | Reseñas de proveedores | ❌ |
| `/collective` | `Collective.tsx` | Compra colectiva grupos | ❌ |
| `/gamification` | `Gamification.tsx` | Dashboard gamificación | ✅ |
| `/education` | `Education.tsx` | Centro educativo | ❌ |
| `/resources` | `Resources.tsx` | Recursos y guías | ❌ |
| `/resources/guia-hogar` | `HomeEnergyGuides.tsx` | Guías del hogar | ❌ |
| `/education/bono-social` | `BonoSocial.tsx` | Guía bono social | ❌ |
| `/chatbot` | `Chatbot.tsx` | Chatbot educativo | ❌ |
| `/bills` | `Bills.tsx` | Gestión de facturas | ✅ |
| `/simuladores/solar` | `SolarSimulator.tsx` | Simulador solar | ❌ |
| `*` | `NotFound.tsx` | Página 404 | ❌ |

**Leyenda:**
- ✅ = Requiere autenticación
- ❌ = Público

### 🔐 Protección de Rutas

**Nota**: Actualmente no hay protección de rutas implementada en el router. Las páginas que requieren autenticación muestran un mensaje de login si el usuario no está autenticado.

**Implementación Recomendada:**
```typescript
// Futuro: ProtectedRoute component
function ProtectedRoute({ children }: { children: ReactNode }) {
  const { isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    return <Navigate to="/" replace />;
  }
  
  return <>{children}</>;
}

// Uso en router
<Route 
  path="/bills" 
  element={
    <ProtectedRoute>
      <BillsPage />
    </ProtectedRoute>
  } 
/>
```

### 🔗 Navegación Programática

**Desde componentes:**
```typescript
import { useNavigate } from "react-router-dom";

function MyComponent() {
  const navigate = useNavigate();
  
  const handleClick = () => {
    navigate("/calculadora");
  };
  
  return <button onClick={handleClick}>Ir a Calculadora</button>;
}
```

**Links declarativos:**
```typescript
import { Link } from "react-router-dom";

<Link to="/matching">Matching de Tarifas</Link>
```

---

## Jerarquía de Componentes

### 🌳 Árbol de Componentes Principal

```
<StrictMode>
  └─ <App>
      └─ <AppProviders>
          ├─ <QueryClientProvider>
          │   └─ React Query cache
          ├─ <TooltipProvider>
          ├─ <AuthProvider>
          │   └─ AuthContext (user, tokens)
          ├─ <Toaster> (shadcn)
          └─ <Sonner> (sonner)
      
      └─ <AppRouter>
          └─ <BrowserRouter>
              └─ <Suspense fallback={<LoadingScreen />}>
                  └─ <Routes>
                      ├─ Route "/" → <IndexPage>
                      ├─ Route "/calculadora" → <CalculatorPage>
                      ├─ Route "/matching" → <MatchingPage>
                      └─ ...más rutas
```

### 📄 Jerarquía de Página Típica

**Ejemplo: Index.tsx (Dashboard Principal)**

```
<PageLayout activeTab="calculator">
  ├─ <a href="#main-content"> (skip link)
  ├─ <Navigation>
  │   ├─ <Logo>
  │   ├─ <NavigationMenu>
  │   │   ├─ Menu Items (Calculadora, Matching, etc.)
  │   │   └─ <DropdownMenu> (User Profile)
  │   ├─ <AuthModal> (Login/Register)
  │   └─ <UserProfileDrawer>
  │
  ├─ <main id="main-content">
  │   ├─ <HeroSection>
  │   │   ├─ Title + Description
  │   │   └─ CTA Buttons
  │   │
  │   └─ <Tabs value={activeTab}>
  │       ├─ <TabsContent value="calculator">
  │       │   └─ <CalculatorSection>
  │       │       └─ <CalculadoraFacturaForm>
  │       │           ├─ Inputs (consumo, potencia)
  │       │           ├─ <Select> (tipo tarifa)
  │       │           └─ <Button> (Calcular)
  │       │
  │       ├─ <TabsContent value="matching">
  │       │   └─ <SmartMatchingDashboard>
  │       │       ├─ <Card> (Estadísticas)
  │       │       └─ Lista de tarifas recomendadas
  │       │
  │       ├─ <TabsContent value="gamification">
  │       │   └─ <GamificationDashboard>
  │       │       ├─ Puntos + Nivel
  │       │       ├─ <Badge> components
  │       │       └─ Leaderboard
  │       │
  │       └─ ...más tabs
  │
  └─ <OnboardingFlow isOpen={showOnboarding}>
      └─ <Dialog>
          └─ Steps (Bienvenida, Perfil, Preferencias)
```

### 🔀 Flujo de Props (Ejemplo: Navigation)

```
<PageLayout>
  props: activeTab, onTabChange
  │
  └─> <Navigation>
       props: activeTab, onTabChange, onShowOnboarding
       state: mobileMenuOpen, showAuthModal, showProfileSheet
       │
       ├─> <Logo>
       │
       ├─> <NavigationMenu>
       │    └─> <NavigationMenuItem>
       │         ├─ onClick → onTabChange(tab)
       │         └─ navigate(routeById[tab])
       │
       ├─> <DropdownMenu>
       │    └─> <DropdownMenuItem>
       │         ├─ onClick → setShowProfileSheet(true)
       │         └─ onClick → logout()
       │
       ├─> <AuthModal isOpen={showAuthModal}>
       │    │  props: onClose, onSuccess
       │    │
       │    └─> <Tabs> (Login / Register)
       │         └─> <Form>
       │              ├─ react-hook-form
       │              ├─ zod validation
       │              └─ onSubmit → authService.login()
       │
       └─> <UserProfileDrawer isOpen={showProfileSheet}>
            └─> Profile form + Avatar upload
```

---

## Componentes Principales

### 🧩 Componentes de Layout

#### **PageLayout**
**Archivo**: `src/app/layouts/page-layout.tsx`

**Propósito**: Layout wrapper para todas las páginas.

**Props:**
```typescript
interface PageLayoutProps {
  children: ReactNode;
  activeTab?: string;           // Tab activo en navegación
  onTabChange?: (value: string) => void;
  withContainer?: boolean;      // Aplicar container mx-auto
  mainClassName?: string;       // Clases custom para <main>
}
```

**Características:**
- ✅ Skip link para accesibilidad
- ✅ Navegación persistente
- ✅ Container responsive
- ✅ Onboarding flow global

---

#### **Navigation**
**Archivo**: `src/components/navigation.tsx`

**Propósito**: Barra de navegación principal con menús desplegables.

**Características:**
- ✅ Logo + menú desktop
- ✅ Menú móvil (Sheet)
- ✅ Dropdown de usuario autenticado
- ✅ Modal de login/registro
- ✅ Drawer de perfil de usuario
- ✅ Integración con AuthContext

---

### 🎯 Componentes de Feature

#### **CalculatorSection**
**Archivo**: `src/components/calculator-section.tsx`

Calculadora de factura eléctrica con inputs de consumo y potencia.

---

#### **SmartMatchingDashboard**
**Archivo**: `src/components/smart-matching-dashboard.tsx`

Dashboard de matching inteligente de tarifas con recomendaciones personalizadas.

---

#### **GamificationDashboard**
**Archivo**: `src/components/gamification-dashboard.tsx`

Dashboard de gamificación con puntos, nivel, badges y leaderboard.

---

#### **CollectiveBuying**
**Archivo**: `src/components/collective-buying.tsx`

Grupos de compra colectiva con listado y join/leave.

---

#### **EducationalChatbot**
**Archivo**: `src/components/educational-chatbot.tsx`

Chatbot educativo con integración a OpenRouter (IA).

---

#### **AuthModal**
**Archivo**: `src/components/auth-modal.tsx`

Modal de autenticación con tabs de Login y Register.

**Características:**
- ✅ Formularios con React Hook Form + Zod
- ✅ Login con email/password
- ✅ Registro con validación
- ✅ Mensajes de error
- ✅ Callback onSuccess para cerrar modal

---

#### **UserProfileDrawer**
**Archivo**: `src/components/user-profile-drawer.tsx`

Panel lateral con perfil del usuario y opciones de edición.

**Características:**
- ✅ Mostrar datos del usuario
- ✅ Editar firstName, lastName, email, phone
- ✅ Upload de avatar
- ✅ Cambio de contraseña
- ✅ Cerrar sesión

---

### 🎨 Componentes UI (shadcn/ui)

**Ubicación**: `src/components/ui/`

**Componentes disponibles (30+):**

| Componente | Descripción |
|------------|-------------|
| `Button` | Botón con variantes (default, outline, ghost, link) |
| `Card` | Tarjeta con header, content, footer |
| `Dialog` | Modal/Dialog accesible |
| `Sheet` | Panel lateral (drawer) |
| `Tabs` | Sistema de pestañas |
| `Form` | Wrapper para React Hook Form |
| `Input` | Input de texto |
| `Select` | Select dropdown |
| `Checkbox` | Checkbox accesible |
| `RadioGroup` | Grupo de radio buttons |
| `Switch` | Toggle switch |
| `Slider` | Slider numérico |
| `Textarea` | Textarea multi-línea |
| `Label` | Label para inputs |
| `Tooltip` | Tooltip hover |
| `Popover` | Popover flotante |
| `DropdownMenu` | Menú desplegable |
| `NavigationMenu` | Menú de navegación |
| `Avatar` | Avatar de usuario |
| `Badge` | Badge/chip |
| `Alert` | Alertas de notificación |
| `Toast` | Toast notifications |
| `Skeleton` | Loading skeleton |
| `Progress` | Barra de progreso |
| `Separator` | Línea separadora |
| `Accordion` | Acordeón expandible |
| `Table` | Tabla de datos |
| `ScrollArea` | Área scrolleable |
| `Collapsible` | Contenido colapsable |
| `AspectRatio` | Ratio de aspecto |

**Todos los componentes:**
- ✅ Construidos sobre Radix UI
- ✅ Totalmente accesibles (WAI-ARIA)
- ✅ Estilizados con Tailwind CSS
- ✅ Soporte para temas (light/dark)
- ✅ TypeScript typed

---

## Hooks Personalizados

### 🪝 Hooks Globales

#### **useAuth**
**Archivo**: `src/app/hooks/use-auth.ts`

Hook para acceder al contexto de autenticación.

```typescript
const {
  user,              // AuthUser | null
  tokens,            // AuthTokens | null
  status,            // "idle" | "ready"
  isAuthenticated,   // boolean
  setSession,        // (payload: SessionState) => void
  logout,            // () => void
  updateUser,        // (user: AuthUser) => void
} = useAuth();
```

---

#### **useMobile**
**Archivo**: `src/hooks/use-mobile.tsx`

Hook para detectar si es dispositivo móvil.

```typescript
const isMobile = useMobile();

if (isMobile) {
  return <MobileView />;
}
```

**Implementación:**
- Usa `window.matchMedia("(max-width: 768px)")`
- Actualiza en resize
- SSR safe

---

#### **useToast**
**Archivo**: `src/hooks/use-toast.ts`

Hook para mostrar notificaciones toast.

```typescript
const { toast } = useToast();

toast({
  title: "Éxito",
  description: "Factura subida correctamente",
  variant: "default", // "default" | "destructive"
});
```

---

## Servicios

### 📡 Servicios de API

**Base Config**: `src/services/config.ts`

```typescript
const API_BASE_URL = import.meta.env.VITE_API_URL || "http://localhost:4005/api";

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

// Interceptor para añadir token
apiClient.interceptors.request.use((config) => {
  const session = loadSession();
  if (session?.tokens?.accessToken) {
    config.headers.Authorization = `Bearer ${session.tokens.accessToken}`;
  }
  return config;
});
```

---

### 🔌 Servicios Disponibles

#### **billsService**
**Archivo**: `src/services/bills.ts`

```typescript
billsService.list(): Promise<Bill[]>
billsService.upload(file: File): Promise<Bill>
billsService.delete(id: string): Promise<void>
```

---

#### **intelligentMatchingService**
**Archivo**: `src/services/intelligentMatching.ts`

```typescript
matchingService.calculate(input: {
  totalConsumptionKwh: number;
  contractedPowerKw: number;
  currentMonthlyCost: number;
}): Promise<Recommendation[]>
```

---

#### **gamificationService**
**Archivo**: `src/services/gamification.ts`

```typescript
gamificationService.getStats(userId: string): Promise<GamificationStats>
gamificationService.claimReward(rewardId: string): Promise<void>
```

---

#### **solarService**
**Archivo**: `src/services/solar.ts`

```typescript
solarService.getBuildingInsights(params: {
  lat: number;
  lng: number;
  peakpower?: number;
}): Promise<SolarInsights>
```

---

## Estilos y UI

### 🎨 Sistema de Diseño

**Framework**: Tailwind CSS v3.4.17

**Configuración**: `tailwind.config.ts`

**Características:**
- ✅ Utility-first approach
- ✅ Variables CSS para temas
- ✅ Dark mode ready (class-based)
- ✅ Responsive breakpoints
- ✅ Custom animations

---

### 🌈 Paleta de Colores

**Archivo**: `src/index.css`

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 221.2 83.2% 53.3%;
  --secondary: 210 40% 96.1%;
  --muted: 210 40% 96.1%;
  --accent: 210 40% 96.1%;
  --destructive: 0 84.2% 60.2%;
  --border: 214.3 31.8% 91.4%;
  --input: 214.3 31.8% 91.4%;
  --ring: 221.2 83.2% 53.3%;
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  /* ...resto de variables dark */
}
```

---

### 📱 Breakpoints Responsive

| Breakpoint | Clase Tailwind | Pixels |
|------------|----------------|--------|
| Mobile | `(default)` | < 640px |
| Tablet | `sm:` | ≥ 640px |
| Laptop | `md:` | ≥ 768px |
| Desktop | `lg:` | ≥ 1024px |
| Wide | `xl:` | ≥ 1280px |
| Ultra Wide | `2xl:` | ≥ 1536px |

---

### ✨ Animaciones

**Archivo**: `tailwind.config.ts`

```javascript
animation: {
  "accordion-down": "accordion-down 0.2s ease-out",
  "accordion-up": "accordion-up 0.2s ease-out",
  "fade-in": "fade-in 0.5s ease-in",
  "slide-in": "slide-in 0.3s ease-out",
}
```

---

## 📊 Resumen de Arquitectura

### Flujo Completo de Autenticación

```
1. Usuario hace click en "Iniciar Sesión"
   ↓
2. <Navigation> abre <AuthModal>
   ↓
3. Usuario completa formulario (React Hook Form + Zod)
   ↓
4. onSubmit → authService.login(email, password)
   ↓
5. API POST /api/auth/login
   ↓
6. Backend retorna { user, tokens }
   ↓
7. authService → setSession({ user, tokens })
   ↓
8. AuthProvider actualiza context
   ↓
9. persistSession(user, tokens) → sessionStorage
   ↓
10. AuthModal ejecuta onSuccess callback
    ↓
11. Modal se cierra
    ↓
12. <Navigation> muestra Avatar + Dropdown
    ↓
13. Componentes re-renderizan con isAuthenticated=true
```

---

### Flujo de Carga de Datos con React Query

```
1. Componente monta → useQuery("bills")
   ↓
2. React Query verifica cache
   ↓
3. Si no hay cache o está stale:
   ↓
4. Ejecuta queryFn → billsService.list()
   ↓
5. billsService → axios.get("/api/bills")
   ↓
6. Interceptor añade Authorization header
   ↓
7. Backend retorna { bills: [...] }
   ↓
8. React Query guarda en cache
   ↓
9. Componente recibe data
   ↓
10. Render de lista de facturas
```

---

## 🚀 Performance y Optimizaciones

### ⚡ Técnicas Implementadas

| Técnica | Implementación | Beneficio |
|---------|----------------|-----------|
| **Lazy Loading** | `React.lazy()` en todas las páginas | Reduce bundle inicial |
| **Code Splitting** | Rutas separadas por chunk | Carga bajo demanda |
| **React Query Cache** | Cache de 60s por defecto | Reduce requests |
| **Memoization** | `useMemo`, `useCallback` | Evita re-renders |
| **Debouncing** | Inputs de búsqueda | Reduce API calls |
| **Image Lazy Load** | `loading="lazy"` en `<img>` | Carga progresiva |
| **Tree Shaking** | Vite + ES Modules | Bundle más pequeño |

---

## 📚 Referencias

- [React Documentation](https://react.dev/)
- [React Router v6](https://reactrouter.com/)
- [TanStack Query](https://tanstack.com/query/latest)
- [React Hook Form](https://react-hook-form.com/)
- [Zod](https://zod.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [shadcn/ui](https://ui.shadcn.com/)
- [Radix UI](https://www.radix-ui.com/)

---

**Documento generado:** Enero 2026  
**Versión Frontend:** 0.0.0  
**Última actualización:** 2026-01-15
