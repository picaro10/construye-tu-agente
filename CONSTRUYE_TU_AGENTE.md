# Construye Tu Propio Agente IA Personal — Guía de Prompts

> Extraído de la construcción real de ORION (20K+ LOC, 233 tools, 32 skills)
> por Lex @ LatenciaTech — hecho 100% con Claude
>
> Esto no es un curso. No es un wrapper de ChatGPT. No es humo.
> Son los prompts reales para que construyas un agente IA personal que hace cosas de verdad.
> Lo que hagas después con él depende de tu imaginación.

---

## Antes de empezar

**Qué vas a necesitar:**
- Una cuenta de Claude (claude.ai) — recomendado Claude Opus o Sonnet
- Un VPS o máquina donde correr tu agente (un servidor barato de Hetzner, DigitalOcean, lo que tengas)
- Una API key de un LLM (OpenAI, Anthropic, Google... una sola, la que prefieras)
- Node.js 22+ instalado
- PostgreSQL y Redis (los instalamos durante el proceso)
- Un bot de Telegram (lo creas en 2 minutos con @BotFather)
- Ganas de construir algo que sea tuyo

**Qué vas a conseguir:**
- Un asistente IA que vive en tu servidor, privado, solo tuyo
- Con sistema de skills modular — puedes crear los que quieras
- Con memoria semántica — recuerda lo que hablasteis
- Que te busca noticias, te lee emails, te genera PDFs, te crea contenido, te pone recordatorios
- Todo desde Telegram, con tu móvil
- Más potente que cualquier "agente" de n8n o wrapper que venden por ahí

**Cómo usar esta guía:**
Cada módulo es un prompt que le pegas a Claude (idealmente en Claude Code, pero también vale Claude chat). Hazlo en orden. Después de cada prompt, Claude te va a generar código — revísalo, pregúntale dudas, itéralo. Esto no es copiar y pegar — es construir hablando.

---

## MÓDULO 1 — La Base (Scaffold + Config + Skill System + Agent + Telegram)

> Este es el corazón. Sin esto no tienes nada. Con esto ya tienes más que el 90% de los "agentes" que venden por ahí.

### Prompt 1.1 — Scaffold del proyecto

```
Necesito que me ayudes a crear la base de un asistente personal de IA en TypeScript.
El proyecto se llama [TU_NOMBRE] y es un agente privado — un solo usuario, máximo poder.
No es un SaaS, no es para vender. Es mi herramienta personal.

Créame el scaffold del proyecto con:

- package.json con las dependencias: typescript, tsx (para desarrollo), pino (logger),
  zod (validación), ioredis, drizzle-orm, pg (postgres), @ai-sdk/openai (o el provider
  que yo use), ai (vercel ai sdk), grammy (telegram). DevDeps: vitest, biome, @types/node
- tsconfig.json estricto (strict: true, ESM, target ES2022)
- biome.json para linting
- .env.example con las variables que va a necesitar:
  - DATABASE_URL (postgres)
  - REDIS_URL
  - LLM_API_KEY (la key del LLM que use)
  - LLM_MODEL (el modelo, ej: gpt-4.1 o claude-sonnet-4)
  - TELEGRAM_BOT_TOKEN
  - TELEGRAM_ADMIN_ID (mi user ID de Telegram, para whitelist)
  - ENCRYPTION_KEY (32 chars para encriptar datos sensibles)
- .gitignore (node_modules, dist, .env, data/)
- Estructura de carpetas:
  src/
    config/      → env.ts (validación Zod de todas las env vars), constants.ts
    core/        → aquí irá el agent, session manager, skill system
    skills/      → cada skill en su carpeta
    channels/    → telegram, y en futuro otros
    llm/         → adaptador del LLM
    db/          → schema y cliente de base de datos
    utils/       → logger (pino con redaction de campos sensibles como password, token, key)
    security/    → prompt firewall básico
  tests/
  data/          → directorio para archivos generados (PDFs, imágenes, etc)

El config/env.ts debe validar TODAS las variables con Zod y fallar al arrancar si falta alguna
obligatoria. Esto es importante: si algo no está configurado, que no arranque.

Crea también una jerarquía de errores tipada:
  AgentError (base) → LLMError, ChannelError, SkillError, SecurityError

Dame todo el código, archivo por archivo.
```

### Prompt 1.2 — Base de datos y Redis

```
Ahora necesito la capa de persistencia.

Base de datos con Drizzle ORM + PostgreSQL. Necesito estas tablas iniciales:

- messages: historial de conversaciones (role, content, channel, tokens_used, model, timestamps)
- memories: memorias semánticas (content, embedding vector, category, confidence, timestamps)
- conversation_summaries: resúmenes automáticos (summary, topics[], tools_used[], message_count)
- preferences: configuración del usuario (key/value, con cache en Redis)
- skill_logs: registro de ejecución de skills (skill, tool, input, output, duration_ms, success)
- audit_log: log de auditoría (action, details, ip, severity)

Crea:
- src/db/schema.ts con todas las tablas en Drizzle
- src/db/client.ts con connection pool
- src/db/migrate.ts para correr migraciones
- src/utils/redis.ts con:
  - Cliente ioredis con reconnexión automática (backoff exponencial, nunca dejar de reintentar)
  - Health check
  - Helper isRedisReady() para que otros módulos verifiquen antes de usar

Para las migraciones usa drizzle-kit. Configura el drizzle.config.ts.

Importante: el Redis debe reconectar SIEMPRE. Nada de retryStrategy que devuelva null
después de X intentos. En producción, Redis puede caer 5 minutos y volver — el agente
tiene que sobrevivir a eso.
```

### Prompt 1.3 — El Sistema de Skills (esto es lo que importa)

```
Ahora lo más importante: el sistema de skills.

Esto es lo que diferencia a mi agente de un chatbot. Cada skill es un módulo independiente
que tiene herramientas (tools) que el agente puede usar. Quiero que sea extensible —
que yo pueda crear un skill nuevo en 5 minutos y el agente lo use automáticamente.

Necesito:

1. Una clase abstracta BaseSkill con:
   - name: string (identificador único)
   - description: string (qué hace el skill)
   - version: string
   - keywords: string[] (palabras clave para activar el skill, en español e inglés)
   - tools: array de herramientas, cada una con:
     - name: string (ej: "web_search")
     - description: string (esto es lo que el LLM lee para decidir si la usa)
     - inputSchema: JSON Schema del input (Zod convertido a JSON Schema)
     - execute: función async que recibe el input validado y devuelve resultado
   - onLoad(): para inicialización async
   - onUnload(): para cleanup

2. Un SkillManifest type que describa el skill sin instanciarlo

3. Un SkillRegistry que:
   - Registra skills por nombre
   - Devuelve la lista de skills registrados
   - Ejecuta un tool de un skill dado
   - Puede descargar un skill (unload)
   - Valida que no haya nombres duplicados

4. Un SkillSelector inteligente que dado un mensaje del usuario:
   - Analiza keywords del mensaje (español e inglés)
   - Selecciona máximo 3-4 skills relevantes para ese mensaje
   - Devuelve máximo 12-15 tools en total (no enviar las 100+ al LLM, eso es tirar dinero)
   - Tiene un sistema de scoring: keyword match + pattern match
   - SIEMPRE incluye el skill "system" (info básica) como fallback

Esto es clave: el selector NO le manda todas las herramientas al LLM en cada turno.
Solo las relevantes. Esto ahorra tokens y mejora la calidad de las respuestas.

Crea los archivos en src/core/:
- base-skill.ts
- skill-registry.ts
- skill-selector.ts

Y crea un primer skill de ejemplo — "system" — en src/skills/system/index.ts con 2 tools:
- system_status: devuelve CPU, RAM, disco, uptime del servidor
- system_info: devuelve versión del agente, skills registrados, uptime del proceso

Así tengo algo que funciona para probar.
```

### Prompt 1.4 — El Agente (el cerebro)

