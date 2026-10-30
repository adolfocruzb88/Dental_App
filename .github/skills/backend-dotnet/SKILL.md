---
name: crm-dental-backend
description: Diseña, implementa y revisa el backend del CRM dental con ASP.NET Core y .NET 10, incluyendo API, dominio, persistencia, seguridad, tareas en segundo plano e integraciones externas.
---

# Skill: Backend del CRM Dental (.NET 10)

## Objetivo

Construir un backend seguro, mantenible y comprobable para el CRM dental de la Dra. Laura. El sistema gestiona pacientes, citas, tratamientos multisesión y seguimientos administrativos. No es un sistema de facturación ni un expediente clínico electrónico.

## Contexto y restricciones del negocio

- La agenda existente usa Google Calendar.
- Se busca reducir inasistencias y trabajo manual de la asistente.
- Los recordatorios se envían mediante una integración oficial con WhatsApp Business Platform o un proveedor compatible.
- Los tratamientos pueden tener varias sesiones; la doctora determina las fechas y los plazos de continuidad.
- No crear diagnósticos, recomendaciones clínicas ni campos de expediente clínico.
- Roles iniciales: Administrador, Doctora y Asistente.
- Priorizar una aplicación modular sencilla. No crear microservicios sin ADR aprobado.
- El proyecto debe compilar y ejecutarse con **.NET 10**. Usar paquetes compatibles con .NET 10 y verificar compatibilidad antes de incorporarlos.

## Principios de arquitectura

1. Separar reglas de negocio de controladores, base de datos y proveedores externos.
2. Preferir un monolito modular con límites explícitos entre módulos.
3. Mantener lógica de negocio fuera de los controladores HTTP.
4. Usar inyección de dependencias y abstraer los servicios externos mediante interfaces.
5. Usar DTOs de entrada/salida; no exponer entidades de persistencia directamente.
6. No crear abstracciones genéricas sin una necesidad concreta.
7. No añadir dependencias, infraestructura o patrones complejos sin explicar su valor.
8. Seguir las convenciones y estructura existentes en el repositorio; antes de introducir una estructura nueva, inspeccionar la solución.

## Stack de referencia

- ASP.NET Core Web API sobre .NET 10.
- C# con nullable reference types habilitados.
- Entity Framework Core con proveedor Pomelo.EntityFrameworkCore.MySql para MariaDB, salvo que una decisión documentada indique otra base de datos.
- Validación explícita de solicitudes.
- OpenAPI para documentar los endpoints.
- Logging estructurado.
- Pruebas con el framework ya adoptado por el repositorio; si no existe, proponer xUnit.
- Trabajos en segundo plano con persistencia y reintentos. Evaluar Hangfire u otra solución compatible, pero no asumir que está instalada.

Antes de agregar paquetes, verificar que la versión sea compatible con .NET 10 y con las dependencias actuales.

## Áreas funcionales

### Pacientes
- Crear, consultar, actualizar y desactivar pacientes.
- Campos mínimos: identificador, nombre, teléfono, correo opcional, preferencias y autorización de comunicación, estado y marcas de tiempo.
- Evitar borrado físico cuando existan relaciones o auditoría.
- Minimizar los datos personales almacenados.

### Citas
- Relacionar la cita con un paciente y, opcionalmente, con un tratamiento.
- Mantener separados el estado de la cita y el estado de confirmación.
- Distinguir cita programada, confirmada, reprogramación solicitada, cancelada, atendida e inasistencia según el modelo aprobado.
- No clasificar automáticamente una cita sin respuesta como inasistencia.
- Las citas canceladas o reprogramadas no deben conservar recordatorios activos del horario anterior.

### Tratamientos y sesiones
- Registrar información administrativa mínima del tratamiento, sesiones previstas y estado informado por la doctora.
- La aplicación no determina si un tratamiento está médicamente completo.
- La doctora define la próxima fecha o regla de seguimiento.
- Una continuidad pendiente puede crear una tarea para la asistente.

### Tareas y comunicaciones
- Registrar responsable, fecha límite, prioridad, estado y resultado de seguimiento.
- Mantener historial de envíos y respuestas sin guardar contenido sensible innecesario.
- Usar identificadores idempotentes para evitar tareas o mensajes duplicados.

