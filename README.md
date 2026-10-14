# Ejercicio de Modelado Arquitectónico: C4 Model para Automatización de Dailies en Microsoft Teams (Acme Corp)

---

## 1. Explicación Preliminar de los Niveles del Modelo C4

El **Modelo C4** es un enfoque de representación visual de arquitectura de software que organiza la estructura de un sistema en niveles jerárquicos de abstracción. Su propósito es brindar claridad y facilitar la comunicación tanto con perfiles técnicos como no técnicos:

* **Nivel 1 — Diagrama de Contexto (C1):** Muestra la vista panorámica del sistema. Establece las fronteras de la aplicación identificando quiénes son los usuarios (integrantes del equipo Scrum y Scrum Master) y los sistemas externos con los que interactúa (plataforma de Microsoft Teams).
* **Nivel 2 — Diagrama de Contenedores (C2):** Describe la arquitectura interna descomponiendo el sistema en sus unidades independientes de ejecución y almacenamiento (APIs, tareas programadas y bases de datos), detallando las responsabilidades de cada contenedor y las interacciones entre ellos.
* **Nivel 3 — Diagrama de Componentes (C3):** Enfoca la vista dentro de un contenedor específico (el Servicio API Principal) para mostrar sus módulos de código internos y la lógica de negocio responsable de procesar la información (validación de entregas, detección de bloqueos 3x y generación de reportes).

---

## 2. Nivel 1: Diagrama de Contexto del Sistema (C1)

### Descripción del Alcance
El sistema **Daily bot** (Aplicación Daily) actúa como el núcleo central para la recolección, validación y difusión asíncrona del trabajo diario de los equipos Scrum en Acme Corp. Los usuarios no interactúan directamente con el servidor de la aplicación, sino que lo hacen a través de la interfaz conversacional y las tarjetas interactivas expuestas en **Microsoft Teams**.

### Tabla de Entidades y Relaciones (C1)

| Identificador | Nombre de la Entidad | Tipo de Elemento | Descripción / Rol | Protocolo / Interfaz |
| :--- | :--- | :--- | :--- | :--- |
| **ACT-01** | **Equipo Scrum** | Persona (Usuario) | Desarrollador o QA que responde el recordatorio diario con sus 3 preguntas y consulta los resúmenes del equipo. | Cliente MS Teams |
| **ACT-02** | **Scrum Master** | Persona (Usuario) | Configura parámetros del equipo y recibe notificaciones grupales o alertas de bloqueos repetitivos. | Cliente MS Teams |
| **SYS-01** | **Daily bot** *(Aplicación Daily)* | **Sistema Principal** | Sistema que gestiona la lógica de las dailies, procesa horarios, valida respuestas y detecta bloqueos. | HTTPS REST / Webhooks |
| **SYS-02** | **Microsoft Teams** | Sistema Externo | Plataforma de comunicación corporativa que actúa como interfaz entre los usuarios y la aplicación. | Bot Framework API / Webhooks |

---

## 3. Nivel 2: Diagrama de Contenedores (C2)

### Descripción Arquitectónica
Al abrir la frontera del sistema **Daily bot**, la solución se descompone en **3 contenedores clave** que separan la recepción de interacciones, la programación temporal y el almacenamiento persistente:

### Tabla de Contenedores y Relaciones (C2)

| Identificador | Contenedor | Tipo de Elemento | Responsabilidad Arquitectónica | Relación / Protocolo |
| :--- | :--- | :--- | :--- | :--- |
| **CON-01** | **Servicio API Principal** | Aplicación API | Núcleo del servidor. Procesa la lógica de las dailies, valida preguntas y ejecuta el cálculo de bloqueos (regla 3x). | Recibe/envía peticiones HTTPS con MS Teams. Lee/escribe en la Base de Datos. |
| **CON-02** | **Planificador de Tareas** | Servicio Background (Worker) | Reloj interno que ejecuta temporizadores automáticos para activar el envío de recordatorios diarios según la zona horaria. | Consulta programación en la Base de Datos y solicita el disparo al Servicio API. |
| **CON-03** | **Base de Datos Relacional** | Base de Datos | Almacena los datos estructurados: usuarios, equipos, configuraciones de horario, respuestas e historial de bloqueos. | Conexión relacional estándar (SQL) con el Servicio API y el Planificador. |

---

## 4. Nivel 3: Diagrama de Componentes (C3)

### Descripción de Componentes Internos
Haciendo un "zoom" exclusivo dentro del contenedor **Servicio API Principal**, la lógica de negocio se organiza en **4 componentes modulares** con responsabilidades únicas:

### Tabla de Componentes Internos (C3)

| Identificador | Componente | Tipo / Rol | Responsabilidad Específica | Dependencias y Conexiones |
| :--- | :--- | :--- | :--- | :--- |
| **CMP-01** | **Controlador de Interacciones** | Controlador API | Punto de entrada que recibe e interpreta peticiones e interacciones enviadas desde Teams o el Planificador. | Recibe eventos de Teams/Planificador y deriva al Validador. |
| **CMP-02** | **Validador de Respuestas** | Lógica de Negocio | Verifica que el reporte del daily cumpla con las 3 preguntas obligatorias antes de registrarlo. | Pasa datos al Detector de Bloqueos y guarda en Base de Datos. |
| **CMP-03** | **Detector de Bloqueos (3x)** | Lógica de Alertas | Analiza el historial para identificar impedimentos repetidos durante 3 días seguidos (Regla 3x). | Consulta la Base de Datos y notifica al Generador de Reportes. |
| **CMP-04** | **Generador de Reportes** | Formateador / Salida | Ensambla y da formato a las tarjetas interactivas de resumen y alertas de estancamiento. | Lee de la Base de Datos y envía tarjetas directamente a Teams. |

---

## 5. Racional de Arquitectura

La arquitectura de **Daily bot** para Acme Corp fue diseñada bajo un patrón modular y desacoplado, enfocado en la simplicidad, mantenibilidad y la alta cohesión de sus partes. Se separó deliberadamente la recepción de eventos de chat (**Servicio API Principal**) de la ejecución de temporizadores automáticos (**Planificador de Tareas**), garantizando que el envío masivo de recordatorios diarios no afecte el tiempo de respuesta en las interacciones activas con los usuarios. La elección de un almacén relacional asegura la integridad y consistencia en la consulta del historial de reportes sin añadir complejidad o costos excesivos de infraestructura.

A nivel de componentes, la lógica interna se organizó siguiendo el principio de responsabilidad única: separar el controlador de entrada, el validador de respuestas, el generador de salida y el detector analítico de bloqueos (regla 3x). Esta separación permite evaluar asíncronamente si un desarrollador lleva tres días con el mismo impedimento sin interferir con el flujo conversacional en Microsoft Teams. La solución satisface al 100% todos los requerimientos funcionales del ejercicio manteniendo una estructura limpia y de fácil implementación técnica.

---

## 6. Instrucciones de Entrega y Repositorio

1. **SharePoint:** Subir el archivo editable `.drawio` exportado desde draw.io con los tres niveles (C1, C2 y C3) y generar el enlace de compartir.
2. **Estructura del Repositorio Git:**
   * **Nombre del Repositorio:** `s2026q2a-c4model-dvillanueva` *(siguiendo la nomenclatura `semillero+assignment+nombre`)*.
   * **Archivos requeridos:**
     * `README.md` *(este documento completo)*.
     * `diagramas_c4_dailies.drawio` *(archivo fuente de draw.io)*.
     * Capturas de pantalla o exportaciones en imagen de los tres diagramas (C1, C2 y C3).
