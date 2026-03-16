# Protocolo de Review --- Code Review Forense

## Proyecto: Voice‑Controlled Autonomous Food Management System

Este protocolo se utiliza para revisar cualquier código generado para el
sistema descrito en el **Architecture Brief del robot de gestión de
alimentos**.

Arquitectura del sistema:

Usuario → Alexa → Alexa Skill → AWS Lambda → Agent Orchestrator → AI
Agents → Supabase → Raspberry Pi → Robot OpenClaw

También incluye:

-   Bot de WhatsApp
-   Procesamiento de PDF
-   RAG con Vector DB
-   LLM APIs

------------------------------------------------------------------------

# 1. Alucinaciones

## Pregunta clave

¿Todo lo que importa el código existe realmente en el ecosistema
tecnológico usado?

La IA puede inventar:

-   librerías
-   funciones
-   endpoints
-   APIs completas

### Librerías esperadas en el proyecto

Python:

-   ask_sdk_core
-   supabase
-   openai
-   fastapi
-   pydantic
-   pgvector

### Qué verificar

**Imports y paquetes**

-   ¿El paquete existe en PyPI?
-   ¿La versión es compatible?
-   ¿Las funciones usadas existen realmente?

**APIs externas**

Verificar documentación oficial de:

-   OpenAI API
-   Supabase API
-   WhatsApp API / Twilio
-   AWS SDK

### Checklist

-   [ ] Todos los imports existen
-   [ ] No hay librerías ficticias
-   [ ] Las APIs usadas existen
-   [ ] Las versiones son compatibles

------------------------------------------------------------------------

# 2. Lógica de negocio

## Pregunta clave

¿La lógica implementa correctamente el comportamiento esperado del
sistema de recetas e inventario?

### Casos borde del sistema de recetas

  Caso                       Ejemplo
  -------------------------- -------------------------
  Receta sin ingredientes    Error
  Ingrediente sin cantidad   Rechazar
  Ingrediente duplicado      Consolidar
  Receta inexistente         Generar sugerencia
  Inventario vacío           Generar lista de compra

### Validaciones necesarias

-   ingredientes no negativos
-   unidades válidas
-   recetas con al menos un ingrediente
-   disponibilidad de ingredientes

### Checklist

-   [ ] Los casos borde están cubiertos
-   [ ] No hay cálculos incorrectos
-   [ ] Los inputs se validan
-   [ ] Los outputs coinciden con el brief

------------------------------------------------------------------------

# 3. Seguridad

## Pregunta clave

¿El código introduce riesgos de seguridad?

### Riesgos principales del sistema

**SQL Injection**

Siempre usar queries parametrizadas.

**Credenciales expuestas**

-   API Keys
-   tokens
-   passwords

Nunca deben aparecer en el código.

**Validación de inputs**

Especialmente:

-   PDFs recibidos
-   mensajes de WhatsApp
-   requests de Alexa

**Seguridad IoT**

Validar comandos enviados a:

Raspberry Pi / Robot OpenClaw

### Checklist

-   [ ] Queries parametrizadas
-   [ ] No hay credenciales en código
-   [ ] Inputs validados
-   [ ] Logs sin información sensible
-   [ ] Endpoints autenticados

------------------------------------------------------------------------

# 4. Respeto al Brief

## Pregunta clave

¿El código respeta la arquitectura definida?

Arquitectura obligatoria:

Alexa → Lambda → Agent Orchestrator → Agents → Supabase → Raspberry Pi

### Qué verificar

  Sección del brief    Qué revisar
  -------------------- --------------------------
  Stack                Python + AWS Lambda
  Arquitectura         Sistema desacoplado
  Contratos de datos   Input / Output definidos
  Agentes              Separación clara
  Storage              Supabase

### Señales de alerta

-   dependencias nuevas no aprobadas
-   hardcoding de endpoints
-   lógica de negocio mezclada con infraestructura
-   agentes acoplados

### Checklist

-   [ ] Stack coincide con el brief
-   [ ] Arquitectura desacoplada
-   [ ] Contratos de datos respetados
-   [ ] No hay dependencias nuevas
-   [ ] No hay hardcodeo

------------------------------------------------------------------------

# 5. Stack del Proyecto

Este punto revisa particularidades del stack del sistema.

## AWS Lambda

-   tiempo de ejecución \< 8s
-   manejo de errores
-   logs estructurados

## Supabase

-   RLS activado
-   migraciones versionadas
-   queries seguras

## AI Agents

-   agentes independientes
-   prompts versionados
-   fallback sin LLM

## RAG

Verificar:

-   chunking correcto
-   embeddings generados
-   retrieval funcional

## Raspberry Pi / Robot

-   comandos validados
-   no ejecución arbitraria

### Checklist

-   [ ] Lambda optimizada
-   [ ] Supabase seguro
-   [ ] Agentes desacoplados
-   [ ] RAG funcional
-   [ ] Robot protegido

------------------------------------------------------------------------

# Resultado del Review

  \#   Punto                Resultado   Detalle
  ---- -------------------- ----------- ---------
  1    Alucinaciones                    
  2    Lógica de negocio                
  3    Seguridad                        
  4    Respeto al brief                 
  5    Stack del proyecto               

------------------------------------------------------------------------

# Veredicto

-   [ ] El código pasa los 5 puntos
-   [ ] Se hicieron correcciones
-   [ ] Código listo para commit

Reviewer:

Fecha:
