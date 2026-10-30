# Skills de desarrollo — CRM Dental

Este paquete contiene cuatro skills reutilizables para orientar a agentes de IA y al equipo de desarrollo del CRM dental de la Dra. Laura.

## Skills incluidos

- `.github\skills\backend-dotnet\SKILL.md`: backend ASP.NET Core sobre .NET 10, C#, EF Core y arquitectura modular.
- `.github\skills\frontend\SKILL.md`: frontend React + TypeScript e integración con la API.
- `.github\skills\diseno-ux\SKILL.md`: experiencia de usuario para una doctora y una asistente con poco tiempo.
- `.github\skills\testing\SKILL.md`: estrategia de pruebas unitarias, integración, aceptación, seguridad y sincronización.

## Cómo usarlos

1. Copia la carpeta `.github\skills\` al repositorio del proyecto.
2. Si tu agente de programación admite Agent Skills, configura o coloca las skills en la ubicación que el agente espera. La estructura `nombre/SKILL.md` sigue el patrón común de Agent Skills.
3. Antes de implementar una tarea, indica al agente qué skill debe aplicar. Para tareas transversales, combina las skills pertinentes.
4. Mantén los requerimientos y decisiones del proyecto en los documentos oficiales del repositorio. Las skills son instrucciones de trabajo, no sustituyen `PROJECT_STATUS.md`, `docs/REQUIREMENTS.md` ni las decisiones de arquitectura.

## Contexto fijo del proyecto

- Producto: CRM pequeño para consultorio dental.
- Objetivo: reducir inasistencias, automatizar recordatorios y dar seguimiento administrativo a tratamientos de varias sesiones.
- No incluye facturación ni expediente clínico electrónico.
- Backend: ASP.NET Core / C# con destino .NET 10.
- Frontend propuesto: React + TypeScript.
- Persistencia propuesta: MariaDB + Entity Framework Core, salvo decisión documentada distinta.
- Integraciones: Google Calendar y proveedor oficial compatible con WhatsApp Business Platform.
- Arquitectura inicial: monolito modular; no introducir microservicios sin una decisión arquitectónica explícita.
- Roles iniciales: Administrador, Doctor y Asistente.
- La doctora define los plazos de seguimiento; el software no hace recomendaciones clínicas.
- Privacidad y seguridad son requisitos transversales.

## Convenciones importantes

- No inventar decisiones de negocio, secretos, endpoints, credenciales ni resultados de pruebas.
- Si falta un dato que cambie arquitectura, seguridad, costo o comportamiento de negocio, registrar la pregunta y proponer opciones.
- Usar español para documentación y textos de interfaz, salvo que el repositorio establezca otra convención.
- Mantener código, pruebas y documentación alineados.
- No declarar una integración real como verificada si solo se probó con mocks o simuladores.
- Responder siempre en español.
