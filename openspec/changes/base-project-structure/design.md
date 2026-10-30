# Design

## Context

Proyecto nuevo (greenfield) para CRM dental. No hay código existente. La propuesta define 4 capacidades: project-structure, data-access, authentication, api-core. Los specs detallan requisitos de comportamiento para cada uno.

Stack objetivo:
- Backend: .NET 10, ASP.NET Core, EF Core 9, Mariadb 13.0.2
- Frontend: React 18, TypeScript 5, Vite, TanStack Query, React Router, Zod
- Infra: Docker, docker-compose, GitHub Actions
- Auth: ASP.NET Core Identity + JWT + refresh tokens

## Goals / Non-Goals

**Goals:**
- Establecer arquitectura Clean Architecture en backend con 5 proyectos
- Configurar frontend modular con feature-based structure
- Implementar autenticación completa con JWT + refresh token rotation
- API REST versionada con validación, manejo de errores, OpenAPI, health checks
- Persistencia con EF Core, repositorios, UoW, auditoría y soft delete
- Pipeline CI/CD que compile, testee, lintee y construya imágenes Docker

**Non-Goals:**
- Funcionalidades de negocio (pacientes, citas, tratamientos, facturación) - vendrán en cambios posteriores
- UI completa del frontend - solo shell, routing, auth context, query client
- Tests de integración exhaustivos - solo setup y smoke tests
- Despliegue a cloud - solo Docker local y CI

## Decisions

### 1. Arquitectura Backend: Clean Architecture con 5 proyectos

**Decisión:** Separar en Dental_App.Api, Dental_App.Application, Dental_App.Domain, Dental_App.Infrastructure, Dental_App.Tests (Unit + Integration).

**Rationale:** Clean Architecture fuerza dependencias hacia adentro. Domain puro (sin dependencias externas), Application con casos de uso (MediatR), Infrastructure con implementaciones concretas (EF Core, Identity, email), Api como entry point delgada.

**Alternativas consideradas:**
- 3 capas (Api, Core, Infrastructure): más simple pero mezcla casos de uso con dominio
- Vertical Slice Architecture: buena para features, pero menos familiar para el equipo y requiere más disciplina

### 2. Orquestación de casos de uso: MediatR

**Decisión:** Usar MediatR para commands/queries en Application. Handlers en Application, no en Api.

**Rationale:** Desacopla controladores de lógica de negocio. Facilita testing (handlers testeables sin HTTP). Pipeline behaviors para validación, logging, transacciones.

**Alternativas:** Minimal APIs con endpoint filters, o controllers con services inyectados directamente. MediatR es estándar en .NET para CQRS ligero.

### 3. Validación: FluentValidation + Pipeline Behavior

**Decisión:** FluentValidation para DTOs, integrado via MediatR pipeline behavior (ValidationBehavior). Retorna ProblemDetails 400.

**Rationale:** Validación declarativa, testeable, separada de handlers. Pipeline behavior la aplica transversalmente.

### 4. Autenticación: ASP.NET Core Identity + JWT Bearer + Refresh Token Rotation

**Decisión:** Identity para user management (password hashing, lockout, email confirmation, 2FA ready). JWT short-lived (15 min) en header Authorization. Refresh token en HTTP-only Secure SameSite=Strict cookie (7 días), rotado en cada uso, revocado en logout.

**Rationale:** Identity maneja complejidad de seguridad (hashing, lockout, tokens). JWT stateless para escalabilidad. Refresh token rotation mitiga robo de token. Cookie HttpOnly previene XSS.

**Alternativas:** Solo JWT largo (riesgo), solo cookies (CSRF risk), IdentityServer/Duende (overkill para monolith).

### 5. Autorización: Policy-based con Claims de Permisos

**Decisión:** Roles (Admin, Dentist, Assistant, Receptionist) mapeados a claims de permisos granulares. Policies en Program.cs: `RequireClaim("permission", "patients.write")`. Handlers verifican permisos via `IAuthorizationService`.

**Rationale:** Más flexible que roles solos. Permite cambiar permisos sin tocar código. Claims vienen en JWT.

### 6. Persistencia: EF Core Code-First + Migraciones

**Decisión:** DbContext en Infrastructure. Entidades en Domain. Configuraciones Fluent API en Infrastructure/Configurations. Migraciones en Infrastructure/Migrations.

**Rationale:** Domain sin dependencia a EF. Migraciones versionadas en código. Fluent API separa mapping de entidad.

### 7. Repositorios: Genérico + Específicos + Unit of Work

