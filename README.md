# 🎵 SintonIA

### De CheTunes a SintonIA: rediseño de un proceso de descubrimiento musical con IA agéntica

**Trabajo Final Integrador** · Diplomatura en Diseño de Procesos de Negocios con Plataformas Visuales de Automatización e IA Agéntica · FCE-UBA · Cohorte 2026

**Alumno:** Kevin Rico · **Docente:** Prof. Diego Parrás

---

## 📌 ¿Qué proceso transforma este proyecto?

Este repositorio documenta el **rediseño de un proceso de descubrimiento musical**. El proceso ya existía de forma digital: en diciembre de 2025 desarrollé **CheTunes**, una app web con IA generativa (Google Gemini) que recomendaba música, identificaba portadas de álbumes por foto y analizaba letras.

CheTunes funcionaba, pero tenía una fricción de fondo: era una **experiencia de "un solo disparo"**. Cada función era un formulario aislado, sin memoria. El usuario cargaba todo su contexto de golpe, recibía una respuesta cerrada y, para ajustar algo, empezaba de cero.

**SintonIA** es la reingeniería de ese mismo proceso, ahora como un **flujo conversacional continuo con memoria de contexto**, construido sobre **Dify** (Chatflow) con `gpt-4o-mini`.

|  | Antes — CheTunes | Ahora — SintonIA |
| :---- | :---- | :---- |
| **Plataforma** | Web en Google AI Studio (código TS) | Dify Chatflow (bajo código) |
| **Modelo** | Google Gemini 2.5 | gpt-4o-mini (OpenAI) |
| **Interacción** | Formulario de un solo disparo | Conversación continua |
| **Memoria** | ❌ Sin memoria entre acciones | ✅ Memoria de sesión |
| **Navegación** | El usuario elige la pestaña | El clasificador enruta solo |

---

## 🎯 ¿Para qué sirve?

SintonIA es un asistente musical conversacional que resuelve tres tareas en un solo chat, sin que el usuario tenga que elegir entre pestañas ni recargar contexto:

- 🎧 **Recomendaciones personalizadas** — Recolecta en diálogo tu estado de ánimo, un artista de referencia y tu preferencia de época, y recién ahí sugiere 3 canciones reales con links de búsqueda a Spotify y YouTube Music.  
- 🖼️ **Identificación de portadas** — Subís la foto de un disco y el bot, con visión multimodal, identifica artista y álbum, y entrega una reseña con contexto histórico y datos curiosos.  
- 📖 **Análisis de letras** — Pegás la letra de una canción y recibís un análisis en 6 dimensiones: identificación, tono emocional, metáforas, significado profundo, lectura alternativa y valoración.

El **valor central del rediseño es la memoria conversacional**: SintonIA mantiene el hilo de la charla, hace preguntas de seguimiento y permite refinar pedidos de forma natural.

---

## 🏗️ Arquitectura del flujo

DISPARADOR: el usuario abre el chat y ve una bienvenida automática

   │

   ▼

┌──────────────┐

│ NODO INICIO  │  captura el mensaje (texto y/o imagen)

└──────┬───────┘

       ▼

┌────────────────────────┐

│ CLASIFICADOR DE         │  ← con memoria de 10 turnos

│ INTENCIONES             │  enruta automáticamente según intención \+ contexto

└───┬────┬────┬────┬──────┘

    │    │    │    │

    (1)  (2)  (3)  (4)

    │    │    │    │

    ▼    ▼    ▼    ▼

┌───────┐ ┌────────┐ ┌────────┐ ┌───────────┐

│ RECOM.│ │PORTADAS│ │ LETRAS │ │ OFF-TOPIC │

└───┬───┘ └───┬────┘ └───┬────┘ └─────┬─────┘

    ▼         ▼          ▼            ▼

  cada rama entrega su RESPUESTA y la conversación CONTINÚA

### Nodos del Chatflow

| Nodo | Tipo | Temp. | Memoria | Función |
| :---- | :---- | :---- | :---- | :---- |
| Inicio | `start` | — | — | Punto de entrada |
| Clasificador | `question-classifier` | 0 | 10 turnos | Enruta a la rama correcta |
| Recomendaciones | `llm` | 0.7 | 20 turnos | Recolecta datos y sugiere canciones |
| Portadas | `llm` (visión) | 0.4 | 10 turnos | Identifica y reseña álbumes |
| Letras | `llm` | 0.6 | 15 turnos | Análisis semiótico de letras |
| Off-topic | `llm` | 0.8 | 8 turnos | Contiene saludos y temas ajenos |

---

## 🛠️ Herramientas utilizadas

