# GitHub Copilot Instructions — CRM Dental

## 1. Propósito del proyecto

Desarrollar un CRM para una clínica dental pequeña, administrada por la Dra. Laura y su asistente.

El sistema debe ayudar a:

- Reducir las inasistencias de los pacientes.
- Administrar pacientes y sus datos de contacto.
- Gestionar citas y su confirmación.
- Dar seguimiento administrativo a tratamientos que requieren varias sesiones.
- Automatizar recordatorios mediante WhatsApp Business.
- Sincronizar las citas con Google Calendar.
- Identificar pacientes que necesitan seguimiento.
- Facilitar el trabajo diario de la doctora y su asistente.

### Funcionalidades fuera del alcance inicial

El sistema no incluirá inicialmente:

- Facturación.
- Contabilidad.
- Expedientes clínicos completos.
- Diagnósticos médicos automatizados.
- Prescripciones médicas.
- Recomendaciones clínicas generadas automáticamente.

No agregar funcionalidades fuera del alcance sin autorización explícita.

---

## 2. Tecnologías del proyecto

### Backend

- .NET 10.
- ASP.NET Core Web API.
- C#.
- Entity Framework Core.
- MariaDB como base de datos predeterminada, salvo que la documentación del proyecto establezca otra decisión.
- Autenticación y autorización mediante mecanismos seguros.
- Pruebas unitarias y de integración.

### Frontend

- React.
- TypeScript.
- Cliente HTTP centralizado para consumir la API.
- Componentes reutilizables.
- Formularios con validación.
- Diseño adaptable a computadoras y tabletas.

### Integraciones

- Google Calendar API para sincronizar citas.
- WhatsApp Business Platform o un proveedor oficial autorizado para enviar mensajes.
- Procesos en segundo plano para recordatorios y tareas programadas.

### Arquitectura

Utilizar inicialmente un **monolito modular**. Evitar microservicios mientras no exista una necesidad técnica real.

Mantener separadas las responsabilidades de presentación, lógica de negocio, persistencia e integraciones externas.

Antes de agregar tecnologías o dependencias, revisar las decisiones documentadas y el código existente.

---

## 3. Fuentes de información del proyecto

Antes de implementar cambios, revisar los archivos existentes y utilizar el siguiente orden de referencia:

1. `PROJECT_STATUS.md`, si existe.
2. `docs/REQUIREMENTS.md`, si existe.
3. `docs/ARCHITECTURE.md` y los registros de decisiones arquitectónicas, si existen.
4. `docs/DATABASE.md`, si existe.
5. Los archivos de habilidades ubicados en `skills/`.
6. El código fuente y las pruebas existentes.

No asumir que estos archivos existen si todavía no han sido creados.

Si hay contradicciones entre los requisitos y el código, señalar la discrepancia antes de realizar cambios que puedan afectar el comportamiento del sistema.

No inventar reglas de negocio. Documentar los supuestos y solicitar aclaraciones cuando una decisión afecte información, permisos, citas o comunicaciones con pacientes.

---

## 4. Reglas de negocio obligatorias

### 4.1 Pacientes

- Cada paciente debe tener un identificador único.
- Validar los datos recibidos en el backend.
- No utilizar el número telefónico como identificador único permanente.
- Evitar registros duplicados mediante validaciones apropiadas.
- No exponer información personal innecesariamente.
- Mantener trazabilidad de los cambios importantes.

### 4.2 Citas

Las citas deben manejar estados explícitos y consistentes.

Los estados iniciales pueden incluir:

- Programada.
- Pendiente de confirmación.
- Confirmada.
- Cancelada.
- Reprogramada, si se decide conservar este estado histórico.
- Atendida.
- Inasistencia.

La lista definitiva debe establecerse en los requisitos del proyecto.

Reglas obligatorias:

- Una cita pendiente de confirmación no debe considerarse automáticamente una inasistencia.
- Una cita cancelada no debe contabilizarse como inasistencia.
- Una cita reprogramada no debe generar recordatorios correspondientes a su horario anterior.
- Antes de enviar un recordatorio, comprobar que la cita sigue vigente.
- No marcar una cita como atendida o como inasistencia sin una acción válida y las reglas correspondientes.
- Evitar que una misma operación genere cambios duplicados.

Separar el estado de la cita del estado de confirmación del paciente cuando sea necesario.

### 4.3 Recordatorios

- Utilizar mensajes previamente aprobados.
- Respetar el consentimiento y las preferencias de comunicación del paciente.
- No enviar mensajes a pacientes que hayan rechazado ese canal de comunicación.
- Evitar recordatorios duplicados.
- No enviar recordatorios de citas canceladas.
- Revalidar la cita inmediatamente antes de programar o ejecutar el envío.
- Registrar el resultado de cada intento.
- Diferenciar entre mensaje solicitado, aceptado por el proveedor, entregado, leído y respondido, según los estados que realmente proporcione la integración.
- No afirmar que un mensaje fue entregado cuando solo se confirmó que el proveedor aceptó la solicitud.
- Implementar reintentos controlados para errores transitorios.