**Decisión:** `IRepository<T>` genérico (CRUD) + repositorios específicos para queries complejas (`IPatientRepository`). `IUnitOfWork` con `SaveChangesAsync()` y transacción implícita.

**Rationale:** Genérico reduce boilerplate. Específicos encapsulan queries de dominio. UoW garantiza atomicidad.

### 8. Auditoría y Soft Delete: Interceptors + Global Query Filters

**Decisión:** `AuditableEntity` base con CreatedAt/By, UpdatedAt/By, IsDeleted, DeletedAt. `SaveChangesInterceptor` puebla auditoría. Global Query Filter `IsDeleted == false` en DbContext. `IncludeDeleted()` para admin.

**Rationale:** Transparente para repositorios. Interceptor centraliza lógica. Query filter evita leaks de datos borrados.

### 9. API: Controllers + Minimal APIs Híbrido

**Decisión:** Controllers para endpoints con lógica de presentación (auth, upload). Minimal APIs para CRUD simples (patients, appointments). Versionado en ruta `/api/v1/`.

**Rationale:** Controllers mejor para auth compleja. Minimal APIs menos ceremony para CRUD. Versionado en ruta es explícito y cacheable.

### 10. Manejo de Errores: ProblemDetails + Exception Middleware

**Decisión:** `app.UseExceptionHandler()` + middleware custom que mapea excepciones a ProblemDetails (RFC 7807). FluentValidation exceptions → 400 con `extensions.errors`. NotFound → 404. Domain exceptions → 409/400. Unhandled → 500 con traceId.

**Rationale:** Estándar interoperable. Cliente puede parsear errores consistentemente. TraceId correlaciona logs.

### 11. Documentación: Swashbuckle + Anotaciones XML

**Decisión:** Swashbuckle.AspNetCore para OpenAPI 3.1. Comentarios XML en controllers/handlers para summaries. `AddSecurityDefinition` para Bearer y Cookie. UI en `/swagger` solo Development.

**Rationale:** Genera spec automática. XML mejora DX. Security schemes documentan auth.

### 12. Health Checks: ASP.NET Core HealthChecks + UI

**Decisión:** `AddHealthChecks()` con checks para DB (Pomelo.EntityFrameworkCore.MySql), self, y dependencias externas. Endpoint `/health` (liveness) y `/health/ready` (readiness). HealthChecksUI en `/health-ui` solo Development.

**Rationale:** K8s/Docker necesitan liveness/readiness separados. UI ayuda debugging local.

### 13. Frontend: Feature-based Structure + TanStack Query

**Decisión:** `src/features/{auth,patients,appointments,...}/` con `api.ts` (endpoints), `hooks.ts` (queries/mutations), `components/`, `types.ts`. TanStack Query para server state. React Router v6 para routing. Zod para validación de formularios (react-hook-form + zodResolver).

**Rationale:** Colocation por feature escala mejor que layers. TanStack Query maneja cache, invalidation, optimistic updates. Zod comparte schemas con backend (posible codegen futuro).

### 14. State Management: Auth Context + TanStack Query

**Decisión:** `AuthProvider` con user, accessToken, login/logout/refresh. Access token en memoria (no localStorage). Refresh via cookie automática. TanStack Query `queryClient` con `Authorization` header interceptor.

**Rationale:** Access token en memoria previene XSS. Cookie HttpOnly para refresh. Interceptor centraliza auth header.

### 15. Docker: Multi-stage Builds + docker-compose

**Decisión:** Dockerfile backend multi-stage (build → runtime). Dockerfile frontend multi-stage (build → nginx). docker-compose.yml con services: api, frontend, maradb, (opcional: pgadmin, mailhog). `.dockerignore` excluye bin/obj, node_modules, .git.

**Rationale:** Imágenes pequeñas en producción. Build reproducible. Compose para dev local completo.

### 16. CI/CD: GitHub Actions

**Decisión:** Workflow `ci.yml` en push/PR: restore, build, test, lint, format check, build docker images. Workflow `cd.yml` en merge a main: build, push images a GHCR, deploy (manual approval).

**Rationale:** Fast feedback en PR. Separación CI/CD. GHCR gratis para private repos.

### 17. Convenciones de Código

**Backend:** .editorconfig + dotnet format (style: Allman, indent: 4, newline: LF). `nullable: enable` global. `TreatWarningsAsErrors` en CI. Analyzers: SonarAnalyzer, Roslynator.

**Frontend:** ESLint (typescript-eslint, react-hooks, jsx-a11y), Prettier, stylelint. `npm run lint` y `npm run format:check` en CI. Husky pre-commit para lint-staged.