```
Ahora el OrionAgent — el cerebro que conecta todo.

Necesito un agent core en src/core/agent.ts que haga esto:

1. Recibe un mensaje del usuario (texto, canal de origen, session ID)
2. Carga el historial de la conversación (últimos 50 mensajes de la DB)
3. Usa el SkillSelector para elegir los skills relevantes para este mensaje
4. Construye el prompt con:
   - System prompt (personalidad del agente, instrucciones)
   - Historial de conversación
   - Las tools seleccionadas para este turno
5. Llama al LLM con el Vercel AI SDK usando tool calling nativo
6. Si el LLM quiere usar una tool:
   - La ejecuta via SkillRegistry
   - Le devuelve el resultado al LLM
   - El LLM puede encadenar más tools (maxSteps: 5)
7. Guarda el mensaje del usuario y la respuesta en la DB
8. Devuelve la respuesta al canal

Usa el Vercel AI SDK con tool() nativo — NO hagas hacks de meter tool results como
mensajes del usuario. El SDK tiene generateText() con tools y maxSteps que maneja
el loop automáticamente.

El system prompt ponlo en src/config/prompts/base.ts. Escríbelo en ESPAÑOL.
Instrucciones importantes para el prompt:
- El agente se comunica en español por defecto
- Tiene acceso real a herramientas y DEBE usarlas cuando sean relevantes
- No debe inventar datos — si no sabe algo, lo busca
- Debe ser directo, útil, sin relleno

El adaptador del LLM en src/llm/provider.ts:
- Usa el Vercel AI SDK (@ai-sdk/openai, @ai-sdk/anthropic, o @ai-sdk/google según
  lo que haya configurado en .env)
- Un solo provider está bien para empezar, lo que el usuario configure
- Retry básico: si falla, reintentar 1 vez con 1s de delay
```

### Prompt 1.5 — Canal de Telegram

```
Ahora conectamos todo a Telegram para que el agente tenga interfaz.

Necesito src/channels/telegram/bot.ts usando grammY:

- Arranca el bot con el TELEGRAM_BOT_TOKEN del .env
- Whitelist de admin: solo responde al TELEGRAM_ADMIN_ID (mi ID de Telegram).
  Cualquier otro usuario que escriba recibe "No autorizado." y se ignora.
  Si TELEGRAM_ADMIN_ID está vacío, el bot NO arranca (fail-closed, no fail-open).
- Maneja mensajes de texto: los pasa al OrionAgent y devuelve la respuesta
- Si la respuesta es muy larga (>4096 chars), la divide en múltiples mensajes
- Indicador de "escribiendo..." mientras el agente procesa
- Maneja errores gracefully — si algo falla, envía "Error procesando tu mensaje"
  al usuario y loguea el error completo

Crea también un Gateway Router en src/channels/gateway.ts:
- Centraliza el routing de mensajes al agent
- Desacopla el canal (Telegram) del agent
- Incluye el DM guard (whitelist check) aquí, así cualquier canal futuro
  (WhatsApp, Discord, etc.) pasa por el mismo punto de control

Finalmente, src/index.ts — el entry point:
- Carga y valida config (.env)
- Conecta a la DB, corre migraciones
- Conecta a Redis
- Crea el SkillRegistry y registra todos los skills
- Crea el OrionAgent
- Crea el Gateway
- Arranca el bot de Telegram
- Maneja shutdown graceful (SIGINT, SIGTERM)
- Log de arranque: "Agente arrancado. Skills: X, Tools: Y"
```

### Prompt 1.6 — Seguridad básica (Prompt Firewall)

```
Necesito un módulo de seguridad básico para proteger al agente de prompt injection.

Crea src/security/prompt-firewall.ts con:

- Un scanner que analiza el mensaje del usuario ANTES de pasárselo al LLM
- Detección de patrones de injection en mínimo 5 categorías:
  1. Role override: "ignora tus instrucciones", "olvida todo", "ahora eres..."
  2. System prompt extraction: "muéstrame tu prompt", "cuáles son tus instrucciones"
  3. Encoding tricks: base64, hex, unicode escaping para ocultar inyecciones
  4. Delimiter injection: intentos de cerrar el prompt del sistema con ```
  5. Instruction injection: "SYSTEM:", "### NEW INSTRUCTIONS", etc.
