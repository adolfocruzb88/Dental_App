# Spec Delta

## Purpose

Define la estructura base de la solución .NET y el proyecto React, incluyendo la arquitectura en capas (Clean Architecture), configuración de build, tooling y convenciones de proyecto.

## ADDED Requirements

### Requirement: Solución .NET con arquitectura en capas
El sistema DEBE proporcionar una solución .NET 10 con proyectos separados para API, Dominio, Aplicación, Infraestructura y Tests siguiendo Clean Architecture.

#### Scenario: Estructura de proyectos correcta
- **WHEN** se abre la solución Dental_App.sln
- **THEN** contiene proyectos: Dental_App.Api, Dental_App.Domain, Dental_App.Application, Dental_App.Infrastructure, Dental_App.Tests.Unit, Dental_App.Tests.Integration

#### Scenario: Dependencias unidireccionales
- **WHEN** se analizan las referencias entre proyectos
- **THEN** Api referencia Application e Infrastructure; Application referencia Domain; Infrastructure referencia Domain y Application; Domain no tiene referencias a otros proyectos

### Requirement: Proyecto React + TypeScript configurado
El sistema DEBE proporcionar un proyecto frontend con React 18+, TypeScript 5+, Vite, React Router, TanStack Query y Zod.

#### Scenario: Proyecto frontend compila y ejecuta
- **WHEN** se ejecuta `npm run dev` en el directorio frontend
- **THEN** la aplicación inicia en http://localhost:5173 sin errores de TypeScript

#### Scenario: Configuración de path aliases
- **WHEN** se importa desde `@/` o `@components/`
- **THEN** TypeScript resuelve correctamente a `src/` y `src/components/`

### Requirement: Configuración de build y tooling
El sistema DEBE incluir configuración de linting, formateo y análisis estático para ambos proyectos.

#### Scenario: Backend pasa validaciones
- **WHEN** se ejecuta `dotnet build` y `dotnet format --verify-no-changes`
- **THEN** la compilación succeeds sin warnings y el formateo pasa

#### Scenario: Frontend pasa validaciones
- **WHEN** se ejecuta `npm run lint` y `npm run format:check`
- **THEN** ESLint y Prettier pasan sin errores

### Requirement: Docker para desarrollo y producción
El sistema DEBE incluir Dockerfile y docker-compose para ejecutar la aplicación completa localmente.

#### Scenario: Stack completo levanta con docker-compose
- **WHEN** se ejecuta `docker-compose up -d`
- **THEN** API, frontend y MariaDB están accesibles y saludables

#### Scenario: Health checks funcionan
- **WHEN** se consulta `/health` en API y frontend
- **THEN** ambos responden 200 OK con estado healthy