### 18. Logging: Serilog + Structured Logging

**Decisión:** Serilog con output JSON a stdout. Enrich: FromLogContext, WithThreadId, WithMachineName. Seq/Grafana Loki en prod. Request logging middleware con correlation ID (X-Request-Id header o generado).

**Rationale:** Structured logs queryables. Correlation ID traza request across services.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Clean Architecture añade complejidad inicial | Documentar bien boundaries. Code reviews enforcen dependencias. |
| MediatR añade indirección | Usar solo para commands/queries. No para todo. |
| Refresh token rotation puede causar race conditions | Implementar "refresh token family" con reuse detection. |
| EF Core global query filter puede afectar performance | Índices en IsDeleted. Monitorear query plans. |
| Versionado en ruta vs header | Ruta es más simple y cacheable. Header requiere Vary header. |
| Frontend feature-based requiere disciplina | ESLint rule para import boundaries. Code review. |
| Docker multi-stage aumenta build time | Cache layers correctamente. Usar BuildKit. |

## Migration Plan

1. Crear solución .NET y proyectos
2. Configurar Domain (entidades base, interfaces)
3. Configurar Application (MediatR, validators, handlers base)
4. Configurar Infrastructure (EF Core, Identity, repositorios, email)
4. Configurar Api (controllers, middleware, Scalar, health)
5. Crear migración inicial y seed data (roles, admin user)
6. Configurar frontend (Vite, TS, Router, Query, Auth context)
7. Dockerfile + docker-compose
8. GitHub Actions CI
9. Smoke tests end-to-end local

Rollback: `docker-compose down -v` elimina volúmenes. Migraciones EF Core son reversibles (`dotnet ef database update <previous>`).

## Decisiones Técnicas Documentadas

Las siguientes decisiones quedan registradas como provisionales hasta su validación final. No se implementarán integraciones reales que dependan de proveedores no seleccionados.

### DEC-001: Proveedor de correo electrónico
**Estado:** Pendiente de selección del proveedor.

**Decisión provisional:**
- Definir una abstracción `IEmailService`.
- Mantener desacoplada la lógica de negocio del proveedor.
- Gestionar credenciales mediante configuración segura.
- No implementar el envío real hasta seleccionar el proveedor.

**Criterio de aceptación:** Cambiar de proveedor no debe requerir modificar la lógica de negocio de pacientes, usuarios o citas.

### DEC-002: Autenticación de dos factores (2FA)
**Estado:** Pendiente de decisión sobre obligatoriedad en producción.

**Decisión provisional:**
- Diseñar la autenticación para admitir 2FA.
- No exigirlo durante el desarrollo local inicial.
- Evaluar su obligatoriedad antes de producción.
- Definir qué roles y operaciones requieren protección adicional.

**Criterio de aceptación:** La decisión sobre 2FA debe estar documentada antes de publicar el sistema.

### DEC-003: Limitación de solicitudes (Rate Limiting)
**Estado:** Decisión técnica propuesta, pendiente de implementación y validación.

**Decisión provisional:**
- Utilizar el middleware integrado de ASP.NET Core.
- Configurar políticas para autenticación, recuperación de contraseñas y endpoints sensibles.
- Devolver HTTP 429 al superar los límites aplicables.
- Evaluar una estrategia compartida si la aplicación utiliza varias instancias.

**Criterio de aceptación:** Las políticas deben contar con pruebas automatizadas y estar configuradas antes de exponer la API a Internet.

### DEC-004: Generación de tipos TypeScript desde OpenAPI
**Estado:** Pendiente de validar la herramienta definitiva.

**Decisión provisional:**
- Evaluar `openapi-typescript` como primera alternativa.
- Mantener OpenAPI como contrato de la API.
- Evaluar Orval si se requiere generar también el cliente HTTP.
- Incorporar un proceso reproducible para generar los tipos.

**Criterio de aceptación:** El proceso de generación debe estar documentado y permitir detectar cambios incompatibles entre el contrato de la API y el frontend.

## Reglas Generales para Decisiones Pendientes

- No presentar las decisiones provisionales como definitivas.
- No implementar integraciones reales que dependan de proveedores aún no seleccionados.
- No bloquear el desarrollo de funcionalidades independientes de estas decisiones.
- Registrar las decisiones definitivas en la documentación de arquitectura cuando sean aprobadas.
- Actualizar los requisitos y las pruebas cuando una decisión afecte el comportamiento del sistema.