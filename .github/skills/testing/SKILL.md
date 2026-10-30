---
name: crm-dental-testing
description: Planifica, implementa y ejecuta pruebas del CRM dental para validar reglas de negocio, API .NET 10, interfaz, integraciones, seguridad, idempotencia y recuperación ante fallos.
---

# Skill: Pruebas y Calidad del CRM Dental

## Objetivo

Verificar que el CRM reduzca trabajo manual sin perder citas, duplicar recordatorios, exponer datos personales ni confundir confirmaciones con inasistencias.

El backend usa ASP.NET Core sobre .NET 10. El frontend propuesto usa React y TypeScript. Ajustar las herramientas a las decisiones reales del repositorio.

## Principios

1. Las pruebas deben ser reproducibles y deterministas.
2. No enviar mensajes reales a pacientes durante pruebas automatizadas.
3. No depender de cuentas reales de Google Calendar en la suite normal.
4. Usar mocks, fakes o simuladores para proveedores externos; mantener pruebas de contrato donde aporten valor.
5. Probar especialmente las reglas que evitan duplicados y mensajes obsoletos.
6. No declarar una prueba ejecutada si solo se escribió.
7. Reportar por separado pruebas unitarias, integración, aceptación y pruebas manuales.
8. No incluir datos personales reales en fixtures, logs ni reportes de pruebas.
9. Probar errores y recuperación, no solo el camino feliz.
10. Ejecutar las pruebas relevantes después de cada cambio y la suite completa antes de integrar.

## Capas de prueba

### 1. Pruebas unitarias
Probar reglas de dominio sin depender de la base de datos o APIs externas.

Casos mínimos:
- Transiciones válidas e inválidas de estado de cita.
- Cita sin respuesta no se convierte automáticamente en inasistencia.
- Una cita cancelada no conserva recordatorios activos.
- Reprogramar invalida trabajos asociados al horario anterior.
- Progreso administrativo de sesiones.
- Generación de seguimiento según la fecha definida por la doctora.
- Tratamiento sin próxima cita crea una tarea solo cuando la regla aprobada lo requiere.
- No se crean tareas activas duplicadas para el mismo evento.
- Cálculo correcto de indicadores.
- Las citas canceladas no cuentan como inasistencias.

### 2. Pruebas de integración del backend
Validar la interacción entre API, persistencia y componentes internos.

Casos mínimos:
- Crear y consultar paciente.
- Crear, actualizar y cancelar cita.
- Relacionar cita, paciente y tratamiento.
- Guardar y consultar tareas.
- Persistir auditoría.
- Ejecutar migraciones en una base de datos de prueba.
- Verificar restricciones e índices importantes.
- Verificar autorización de endpoints.
- Verificar formato uniforme de errores.
- Comprobar transacciones y concurrencia en operaciones relevantes.

Preferir una base de datos real de prueba en contenedor cuando sea viable, para no depender exclusivamente de un proveedor EF Core distinto al de producción.

### 3. Pruebas de integración de Google Calendar
Usar un adaptador simulado en la suite normal.

Escenarios:
- Crear evento remoto y guardar su identificador.
- Modificar evento.
- Cancelar evento.
- Recibir una modificación externa.
- Timeout del proveedor.
- Error de autorización.
- Evento eliminado externamente.
- Reintento de sincronización.
- Conflicto de cambios.
- Evitar eventos duplicados después de reintentar.
- Reconciliar cambios pendientes tras reiniciar el backend.

Las pruebas contra una cuenta de Google real deben ser explícitas, aisladas, opcionales y ejecutarse solo con credenciales de prueba autorizadas.

### 4. Pruebas de WhatsApp y mensajería
No enviar mensajes reales a pacientes en CI.

Escenarios:
- Crear trabajo de recordatorio.
- Enviar al proveedor simulado.
- Registrar fallo temporal.
- Reintentar con política limitada.
- Procesar estado de entrega.
- Procesar confirmación del paciente.
- Procesar solicitud de reprogramación.
- Rechazar webhook inválido.
- Procesar el mismo webhook varias veces sin duplicar operaciones.
- No enviar si la autorización de comunicación no está vigente.
- No enviar recordatorio a cita cancelada.
- No enviar recordatorio del horario anterior después de reprogramar.
- No confundir aceptación del proveedor con entrega o confirmación.
- No filtrar tokens ni datos personales en logs.

