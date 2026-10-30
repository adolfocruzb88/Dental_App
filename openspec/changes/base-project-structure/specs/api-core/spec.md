# Spec Delta

## Purpose

Define la API REST base con controladores, middleware, validación, manejo de errores, versionado, documentación OpenAPI/Swagger y health checks.

## ADDED Requirements

### Requirement: API REST con versionado y convenciones
El sistema DEBE exponer API RESTful versionada (v1) con convenciones consistentes: rutas en kebab-case, JSON camelCase, paginación cursor-based, filtrado y ordenamiento estándar.

#### Scenario: Endpoint versionado accesible
- **WHEN** cliente llama GET /api/v1/health
- **THEN** responde 200 con versión y timestamp

#### Scenario: Convenciones de respuesta consistentes
- **WHEN** cualquier endpoint exitoso responde
- **THEN** body: { data: T, meta?: PaginationMeta }, headers: X-Request-Id

#### Scenario: Paginación cursor-based
- **WHEN** GET /api/v1/patients?cursor=abc&limit=20
- **THEN** responde con data[], nextCursor, hasMore

### Requirement: Validación de entrada con FluentValidation
El sistema DEBE validar todos los DTOs de entrada con FluentValidation, retornando errores estructurados.

#### Scenario: Validación fallida retorna 400 estructurado
- **WHEN** POST /api/v1/patients con email inválido
- **THEN** responde 400 con { errors: [{ field: "email", code: "invalid_email", message: "..." }] }

#### Scenario: Validación de reglas de negocio
- **WHEN** se intenta crear paciente con DNI duplicado
- **THEN** responde 409 Conflict con error code "duplicate_dni"

### Requirement: Manejo global de errores y ProblemDetails
El sistema DEBE implementar middleware global que capture excepciones y retorne RFC 7807 ProblemDetails.

#### Scenario: Excepción no controlada retorna 500
- **WHEN** ocurre excepción inesperada
- **THEN** responde 500 con ProblemDetails (type, title, status, detail, instance, traceId)

#### Scenario: Excepción de validación retorna 400
- **WHEN** FluentValidation falla
- **THEN** responde 400 con ProblemDetails y extensions.errors

#### Scenario: NotFound retorna 404
- **WHEN** recurso no existe
- **THEN** responde 404 con ProblemDetails

### Requirement: Documentación OpenAPI/Swagger
El sistema DEBE generar especificación OpenAPI 3.1 completa con ejemplos, esquemas de autenticación y agrupación por tags.

#### Scenario: Swagger UI accesible
- **WHEN** se navega a /swagger en desarrollo
- **THEN** UI carga con todos los endpoints documentados

#### Scenario: Esquema de autenticación documentado
- **WHEN** se inspecciona spec OpenAPI
- **THEN** incluye securitySchemes: BearerAuth (JWT) y CookieAuth (refresh)

### Requirement: Health checks y observabilidad
El sistema DEBE exponer health checks para API, base de datos y dependencias externas.

#### Scenario: Health check básico
- **WHEN** GET /health
- **THEN** responde 200 con { status: "Healthy", checks: [...] }

#### Scenario: Health check de base de datos
- **WHEN** base de datos inaccesible
- **THEN** /health responde 503 con status "Unhealthy" y check db fallido

#### Scenario: Request logging con correlation ID
- **WHEN** cualquier request procesado
- **THEN** logs incluyen X-Request-Id, método, path, status, duración