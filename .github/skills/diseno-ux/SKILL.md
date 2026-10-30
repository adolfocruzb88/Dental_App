---
name: crm-dental-ux
description: Define y evalúa la experiencia de usuario del CRM dental, priorizando flujos simples para la doctora y su asistente, accesibilidad, prevención de errores y reducción del trabajo manual.
---

# Skill: UX/UI del CRM Dental

## Misión

Diseñar una experiencia que ayude a la Dra. Laura y a su asistente a reducir inasistencias y completar el seguimiento administrativo de tratamientos, con el mínimo esfuerzo diario.

La interfaz no debe sentirse como un sistema hospitalario complejo. Debe ser un CRM pequeño, predecible y orientado a tareas.

## Usuarios y objetivos

### Asistente
Necesita:
- Ver rápidamente las citas que requieren confirmación.
- Identificar pacientes que solicitaron reprogramar.
- Resolver tareas pendientes.
- Detectar tratamientos sin próxima cita.
- Registrar el resultado de llamadas o mensajes.
- Corregir errores sin repetir todo el proceso.

### Doctora
Necesita:
- Consultar agenda y estado de citas.
- Revisar tratamientos que requieren continuidad.
- Definir o ajustar fechas de seguimiento.
- Consultar indicadores de inasistencia y continuidad.
- Supervisar el trabajo sin gestionar manualmente cada recordatorio.

### Administrador
Necesita:
- Administrar usuarios, permisos y configuración.
- Configurar integraciones sin exponer credenciales.
- Consultar fallos operativos y auditoría según sus permisos.

## Principios de diseño

1. **Priorizar lo accionable:** el dashboard debe responder “¿qué requiere mi atención ahora?”.
2. **Reducir carga cognitiva:** mostrar información relevante, agrupada por tarea.
3. **Usar lenguaje cotidiano:** “Pendiente de confirmar”, “Solicitó cambio” y “Seguimiento vencido”.
4. **Prevenir errores:** confirmar cancelaciones y advertir cuando una acción afecta recordatorios.
5. **Mantener trazabilidad:** mostrar quién cambió el estado y cuándo cuando sea útil.
6. **Evitar alarmas falsas:** una cita sin respuesta no equivale a inasistencia.
7. **No castigar al usuario:** los errores deben ofrecer una acción de recuperación.
8. **No crear trabajo innecesario:** evitar tareas duplicadas y notificaciones redundantes.
9. **Respetar privacidad:** mostrar solo los datos necesarios.
10. **No hacer inferencias clínicas:** el sistema organiza el seguimiento que define la doctora.

## Navegación propuesta

Menú principal sugerido:
- Inicio
- Agenda
- Pacientes
- Tratamientos
- Tareas de seguimiento
- Comunicaciones
- Reportes
- Configuración (según permisos)

Mantener navegación corta y nombres consistentes. Evitar submenús profundos para tareas diarias.

## Flujos críticos

### Flujo A: confirmar citas del día siguiente
1. La asistente abre “Citas por confirmar”.
2. Ve paciente, hora, estado de mensaje y fecha de cita.
3. Distingue quién confirmó, quién pidió reprogramar y quién no respondió.
4. Ejecuta la acción permitida: revisar, contactar o registrar resultado.
5. La tarea queda actualizada y auditable.

No asumir que la ausencia de respuesta significa que el paciente no acudirá.

### Flujo B: registrar una cita
1. Buscar primero al paciente para evitar duplicados.
2. Seleccionar o crear paciente.
3. Elegir fecha y hora disponibles según la fuente de disponibilidad acordada.
4. Revisar el resumen antes de guardar.
5. Mostrar si la sincronización con Google Calendar se completó o quedó pendiente.
6. Informar qué ocurrirá con los recordatorios.

### Flujo C: seguimiento de tratamiento
1. Abrir la lista de tratamientos que requieren continuidad.
2. Identificar última sesión, próxima fecha indicada y estado.
3. Crear o completar una tarea de contacto.
4. Registrar resultado: contactado, sin respuesta, solicita reprogramación, pausado por indicación de la doctora u otro resultado configurado.
5. Si se programa una cita, reflejarla en el tratamiento y en la agenda.

No usar términos que interpreten el resultado médico del tratamiento.

### Flujo D: error de integración
1. Mostrar el error en lenguaje comprensible.
2. Explicar si la cita o el mensaje quedó guardado localmente.
3. Ofrecer reintento seguro cuando corresponda.
4. Evitar que el usuario cree un duplicado al intentar de nuevo.
5. Proporcionar una referencia técnica para soporte sin revelar secretos.

## Dashboard

Orden recomendado:
1. Tareas urgentes o vencidas.
2. Citas de hoy.
3. Citas próximas pendientes de confirmación.
4. Tratamientos sin próxima cita cuando corresponda.
5. Mensajes o sincronizaciones fallidos.
6. Indicadores del periodo.

Cada tarjeta o indicador debe llevar a una lista filtrada que permita actuar. No mostrar un KPI sin definir su fórmula ni ofrecer una manera de investigar los registros que lo componen.

## Estados y microcopy

Usar etiquetas precisas:
- “Pendiente de confirmación” no es “No asistió”.
- “Enviado al proveedor” no es “Entregado”.
- “Entregado” no significa “Leído” ni “Confirmado”.
- “Tratamiento sin próxima cita” no significa “Tratamiento abandonado” hasta que la doctora o la regla administrativa aprobada así lo determine.
- “Seguimiento vencido” significa que venció una fecha administrativa configurada, no que exista una urgencia clínica.

Mensajes de error:
- Explicar qué ocurrió.
- Indicar si los datos se guardaron.
- Dar un siguiente paso concreto.
- Evitar mostrar trazas, SQL o información técnica sensible.

## Formularios

- Solicitar solo los datos necesarios.
- Agrupar campos relacionados.
- Marcar obligatorios claramente.
- Mantener los datos si ocurre un error.
- Ofrecer búsqueda de paciente antes de crear uno nuevo.
- Evitar borrar información accidentalmente.
- Usar confirmaciones para cancelar o reprogramar citas.
- Mostrar fecha, hora y zona horaria de forma inequívoca.

## Accesibilidad y responsive

- Contraste suficiente.
- Texto legible y controles cómodos.
- Navegación por teclado.
- Foco visible.
- No depender solo del color para indicar estados.
- Etiquetas accesibles.
- Tablas utilizables en pantallas pequeñas o con alternativa de tarjetas.
- No usar animaciones que dificulten el trabajo cotidiano.

## Validación de UX

Antes de dar por terminado un flujo, comprobar:
- ¿La asistente puede entender qué debe hacer sin capacitación extensa?
- ¿Se distinguen claramente los estados?
- ¿Puede recuperarse de un error sin duplicar una cita o mensaje?
- ¿Se evita introducir información innecesaria?
- ¿Se puede completar el flujo con teclado?
- ¿La interfaz aclara cuándo una operación está pendiente de sincronización?
- ¿Se evita exponer datos personales a usuarios no autorizados?

Cuando sea posible, proponer pruebas de usabilidad breves con tareas realistas y registrar dificultades observadas.

## Entregables

Para una tarea UX, entregar según corresponda:
- Flujos de usuario.
- Inventario de pantallas y estados.
- Wireframes descritos o implementables.
- Etiquetas y microcopy en español.
- Criterios de accesibilidad.
- Casos de error y recuperación.
- Criterios de aceptación observables.

No generar imágenes decorativas como sustituto de especificaciones funcionales ni afirmar que una prueba de usabilidad se realizó si no se hizo.