| Herramienta | Rol en el rediseño |
| :---- | :---- |
| **Dify** (Chatflow) | Plataforma visual que orquesta todo el flujo: nodos, memoria, enrutamiento y respuesta. |
| **gpt-4o-mini** (OpenAI) | Modelo base de los 5 nodos LLM. Elegido por bajo costo, velocidad y visión multimodal. |
| **Question Classifier** (Dify) | Nodo nativo que clasifica cada mensaje en 1 de 4 intenciones, con memoria de contexto. |
| **Visión multimodal** | Habilitada en el nodo de portadas para leer imágenes de álbumes. |
| **Claude Opus 4.8, Claude Opus 4.6** | Usado para generar y depurar el archivo DSL de Dify. |

---

## 🚀 Cómo usarlo

### Requisitos

- Una cuenta en [Dify](https://dify.ai) (el plan gratuito alcanza).  
- Un proveedor de modelo configurado en Dify con acceso a `gpt-4o-mini` (OpenAI).

### Importar el flujo

1. Cloná o descargá este repositorio.  
2. En Dify, entrá a tu workspace y hacé clic en **"Crear aplicación"** → **"Importar archivo DSL"**.  
3. Subí el archivo [`sintonia_dify_chatflow.yml`](http://./sintonia_dify_chatflow.yml).  
4. Verificá que `gpt-4o-mini` esté disponible en tu proveedor de modelos.  
5. En el nodo **Identificación de Portadas**, confirmá que la opción **Vision** esté activada.  
6. Ejecutá la vista previa y probá los casos de test de abajo.

### Casos de prueba

| Escribí esto | Debería... |
| :---- | :---- |
| `quiero música para relajarme` | Ir a Recomendaciones y preguntar tu mood |
| `algo triste tipo Radiohead de los 90` | Detectar los 3 datos y recomendar directo |
| *(subís una foto de un disco)* | Ir a Portadas y analizar la imagen |
| `quiero identificar una portada` | Ir a Portadas y pedirte la imagen |
| *(pegás una letra)* | Ir a Letras y analizarla |
| `hola, ¿cómo estás?` | Ir a Off-topic y redirigirte con calidez |

---

## 📂 Estructura del repositorio

sintonia/

├── README.md                       ← este archivo

├── sintonia\_dify\_chatflow.yml      ← el flujo completo, listo para importar

├── informe/

│   └── Informe\_SintonIA.pdf        ← informe escrito del TP

├── docs/

│   ├── proceso-antes.md            ← diagnóstico de CheTunes

│   ├── proceso-ahora.md            ← diseño de SintonIA

│   └── diagramas/                  ← flujos antes/después

└── evolucion/                      ← versiones intermedias del flujo (v1 → v4)

    ├── v1\_flujo\_base.yml

    ├── v2\_sin\_greeting.yml

    ├── v3\_ids\_corregidos.yml

    └── v4\_final\_con\_memoria.yml

💡 La carpeta `evolucion/` muestra el proceso de trabajo, no solo el resultado: cada versión corrigió un problema concreto detectado en la anterior.

---

## 🔄 Evolución del proyecto (proceso de trabajo)

El flujo no salió bien de una. Estas fueron las iteraciones y qué corrigió cada una:

| Versión | Problema detectado | Corrección |
| :---- | :---- | :---- |
| **v1** | Flujo base con 4 ramas incluyendo greeting | Arquitectura inicial completa |
| **v2** | El flujo era demasiado rígido | Simplificación a bienvenida → clasificador → ramas |
| **v3** | La respuesta mostraba texto crudo `{{#node.text#}}` | IDs de nodo reescritos a formato numérico nativo de Dify |
| **v4** | Responder "Radiohead" desviaba a Portadas | Memoria \+ regla de continuidad en el clasificador; banco de 27 preguntas dinámicas |

---

## 📊 Evaluación AIBPS (resumen)

- **⚡ Agilidad** — Más ágil en la exploración iterativa: refinar un pedido es un mensaje más, no reiniciar un formulario.  
- **💧 Fluidez** — Desaparece la fricción de elegir pestaña; la interacción imita una charla natural.  
- **🔒 Protección** — Conversación efímera, sin datos personales sensibles; ejecuciones trazables en los logs de Dify.  
- **🎛️ Control** — Diseño humano de las reglas; la IA solo sugiere, nunca ejecuta acciones irreversibles; flujo auditable nodo por nodo.

El análisis completo está en el [informe PDF](http://./informe/Informe_SintonIA.pdf).

---

## 📝 Licencia y créditos

Proyecto académico desarrollado para la Diplomatura en Diseño de Procesos de Negocios de la FCE-UBA (Cohorte 2026).

*"De una web que respondía una vez, a una conversación que acompaña. Ese fue el verdadero rediseño."*  
