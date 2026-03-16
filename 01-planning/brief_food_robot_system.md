# Sistema Autónomo de Gestión Alimentaria Controlado por Voz

## Documento de Arquitectura Técnica

------------------------------------------------------------------------

  

# 1. Resumen

  

Este documento define la **arquitectura de referencia** de un

sistema de gestión alimentaria doméstica controlado por voz que permite:

  

- Gestionar el inventario de ingredientes

- Procesar los PDF de recetas semanales del nutricionista

- Sugerir recetas según los ingredientes disponibles

- Generar listas de la compra de forma automática

- Soportar interacción por voz mediante Alexa

- Procesar documentos a través de WhatsApp

- Opcionalmente controlar un asistente de cocina robótico

  

La arquitectura integra:

  

- Interfaces de voz

- Agentes de IA

- Generación aumentada por recuperación (RAG)

- Almacenamiento de datos estructurados

- Control robótico

  

Interfaces principales del sistema:

  

- Skill Alexa + Adaptador Lambda

- Bot de ingesta por WhatsApp

- Servicio API de agentes

- Base de datos Supabase

- Controlador robótico Raspberry Pi

  

------------------------------------------------------------------------

  

# 2. Enunciado del problema

  

La gestión alimentaria doméstica actual presenta varias ineficiencias:

  

- No existe un inventario centralizado de ingredientes

- Las recetas semanales del nutricionista llegan en PDF

- Comprobación manual de los ingredientes que faltan

- No hay automatización de las listas de la compra

- No hay memoria ni base de conocimiento de recetas

- La integración actual con Alexa está fuertemente acoplada a una única Lambda (lógica de negocio y orquestación dentro del mismo componente)

  

Limitaciones técnicas adicionales:

  

- No hay arquitectura modular

- No hay sistema de recuperación RAG

- No hay capa de orquestación de agentes

- No hay abstracción para robótica

  

------------------------------------------------------------------------

  

# 3. Objetivos del sistema

  

El sistema debe:

  

1. Ingerir los PDF de recetas recibidos por WhatsApp

2. Extraer datos estructurados de las recetas

3. Almacenar recetas e ingredientes en Supabase

4. Permitir consultas por voz mediante Alexa

5. Sugerir recetas según los ingredientes disponibles

6. Generar automáticamente listas de la compra

7. Opcionalmente traducir recetas en acciones robóticas

  

Objetivos secundarios:

  

- Agentes de IA modulares

- Arquitectura extensible

- Respuestas de baja latencia

- Integración con robótica

  

------------------------------------------------------------------------

  

# 4. Arquitectura objetivo (alto nivel)

  

Usuario\

↓\

Alexa / WhatsApp\

↓\

Capa de integración (Skill Alexa + Lambda, Bot WhatsApp)\

↓\

Servicio API de agentes\

↓\

Sistema de agentes de IA\

↓\

Capa de datos (Supabase + base vectorial)\

↓\

Capa robótica (Raspberry Pi)

  

------------------------------------------------------------------------

  

# 5. Arquitectura C4

  

## 5.1 Contexto del sistema (nivel 1)

  

``` mermaid

flowchart LR

  

User((Usuario))

  

Alexa[Dispositivo Alexa]

WhatsApp[WhatsApp]

  

FoodSystem[Sistema de Gestión Alimentaria]

  

Supabase[(Base de datos Supabase)]

Robot[Robot Raspberry Pi]

  

User --> Alexa

User --> WhatsApp

  

Alexa --> FoodSystem

WhatsApp --> FoodSystem

  

FoodSystem --> Supabase

FoodSystem --> Robot

```

  

------------------------------------------------------------------------

  

## 5.2 Arquitectura de contenedores (nivel 2)

  

