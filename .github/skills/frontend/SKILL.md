---
name: crm-dental-frontend
description: Implementa y mantiene el frontend React y TypeScript del CRM dental, incluyendo flujos de pacientes, citas, tratamientos, tareas y dashboard con integración segura con la API.
---

# Skill: Frontend del CRM Dental

## Objetivo

Construir una interfaz clara, rápida, accesible y adaptable para una doctora y una asistente de consultorio pequeño. La interfaz debe reducir trabajo administrativo y hacer evidentes las acciones pendientes.

## Contexto fijo

- Backend: ASP.NET Core Web API sobre .NET 10.
- Frontend propuesto: React + TypeScript.
- Producto: CRM administrativo dental; no facturación ni expediente clínico.
- Funciones centrales: pacientes, citas, confirmaciones, tratamientos multisesión, tareas y dashboard.
- Integraciones externas: Google Calendar y WhatsApp, gestionadas por backend.
- Idioma inicial de interfaz: español.
- No guardar secretos de proveedores en el frontend.

## Principios de UX y frontend

1. Priorizar tareas frecuentes sobre opciones poco usadas.
2. Mostrar estados de forma explícita y comprensible.
3. Evitar formularios largos y campos que no sean necesarios.
4. No duplicar reglas de negocio críticas en frontend como sustituto de validación backend.
5. No mostrar datos clínicos ni inventar recomendaciones clínicas.
6. Diseñar para uso cotidiano en escritorio y tablet; adaptar también a móvil.
7. Proporcionar estados de carga, vacío, error, éxito y permisos insuficientes.
8. Mantener consistencia visual y de interacción entre módulos.
9. Respetar accesibilidad y navegación por teclado.
10. No asumir que una acción fue exitosa hasta recibir confirmación del backend.

## Stack y convenciones

- React con TypeScript en modo estricto.
- Usar la herramienta de construcción y el gestor de paquetes ya presentes en el repositorio.
- Si el repositorio no tiene stack frontend, proponer Vite y npm como opción simple.
- Usar un sistema de componentes consistente; no incorporar una biblioteca visual pesada sin necesidad.
- Centralizar el acceso HTTP a la API.
- Tipar las respuestas y solicitudes de la API.
- No usar `any` para evitar definir tipos; justificar cualquier excepción.
- Usar variables de entorno públicas únicamente para configuración no secreta, por ejemplo la URL base de la API.
- No incluir credenciales OAuth privadas, tokens de Google ni secretos de WhatsApp en el bundle del navegador.

## Pantallas principales

### 1. Inicio / dashboard
Mostrar:
- Citas de hoy y mañana.
- Citas pendientes de confirmación.
- Solicitudes de reprogramación.
- Seguimientos vencidos.
- Tratamientos que requieren próxima cita.
- Mensajes fallidos que requieren intervención.
- Tareas pendientes de la asistente.

Evitar gráficos decorativos que no ayuden a tomar una decisión. Dar acceso directo desde cada indicador a la lista correspondiente.

### 2. Pacientes
- Tabla o lista con búsqueda por nombre y teléfono.
- Formulario simple para alta y edición.
- Vista de detalle con citas, tratamientos administrativos y tareas.
- Mostrar preferencias y estado de autorización de comunicaciones claramente.
- No mostrar más datos personales de los necesarios para la tarea actual.

### 3. Agenda
- Vista por día y semana, si lo soporta el alcance aprobado.
- Mostrar fecha, hora, paciente y estado.
- Acciones visibles para confirmar, reprogramar, cancelar y registrar resultado según permisos.
- Indicar el estado de sincronización con Google Calendar.
- Advertir cuando la sincronización falle; no ocultar el problema.
- Confirmar las acciones destructivas o difíciles de revertir.
- Mostrar zona horaria de forma coherente.

### 4. Tratamientos
- Mostrar progreso administrativo de sesiones.
- Distinguir sesiones completadas, pendientes y próximas.
- Mostrar si existe próxima cita.
- Facilitar la creación de una tarea de seguimiento.
- No presentar el porcentaje de sesiones como evaluación clínica del tratamiento.

### 5. Bandeja de tareas
- Filtros por vencimiento, prioridad, responsable y tipo.
- Acción rápida para registrar resultado del contacto.
- Mostrar tareas vencidas sin alarmismo innecesario.
- Evitar acciones masivas que puedan enviar mensajes sin confirmación o autorización.

### 6. Comunicaciones
- Mostrar estado del mensaje: pendiente, enviado al proveedor, entregado si existe confirmación, fallido o respuesta recibida.
- No confundir envío con entrega ni entrega con lectura.
- Mostrar mensajes fallidos con una acción clara para revisar o reintentar, respetando las reglas de idempotencia del backend.

## Formularios y validación

- Etiquetar cada campo de forma visible.
- Indicar campos obligatorios antes de enviar.
- Validar formatos en cliente para mejorar la experiencia, pero aceptar que el backend es la autoridad final.
- Mantener los valores ingresados cuando una solicitud falla.
- Mostrar errores junto al campo y un resumen cuando sea útil.
- Prevenir doble envío mientras una operación está en curso.
- Confirmar la navegación si hay cambios sin guardar.
- Normalizar teléfonos según la estrategia acordada por el backend; no asumir reglas de país sin configuración.

## Estados de interfaz

Toda vista que cargue datos remotos debe contemplar:
- Cargando.
- Datos disponibles.
- Sin resultados.
- Error recuperable.
- Sesión expirada o no autenticada.
- Acceso denegado.
- Operación en progreso.
- Operación completada.

Los mensajes de error deben ser útiles, en español y sin detalles técnicos sensibles.

## Seguridad frontend

- Aplicar controles de acceso de presentación por rol, sabiendo que no reemplazan la autorización backend.
- No almacenar tokens sensibles de larga duración en `localStorage`.
- No renderizar contenido externo sin sanitización.
- Evitar registrar datos personales en consola.
- Proteger las rutas privadas y gestionar la expiración de sesión.
- No incluir secretos en variables `VITE_*` ni equivalentes.
- Restringir las acciones visibles según permisos, pero validar siempre en backend.

## Accesibilidad

- Usar elementos HTML semánticos.
- Mantener etiquetas accesibles para campos y controles.
- Permitir navegación por teclado.
- Mantener foco visible.
- No comunicar estados únicamente mediante color.
- Usar contraste suficiente y tamaños legibles.
- Añadir nombres accesibles a iconos y botones sin texto.
- Asociar mensajes de error con sus campos.

## Calidad y mantenibilidad

- Dividir componentes por responsabilidad.
- Separar componentes visuales, lógica de página y servicios de API.
- Evitar componentes monolíticos.
- Reutilizar componentes cuando exista un patrón real, no por anticipación.
- Mantener los tipos sincronizados con los contratos de API.
- No introducir estado global si el estado local o de consulta es suficiente.
- No duplicar cálculos de negocio que deberían provenir del backend.
- Usar formatos de fecha y hora coherentes con la zona horaria configurada.

## Flujo de trabajo

1. Leer requerimientos, `PROJECT_STATUS.md`, decisiones de API y skill de UX.
2. Inspeccionar estructura, librerías y convenciones del frontend.
3. Confirmar contratos de endpoints antes de asumir campos.
4. Diseñar estados y flujo de interacción.
5. Implementar interfaz y manejo de errores.
6. Añadir pruebas de componentes o flujos según las herramientas del repositorio.
7. Ejecutar lint, typecheck, build y pruebas disponibles.
8. Verificar accesibilidad básica y comportamiento responsive.
9. Documentar variables de entorno no secretas y cambios de interfaz.

## Criterios de aceptación

- TypeScript compila sin errores.
- La interfaz funciona con los contratos reales o mocks claramente identificados.
- Se manejan correctamente carga, vacío y error.
- Las operaciones no se duplican por doble clic.
- No se filtran secretos ni datos personales en consola.
- Las acciones sensibles se controlan en frontend y backend.
- Los flujos críticos pueden completarse con teclado.
- La interfaz no sugiere que una cita está confirmada si no existe confirmación registrada.

## Entrega esperada

Reportar:
- Pantallas y componentes implementados.
- Endpoints consumidos.
- Variables de entorno requeridas.
- Pruebas, lint, typecheck y build ejecutados.
- Limitaciones conocidas.