Los procesos de envío deben ser idempotentes: repetir una operación no debe provocar mensajes duplicados.

### 4.4 Tratamientos de varias sesiones

El CRM debe permitir registrar administrativamente un tratamiento que requiera varias citas.

Debe poder identificarse:

- El paciente asociado.
- Las sesiones planificadas.
- Las citas relacionadas.
- Las sesiones realizadas.
- Las sesiones pendientes.
- La fecha del último seguimiento.
- La fecha sugerida para el siguiente contacto, cuando corresponda.

La doctora determina los intervalos y las indicaciones de seguimiento.

El sistema no debe decidir por sí mismo si un tratamiento está clínicamente terminado ni generar recomendaciones médicas.

No considerar un tratamiento terminado únicamente porque haya transcurrido cierto tiempo.

### 4.5 Google Calendar

- Mantener identificada la relación entre una cita local y su evento externo.
- Evitar crear eventos duplicados.
- Actualizar o cancelar el evento correspondiente cuando cambie la cita.
- Gestionar los errores de sincronización sin perder la cita registrada en el CRM.
- Registrar los errores que requieran intervención.
- Evitar sobrescribir cambios del usuario sin una regla de sincronización definida.
- No guardar credenciales en el código fuente.

La sincronización debe diseñarse considerando que las operaciones externas pueden fallar o responder después de un tiempo prolongado.

### 4.6 Indicadores

Los indicadores deben calcularse con reglas explícitas y documentadas.

Por ejemplo, la tasa de inasistencia debe excluir las citas canceladas y no debe considerar automáticamente como inasistencia las citas pendientes de confirmación.

Definir con precisión qué citas se incluyen en el numerador y denominador de cada indicador.

No presentar datos estimados como si fueran resultados definitivos.

---

## 5. Seguridad y privacidad

La seguridad es un requisito transversal.

### Obligaciones

- Aplicar el principio de mínimo privilegio.
- Validar permisos en el backend, no solamente en la interfaz.
- Separar los permisos de administración, doctora y asistente.
- Validar todos los datos recibidos de clientes y servicios externos.
- Utilizar DTO específicos para las operaciones de entrada y salida.
- No exponer entidades internas de la base de datos directamente.
- No almacenar contraseñas en texto plano.
- No incluir secretos, tokens ni credenciales en el repositorio.
- No enviar información sensible a registros de aplicación innecesarios.
- Proteger los endpoints contra accesos no autorizados.
- Configurar HTTPS y una política CORS restrictiva en producción.
- Aplicar limitación de solicitudes cuando corresponda.
- Validar la autenticidad de webhooks y protegerlos contra la repetición de solicitudes.
- Registrar las modificaciones importantes para facilitar su auditoría.

Minimizar los datos personales almacenados y utilizados.

Considerar las obligaciones aplicables en México en materia de protección de datos personales y privacidad. No afirmar que el sistema cumple legalmente con todos los requisitos sin una revisión específica.

### Manejo de secretos

Las credenciales deben gestionarse mediante variables de entorno o un almacén de secretos apropiado para el entorno.

Nunca incluir credenciales reales en:

- Código fuente.
- Archivos de configuración versionados.
- Pruebas automatizadas.
- Documentación pública.
- Registros de errores.

Los archivos de ejemplo deben utilizar valores ficticios.

---

## 6. Convenciones del backend

### Diseño

- Mantener los controladores delgados.
- Ubicar la lógica de negocio en servicios o componentes de aplicación apropiados.
- Separar el acceso a datos de las reglas de negocio.
- Utilizar inyección de dependencias.
- Mantener las integraciones externas detrás de interfaces.
- Evitar dependencias innecesarias entre módulos.
- Preferir soluciones simples y mantenibles.

### C#

- Utilizar nombres descriptivos.
- Seguir las convenciones estándar de C#.
- Habilitar y respetar los tipos de referencia anulables cuando la configuración del proyecto lo permita.
- Utilizar operaciones asíncronas para I/O.
- Propagar `CancellationToken` cuando corresponda.
- No utilizar `.Result` o `.Wait()` para bloquear operaciones asíncronas.
- Evitar excepciones genéricas para controlar el flujo normal del negocio.
- No capturar excepciones para ignorarlas silenciosamente.

### Persistencia

- Utilizar Entity Framework Core de acuerdo con los patrones del proyecto.
- Crear migraciones para los cambios de esquema.
- Mantener las relaciones e índices necesarios.
- Definir las restricciones importantes en la base de datos cuando corresponda.
- Evitar consultas N+1.
- Aplicar paginación en listados que puedan crecer.
- Utilizar transacciones cuando una operación requiera consistencia entre varios cambios.

No mantener una transacción de base de datos abierta durante llamadas lentas a servicios externos.

### API

- Utilizar códigos HTTP adecuados.
- Mantener contratos de entrada y salida consistentes.
- Validar datos en el servidor.
- Devolver errores estructurados, preferentemente mediante `ProblemDetails`.
- Evitar exponer detalles internos de excepciones.
- Documentar los endpoints cuando exista OpenAPI.
- No cambiar contratos públicos sin evaluar el impacto en el frontend y las integraciones.

---

## 7. Convenciones del frontend

### React y TypeScript

- Utilizar TypeScript con tipado estricto cuando sea compatible con la configuración del proyecto.
- Evitar el uso de `any` salvo que exista una justificación concreta.
- Crear componentes reutilizables cuando aporten claridad.
- Mantener separadas las responsabilidades de presentación y acceso a datos.
- Centralizar la configuración del cliente HTTP.
- No duplicar reglas de negocio que deban aplicarse en el backend.
- No incluir secretos ni credenciales privadas en el código del navegador.

### Experiencia de usuario

Las pantallas deben facilitar el trabajo cotidiano de la doctora y su asistente.

Cada operación asíncrona debe contemplar, según corresponda:

- Estado de carga.
- Resultado exitoso.
- Estado vacío.
- Error recuperable.
- Error de validación.
- Sesión expirada.
- Acceso denegado.

Además:

- Evitar envíos duplicados de formularios.
- Mostrar mensajes de error comprensibles.
- Solicitar confirmación para operaciones destructivas.
- Evitar perder datos introducidos cuando sea posible.
- Mostrar claramente los estados de las citas.
- Mantener un diseño accesible y adaptable.
- No indicar que una operación fue exitosa hasta que el backend confirme su resultado.

---

## 8. Integraciones y procesos en segundo plano

Los servicios externos deben implementarse mediante abstracciones para facilitar las pruebas y permitir cambiar de proveedor.

Por ejemplo:

- Servicio de calendario.
- Servicio de mensajería.
- Servicio de envío de recordatorios.
- Servicio de seguimiento de tratamientos.

Los procesos en segundo plano deben contemplar:

- Persistencia de las tareas cuando sea necesaria.
- Reintentos con límites.
- Esperas progresivas ante errores transitorios.
- Idempotencia.
- Registro de errores.
- Recuperación después de reinicios.
- Detección de tareas pendientes o fallidas.
- Evitar envíos duplicados.
- Evitar ejecutar tareas con información obsoleta.

No depender exclusivamente de tareas en memoria para procesos que no deban perderse cuando se reinicie la aplicación.

No inventar endpoints ni contratos de proveedores. Consultar la documentación oficial de la integración antes de implementar detalles específicos.

Diferenciar claramente las implementaciones simuladas de las integraciones reales.

---

## 9. Pruebas obligatorias

Los cambios de lógica de negocio deben incluir pruebas apropiadas.

### Citas y recordatorios

Verificar, como mínimo:

- Una cita pendiente de confirmación no se contabiliza automáticamente como inasistencia.
- Una cita cancelada no genera recordatorios.
- Una cita reprogramada no recibe recordatorios del horario anterior.
- Un recordatorio repetido no genera mensajes duplicados.
- Un error temporal permite reintentar sin duplicar el envío.
- No se envían mensajes cuando no existe el consentimiento requerido.
- Una cita modificada después de programar un recordatorio se vuelve a validar antes del envío.

### Webhooks e integraciones

Verificar:

- La recepción repetida de un mismo evento no duplica los cambios.
- Los eventos inválidos son rechazados.
- Los errores externos no provocan la pérdida de una cita local.
- Los fallos de sincronización pueden identificarse y recuperarse.
- Las respuestas de los proveedores se interpretan de acuerdo con su significado real.

### Tratamientos

Verificar:

- La relación entre paciente, tratamiento y sesiones.
- El registro de sesiones pendientes y realizadas.
- El seguimiento de fechas.
- La actualización de los datos sin perder el historial requerido.

### Seguridad

Verificar:

- Los usuarios no pueden ejecutar operaciones para las que carecen de permisos.
- La API valida los datos independientemente de las validaciones del frontend.
- Los datos sensibles no se exponen en respuestas o registros innecesarios.
- Las credenciales no aparecen en el repositorio.

### Indicadores

Verificar los cálculos con datos controlados, incluyendo citas canceladas, pendientes de confirmación, atendidas y con inasistencia confirmada.