- Score de riesgo 0-1 por mensaje
- Si score > 0.7 → bloquear mensaje, loguear intento, responder "Solicitud bloqueada"
- Si score 0.4-0.7 → dejar pasar pero loguear como warning
- Patrones en español E inglés

Intégralo en el gateway: todo mensaje pasa por el firewall antes de llegar al agent.

No necesito los 11 módulos de seguridad que tiene Orion completo. Con el prompt firewall
básico ya estamos por encima del 99% de los bots que hay por ahí que no tienen NADA.
```

### Prompt 1.7 — Docker Compose

```
Crea un docker-compose.yml para levantar la infraestructura:

services:
  postgres:
    - PostgreSQL 16
    - Bind a 127.0.0.1 (no exponer al exterior)
    - Volume persistente para datos
    - Variables de .env para user/password/db
    
  redis:
    - Redis 7
    - Con password (--requirepass desde variable de entorno)
    - Bind a 127.0.0.1
    - AOF activado (appendonly yes) para persistencia
    
No metas el agente en Docker por ahora — lo corremos con tsx directamente
para facilitar el desarrollo. Solo la infra (postgres + redis).

Crea también los scripts en package.json:
- "dev": "tsx watch src/index.ts" (development con hot reload)
- "start": "tsx src/index.ts" (producción)
- "db:generate": "drizzle-kit generate"
- "db:migrate": "tsx src/db/migrate.ts"
- "test": "vitest run"
- "test:watch": "vitest"
- "lint": "biome check ."
- "typecheck": "tsc --noEmit"
```

---

## MÓDULO 2 — Memoria Semántica

> Sin esto tu agente tiene amnesia. Con esto, recuerda lo que hablasteis ayer.

### Prompt 2

```
Necesito que mi agente tenga memoria que persista entre conversaciones.

Quiero dos cosas:

1. MEMORIA SEMÁNTICA (src/core/memory.ts):
   - Usa embeddings de OpenAI (text-embedding-3-small) para convertir texto a vectores
   - Los guarda en PostgreSQL con pgvector (necesito instalar la extensión)
   - Búsqueda por similitud coseno — le preguntas "qué sé sobre servidores"
     y te devuelve las memorias más relevantes
   - Un skill "memory" con tools:
     - memory_save: guardar un hecho/dato manualmente
     - memory_search: buscar en la memoria por texto
     - memory_list: ver las últimas memorias guardadas
     - memory_delete: borrar una memoria por ID

2. AUTO-RESUMEN (src/core/summarizer.ts):
   - Cada 10 mensajes de conversación, automáticamente genera un resumen
     usando el LLM
   - Guarda el resumen en la tabla conversation_summaries con:
     - El resumen en texto
     - Los topics principales (array de strings)
     - Los tools que se usaron en esa ventana
   - Esto NO bloquea la conversación — se ejecuta async

3. MEMORIA PROACTIVA (src/core/memory-retriever.ts):
   - ANTES de cada respuesta, busca automáticamente en la memoria semántica
     usando el mensaje del usuario como query
   - También carga los últimos 3 resúmenes de conversación
   - Inyecta las memorias encontradas en el system prompt como contexto
   - El agente VE sus memorias sin necesidad de hacer tool call
   - Esto se ejecuta en PARALELO con la carga del historial (Promise.all)
   - Timeout de 3 segundos máximo — si la búsqueda tarda, se continúa sin ella

4. EXTRACCIÓN AUTOMÁTICA (src/core/entity-extractor.ts):
   - Después de cada respuesta (async, no bloquea), analiza el mensaje del usuario
   - Detecta hechos que vale la pena recordar:
     - "mi servidor es 10.0.0.1" → guarda
     - "trabajo en X" → guarda
     - "prefiero JSON" → guarda
   - NUNCA guarda passwords, API keys, tokens (blacklist de patrones sensibles:
     sk-*, ghp_*, Bearer, password, contraseña)
   - Antes de guardar, busca si ya existe algo similar (>85% similitud)
     para no duplicar

