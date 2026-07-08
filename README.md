# 🎵 SintonIA

### De CheTunes a SintonIA: rediseño de un proceso de descubrimiento musical con IA agéntica

**Trabajo Final Integrador** · Diplomatura en Diseño de Procesos de Negocios con Plataformas Visuales de Automatización e IA Agéntica · FCE-UBA · Cohorte 2026

**Alumno:** Kevin Rico · **Docente:** Prof. Diego Parrás

---

## 📌 ¿Qué proceso transforma este proyecto?

Este repositorio documenta el **rediseño de un proceso de descubrimiento musical**. El proceso ya existía de forma digital: en diciembre de 2025 desarrollé **CheTunes**, una app web con IA generativa (Google Gemini) que recomendaba música, identificaba portadas de álbumes por foto y analizaba letras.

CheTunes funcionaba, pero tenía una fricción de fondo: era una **experiencia de "un solo disparo"**. Cada función era un formulario aislado, sin memoria. El usuario cargaba todo su contexto de golpe, recibía una respuesta cerrada y, para ajustar algo, empezaba de cero.

**SintonIA** es la reingeniería de ese mismo proceso, ahora como un **flujo conversacional continuo con memoria de contexto**, construido sobre **Dify** (Chatflow) con una **arquitectura híbrida de modelos**. La página de CheTunes sigue activa, con la diferencia de que ahora integra este chatbot.

|  | Antes — CheTunes | Ahora — SintonIA |
| :---- | :---- | :---- |
| **Plataforma** | Web en Google AI Studio (código TS) | Dify Chatflow (bajo código) |
| **Modelo** | Google Gemini 2.5 | gpt-4o-mini \+ Gemini 2.5 Flash (híbrido) |
| **Interacción** | Formulario de un solo disparo | Conversación continua |
| **Memoria** | ❌ Sin memoria entre acciones | ✅ Memoria de sesión |

---

## 🎯 ¿Para qué sirve?

SintonIA es un asistente musical conversacional que resuelve tres tareas en un solo chat, sin que el usuario tenga que elegir entre pestañas ni recargar contexto:

- 🎧 **Recomendaciones personalizadas** — Recolecta en diálogo tu estado de ánimo, un artista de referencia y tu preferencia de época, y recién ahí sugiere 3 canciones reales con links de búsqueda a Spotify y YouTube Music.  
- 🖼️ **Identificación de portadas** — Subís la foto de un disco y el bot, con visión multimodal, identifica artista y álbum, y entrega una reseña con contexto histórico y datos curiosos.  
- 📖 **Análisis de letras** — Pegás la letra de una canción y recibís un análisis en dimensiones (identificación, tono emocional, metáforas, significado profundo, lectura alternativa y valoración). Este nodo usa **búsqueda web en tiempo real (grounding)** para identificar la canción real y no inventar.

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

| Nodo | Tipo | Modelo | Temp. | Memoria | Función |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Inicio | `start` | — | — | — | Punto de entrada |
| Clasificador | `question-classifier` | gpt-4o-mini | 0 | 10 turnos | Enruta a la rama correcta |
| Recomendaciones | `llm` | gpt-4o-mini | 0.5 | 20 turnos | Recolecta datos y sugiere canciones |
| Portadas | `llm` (visión) | gpt-4o-mini | 0.3 | 10 turnos | Identifica y reseña álbumes |
| Letras | `llm` | **Gemini 2.5 Flash \+ Grounding** | 0.3 | 15 turnos | Análisis \+ búsqueda web de la canción |
| Off-topic | `llm` | gpt-4o-mini | 0.8 | 8 turnos | Contiene saludos y temas ajenos |

---

## 🛠️ Herramientas utilizadas

| Herramienta | Rol en el rediseño |
| :---- | :---- |
| **Gemini 3.5 Flash** | Modelo usado al inicio, en la ideación, para detectar qué proceso de CheTunes convenía automatizar. |
| **Claude Opus 4.6** | Copiloto en la etapa intermedia del armado del flujo (con errores de configuración que se corrigieron después). |
| **Claude Opus 4.8** | Copiloto para generar y depurar la versión final del archivo DSL de Dify. |
| **Dify** (Chatflow) | Plataforma visual que orquesta todo el flujo: nodos, memoria, enrutamiento y respuesta. |
| **gpt-4o-mini** (OpenAI) | Modelo base de 4 de los 5 nodos LLM. Elegido por bajo costo, velocidad y visión multimodal. |
| **Gemini 2.5 Flash \+ Grounding** | Modelo del nodo de Letras. El grounding le da búsqueda web de Google para identificar la canción real y evitar alucinaciones. |
| **Question Classifier** (Dify) | Nodo nativo que clasifica cada mensaje en 1 de 4 intenciones, con memoria de contexto. |
| **Visión multimodal** | Habilitada en el nodo de portadas para leer imágenes de álbumes. |

