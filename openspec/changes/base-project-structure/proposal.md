# Propuesta

## Por qué

El CRM dental necesita una estructura base de proyecto bien definida para ASP.NET Core (.NET 10) en el backend y React + TypeScript en el frontend. Actualmente no existe ninguna estructura de proyecto, lo que impide el desarrollo ordenado de funcionalidades como gestión de pacientes, citas, tratamientos, facturación y dashboard.

## Qué cambios

- Crear solución .NET con proyectos para API, Dominio, Infraestructura y Tests
- Configurar proyecto React + TypeScript + Vite para el frontend
- Establecer arquitectura limpia (Clean Architecture) con separación de capas
- Configurar Entity Framework Core con MariaDB
- Configurar autenticación/autorización con JWT y ASP.NET Core Identity
- Establecer pipeline de CI/CD básico
- Configurar Docker para desarrollo y producción
- Establecer convenciones de código, linting y testing

## Capacidades

### Nuevas Capacidades

- `project-structure`: Estructura base de la solución .NET y proyecto React, arquitectura en capas, configuración de build y tooling
- `data-access`: Capa de persistencia con Entity Framework Core, migraciones, repositorios y unidad de trabajo
- `authentication`: Autenticación y autorización con JWT, ASP.NET Core Identity, roles y permisos
- `api-core`: API REST base con controladores, middleware, validación, manejo de errores y documentación OpenAPI/Swagger

### Capacidades Modificadas

*Ninguna - es un proyecto nuevo*

## Impacto

- **Código**: Nueva solución completa en `src/` (backend) y `frontend/` (React)
- **APIs**: Endpoints base de salud, autenticación y versión
- **Dependencias**: Paquetes NuGet para EF Core, Identity, MediatR, FluentValidation, etc. Paquetes npm para React, React Router, TanStack Query, Zod, etc.
- **Sistemas**: MariaDB, Docker, GitHub Actions
- **Infraestructura**: docker-compose.yml, Dockerfile, .github/workflows/