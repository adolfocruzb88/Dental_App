# Spec Delta

## Purpose

Define la capa de persistencia con Entity Framework Core, incluyendo migraciones, repositorios, unidad de trabajo, y patrones de acceso a datos para MariaDB.

## ADDED Requirements

### Requirement: Entity Framework Core configurado con MariaDB
El sistema DEBE configurar EF Core 9+ con proveedor Pomelo.EntityFrameworkCore.MySql para MariaDB, incluyendo DbContext, configuración de convenciones y migraciones.

#### Scenario: Migración inicial se aplica
- **WHEN** se ejecuta `dotnet ef database update`
- **THEN** la base de datos se crea con esquema inicial sin errores

#### Scenario: Convenciones de nomenclatura aplicadas
- **WHEN** se inspecciona el esquema generado
- **THEN** tablas en snake_case, columnas en snake_case, claves foráneas con nombres consistentes

### Requirement: Patrón Repositorio y Unidad de Trabajo
El sistema DEBE implementar repositorios genéricos y específicos, y unidad de trabajo para transacciones atómicas.

#### Scenario: Repositorio genérico CRUD
- **WHEN** se usa `IRepository<TEntity>` para una entidad
- **THEN** expone GetById, GetAll, Add, Update, Delete, ExistsAsync

#### Scenario: Unidad de trabajo confirma transacción
- **WHEN** se llama `await unitOfWork.SaveChangesAsync()`
- **THEN** todos los cambios pendientes se persisten en una sola transacción

#### Scenario: Rollback en error
- **WHEN** ocurre excepción durante SaveChangesAsync
- **THEN** ningún cambio parcial se persiste

### Requirement: Configuración de entidades base
El sistema DEBE definir entidad base con auditoría (CreatedAt, UpdatedAt, CreatedBy, UpdatedBy) y soft delete (IsDeleted, DeletedAt).

#### Scenario: Auditoría automática en creación
- **WHEN** se agrega nueva entidad y se guarda
- **THEN** CreatedAt y CreatedBy se poblan automáticamente

#### Scenario: Auditoría automática en actualización
- **WHEN** se modifica entidad existente y se guarda
- **THEN** UpdatedAt y UpdatedBy se actualizan automáticamente

#### Scenario: Soft delete filtra consultas
- **WHEN** se consulta repositorio con entidad soft-deleted
- **THEN** la entidad no aparece en resultados normales (solo con IncludeDeleted)