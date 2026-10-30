# Tasks

## 1. Backend: Solución y Estructura de Proyectos

- [ ] 1.1 Crear solución `Dental_App.sln` y proyectos: `Dental_App.Api`, `Dental_App.Application`, `Dental_App.Domain`, `Dental_App.Infrastructure`, `Dental_App.Tests.Unit`, `Dental_App.Tests.Integration` — verificar con `dotnet sln list`
- [ ] 1.2 Configurar referencias entre proyectos según Clean Architecture (Api → Application, Infrastructure; Application → Domain; Infrastructure → Domain, Application; Domain sin referencias) — verificar con `dotnet build`
- [ ] 1.3 Agregar paquetes NuGet base: `MediatR`, `FluentValidation`, `FluentValidation.DependencyInjectionExtensions`, `Mapster`, `Serilog.AspNetCore`, `Serilog.Sinks.Console`, `Serilog.Enrichers.Environment` — verificar `dotnet restore`
- [ ] 1.4 Configurar `.editorconfig`, `Directory.Build.props` (nullable, warnings as errors, analyzers: SonarAnalyzer, Roslynator) — verificar `dotnet format --verify-no-changes` y `dotnet build` sin warnings
- [ ] 1.5 Configurar `global.json` para .NET 10 SDK — verificar `dotnet --version` en el directorio

## 2. Backend: Domain (Entidades Base e Interfaces)

- [ ] 2.1 Crear `AuditableEntity` abstracta con `Id`, `CreatedAt`, `CreatedBy`, `UpdatedAt`, `UpdatedBy`, `IsDeleted`, `DeletedAt` — verificar compilación
- [ ] 2.2 Crear interfaces `IRepository<T>`, `IUnitOfWork`, `IAuditableEntity` en `Domain/Common` — verificar compilación
- [ ] 2.3 Crear excepciones de dominio: `DomainException`, `NotFoundException`, `ConflictException`, `ValidationException` — verificar compilación
- [ ] 2.4 Definir enums base: `UserRole` (Admin, Dentist, Assistant, Receptionist), `Permission` (claims granulares) — verificar compilación

## 3. Backend: Application (MediatR, Validators, Behaviors)

- [ ] 3.1 Configurar MediatR en `Application/DependencyInjection.cs` con `AddMediatR(cfg => cfg.RegisterServicesFromAssemblyContaining<Program>())` — verificar compilación
- [ ] 3.2 Crear `ValidationBehavior<TRequest, TResponse>` pipeline behavior usando FluentValidation — verificar test unitario que valida request inválido retorna error
- [ ] 3.3 Crear `LoggingBehavior<TRequest, TResponse>` para logging estructurado con Serilog — verificar logs en test
- [ ] 3.4 Crear `TransactionBehavior<TRequest, TResponse>` que usa `IUnitOfWork` para commit/rollback automático — verificar test de transacción
- [ ] 3.5 Crear validadores base abstractos y extension methods para reglas comunes (email, DNI, phone) — verificar tests de validadores

## 4. Backend: Infrastructure (EF Core, Identity, Repositorios)

- [ ] 4.1 Crear `ApplicationDbContext` heredando de `IdentityDbContext<ApplicationUser, ApplicationRole, Guid>` — verificar compilación
- [ ] 4.2 Configurar convenciones EF Core: snake_case para tablas/columnas, global query filter `IsDeleted == false` — verificar migración generada
- [ ] 4.3 Crear configuraciones Fluent API para `ApplicationUser`, `ApplicationRole`, `AuditableEntity` en `Infrastructure/Configurations` — verificar migración
- [ ] 4.4 Implementar `Repository<T>` genérico y `UnitOfWork` con `SaveChangesAsync()` — verificar tests de integración CRUD
- [ ] 4.5 Implementar `SaveChangesInterceptor` para poblar auditoría (CreatedAt/By, UpdatedAt/By) — verificar test de auditoría automática
- [ ] 4.6 Configurar ASP.NET Core Identity: password policy, lockout (5 intentos, 15 min), email confirmation, token providers — verificar test de login/lockout
- [ ] 4.7 Crear migración inicial `InitialCreate` y seed data (roles, usuario admin por defecto) — verificar `dotnet ef database update` crea schema correcto
- [ ] 4.8 Implementar `IEmailService` con proveedor configurable (SMTP/Mailhog para dev) — verificar test de envío email

## 5. Backend: API (Controllers, Middleware, Swagger, Health)

- [ ] 5.1 Configurar `Program.cs`: DI, MediatR, EF Core, Identity, JWT Bearer, CORS, Serilog, HealthChecks, Swagger — verificar `dotnet run` inicia sin errores
- [ ] 5.2 Implementar middleware global de excepciones que retorna ProblemDetails (RFC 7807) con traceId — verificar test: excepción → 500 ProblemDetails
- [ ] 5.3 Configurar autenticación JWT: `AddJwtBearer` con validación de issuer, audience, lifetime; refresh token en cookie HttpOnly — verificar test: login → access token + refresh cookie
- [ ] 5.4 Implementar endpoints Auth: `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/logout`, `POST /api/v1/auth/forgot-password`, `POST /api/v1/auth/reset-password` — verificar tests de integración auth flow
- [ ] 5.5 Implementar policies de autorización basadas en claims de permisos (`patients.read`, `patients.write`, `appointments.*`, `billing.*`, `admin.*`) — verificar test: usuario sin permiso → 403
- [ ] 5.6 Configurar Swagger/OpenAPI 3.1 con security schemes (BearerAuth, CookieAuth), XML comments, UI en `/swagger` (solo Development) — verificar `/swagger/v1/swagger.json` válido
- [ ] 5.7 Configurar Health Checks: `/health` (liveness), `/health/ready` (readiness con DB check), HealthChecksUI en `/health-ui` (Development) — verificar endpoints responden correctamente
- [ ] 5.8 Implementar request logging middleware con correlation ID (X-Request-Id header o generado) — verificar logs incluyen correlation ID
- [ ] 5.9 Crear controladores base: `BaseApiController` con helpers para ProblemDetails, paginación cursor-based — verificar test de paginación

