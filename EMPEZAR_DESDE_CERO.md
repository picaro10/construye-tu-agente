# Empezar Desde Cero — Guía para Principiantes

> Si nunca has programado un agente, si no sabes qué es una API key,
> o si la terminal te da respeto — esta guía es para ti.
> Léela antes de tocar nada. En 15 minutos vas a entender todo lo que necesitas.

---

## Qué es esto y qué vas a construir

Vas a construir un **asistente personal de inteligencia artificial** que:
- Vive en un servidor tuyo (no depende de nadie)
- Te responde por Telegram desde tu móvil
- Recuerda lo que le dices
- Puede buscar en internet, leer tus emails, generar documentos, ponerte recordatorios
- Y puedes añadirle las funciones que quieras

No necesitas saber programar de memoria. Vas a **hablar con Claude** (una IA) y Claude va a escribir el código por ti. Tu trabajo es guiarlo con los prompts de esta guía, entender qué está pasando, y corregir cuando algo falle.

---

## Qué es Claude y cómo se usa

**Claude** es una inteligencia artificial creada por Anthropic. Es como ChatGPT pero (para este proyecto) mejor con código.

Tienes dos formas de usarlo:

### Opción A: Claude Chat (lo más fácil para empezar)
- Entra en [claude.ai](https://claude.ai)
- Crea una cuenta
- Abre un chat nuevo
- Pega el prompt del módulo que toque
- Claude te responde con el código

### Opcion B: Claude Code (lo recomendado)
- Es Claude pero en tu terminal (la pantalla negra donde escribes comandos)
- Tiene acceso directo a tus archivos — puede crear, editar, y ejecutar código
- Es más rápido y comete menos errores porque ve tu proyecto real
- Para instalarlo: [claude.ai/code](https://claude.ai/code)

**Para esta guía, cualquiera de las dos vale.** Si nunca has usado una terminal, empieza con Claude Chat. Cuando te sientas cómodo, pasa a Claude Code.

---

## Qué necesitas instalar (antes de empezar con los módulos)

### 1. Una terminal
- **Windows**: Instala [Windows Terminal](https://aka.ms/terminal) desde la Microsoft Store
- **Mac**: Ya tienes una — busca "Terminal" en Spotlight
- **Linux**: Ya sabes dónde está

### 2. Node.js (el motor que ejecuta el código)
- Ve a [nodejs.org](https://nodejs.org)
- Descarga la versión **22 o superior** (la que diga "Current")
- Instálalo como cualquier programa
- Verifica que funciona abriendo la terminal y escribiendo:
  ```
  node --version
  ```
  Si te dice `v22.algo` o superior, estás bien

### 3. Un editor de código
- Recomendado: [Visual Studio Code](https://code.visualstudio.com/) (gratis)
- Es donde vas a ver y editar los archivos que Claude genere
- No es obligatorio pero te va a hacer la vida mucho más fácil

### 4. Git (para gestionar tu código)
- **Windows**: Descarga desde [git-scm.com](https://git-scm.com/)
- **Mac**: Escribe `git` en la terminal y te ofrece instalarlo
- **Linux**: `sudo apt install git` o `sudo dnf install git`

### 5. Un bot de Telegram
- Abre Telegram en tu móvil
- Busca `@BotFather` (es un bot oficial de Telegram)
- Escríbele `/newbot`
- Sigue los pasos — te pide nombre y username para tu bot
- Te da un **token** (un texto largo). Guárdalo, lo vas a necesitar
- Para saber tu **ID de Telegram**: busca el bot `@userinfobot`, escríbele cualquier cosa, y te dice tu ID numérico

### 6. Una API key de un LLM
Necesitas una "llave" para que tu agente pueda pensar. Elige UNA:

| Proveedor | Cómo conseguirla |
|-----------|-----------------|
| **OpenAI** | Crea cuenta en [platform.openai.com](https://platform.openai.com), ve a API Keys, crea una nueva |
| **Anthropic** | Crea cuenta en [console.anthropic.com](https://console.anthropic.com), ve a API Keys |
| **Google** | Crea cuenta en [aistudio.google.com](https://aistudio.google.com), genera una API key |

Las tres funcionan. OpenAI es la más popular. Anthropic (Claude) es la que usamos para construir. Google es la más barata para empezar.

> **Importante**: Las API keys cuestan dinero por uso. No mucho (unos centavos por conversación), pero no es gratis. Pon un límite de gasto en tu cuenta para no llevarte sustos.

### 7. Un servidor (VPS) — esto puede esperar
Para el deploy final necesitas un servidor. Pero para desarrollo puedes usar tu propio ordenador. No compres nada hasta que tengas el agente funcionando en local.

---

## Glosario — Palabras que vas a ver mucho

| Término | Qué significa |
|---------|--------------|
| **API** | Una puerta para que programas hablen entre sí. Cuando tu agente "llama a OpenAI", está usando su API. |
| **API Key** | La llave que abre esa puerta. Sin ella, no te dejan pasar. Es secreta — no la compartas nunca. |
| **Terminal / Consola** | La pantalla negra donde escribes comandos. No muerde. |
| **npm** | El "instalador de paquetes" de Node.js. Cuando escribes `npm install`, descarga las librerías que tu proyecto necesita. |
| **.env** | Un archivo donde guardas tus secretos (API keys, contraseñas). NUNCA lo subas a internet. |
| **Docker** | Un programa que crea "contenedores" — como mini-servidores aislados. Lo usamos para la base de datos. |
| **PostgreSQL** | Una base de datos. Donde tu agente guarda mensajes, memorias, etc. |
| **Redis** | Una base de datos ultra-rápida que vive en memoria. Para cache y recordatorios. |
| **TypeScript** | El lenguaje en el que está escrito el agente. Es JavaScript pero con tipos (más seguro). |
| **VPS** | Virtual Private Server. Un ordenador en la nube que alquilas por ~5€/mes para que tu agente viva ahí 24/7. |
| **Git** | Sistema para guardar versiones de tu código. Como un "historial de cambios" infinito. |
| **GitHub** | Una web donde subes tu código con Git. Como Google Drive pero para programadores. |
| **LLM** | Large Language Model. El "cerebro" de la IA (GPT-4, Claude, Gemini...). |
| **Token** | Una unidad de texto para el LLM. Aproximadamente 1 token = 1 palabra. Los LLM cobran por tokens. |
| **Prompt** | El texto que le das a la IA para que haga algo. Los prompts de esta guía son las instrucciones para Claude. |
| **Skill** | Una habilidad de tu agente. "Buscar en internet" es un skill. "Leer emails" es otro. Son módulos independientes. |
| **Tool** | Una herramienta dentro de un skill. El skill "email" tiene tools como "leer email", "enviar email", etc. |
| **Webhook** | Una URL que recibe datos automáticamente cuando algo pasa. Telegram usa webhooks para avisarte de mensajes. |
| **Deploy** | Poner tu código a funcionar en un servidor real, accesible 24/7. |

---

## Cómo usar los prompts de la guía

### Paso a paso

1. **Abre Claude** (chat o Claude Code)
2. **Copia el prompt** del módulo que toque (empieza por el 1.1, en orden)
3. **Pégalo en Claude** y envíalo
4. **Claude te genera código** — varios archivos con explicaciones
5. **Crea los archivos** en tu proyecto con el código que Claude te da
6. **Prueba que funciona** antes de pasar al siguiente módulo
7. **Si algo falla**, pregúntale a Claude (ver sección siguiente)
8. **Cuando funcione**, pasa al siguiente prompt

### Reglas importantes

- **Ve en orden.** El módulo 2 necesita el 1. El 3 necesita el 1 y el 2. No te saltes nada.
- **No pases al siguiente hasta que el actual funcione.** Si el módulo 1 no arranca, no metas el 2.
- **Lee lo que Claude genera.** No copies a ciegas. Aunque no entiendas todo, léelo. Poco a poco irás pillando qué hace cada cosa.
- **Personaliza los prompts.** Donde dice `[TU_NOMBRE]`, pon el nombre que quieras para tu agente.

---

## Cómo hablar con Claude cuando algo falla

Esto es lo más importante de esta guía. **Todo va a fallar en algún momento.** Es normal. Lo que importa es cómo le preguntas a Claude para que te ayude.

### La regla de oro: DALE CONTEXTO

No hagas esto:
```
No funciona
```

Haz esto:
```
Al ejecutar npm run dev me sale este error:

[pega aquí el error completo de la terminal]

Estoy en el módulo 1.5 de la guía. Acabo de crear el bot de Telegram.
El archivo que da error es src/channels/telegram/bot.ts
```

### Plantillas para pedir ayuda

**Cuando tienes un error:**
```
Me sale este error al [lo que estabas haciendo]:

[pega el error COMPLETO — no resumas, no recortes]

El archivo es [nombre del archivo].
Estoy siguiendo el módulo [número] de una guía de construcción de agente IA.
```

**Cuando algo no hace lo que esperas:**
```
Ejecuto [comando] y esperaba que pasara [esto],
pero en vez de eso pasa [esto otro].

El código relevante es este:
[pega el trozo de código]
```

**Cuando no entiendes algo:**
```
En este código:
[pega el trozo]

No entiendo qué hace [la parte que no entiendes].
Explícamelo como si fuera la primera vez que lo veo.
```

**Cuando Claude genera código que no sabes dónde poner:**
```
Me has generado este código pero no sé en qué archivo va
ni dónde lo pongo dentro del proyecto. ¿Puedes decirme
la ruta exacta del archivo y qué archivos tengo que modificar?
```

### Trucos que te van a salvar

1. **Pega el error completo.** La terminal te da 20 líneas de error — pégalas TODAS. Claude lee rápido, tú no tienes que filtrar.

2. **Pregunta "por qué" además de "cómo".** No solo preguntes cómo arreglarlo, pregunta por qué falló. Así aprendes.

3. **Si Claude te da una solución y no funciona, dile.** "He probado lo que me dijiste y ahora me sale este otro error: [error]". Claude ajusta.

4. **No tengas miedo de preguntar cosas "tontas".** Claude no te juzga. Pregunta qué es un `import`, qué hace `async`, por qué hay un `.` antes de `env`. Todo vale.

5. **Si te pierdes, pídele que te haga un resumen.** "Resúmeme qué archivos tengo que tener creados hasta ahora y qué hace cada uno."

6. **Un chat por módulo.** Si el chat se hace muy largo y Claude empieza a confundirse, abre uno nuevo y dile: "Estoy construyendo un agente IA personal en TypeScript. Voy por el módulo [X]. Estos son los archivos que tengo: [lista]. Necesito ayuda con [problema]."

---

## Errores comunes y cómo resolverlos

### "command not found: node"
Node.js no está instalado o la terminal no lo encuentra. Cierra la terminal, ábrela de nuevo, y escribe `node --version`. Si no funciona, reinstala Node.js.

### "command not found: npm"
Mismo que arriba — npm viene con Node.js.

### "EACCES: permission denied"
En Mac/Linux, a veces necesitas permisos. Prueba con `sudo` delante del comando. Si es un npm install, mejor configura npm para que no necesite sudo: pregúntale a Claude "cómo configuro npm sin sudo en [tu sistema operativo]".

### "Cannot find module '...'"
Te falta una dependencia. Ejecuta `npm install` en la carpeta del proyecto.

### "ECONNREFUSED" (al conectar a la base de datos o Redis)
PostgreSQL o Redis no están arrancados. Si usas Docker: `docker compose up -d`. Si los instalaste directo: busca cómo arrancarlos en tu sistema.

### "Invalid API key" o "Authentication error"
Tu API key está mal. Revisa el archivo `.env`, verifica que no haya espacios extra, y que la key sea la correcta.

### El bot de Telegram no responde
- Verifica que el token es correcto en `.env`
- Verifica que tu ID de Telegram es el correcto en `TELEGRAM_ADMIN_ID`
- Verifica que el agente está corriendo sin errores en la terminal

---

## Cómo verificar que cada módulo funciona

Después de cada módulo, haz estas comprobaciones:

### Tras el Módulo 1 (La Base)
```bash
npm run dev
```
- La terminal debería decir algo como "Agente arrancado. Skills: 1, Tools: 2"
- Escríbele algo al bot en Telegram
- Debería responderte (aunque sea algo básico)
- Si dice "No autorizado" a alguien que no seas tú, la seguridad funciona

### Tras el Módulo 2 (Memoria)
- Dile al bot: "Recuerda que mi color favorito es el azul"
- Espera unos segundos
- Dile: "Cuál es mi color favorito?"
- Si se acuerda, la memoria funciona

### Tras el Módulo 3 (Web Search)
- Dile: "Busca las últimas noticias sobre inteligencia artificial"
- Debería devolverte resultados reales de internet

### Tras el Módulo 7 (Reminders)
- Dile: "Recuérdame en 2 minutos que esto funciona"
- Espera 2 minutos
- Debería llegarte un mensaje proactivo en Telegram

---

## Cuánto tiempo lleva esto

Siendo realista:

| Tu nivel | Tiempo estimado |
|----------|----------------|
| Sabes TypeScript/Node | 1-2 días para todo |
| Sabes programar pero no TypeScript | 3-5 días, preguntándole mucho a Claude |
| Primer proyecto serio | 1-2 semanas, con paciencia |

No es una carrera. Si tardas un mes y al final tienes tu agente funcionando, has ganado.

---

## Recursos útiles

- [Node.js docs](https://nodejs.org/docs/latest/api/) — Documentación oficial de Node
- [TypeScript en 5 minutos](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) — Si nunca tocaste TypeScript
- [Telegram Bot API](https://core.telegram.org/bots/api) — Documentación de bots de Telegram
- [Vercel AI SDK](https://sdk.vercel.ai/docs) — El SDK que usa tu agente para hablar con el LLM

---

## Una cosa más

Vas a tener momentos de frustración. Cosas que no funcionan, errores que no entiendes, horas perdidas en algo que era un punto y coma mal puesto.

Es normal. Le pasa a todo el mundo. Le pasó al que escribió esta guía.

La diferencia entre alguien que construye algo y alguien que no, no es el talento — es la paciencia de seguir preguntando hasta que funciona.

Tienes a Claude ahí. Pregúntale todo. Las veces que haga falta.

Ahora ve al [Módulo 1](./CONSTRUYE_TU_AGENTE.md#m%C3%B3dulo-1--la-base-scaffold--config--skill-system--agent--telegram) y empieza a construir.

---

*Si algo de esta guía no se entiende, abre un issue y lo mejoramos entre todos.*
