# Escoge tu Energía – Monorepo (React + Express + MySQL)

Aplicación completa para comparar tarifas energéticas, gestionar usuarios y operar con facturas. Incluye frontend en React + Vite + TypeScript y backend en Node/Express con Prisma sobre MySQL.

##  Documentación Técnica Completa

Este proyecto cuenta con una suite completa de documentación técnica (5,738 líneas, ~114 páginas):

| Documento | Descripción | Tamaño |
|-----------|-------------|--------|
| [README_TECNICO.md](README_TECNICO.md) | Tech stack completo, arquitectura, estructura de proyecto y guía de inicio | 38 KB |
| [BASE_DE_DATOS.md](BASE_DE_DATOS.md) | Esquema ER (Mermaid), 15 tablas, diccionario de datos, índices y relaciones | 30 KB |
| [BACKEND_API.md](BACKEND_API.md) | Endpoints REST, 9 servicios principales, autenticación JWT, ejemplos de uso | 33 KB |
| [FRONTEND.md](FRONTEND.md) | Arquitectura React, 30+ componentes, estado con Context/Query, rutas y hooks | 32 KB |
| [INFRAESTRUCTURA.md](INFRAESTRUCTURA.md) | Docker/Compose, despliegue en producción, variables .env, CI/CD, troubleshooting | 31 KB |

**Inicia por:** [README_TECNICO.md](README_TECNICO.md) para una visión general, luego consulta los documentos específicos según necesites.

---

## Documentación rápida para terceros

- `docs/PROJECT_OVERVIEW.md`: visión general del proyecto, dependencias, servicios, variables de entorno, flujos y puntos a mejorar.

## Arquitectura

- **Frontend**: React 18, Vite, TypeScript, Tailwind/shadcn, React Query, React Router con carga perezosa, contexto de autenticación, Atomic/Container + hooks y servicios desacoplados.
- **Backend**: Express 5, Prisma (MySQL), capas separadas (`routes` → `services` → `repositories`), middlewares de seguridad (helmet, hpp, rate limit, compression, CORS restringido), validación con Zod, DTOs y transformadores.
- **Infra/DevOps**: GitHub Actions CI (lint/test/build), Husky + lint-staged, Vitest para FE/BE.
- **Integraciones**: Sincronización de consumo horario vía Datadis (login + descarga últimos 30 días y almacenamiento en `hourly_consumptions`).

## Requisitos

- Node.js 20+
- npm 10+
- MySQL 8.0+ instalado localmente (sin contenedores)

## Instalación y scripts

```bash
# Frontend
npm ci
npm run dev          # http://localhost:5173
npm run lint
npm run test
npm run build

# Backend
cd server
npm ci
npm run prisma:migrate  # crear/actualizar BD
npm run dev             # API en http://localhost:4005
npm run lint
npm run test
npm run build
```

### Variables de entorno

Frontend (`.env`):  
`VITE_API_URL`, `VITE_GOOGLE_CLIENT_ID`

Backend (`server/.env` - ver `.env.example`):  
`DATABASE_URL`, `PORT`, `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `FRONTEND_APP_URL`, `GOOGLE_*`, `FACEBOOK_*`, `BCRYPT_SALT_ROUNDS`, `ADMIN_EMAILS`, `UPLOAD_DIR`, `ACCESS_TOKEN_EXPIRES_IN`, `REFRESH_TOKEN_EXPIRES_IN`.

### Base de datos MySQL local

1) Arranca tu instancia MySQL 8.0+ local (sin contenedores).
2) Crea la base de datos `escogetuenergia` (o la que definas en `DATABASE_URL`): `CREATE DATABASE escogetuenergia;`.
3) Ajusta `DATABASE_URL` en `server/.env` apuntando a tu instancia, por ejemplo `mysql://root:root@localhost:3306/escogetuenergia`.
4) Ejecuta `cd server && npm run prisma:migrate` para aplicar el esquema antes de levantar la API.

## Estructura de carpetas

```
src/
  app/ (router, providers, layouts)
  components/ (ui + features)
  pages/ (rutas con PageLayout + lazy loading)
  services/, lib/, types/
server/
  src/
    routes/        # controladores HTTP
    services/      # lógica de dominio
    repositories/  # acceso a datos Prisma
    middleware/, utils/, config/
  prisma/schema.prisma
.github/workflows/ci.yml
```

## Testing

- **Frontend**: Vitest + Testing Library (`npm run test`). Ejemplo: `src/lib/session.test.ts` valida persistencia de sesión.
- **Backend**: Vitest (`npm run test --prefix server`). Ejemplo: `src/utils/password.test.ts` verifica hashing/validación.
- Separar unit vs integración: mockear fetch/servicios en FE; en BE mockear Prisma/repositorios para servicios.
- Sincronización Datadis: probar con credenciales reales; endpoint `POST /api/datadis/sync` (autenticado) recibe `{ username, password }` y descarga últimos 30 días.

## CI/CD

GitHub Actions (`ci.yml`) ejecuta lint + test + build para frontend y backend en cada push/PR.

## Flujo recomendado

1) Crear `.env` y `server/.env` desde los ejemplos.  
2) `npm ci` (frontend) y `npm ci` en `server/`.  
3) Arrancar MySQL local y ejecutar `npm run prisma:migrate --prefix server` para aplicar el esquema.  
4) `npm run dev` (frontend) y `npm run dev --prefix server` para API.  
5) Tests: `npm run test` (FE) y `npm run test --prefix server` (BE).  
6) Build: `npm run build` / `npm run build --prefix server`.  

## Notas de diseño

- Navegación y layout unificados con `PageLayout` (skip-link accesible, Onboarding global, cabecera fija).
- Autenticación centralizada con `AuthProvider` + almacenamiento de sesión seguro (sessionStorage), refresco controlado y perfil en `UserProfileDrawer`.
- Rutas con `React.lazy`/`Suspense` para code splitting y optimización de rendimiento.
- Backoffice separado en servicios/repositorios para reducir acoplamiento, trazabilidad y testabilidad.
- Middlewares de seguridad: helmet, hpp, CORS restringido, rate limiting y compresión.
