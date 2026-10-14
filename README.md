# Ejercicio de Modelado Arquitectónico: C4 Model para Automatización de Dailies en Microsoft Teams (Acme Corp)

---

## 1. Explicación Preliminar de los Niveles del Modelo C4

El **Modelo C4** es un enfoque de representación visual de arquitectura de software que organiza la estructura de un sistema en niveles jerárquicos de abstracción. Su propósito es brindar claridad y facilitar la comunicación tanto con perfiles técnicos como no técnicos:

**Nivel 1 — Diagrama de Contexto (C1):** El nivel C1 muestra el sistema desde una perspectiva general. Permite identificar el sistema que se está construyendo, los usuarios que interactúan con él y los sistemas externos con los que se relaciona.

**Nivel 2 — Diagrama de Contenedores (C2):** El nivel C2 muestra cómo está estructurado internamente el sistema, dividiéndolo en contenedores que representan las principales partes de la solución y sus responsabilidades.

**Nivel 3 — Diagrama de Componentes (C3):** El nivel C3 profundiza en los contenedores, mostrando sus principales componentes y cómo estos se relacionan para cumplir las responsabilidades definidas en el nivel anterior.

---

## 2. Lógica de estructuración del sistema
La solución se estructuró alrededor de un sistema de control de dailies (**Daily bot**) integrado con **Microsoft Teams**.

En **C1** se identificaron los actores principales, **Equipo Scrum** y **Scrum Master**, junto con **Microsoft Teams** como el sistema externo de comunicación.

En **C2**, el sistema se dividió en tres contenedores principales: **Servicio API Principal**, **Planificador de Tareas** y **Base de Datos Relacional**, separando la interacción y validación con los usuarios, la ejecución de tareas programadas (recordatorios) y el almacenamiento persistente de la información.

En **C3** se detallaron los componentes dentro del **Servicio API Principal**:

* **Controlador de Interacciones:** Captura e interpreta los eventos provenientes de Teams o del Planificador.
* **Validador de Respuestas:** Verifica que el reporte cumpla con las 3 preguntas obligatorias del daily.
* **Detector de Bloqueos (3x):** Evalúa el historial para identificar impedimentos repetidos durante 3 días consecutivos y notificar alertas.
* **Generador de Reportes:** Construye y despacha las tarjetas interactivas de resumen y alertas directamente a Microsoft Teams.