``` mermaid

flowchart LR

  

User((Usuario))

  

Alexa[Dispositivo Alexa]

WhatsApp[WhatsApp]

  

Skill[Skill Alexa]

Lambda[Adaptador AWS Lambda]

BotWhatsApp[Bot WhatsApp]

  

AgentAPI[Servicio API de agentes]

  

RAG[Capa de recuperación RAG]

  

Supabase[(PostgreSQL)]

VectorDB[(Base vectorial)]

  

RobotController[Raspberry Pi]

  

User --> Alexa

Alexa --> Skill

Skill --> Lambda

Lambda --> AgentAPI

  

User --> WhatsApp

WhatsApp --> BotWhatsApp

BotWhatsApp --> AgentAPI

  

AgentAPI --> Supabase

AgentAPI --> RAG

  

RAG --> VectorDB

  

AgentAPI --> RobotController

```

  

------------------------------------------------------------------------

  

## 5.3 Arquitectura de componentes (nivel 3)

  

``` mermaid

flowchart LR

  

AgentAPI[Servicio API de agentes]

  

subgraph Agent System

  

InventoryAgent[Agente de inventario alimentario]

RecipeAgent[Agente de recomendación de recetas]

ShoppingAgent[Agente de lista de la compra]

RobotAgent[Agente de comandos robóticos]

  

end

  

PDFParser[Analizador PDF]

Fragmentador[Fragmentador]

EmbeddingService[Servicio de embeddings]

Retriever[Recuperador vectorial]

LLM[Servicio LLM]

  

DB[(Supabase DB)]

VectorDB[(Almacén vectorial)]

  

AgentAPI --> PDFParser

AgentAPI --> InventoryAgent

AgentAPI --> RecipeAgent

AgentAPI --> ShoppingAgent

AgentAPI --> RobotAgent

  

PDFParser --> Fragmentador

PDFParser --> InventoryAgent

Fragmentador --> EmbeddingService

  

InventoryAgent --> DB

RecipeAgent --> DB

ShoppingAgent --> DB

  

EmbeddingService --> VectorDB

Retriever --> VectorDB

  

RecipeAgent --> Retriever

Retriever --> LLM

RecipeAgent --> LLM

  

RobotAgent --> RobotController

```

  

------------------------------------------------------------------------

  

# 6. Capas principales de la arquitectura

  

## 6.1 Capa de integración

  

Gestiona los puntos de entrada al sistema.

  

Canales:

  

- Skill Alexa → Adaptador AWS Lambda

- Bot WhatsApp → API de agentes

  

Responsabilidades:

  

- Enrutado de intenciones

- Validación de entradas

- Normalización de peticiones

  

------------------------------------------------------------------------

  

## 6.2 Servicio API de agentes

  

Servicio central de orquestación.

  

Responsabilidades:

  

- Gestionar los agentes de IA

- Coordinar flujos de trabajo

- Aplicar reglas de negocio

- Exponer endpoints REST

  

Stack recomendado:

  

- FastAPI

- Python 3.10+

- Ejecución asíncrona

- Despliegue en contenedores

  

------------------------------------------------------------------------

  

## 6.3 Sistema de agentes de IA

  

Agentes de IA especializados realizan las tareas de dominio.

  

### Agente de inventario alimentario

  

Procesa los PDF del nutricionista.

  

Responsabilidades:

  

- Analizar PDF

- Extraer recetas

- Normalizar ingredientes

- Almacenar datos estructurados

  

------------------------------------------------------------------------

  

### Agente de recomendación de recetas

  

Ofrece sugerencias de recetas.

  

Capacidades:

  

- Coincidencia de ingredientes

- Ordenación de recetas

- Generación opcional de recetas

  

------------------------------------------------------------------------

  

### Agente de lista de la compra

  

Genera listas de la compra.

  

Responsabilidades:

  

- Detectar ingredientes faltantes

- Agregar necesidades semanales

- Optimizar listas de la compra

  

------------------------------------------------------------------------

  

### Agente de comandos robóticos

  

Integración opcional con robótica.

  

Responsabilidades:

  

- Convertir pasos de receta en acciones del robot

- Comunicarse con Raspberry Pi

  

Posibles protocolos:

  

- MQTT

- WebSocket

  

------------------------------------------------------------------------

  

# 7. Arquitectura de datos

  

## Base de datos principal

  