Para pgvector: dame el SQL para habilitar la extensión y un campo
vector(1536) en la tabla memories.

Importante: la memoria debe ser ÚTIL, no decoración. Si el agente no la usa
automáticamente antes de responder, no sirve para nada.
```

---

## MÓDULO 3 — Skill: Web Search

> Para que tu agente pueda buscar información actual, no solo lo que sabe el LLM.

### Prompt 3

```
Crea un skill de búsqueda web en src/skills/web-search/index.ts

Tools:
1. web_search: busca en DuckDuckGo (API HTML, no necesita key)
   - Input: query (string), max_results (number, default 5)
   - Hace fetch a DuckDuckGo, parsea los resultados
   - Devuelve: título, URL, snippet de cada resultado

2. web_fetch: obtiene el contenido de una URL
   - Input: url (string)
   - Hace fetch de la página, extrae el texto principal (quita HTML/scripts/styles)
   - Devuelve el texto limpio, truncado a 4000 chars
   - Protección SSRF: bloquear URLs que apunten a localhost, IPs privadas
     (10.x, 172.16-31.x, 192.168.x), metadata endpoints (169.254.169.254)

Keywords del skill: buscar, search, noticias, news, investigar, qué es, quién es,
dime sobre, find, lookup, web

Que las descriptions de los tools sean claras para el LLM:
"Busca información actual en internet. Úsalo cuando necesites datos recientes,
noticias, o información que no tengas."
```

---

## MÓDULO 4 — Skill: Email (Gmail)

> Para que puedas decirle "lee mis correos" o "envía un email a X" desde Telegram.

### Prompt 4

```
Crea un skill de email usando Gmail API en src/skills/email/index.ts

Prerrequisito: el usuario necesita configurar Gmail OAuth2:
- Crear proyecto en Google Cloud Console
- Habilitar Gmail API
- Crear credenciales OAuth2 (tipo Desktop)
- Obtener refresh token con scopes: https://mail.google.com/
- Poner GMAIL_CLIENT_ID, GMAIL_CLIENT_SECRET, GMAIL_REFRESH_TOKEN en .env

Añade esas variables al env.ts (opcionales — si no están, el skill no se registra).

Tools:
1. email_list: listar últimos emails (default 20, max 50)
   - Devuelve: subject, from, date, snippet, id
2. email_read: leer un email por ID
   - Devuelve: subject, from, to, date, body (texto plano)
3. email_send: enviar email
   - Input: to, subject, body
   - Usar nodemailer o Gmail API directamente
4. email_draft: crear borrador
   - Input: to, subject, body
5. email_search: buscar emails
   - Input: query (usa la sintaxis de búsqueda de Gmail)
   - Devuelve lista como email_list

Usa googleapis (npm) para la API. El token refresh se gestiona automáticamente.

Importante: si las credenciales no están configuradas, el skill simplemente
no se registra. No crashear el agente por un skill opcional.
```

---

## MÓDULO 5 — Skill: Content

> Para generar contenido: posts, traducciones, resúmenes.

### Prompt 5

```
Crea un skill de contenido en src/skills/content/index.ts

Tools:
1. content_generate: generar contenido
   - Input: type (tweet, linkedin_post, email, article, blog_post), topic, tone (opcional), length (opcional)
   - Usa el LLM del agente para generar el contenido
   - Devuelve el texto generado

2. content_translate: traducir texto
   - Input: text, target_language
   - Usa el LLM para traducir
   - Devuelve la traducción

3. content_summarize: resumir texto
   - Input: text, max_length (opcional)
   - Devuelve el resumen

4. content_rewrite: reescribir texto con otro tono/estilo
   - Input: text, style (formal, casual, técnico, creativo)
   - Devuelve el texto reescrito

Keywords: escribir, redactar, generar, crear post, tweet, traducir, translate,
resumir, summarize, reescribir, content

Estas tools usan el LLM internamente — eso está bien. El agente usa su propio
cerebro como herramienta para generar contenido.
```

---

## MÓDULO 6 — Skill: PDF

> Para generar documentos profesionales.

### Prompt 6

```
Crea un skill de generación de PDF en src/skills/pdf/index.ts usando PDFKit.

Tools:
1. pdf_create: crear un documento PDF
   - Input: title, sections (array de {heading, body}), author (opcional)
   - Genera un PDF con formato profesional: título, secciones con headers, footer con fecha
   - Guarda en data/pdf/ con nombre basado en el título

2. pdf_invoice: crear una factura
   - Input: from (nombre, dirección, NIF), to (nombre, dirección, NIF),
     items (array de {description, quantity, unit_price}), tax_rate (default 21%)
   - Calcula subtotal, IVA, total
   - Formato profesional con tabla de items
   - Guarda en data/pdf/

3. pdf_report: crear un reporte con tablas
   - Input: title, sections (array de {heading, body}),
     tables (array de {title, headers[], rows[][]})
   - Formato con tablas formateadas
   - Guarda en data/pdf/

Cada tool devuelve la ruta del archivo generado.

MUY IMPORTANTE: los JSON Schemas de cada tool deben estar COMPLETOS.
Si un campo es un array, DEBE tener "items" con la estructura del elemento.
Si un campo es un object, DEBE tener "properties" definidas.
Si el schema está incompleto, el Vercel AI SDK lo rechaza silenciosamente
y el LLM nunca ve la herramienta. Esto me costó horas de debugging.
```

---

## MÓDULO 7 — Skill: Reminders

> "Recuérdame en 2 horas que llame a Carlos" — y te avisa por Telegram.

### Prompt 7

```
Crea un skill de recordatorios en src/skills/reminder/index.ts

Usa Redis sorted sets para almacenar los reminders (key: timestamp, value: reminder data).

Tools:
1. reminder_add: crear un recordatorio
   - Input: message (qué recordar), when (cuándo)
   - El "when" debe entender formatos flexibles:
     - Relativos: "5m", "2h", "1d", "30min"
     - Español: "mañana", "pasado mañana", "en una hora", "esta noche"
     - Hora exacta: "14:30", "a las 8"
   - Guarda en Redis sorted set con el timestamp como score
   - Devuelve confirmación con la hora exacta del reminder

2. reminder_list: ver recordatorios pendientes
   - Devuelve todos los reminders futuros ordenados por fecha

3. reminder_delete: borrar un recordatorio por ID

Background checker (importante):
- Un setInterval cada 30 segundos que:
  - Consulta Redis por reminders cuyo timestamp ya pasó
  - Para cada uno, envía un mensaje PROACTIVO al canal de Telegram del admin
  - Borra el reminder de Redis tras enviarlo
  - Verifica isRedisReady() antes de consultar (no crashear si Redis no está)

Para enviar mensajes proactivos desde el reminder checker, necesito acceso
al bot de Telegram. Pasa una función sendNotification al skill desde index.ts
que use bot.api.sendMessage(adminId, mensaje).

Keywords: recordar, recuérdame, reminder, avísame, no olvides, remind me,
alarma, alerta
```

---

## MÓDULO 8 — Skill: File Operations

> Acceso al filesystem del servidor — leer, escribir, buscar archivos.

### Prompt 8

```
Crea un skill de operaciones con archivos en src/skills/file-ops/index.ts

Tools:
1. file_read: leer un archivo
   - Input: path
   - Devuelve el contenido (truncado a 10K chars si es muy grande)

2. file_write: escribir/crear un archivo
   - Input: path, content
   - Crea directorios intermedios si no existen

3. file_list: listar archivos en un directorio
   - Input: path, recursive (boolean, default false)
   - Devuelve: nombre, tamaño, tipo, fecha modificación

4. file_search: buscar archivos por nombre
   - Input: directory, pattern (glob o texto)
   - Busca recursivamente

5. file_info: información de un archivo
   - Input: path
   - Devuelve: tamaño, permisos, fecha creación/modificación, tipo

6. file_delete: borrar un archivo
   - Input: path
   - IMPORTANTE: esta tool debe tener requiresConfirmation: true
   - El agente debe pedir confirmación antes de borrar

