---
name: pmbok8-project-management
description: Sistema de agentes para generación automática de artefactos de gestión de proyectos basado en PMBOK 8 del PMI. Usar cuando se necesite crear documentación de proyectos como Actas de Constitución, WBS, Registros de Riesgos, Matrices RACI, Product Backlogs, Cronogramas, Presupuestos y cualquier otro entregable de gestión de proyectos. Soporta enfoques predictivos, ágiles e híbridos adaptando los artefactos según el ciclo de vida del proyecto. NUEVO: Incluye soporte multi-proveedor para Claude (narrativa/análisis) y Gemini (datos estructurados/cuantitativos).
---

# Sistema de Agentes para Gestión de Proyectos PMBOK 8

Este skill proporciona un sistema de agentes especializados para generar artefactos de gestión de proyectos alineados con el PMBOK 8 del Project Management Institute.

## Cuándo Usar Este Skill

Utilice este skill cuando necesite generar documentación de gestión de proyectos, incluyendo documentos de inicio como el Acta de Constitución del Proyecto o el Business Case, documentos de planificación como WBS, Cronogramas, Presupuestos o Planes de Gestión, documentos de ejecución como Informes de Avance o Solicitudes de Cambio, documentos de monitoreo como Análisis de Valor Ganado o Actualizaciones de Riesgos, documentos de cierre como Lecciones Aprendidas o Informes de Cierre, y artefactos ágiles como Product Backlogs, Historias de Usuario o Sprint Backlogs.

## Estructura del Sistema

El sistema está organizado en tres niveles con soporte multi-proveedor. El agente coordinador (agent.md) recibe los requerimientos del proyecto y determina qué artefactos generar según la fase del ciclo de vida. Los subagentes especializados en el directorio subagents contienen las instrucciones detalladas para cada dominio de desempeño del PMBOK 8. Las referencias y plantillas en el directorio references proporcionan las plantillas base para cada tipo de artefacto. El directorio providers contiene instrucciones optimizadas para diferentes proveedores de IA (Claude, Gemini).

## Flujo de Trabajo

Para generar artefactos de gestión de proyectos, siga estos pasos:

Primero, seleccione el proveedor de IA apropiado. Use providers/INSTRUCCION-INICIO-CLAUDE.md para documentos narrativos y análisis, o providers/INSTRUCCION-INICIO-GEMINI.md para datos estructurados y análisis cuantitativos.

Segundo, identifique la fase del ciclo de vida del proyecto. Las fases posibles son Inicio, Planificación, Ejecución, Monitoreo y Control, o Cierre.

Tercero, determine el enfoque del proyecto. Los enfoques disponibles son Predictivo (tradicional/cascada), Ágil (Scrum, Kanban), o Híbrido (combinación).

Cuarto, consulte el agente coordinador leyendo el archivo agent.md para entender la lógica de coordinación.

Quinto, active los subagentes correspondientes. Para la fase de Inicio utilice gobernanza-agent.md e interesados-agent.md. Para Planificación utilice alcance-agent.md, cronograma-agent.md, finanzas-agent.md, recursos-agent.md y riesgo-agent.md. Para proyectos ágiles utilice agile-agent.md.

Sexto, utilice las plantillas del directorio references para generar los documentos con la estructura correcta.

## Artefactos por Fase del Ciclo de Vida

### Fase de Inicio

En esta fase se generan el Acta de Constitución del Proyecto, el Registro de Interesados inicial, la identificación de riesgos de alto nivel y, para proyectos ágiles, la Visión del Producto y el Product Backlog inicial.

### Fase de Planificación

En esta fase se generan el Plan de Gestión del Proyecto, la Estructura de Desglose del Trabajo (WBS), el Diccionario de la WBS, el Cronograma del Proyecto, el Presupuesto, el Registro de Riesgos completo, el Plan de Comunicaciones, la Matriz RACI, las Líneas Base de alcance, tiempo y costo. Para proyectos ágiles se genera el Product Backlog refinado, la Definition of Done y el Roadmap de Releases.

### Fase de Ejecución

En esta fase se gestionan las actualizaciones del Registro de Riesgos, los Informes de Avance, las Solicitudes de Cambio, los registros de Lecciones Aprendidas en progreso y, para proyectos ágiles, los Sprint Backlogs y Burndown Charts.

### Fase de Monitoreo y Control

En esta fase se realizan los Informes de Estado del Proyecto, el Análisis de Valor Ganado, el Control de Cambios, las actualizaciones a planes y líneas base, y los Informes de Desempeño.

### Fase de Cierre

En esta fase se generan el Informe de Cierre del Proyecto, las Lecciones Aprendidas finales, la documentación de Aceptación de Entregables y el Archivo del Proyecto.

## Dominios de Desempeño PMBOK 8

El sistema cubre los siete dominios de desempeño establecidos en el PMBOK 8. El dominio de Gobernanza abarca la supervisión, autorización y toma de decisiones del proyecto. El dominio de Alcance define qué trabajo está incluido y excluido del proyecto. El dominio de Cronograma gestiona la planificación y control temporal del proyecto. El dominio de Finanzas maneja la estimación, presupuestación y control de costos. El dominio de Interesados identifica, analiza y gestiona el compromiso de los stakeholders. El dominio de Recursos planifica y gestiona los recursos humanos y materiales. El dominio de Riesgo identifica, analiza y responde a la incertidumbre del proyecto.

## Principios PMBOK 8

Todos los artefactos generados están alineados con los seis principios fundamentales del PMBOK 8. El principio de Adoptar una Visión Holística implica considerar el proyecto como un sistema integrado. El principio de Enfocarse en el Valor asegura que cada entregable aporte valor medible. El principio de Integrar la Calidad embebe la calidad en procesos y entregables. El principio de Liderazgo Responsable promueve la toma de decisiones éticas y responsables. El principio de Integrar Sostenibilidad considera los impactos ambientales, sociales y económicos. El principio de Construir Cultura Empoderada fomenta equipos colaborativos y empoderados.

## Ejemplo de Uso

Para iniciar un nuevo proyecto, proporcione la siguiente información:

El nombre del proyecto con una descripción breve. La justificación del proyecto explicando el problema u oportunidad que aborda. La fase actual del ciclo de vida en que se encuentra el proyecto. El enfoque de desarrollo que puede ser predictivo, ágil o híbrido. Los entregables específicos que se requieren generar.

Por ejemplo, si el usuario indica que tiene un proyecto de implementación de CRM para el departamento de ventas, con el objetivo de incrementar las ventas en un 15% mediante mejor gestión de clientes, encontrándose en fase de inicio con enfoque híbrido, necesitando el Acta de Constitución y el Product Backlog inicial, el sistema activará el subagente de gobernanza para generar el Acta de Constitución y el subagente ágil para generar el Product Backlog inicial, ambos adaptados al contexto híbrido del proyecto.

## Ubicación de Archivos

El archivo agent.md contiene la lógica del agente coordinador principal. El archivo skills.md describe las capacidades detalladas del sistema. El directorio providers contiene las instrucciones optimizadas para cada proveedor de IA. El directorio subagents contiene los agentes especializados por dominio. El directorio references contiene las plantillas y guías de referencia organizadas por dominio.