Supabase PostgreSQL.

  

Almacena:

  

- Ingredientes

- Recetas

- Preferencias de usuario

- Inventario

  

------------------------------------------------------------------------

  

## Base de datos vectorial

  

Se usa para recuperación semántica.

  

Opciones recomendadas:

  

- pgvector (Supabase)

- Pinecone

- Weaviate

  

------------------------------------------------------------------------

  

## Esquema de datos (simplificado)

  

### Recetas

  

Campo         Tipo

------------- -----------

id            UUID

name          TEXT

description   TEXT

created_at    TIMESTAMP

  

### Ingredientes

  

Campo         Tipo

------------- -----------

id            UUID

name          TEXT

unit          TEXT

created_at    TIMESTAMP

  

### IngredientesPorReceta

  

Campo           Tipo

--------------- -------

recipe_id       UUID

ingredient_id   UUID

quantity        FLOAT

  

------------------------------------------------------------------------

  

# 8. Generación aumentada por recuperación (RAG)

  

Canal de conocimiento:

  

1. Ingesta de PDF de recetas

2. Análisis de documentos

3. Generación de fragmentos

4. Creación de embeddings

5. Almacenamiento vectorial

6. Recuperación semántica

7. Generación de respuestas con LLM

  

Arquitectura:

  

PDF → Analizador → Fragmentador → Embeddings → VectorDB → Recuperador → LLM

  

Componentes explícitos: **Analizador** (extracción de texto), **Fragmentador** (generación de fragmentos para búsqueda semántica), **Servicio de embeddings**, **Almacén vectorial**, **Recuperador** (búsqueda por similitud), **Servicio LLM** (generación de respuestas).

  

------------------------------------------------------------------------

  

# 9. Restricciones de rendimiento

  

Restricción crítica del sistema:

  

Tiempo máximo de respuesta de Alexa: **~8 segundos**

  

Bot WhatsApp (respuesta asíncrona): tiempo máximo de **1 segundo** para acusar recibo al usuario; el procesamiento del PDF continúa en segundo plano.

  

Estrategias de mitigación:

  

- Cachear consultas frecuentes

- Precalcular embeddings

- Minimizar llamadas síncronas al LLM

- Consultas directas a BD para inventario

  

------------------------------------------------------------------------

  

# 10. Consideraciones de seguridad

  

Requisitos de seguridad:

  

- Autenticación de la API

- Verificación del bot de WhatsApp

- Almacenamiento seguro de tokens

- Aislamiento de red para dispositivos robóticos

- Sanitización de entradas

- Control de acceso y autorización por rol o por usuario (qué acciones puede realizar cada identidad)

- Tratamiento de datos personales y de salud: confidencialidad, minimización de datos y cumplimiento normativo aplicable (p. ej. recetas del nutricionista)

  

------------------------------------------------------------------------

  

# 11. Observabilidad

  

Stack de observabilidad requerido:

  

Registro (logging):

  

- AWS CloudWatch

  

Métricas:

  

- Latencia de peticiones

- Tiempo de ejecución de agentes

  

Trazabilidad:

  

- OpenTelemetry

  

------------------------------------------------------------------------

  

# 12. Definición de hecho

  

Una funcionalidad se considera terminada cuando:

  

- Se pueden subir PDF de recetas por WhatsApp

- Las recetas se analizan y almacenan

- Alexa puede consultar recetas

- La coincidencia de ingredientes funciona

- Las listas de la compra se generan automáticamente

- *(Opcional)* El agente de comandos robóticos puede recibir una receta y enviar acciones al Raspberry Pi

  

Requisitos de ingeniería:

  

- Cobertura de pruebas unitarias ≥ 90%

- Pruebas de integración en verde

- Diagramas de arquitectura actualizados

- Revisión de código completada

  

------------------------------------------------------------------------

  

# 13. Mejoras futuras

  

Evolución prevista del sistema:

  

- Robot de cocina autónomo

- IA de planificación de comidas

- Integración con nevera inteligente

- Sensores de peso para ingredientes

- Paneles de nutrición