SEGURIDAD — Path validation:
- Todas las rutas deben resolverse a paths dentro de un directorio permitido
  (el home del proceso o data/)
- Bloquear traversal (../../../etc/passwd)
- Bloquear acceso a: /etc, /var, /root, .env, node_modules
- Si la ruta apunta fuera del directorio permitido → error

Keywords: archivo, file, leer, escribir, crear, borrar, buscar, directorio,
carpeta, folder
```

---

## MÓDULO 9 — Generación de imágenes

> "Genera una imagen de un gato cyberpunk" — DALL-E desde Telegram.

### Prompt 9

```
Crea un skill de generación de imágenes en src/skills/image/index.ts

Usa DALL-E 3 de OpenAI (ya tienes la API key si usas OpenAI como LLM).

Tools:
1. image_generate: generar una imagen
   - Input: prompt (descripción de la imagen),
     size ("1024x1024" | "1792x1024" | "1024x1792", default "1024x1024"),
     quality ("standard" | "hd", default "standard"),
     style ("vivid" | "natural", default "vivid")
   - Llama a la API de OpenAI Images
   - Descarga la imagen y la guarda en data/images/ con nombre timestamp
   - Devuelve la ruta local del archivo

Si el usuario usa otro provider LLM que no sea OpenAI, necesitará igualmente
la OPENAI_API_KEY para las imágenes. Añade como variable opcional al .env.

Keywords: imagen, image, genera imagen, create image, dibujar, draw,
foto, picture, ilustración

Cuando devuelvas la ruta del archivo, el canal de Telegram debería poder
enviar la imagen como foto. Actualiza el gateway/telegram para que si la
respuesta del agent incluye una ruta a un archivo de imagen, lo envíe
como foto además del texto.
```

---

## MÓDULO 10 — Tests

> Porque si no lo testeas, no sabes si funciona.

### Prompt 10

```
Necesito tests con Vitest para los módulos críticos:

1. tests/unit/skill-selector.test.ts
   - Verifica que selecciona los skills correctos por keywords
   - Verifica el límite de max skills y max tools
   - Verifica que siempre incluye el skill system

2. tests/unit/skill-registry.test.ts
   - Registrar skill, ejecutar tool, desregistrar
   - Duplicados (debe fallar)
   - Tool que no existe (debe fallar)

3. tests/unit/prompt-firewall.test.ts
   - Mensajes normales → score bajo
   - Injection attempts → score alto
   - Patrones en español e inglés

4. tests/unit/config.test.ts
   - Variables válidas → ok
   - Variable faltante → error claro

5. tests/unit/memory.test.ts (si implementaste el módulo 2)
   - Entity extractor detecta hechos
   - Blacklist de secretos funciona
   - Deduplicación funciona

Corre con: npm test

No necesito 100% coverage. Necesito que los módulos CRÍTICOS estén cubiertos:
el skill system, el firewall, y la config. Si eso funciona, el resto se puede
iterar en producción.
```

---

## MÓDULO 11 — Deploy

> Ponerlo a correr en tu servidor de verdad.

### Prompt 11

```
Ayúdame a preparar el deploy en un VPS con Ubuntu.

Necesito:

1. Un script setup.sh que:
   - Instale Node.js 22 via nvm
   - Instale PostgreSQL 16 y cree el user y database
   - Instale la extensión pgvector en PostgreSQL
   - Configure PostgreSQL para escuchar solo en 127.0.0.1
   - Instale Redis y configure: password, bind 127.0.0.1, appendonly yes
   - Instale dependencias del proyecto (npm install)
   - Corra las migraciones de base de datos

2. Un archivo orion.service para systemd:
   - WorkingDirectory: donde esté el proyecto
   - ExecStart: con el PATH correcto de nvm
   - Restart=on-failure, RestartSec=5
   - EnvironmentFile apuntando al .env
   - Que se habilite al boot (enable)

3. Un script de backup:
   - pg_dump diario a las 3 AM (cron)
   - Retención de 7 días
   - Guarda en data/backups/

