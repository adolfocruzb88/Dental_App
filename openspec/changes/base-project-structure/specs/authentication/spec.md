# Spec Delta

## Purpose

Define la autenticación y autorización con JWT, ASP.NET Core Identity, roles, permisos y políticas de acceso para el CRM dental.

## ADDED Requirements

### Requirement: Autenticación JWT con ASP.NET Core Identity
El sistema DEBE implementar autenticación basada en tokens JWT con refresh tokens, usando ASP.NET Core Identity para gestión de usuarios.

#### Scenario: Login exitoso retorna tokens
- **WHEN** usuario envía credenciales válidas a POST /api/auth/login
- **THEN** responde 200 con access token (JWT, 15 min) y refresh token (HTTP-only cookie, 7 días)

#### Scenario: Refresh token renueva access token
- **WHEN** cliente envía refresh token válido a POST /api/auth/refresh
- **THEN** responde 200 con nuevo access token y nuevo refresh token (rotación)

#### Scenario: Credenciales inválidas rechazadas
- **WHEN** usuario envía credenciales incorrectas
- **THEN** responde 401 Unauthorized sin revelar si usuario existe

#### Scenario: Access token expirado rechazado
- **WHEN** cliente usa access token expirado en header Authorization
- **THEN** responde 401 con código token_expired

### Requirement: Autorización por roles y permisos
El sistema DEBE implementar control de acceso basado en roles (RBAC) con permisos granulares: Admin, Dentist, Assistant, Receptionist.

#### Scenario: Admin accede a todo
- **WHEN** usuario con rol Admin accede a cualquier endpoint
- **THEN** acceso permitido

#### Scenario: Dentist gestiona pacientes y tratamientos
- **WHEN** usuario con rol Dentist accede a endpoints de pacientes/tratamientos
- **THEN** acceso permitido; endpoints de facturación/configuración denegados (403)

#### Scenario: Assistant solo lectura en pacientes
- **WHEN** usuario con rol Assistant accede a GET /api/patients
- **THEN** acceso permitido; POST/PUT/DELETE denegados (403)

#### Scenario: Receptionist gestiona citas
- **WHEN** usuario con rol Receptionist accede a endpoints de citas
- **THEN** acceso CRUD completo en citas; pacientes solo lectura

### Requirement: Registro y gestión de usuarios
El sistema DEBE permitir registro de usuarios por Admin, confirmación de email, reset de contraseña y bloqueo por intentos fallidos.

#### Scenario: Admin crea usuario
- **WHEN** Admin POST /api/users con email, rol, datos
- **THEN** usuario creado, email de bienvenida enviado, contraseña temporal generada

#### Scenario: Bloqueo tras 5 intentos fallidos
- **WHEN** 5 logins fallidos consecutivos
- **THEN** cuenta bloqueada 15 minutos, email de alerta enviado

#### Scenario: Reset de contraseña
- **WHEN** usuario solicita reset vía POST /api/auth/forgot-password
- **THEN** email con token de un uso (30 min) enviado; POST /api/auth/reset-password con token válido actualiza contraseña