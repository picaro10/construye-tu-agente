# 🤖 Construye Tu Propio Agente IA Personal

### De cero a asistente inteligente en tu servidor — con prompts y Claude.

> No es un wrapper de ChatGPT. No es un bot de Telegram con `fetch`.
> Es un agente real con sistema de skills, memoria semántica, y herramientas que ejecutan acciones de verdad.


---

## ¿Qué es esto?

Una guía de **11 prompts en lenguaje natural** que le das a [Claude](https://claude.ai) para construir paso a paso un asistente IA personal que:

- 🧠 **Tiene memoria** — recuerda lo que hablasteis ayer (embeddings + pgvector)
- 🔧 **Usa herramientas reales** — lee tus emails, genera PDFs, busca en internet, crea imágenes
- ⏰ **Es proactivo** — te pone recordatorios y te avisa por Telegram
- 🛡️ **Se protege** — prompt firewall contra inyecciones
- 🧩 **Es extensible** — sistema de skills modular, creas uno nuevo en 5 minutos
- 🔒 **Es privado** — corre en TU servidor, solo responde a TI

Todo desde tu Telegram. Sin SaaS. Sin suscripción. Sin humo.

---

## ¿Por qué es diferente a lo que hay por ahí?

Porque tiene **sistema de skills real**. Eso es lo que separa un juguete de una herramienta.

La mayoría de "agentes IA" que se venden son esto:

```
while (true) {
  mensaje = leer_telegram()
  respuesta = llamar_openai(mensaje)
  enviar_telegram(respuesta)
}
```

Un loro con API key. Sin skills. Sin memoria. Sin herramientas reales.

Lo que tú vas a construir tiene esto:

```
Usuario → Gateway (seguridad) → Skill Selector → Agent Core → LLM con Tool Calling
                                      ↓
                          Skill Registry (módulos independientes)
                          ├── web-search (busca noticias)
                          ├── email (lee/envía Gmail)
                          ├── pdf (genera documentos)
                          ├── memory (recuerda todo)
                          ├── reminders (te avisa)
                          ├── content (genera posts)
                          ├── image (crea imágenes)
                          ├── file-ops (gestiona archivos)
                          └── [tu skill aquí]
```

Y el selector inteligente no le manda 100 herramientas al LLM — elige las 3-4 relevantes para cada mensaje. Ahorra tokens y mejora respuestas.

---

## ¿Qué necesito?

| Requisito | Detalle |
|-----------|---------|
| **Claude** | Cuenta en [claude.ai](https://claude.ai) (recomendado Opus o Sonnet) |
| **VPS** | Cualquier servidor Linux (Hetzner, DigitalOcean, lo que tengas) |
| **API Key LLM** | OpenAI, Anthropic, o Google — una sola, la que prefieras |
| **Node.js 22+** | Runtime |
| **Bot Telegram** | Lo creas en 2 min con @BotFather |
| **Ganas** | De construir algo que sea tuyo |

---

## Los 11 Módulos

| # | Módulo | Qué consigues |
|---|--------|---------------|
| 1 | **La Base** | Scaffold + Config + Skill System + Agent Core + Telegram + Seguridad + Docker |
| 2 | **Memoria** | Memoria semántica con embeddings + auto-resumen + retrieval proactivo + extracción de entidades |
| 3 | **Web Search** | Buscar información actual en internet |
| 4 | **Email** | Leer y enviar emails de Gmail desde Telegram |
| 5 | **Content** | Generar posts, traducir, resumir, reescribir |
| 6 | **PDF** | Crear documentos y facturas profesionales |
| 7 | **Reminders** | Recordatorios con notificación proactiva por Telegram |
| 8 | **File Ops** | Leer, escribir, buscar archivos en tu servidor |
| 9 | **Imágenes** | Generar imágenes con DALL-E |
| 10 | **Tests** | Tests de los módulos críticos con Vitest |
| 11 | **Deploy** | systemd + nginx + HTTPS + backups automáticos |

👉 **[Ver la guía completa con todos los prompts](./GUIA_COMPLETA.md)**

👉 **[Prompts individuales por módulo](./prompts/)**

---

## Cómo funciona

No es copiar y pegar. Es **construir hablando con Claude**.

1. Abres Claude (chat o Claude Code)
2. Le pegas el prompt del módulo 1
3. Claude te genera el código
4. Lo revisas, le preguntas dudas, iteras
5. Cuando funcione, pasas al módulo 2
6. Repites hasta tener tu agente completo

Cada prompt está escrito en lenguaje natural, como hablarías con un compañero. No son plantillas de ingeniería. Son conversaciones.

---

## La historia detrás

Estos prompts están extraídos de la construcción real de **ORION** — un asistente IA personal con +20K líneas de código, 233 herramientas, 32 skills, 5 canales, memoria semántica, voice calls en tiempo real, sistema de seguridad de 11 módulos, y un agente que aprende de sus propios errores.

ORION se construyó **hablando con Claude**. Sin prompts de ingeniería, sin plantillas. Preguntando "¿y si le meto un sistema de skills?" y Claude respondiendo "se puede, el mejor camino es X". Pieza por pieza, sesión por sesión.

Lo que comparto aquí es la base — lo suficiente para que tengas un agente personal más potente que cualquier wrapper de ChatGPT o bot de n8n. Lo que hagas después depende de tu imaginación.

---

## Lecciones del camino

> Estas no están en ningún tutorial. Son cicatrices reales.

1. **Los JSON Schemas incompletos matan herramientas en silencio.** Si un array no tiene `items`, el SDK lo descarta sin error. Horas perdidas.
2. **El idioma del system prompt = el idioma de las respuestas.** Prompt en inglés → respuestas mezcladas. Escríbelo en tu idioma.
3. **Las descriptions de los tools son instrucciones para el LLM.** Si dice "Public URL required", el LLM rechazará paths locales aunque el código los soporte.
4. **No envíes 100 tools al LLM.** Selector inteligente: max 4 skills, 15 tools por turno.
5. **La memoria sin auto-retrieval es decoración.** Inyéctala antes de cada respuesta, no esperes que el LLM decida buscar.
6. **Redis: `retryStrategy` que devuelve `null` = muerte.** Siempre reconectar con backoff.
7. **systemd > nohup.** Siempre.
8. **NUNCA guardes passwords en la memoria semántica.** Blacklist desde el día 1.
9. **Absorber > conectar.** Integrar directo > depender de servicio externo.
10. **Dos LLM providers estables > cinco inestables.**

---

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Runtime | Node.js 22 + TypeScript |
| LLM | Vercel AI SDK (compatible con OpenAI, Anthropic, Google) |
| Base de datos | PostgreSQL 16 + Drizzle ORM + pgvector |
| Cache | Redis 7 |
| Canal | Telegram (grammY) |
| PDF | PDFKit |
| Imágenes | DALL-E 3 (OpenAI API) |
| Validación | Zod |
| Tests | Vitest |
| Deploy | systemd + nginx + Let's Encrypt |

---

## Estructura del repo

```
├── README.md                ← Estás aquí
├── GUIA_COMPLETA.md         ← La guía con los 11 módulos en un solo documento
├── prompts/                 ← Prompts individuales por módulo (para copiar fácil)
│   ├── 01-base-scaffold.md
│   ├── 02-base-database.md
│   ├── 03-base-skills.md
│   ├── 04-base-agent.md
│   ├── 05-base-telegram.md
│   ├── 06-base-seguridad.md
│   ├── 07-base-docker.md
│   ├── 08-memoria.md
│   ├── 09-web-search.md
│   ├── 10-email.md
│   ├── 11-content.md
│   ├── 12-pdf.md
│   ├── 13-reminders.md
│   ├── 14-file-ops.md
│   ├── 15-imagenes.md
│   ├── 16-tests.md
│   └── 17-deploy.md
└── LICENSE
```

---

## Contribuir

¿Construiste un skill nuevo? ¿Mejoraste algún prompt? ¿Encontraste un error?

Abre un PR o un issue. Esto es de la comunidad.

---

## Licencia

MIT — Haz lo que quieras con esto. Úsalo, modifícalo, compártelo.
Solo no lo vendas como si fuera tuyo sin darle crédito, ¿vale?

---