### 5. Pruebas de frontend
Según las herramientas del repositorio, cubrir:
- Renderizado de carga, error, vacío y datos.
- Formularios con campos válidos e inválidos.
- Mensajes de error accesibles.
- Prevención de doble envío.
- Permisos y estados visibles.
- Búsqueda y selección de paciente.
- Reprogramación y cancelación.
- Tareas de seguimiento.
- Indicadores y filtros.
- Navegación por teclado en flujos críticos.

Usar pruebas centradas en el comportamiento observable, no en detalles internos de implementación.

### 6. Pruebas end-to-end / aceptación
Cubrir los flujos de mayor valor:
1. Crear paciente y cita.
2. Confirmar que el evento se sincroniza o queda en estado pendiente explícito.
3. Generar un recordatorio simulado.
4. Registrar confirmación.
5. Reprogramar cita y comprobar que el recordatorio anterior queda invalidado.
6. Registrar inasistencia y comprobar métricas.
7. Crear tratamiento de varias sesiones.
8. Completar una sesión y generar seguimiento cuando corresponde.
9. Registrar resultado de contacto.
10. Verificar que un usuario sin permisos no pueda realizar acciones restringidas.

## Matriz de reglas críticas

| Regla | Prueba mínima |
|---|---|
| Pendiente de confirmación no equivale a inasistencia | Prueba unitaria y de aceptación |
| Cita cancelada no genera recordatorios | Unitaria e integración de trabajos |
| Reprogramación invalida recordatorios antiguos | Unitaria, integración y aceptación |
| Reintento no duplica mensajes | Integración de mensajería |
| Webhook repetido no duplica confirmaciones | Integración de webhook |
| Sin autorización no se envía mensaje | Unitaria e integración |
| Tratamiento sin próxima cita genera tarea cuando corresponde | Unitaria e integración |
| No se permite acceso no autorizado | Integración de API |
| Cálculo de inasistencia excluye cancelaciones | Unitaria |
| Fallo externo no pierde la cita | Integración y recuperación |

## Seguridad

Incluir pruebas para:
- Acceso anónimo a endpoints protegidos.
- Acceso con rol insuficiente.
- Manipulación de identificadores para acceder a registros de otros usuarios o ámbitos no autorizados.
- Validación de datos de entrada.
- Rate limiting donde aplique.
- Webhooks con firma inválida o eventos repetidos.
- Manejo de errores sin filtración de stack traces.
- Ausencia de secretos en repositorio y logs.
- Configuración CORS.
- Dependencias vulnerables mediante las herramientas disponibles.

No afirmar que el sistema es seguro solo porque las pruebas pasan. Documentar el alcance real de las verificaciones.

## Datos de prueba

- Usar pacientes ficticios y números telefónicos no reales.
- Evitar datos clínicos.
- Crear fixtures legibles y reutilizables.
- No copiar producción a ambientes de prueba sin un proceso de anonimización aprobado.
- Mantener los secretos de prueba fuera del repositorio.

## Automatización y comandos

Primero inspeccionar los archivos de solución y scripts existentes. No inventar comandos específicos si no se ha verificado la estructura.

Para proyectos .NET, considerar:
- `dotnet restore`
- `dotnet build`
- `dotnet test`

Ejecutar los comandos apropiados desde la solución o proyectos correspondientes y reportar el resultado real.

Para frontend, ejecutar los scripts definidos en `package.json`, como lint, typecheck, test y build, si existen.

No modificar archivos de configuración de CI sin revisar el pipeline actual.

## Criterios de salida

Una tarea puede considerarse lista para revisión cuando:
- Se añadieron pruebas de las reglas modificadas.
- Las pruebas relevantes se ejecutaron.
- No hay fallos conocidos sin documentar.
- Se cubrieron al menos los escenarios negativos importantes.
- Los mocks y simuladores están claramente diferenciados de las integraciones reales.
- Se informó cualquier prueba omitida y el motivo.

## Informe de pruebas

Al finalizar, proporcionar:
- Comandos ejecutados.
- Resultado de build.
- Total de pruebas aprobadas, fallidas y omitidas, si la herramienta lo reporta.
- Escenarios cubiertos.
- Dependencias de servicios externos.
- Fallos conocidos.
- Riesgos no cubiertos.
- Confirmación explícita de si las pruebas usaron mocks o servicios reales.