💡 **Arquitectura híbrida:** no todos los nodos usan el mismo modelo. Cada nodo usa el modelo óptimo para su tarea: gpt-4o-mini donde funcionaba bien y era barato, y Gemini 2.5 Flash con grounding solo en Letras, el único nodo que necesitaba búsqueda web real.

---

## 🚀 Cómo usarlo

### Requisitos

- Una cuenta en [Dify](https://dify.ai) (el plan gratuito alcanza).  
- Un proveedor **OpenAI** configurado con acceso a `gpt-4o-mini`.  
- Un proveedor **Google/Gemini** configurado con acceso a `gemini-2.5-flash` (para el nodo de Letras).

### Importar el flujo

1. Cloná o descargá este repositorio.  
2. En Dify, entrá a tu workspace y hacé clic en **"Crear aplicación"** → **"Importar archivo DSL"**.  
3. Subí el archivo **`SintonIA.yml`.**  
4. Verificá que `gpt-4o-mini` y `gemini-2.5-flash` estén disponibles en tus proveedores de modelo.  
5. En el nodo **Análisis de Letras**, confirmá que el modelo sea **Gemini 2.5 Flash** y que la opción **Grounding** esté en **True** (es lo que activa la búsqueda web).  
6. En el nodo **Identificación de Portadas**, confirmá que la opción **Vision** esté activada.  
7. Ejecutá la vista previa y probá los casos de test de abajo.
8. Tambien podes probar el chat desde la Web de [**CheTunes**](https://chetunes-ai-68023813385.us-west2.run.app/)

### Casos de prueba

| Escribí esto | Debería... |
| :---- | :---- |
| `quiero música para relajarme` | Ir a Recomendaciones y preguntar tu mood |
| `algo triste tipo Radiohead de los 90` | Detectar los 3 datos y recomendar directo |
| *(subís una foto de un disco)* | Ir a Portadas y analizar la imagen |
| `quiero identificar una portada` | Ir a Portadas y pedirte la imagen |
| *(pegás una letra real)* | Ir a Letras, identificar la canción con búsqueda web y analizarla |
| `hola, ¿cómo estás?` | Ir a Off-topic y redirigirte con calidez |

---

## 📂 Estructura del repositorio

sintonia/

├── README.md                       ← este archivo

├── SintonIA.yml                    ← el flujo completo, listo para importar

├── informe/

│   └── Informe\_SintonIA.pdf        ← informe escrito del TP

---

## 🔄 Evolución del proyecto (proceso de trabajo)

El flujo no salió bien de una. Estas fueron las iteraciones y qué corrigió cada una:

| Versión | Problema detectado | Corrección |
| :---- | :---- | :---- |
| **v1** | Flujo base con 4 ramas incluyendo greeting | Arquitectura inicial completa |
| **v2** | El flujo era demasiado rígido | Simplificación a bienvenida → clasificador → ramas |
| **v3** | La respuesta mostraba texto crudo `{{#node.text#}}` | IDs de nodo reescritos a formato numérico nativo de Dify |
| **v4** | Responder "Radiohead" desviaba a Portadas | Memoria \+ regla de continuidad en el clasificador; banco de 27 preguntas dinámicas |
| **v5** | El análisis de letras inventaba canciones (alucinación) | Nodo de Letras migrado a **Gemini 2.5 Flash con Grounding** (búsqueda web), adoptando una arquitectura híbrida de modelos |

---

## 📊 Evaluación AIBPS (resumen)

- **⚡ Agilidad** — Más ágil en la exploración iterativa: refinar un pedido es un mensaje más, no reiniciar un formulario. El nodo de Letras con grounding tarda un poco más porque busca en la web antes de responder.  
- **💧 Fluidez** — Desaparece la fricción de elegir pestaña; la interacción imita una charla natural.  
- **🔒 Protección** — Conversación efímera, sin datos personales sensibles; ejecuciones trazables en los logs de Dify.  
- **🎛️ Control** — Diseño humano de las reglas, incluida la decisión de qué modelo usa cada rama; la IA solo sugiere, nunca ejecuta acciones irreversibles; flujo auditable nodo por nodo.

- El análisis completo está en el [informe PDF](https://drive.google.com/file/d/1E2mtw4vJjYXAAz8-Td9JIN7pQD2hnTns/view?usp=drive_link).  
- [Podcast de SintonIA](https://drive.google.com/file/d/14BAWW_n_0FzAxGEYAeFoj6STHFrW7VfM/view?usp=drive_link)  
- [Web Chetunes](https://chetunes-ai-68023813385.us-west2.run.app/)

---

## 📝 Licencia y créditos

Proyecto académico desarrollado para la Diplomatura en Diseño de Procesos de Negocios de la FCE-UBA (Cohorte 2026).

*"De una web que respondía una vez, a una conversación que acompaña. Ese fue el verdadero rediseño."*  