### Integraciones
- Google Calendar: encapsular la API detrás de interfaces, mantener identificadores externos, sincronización y estrategia de reconciliación.
- WhatsApp: utilizar proveedor oficial, webhooks validados y estados de entrega según evidencia real del proveedor.
- Ninguna integración externa debe estar acoplada directamente a las reglas del dominio.

## Seguridad obligatoria

- Aplicar autorización en el backend a cada operación protegida.
- Validar datos en los límites del sistema.
- Usar consultas parametrizadas y EF Core correctamente.
- No confiar en validaciones exclusivas del frontend.
- Guardar secretos en un gestor de secretos o configuración segura; nunca en el repositorio.
- No registrar contraseñas, tokens, secretos ni datos personales innecesarios.
- Proteger webhooks contra falsificación y repetición, de acuerdo con las capacidades del proveedor.
- Aplicar límites de solicitudes y CORS restrictivo según el despliegue.
- Usar HTTPS en producción.
- Auditar cambios sensibles y administrativos.
- Diseñar la privacidad considerando las obligaciones aplicables en México.

## API y manejo de errores

- Diseñar endpoints REST consistentes y documentados.
- Usar códigos HTTP apropiados.
- Devolver errores con un formato uniforme, preferentemente Problem Details.
- No filtrar stack traces, SQL ni secretos al cliente.
- Validar paginación, ordenamiento y filtros.
- No devolver entidades internas, secretos o datos no autorizados.
- Mantener compatibilidad de API cuando sea posible; documentar cambios incompatibles.

## Persistencia y consistencia

- Usar migraciones versionadas.
- Definir claves, restricciones e índices según consultas reales.
- Gestionar concurrencia cuando dos usuarios puedan modificar la misma cita.
- Usar transacciones en operaciones que deban persistir como una unidad.
- No mantener transacciones de base de datos abiertas mientras se espera una API externa.
- Diseñar sincronizaciones externas para tolerar fallos parciales.
- Registrar estado de sincronización y permitir reintento/reconciliación.

## Trabajos en segundo plano

- Persistir los trabajos críticos para sobrevivir a reinicios.
- Evitar depender de un proceso en memoria para recordatorios que no deben perderse.
- Implementar reintentos limitados, backoff y manejo de errores.
- Hacer los trabajos idempotentes.
- Verificar de nuevo el estado vigente de la cita antes de enviar un recordatorio.
- Evitar que una cita cancelada o modificada genere un mensaje obsoleto.
- No marcar un mensaje como entregado solo porque la solicitud al proveedor tuvo éxito.

## Calidad de código

- Usar nombres claros y coherentes.
- Mantener métodos pequeños con una responsabilidad clara.
- Evitar lógica de negocio duplicada.
- Usar async/await para I/O y propagar CancellationToken en operaciones compatibles.
- No capturar excepciones para ignorarlas.
- No usar `async void` salvo eventos UI, que no son parte del backend.
- Evitar consultas N+1 y cargar únicamente los datos necesarios.
- Documentar decisiones no obvias, no cada línea de código.

## Flujo de trabajo para cada tarea

1. Leer `PROJECT_STATUS.md`, requerimientos, reglas de negocio y ADR relevantes.
2. Inspeccionar la solución y las convenciones existentes.
3. Resumir el objetivo y listar supuestos o preguntas que bloqueen la implementación.
4. Diseñar el cambio mínimo necesario.
5. Implementar dominio, persistencia, API e integración en capas apropiadas.
6. Añadir o actualizar pruebas.
7. Ejecutar build, pruebas y análisis disponibles.
8. Actualizar documentación y `PROJECT_STATUS.md` si cambió el estado del proyecto.
9. Informar archivos modificados, comandos ejecutados, resultados reales y pendientes.

## Criterios de aceptación

Una tarea backend no se considera terminada si:
- No compila con .NET 10.
- No tiene pruebas para reglas de negocio importantes.
- La autorización existe solo en frontend.
- Los errores externos pueden duplicar operaciones críticas.
- Se exponen secretos o datos innecesarios.
- La documentación contradice el comportamiento implementado.
- Se afirma que una integración real funciona cuando solo se usaron mocks.

## Entrega esperada

Al finalizar cada tarea, reportar:
- Resumen de implementación.
- Archivos modificados.
- Cambios de base de datos.
- Endpoints afectados.
- Pruebas ejecutadas y resultado.
- Configuración o secretos requeridos, sin revelar valores.
- Riesgos y trabajo pendiente.