Las pruebas automatizadas no deben enviar mensajes reales a pacientes ni modificar calendarios de producción.

---

## 10. Manejo de errores y observabilidad

- Registrar errores con información suficiente para investigarlos.
- No registrar contraseñas, tokens ni datos personales innecesarios.
- Utilizar identificadores de correlación cuando resulte útil.
- Distinguir errores de validación, permisos, persistencia e integraciones externas.
- Proporcionar mensajes comprensibles al usuario.
- Conservar los detalles técnicos en los registros internos protegidos.
- Evitar bloques `catch` vacíos.
- No ocultar fallos de sincronización o mensajería.

Los registros deben facilitar la investigación sin comprometer la privacidad.

---

## 11. Forma de trabajar con Copilot

Antes de modificar código:

1. Inspeccionar la estructura del repositorio.
2. Leer los requisitos y las decisiones arquitectónicas existentes.
3. Identificar el módulo afectado.
4. Determinar las reglas de negocio involucradas.
5. Revisar el comportamiento actual y las pruebas relacionadas.
6. Identificar supuestos y riesgos importantes.

Durante la implementación:

1. Realizar cambios pequeños y coherentes.
2. Reutilizar patrones existentes cuando sean apropiados.
3. Evitar refactorizaciones ajenas al objetivo.
4. Agregar o actualizar pruebas.
5. Actualizar la documentación cuando cambie el comportamiento.
6. No agregar paquetes innecesarios.
7. No reemplazar una implementación real por un mock sin indicarlo expresamente.
8. No modificar datos o configuraciones de producción.

Antes de finalizar:

1. Ejecutar las pruebas disponibles.
2. Compilar los proyectos afectados.
3. Ejecutar los análisis estáticos, verificaciones de tipos o lint disponibles.
4. Revisar el diff para detectar cambios accidentales.
5. Confirmar que no se hayan agregado secretos.
6. Actualizar `PROJECT_STATUS.md`, si existe y corresponde.

Si una verificación no pudo ejecutarse, indicarlo expresamente. Nunca afirmar que una compilación o prueba fue exitosa sin haberla ejecutado.

---

## 12. Criterios de aceptación

Una implementación se considera terminada cuando:

- Cumple el requisito solicitado.
- Respeta las reglas de negocio.
- Aplica las validaciones necesarias en el backend.
- Respeta los permisos correspondientes.
- Contempla los errores esperados.
- Incluye las pruebas pertinentes.
- No introduce secretos ni datos reales de pacientes en el repositorio.
- No rompe funcionalidades existentes.
- Mantiene actualizada la documentación afectada.
- Se informa qué archivos se modificaron y qué verificaciones se ejecutaron.

Si falta una decisión importante, debe documentarse como pendiente en lugar de inventar una regla de negocio.

---

## 13. Idioma y comunicación

El idioma predeterminado y obligatorio para la comunicación del proyecto es el español.

Estas instrucciones aplican a todas las interacciones y entregables relacionados con el proyecto:

- **Respuestas al usuario:** responder siempre en español, independientemente del idioma utilizado en la pregunta.
- **Explicaciones técnicas:** explicar en español las decisiones de arquitectura, implementación, errores y soluciones.
- **Documentación:** redactar en español los archivos Markdown, manuales, especificaciones, instrucciones y documentación técnica.
- **Comentarios del código:** escribir los comentarios en español cuando sean necesarios y aporten valor.
- **Mensajes de error:** utilizar español para los mensajes mostrados a los usuarios finales.
- **Pruebas:** redactar en español los nombres descriptivos de los casos de prueba, manteniendo las convenciones y restricciones del lenguaje y las herramientas utilizadas.
- **Git:** redactar en español los mensajes de commit, cuando el proyecto permita definir esta convención.
- **Nombres técnicos:** conservar en inglés los identificadores, clases, métodos, variables, propiedades, namespaces y términos propios de las tecnologías cuando sea apropiado o necesario para mantener las convenciones de programación.
- **APIs y dependencias:** respetar los nombres, contratos y documentación oficial de las bibliotecas y servicios externos.

No traducir nombres oficiales de tecnologías, clases de frameworks, métodos de API ni términos que deban conservarse por compatibilidad técnica.

Si el usuario solicita expresamente otro idioma para una respuesta o documento específico, respetar esa solicitud para ese entregable.

**Regla principal:** salvo petición expresa en contrario, toda comunicación y documentación generada para este proyecto debe estar en español.

## 14. Principio rector

**Priorizar la seguridad, la privacidad, la integridad de las citas y la facilidad de uso por encima de la complejidad técnica.**

El objetivo del CRM es reducir las inasistencias y facilitar el seguimiento administrativo de los pacientes, no convertir una clínica pequeña en una operación difícil de administrar.