## 6. Frontend: Setup y Configuración Base

- [ ] 6.1 Crear proyecto Vite + React + TypeScript en `frontend/` — verificar `npm run dev` inicia en localhost:5173
- [ ] 6.2 Instalar dependencias: `react-router-dom`, `@tanstack/react-query`, `axios`, `zod`, `@hookform/resolvers`, `react-hook-form`, `clsx`, `tailwind-merge`, `lucide-react` — verificar `npm install` sin errores
- [ ] 6.3 Configurar TypeScript strict, path aliases (`@/`, `@components/`, `@features/`, `@hooks/`, `@lib/`), ESLint (typescript-eslint, react-hooks, jsx-a11y), Prettier, stylelint — verificar `npm run lint` y `npm run format:check` pasan
- [ ] 6.4 Configurar Tailwind CSS v4 con diseño system (colores, spacing, typography) — verificar build de CSS
- [ ] 6.5 Configurar TanStack Query `QueryClientProvider` con `axios` interceptor para Authorization header y error handling global — verificar request incluye token
- [ ] 6.6 Crear `AuthContext` con `user`, `accessToken`, `login()`, `logout()`, `refresh()`, `isAuthenticated` — verificar login/logout flow en browser
- [ ] 6.7 Implementar routing con React Router v6: rutas públicas (login), protegidas (dashboard), layout con sidebar/header — verificar navegación y guards
- [ ] 6.8 Crear componentes base UI: `Button`, `Input`, `Card`, `Table`, `Modal`, `Toast`, `Spinner`, `Avatar`, `Badge` — verificar Storybook o visual test

## 7. Frontend: Feature Auth (Login, Registro, Password Reset)

- [ ] 7.1 Crear feature `auth` con `api.ts` (endpoints tipados con Zod), `hooks.ts` (useLogin, useRegister, useForgotPassword, useResetPassword), `types.ts` — verificar types compilan
- [ ] 7.2 Implementar página `LoginPage` con formulario validado (Zod + react-hook-form), manejo de errores, loading state — verificar login exitoso redirige a dashboard
- [ ] 7.3 Implementar página `ForgotPasswordPage` y `ResetPasswordPage` — verificar flow completo de reset
- [ ] 7.4 Implementar `ProtectedRoute` wrapper que verifica auth y redirige a login — verificar acceso sin token redirige a login

## 8. Docker y Docker Compose

- [ ] 8.1 Crear `Dockerfile` backend multi-stage (build → runtime alpine) con non-root user — verificar `docker build -t dental-app-api .` produce imagen < 200MB
- [ ] 8.2 Crear `Dockerfile` frontend multi-stage (build → nginx alpine) con config nginx para SPA routing — verificar `docker build -t dental-app-frontend .` produce imagen < 50MB
- [ ] 8.3 Crear `docker-compose.yml` con services: `api` (puerto 8080), `frontend` (puerto 80), `postgres` (puerto 5432), `mailhog` (puerto 1025/8025), `pgadmin` (opcional) — verificar `docker-compose up -d` levanta todo saludable
- [ ] 8.4 Configurar health checks en docker-compose para api (`/health`), postgres (`pg_isready`), frontend (nginx) — verificar `docker-compose ps` muestra healthy
- [ ] 8.5 Crear `.dockerignore` excluyendo `bin/`, `obj/`, `node_modules/`, `.git/`, `*.md`, `Dockerfile*`, `docker-compose*` — verificar contexto de build pequeño

## 9. CI/CD: GitHub Actions

- [ ] 9.1 Crear `.github/workflows/ci.yml`: checkout, setup .NET, restore, build, test (unit + integration), lint (dotnet format), build docker images — verificar workflow pasa en PR
- [ ] 9.2 Crear `.github/workflows/cd.yml`: trigger en merge a main, build, push imágenes a GHCR, deploy manual approval — verificar workflow ejecuta en merge
- [ ] 9.3 Configurar secrets en GitHub: `POSTGRES_PASSWORD`, `JWT_SECRET`, `GHCR_TOKEN` — verificar workflows acceden a secrets

## 10. Tests y Verificación End-to-End

- [ ] 10.1 Configurar proyecto `Tests.Unit` con xUnit, Moq, FluentAssertions, AutoFixture — verificar `dotnet test` pasa
- [ ] 10.2 Configurar proyecto `Tests.Integration` con Testcontainers (MariaDB), WebApplicationFactory — verificar test de integración auth flow pasa
- [ ] 10.3 Escribir tests unitarios para: validadores, domain exceptions, repository/UoW, interceptors — verificar coverage > 80% en Domain/Application
- [ ] 10.4 Escribir tests de integración para: login/refresh/logout, registro usuario, health checks, CRUD básico — verificar tests pasan contra Testcontainers
- [ ] 10.5 Configurar frontend tests: Vitest + React Testing Library, MSW para API mocking — verificar `npm run test` pasa
- [ ] 10.6 Smoke test E2E local: `docker-compose up -d`, verificar login en frontend, crear usuario via API, health checks — verificar todo funciona integrado

## Workflow follow-up

- Archivar el cambio después de que los requisitos de revisión del proyecto se cumplan.
- Verificar el resultado archivado.