4. Si voy a exponer el dashboard o API web, configuración nginx:
   - Reverse proxy a localhost:3000
   - WebSocket support (headers Upgrade/Connection)
   - Let's Encrypt para HTTPS (certbot)

Cosas importantes del deploy que aprendí construyendo Orion:
- El .env de local NO es el .env del server. Revisa CADA variable.
- Si usas systemd con ProtectHome=read-only, los skills que escriben
  en ~/Documents van a fallar. Todo output debe ir dentro del WorkingDirectory/data/
- Si tienes un honeypot en el puerto 6379, Redis no va a conectar.
  Verifica que no haya conflictos de puertos.
- Google OAuth en modo "Testing" = refresh token caduca en 7 días.
  Hay que pasarlo a Production o regenerar semanalmente.
```

---

## Y después, ¿qué?

Con estos 11 módulos tienes un agente personal corriendo en tu servidor que:

- Responde por Telegram (solo a ti, nadie más)
- Tiene un sistema de skills extensible (puedes crear los que quieras)
- Recuerda lo que le dices (memoria semántica + auto-resumen)
- Busca información en internet
- Lee y envía tus emails
- Genera contenido (posts, traducciones, resúmenes)
- Genera PDFs profesionales y facturas
- Te pone recordatorios y te avisa proactivamente
- Genera imágenes con DALL-E
- Lee y escribe archivos en tu servidor
- Tiene protección contra prompt injection
- Corre como servicio en tu VPS con backups automáticos

**Esto ya es más que cualquier "agente IA" que venden en cursos de 300€.**

Y a partir de aquí, usa tu imaginación:

- ¿Quieres que audite la seguridad de tus servidores? Crea un skill.
- ¿Quieres que gestione tu calendario? Crea un skill.
- ¿Quieres que controle tu Docker? Crea un skill.
- ¿Quieres que hable por voz? Añade Whisper + TTS.
- ¿Quieres que responda por WhatsApp, Discord, Slack? Añade más canales.
- ¿Quieres que aprenda de sus errores? Construye un feedback loop.

El límite es tu creatividad. El skill system está diseñado para que cada
módulo nuevo sea independiente — no tienes que tocar el core.

---

## Lecciones del camino (las que me ahorraron horas)

1. **Los JSON Schemas de las tools deben estar COMPLETOS.** Si un array no tiene `items` o un object no tiene `properties`, el AI SDK lo rechaza en silencio y la herramienta desaparece. Sin errores, sin logs. Horas perdidas hasta descubrir esto.

2. **El idioma del system prompt determina el idioma de las respuestas.** Si le dices "responde en español" dentro de un prompt en inglés, va a mezclar idiomas. Escribe el prompt entero en el idioma que quieras.

3. **Las descriptions de las tools son instrucciones para el LLM.** Si el description dice "Public URL required", el LLM se va a negar a pasar un path local aunque el código lo soporte. Sé preciso en las descriptions.

4. **`retryStrategy` que devuelve `null` mata la conexión de Redis para siempre.** En producción, SIEMPRE reconectar con backoff exponencial.

5. **No envíes 100+ tools al LLM en cada turno.** Usa un selector inteligente. Máximo 4 skills, 15 tools. Ahorra tokens y mejora respuestas.

6. **La memoria sin auto-retrieval es decoración.** Si el LLM tiene que decidir buscar en la memoria, no lo va a hacer. Inyecta las memorias relevantes automáticamente antes de cada respuesta.

7. **Absorber > conectar.** Si puedes integrar la funcionalidad directamente en tu agente en vez de conectar un servicio externo, hazlo. Menos dependencias, menos puntos de fallo.

8. **Dos providers LLM estables > cinco inestables.** No metas 5 providers si no los necesitas. Uno bueno y un fallback es suficiente.

9. **systemd > nohup.** Un `nohup` no sobrevive reboot. systemd sí, y se reinicia si crashea.

10. **NUNCA guardes passwords o API keys en la memoria semántica.** Blacklist de patrones sensibles desde el día 1.

---

*Hecho con Claude. Si no es con Claude, no sale buen agente.*
*— Lex @ LatenciaTech*
