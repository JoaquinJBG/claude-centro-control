# Mejoras del Centro de Control: investigación y propuesta (2026-11-02)

**El problema, con palabras del usuario:** "siento que pierdo control de los proyectos, porque las conversaciones
avanzan y si me has preguntado algo y otra sesión te habla, pues yo ya no lo veo. [...] El dashboard no lo miro
apenas, hay que mejorarlo."

**Conclusión en una línea:** las preguntas se pierden porque hoy solo viven en el hilo del chat. Hay que sacarlas a
una **bandeja con id fijo** que se repite arriba de cada respuesta y en el cuadro de siguiente paso, y que también
salga en un dashboard **en vivo** (base de datos, no archivos subidos a mano) con un **semáforo por jefe**.

---

## 1. Qué hace la gente (con enlaces)

### Herramientas para varias sesiones

| Herramienta | Qué es | Lo que interesa copiar |
|---|---|---|
| **Agent view de Claude Code** (`claude agents`), oficial | Lista en vivo de todas las sesiones en segundo plano | Estados fijos: **Trabajando · Necesita respuesta (amarillo) · Inactiva · Completada (verde) · Falló (rojo) · Parada (gris)**. Las que **necesitan respuesta van siempre arriba**, y la fila enseña **la pregunta literal**. Se contesta desde la lista sin abrir la sesión. Los permisos no se pueden contestar desde fuera: hay que abrir la sesión. [docs](https://code.claude.com/docs/en/agent-view) |
| **ccm** (gestor de atención) | tmux + panel para sesiones en paralelo | Prioridad fija **PERMISO > TRABAJANDO > INACTIVA**: el que espera sale arriba aunque llegue otra cosa. No escribe en una sesión que está esperando un permiso. [artículo](https://yohasebe.com/posts/2026-05-04-ccm-attention-manager/) |
| **ccmanager** | TUI de sesiones por worktree | Tres estados (inactiva, ocupada, esperando) y un *hook* que lanza un comando al cambiar de estado. [ficha](https://awesome.ecosyste.ms/projects/github.com%2Fkbwo%2Fccmanager) |
| **Claude Squad** | TUI, una sesión por worktree | Aislamiento por rama; no resuelve las preguntas perdidas. [repo](https://github.com/smtg-ai/claude-squad) |
| **Conductor** | App de Mac, worktrees + revisión de diff | "Ver quién trabaja y qué necesita atención" sin cambiar de ventana. Solo Mac. [comparativa](https://conductor.build/compare/claude-code) |
| **Vibe Kanban** | Tablero web por tareas | Columna **"En revisión" = el agente terminó y espera a una persona**. Es lo mismo que nuestro ✅. [docs](https://vibekanban.com/docs/core-features/monitoring-task-execution) |
| **Crystal** | Retirado en febrero de 2026 | — [comparativa](https://agentsroom.dev/fr/blog/best-multi-agent-coding-tools) |
| **Agent teams** de Claude Code (experimental) | Un líder con compañeros, lista de tareas compartida y buzón por agente | Cada tarea tiene un estado (pendiente, en curso o hecha) y un buzón por agente con entrega confirmada. Pero solo hay un equipo por sesión, no se pueden anidar, va por terminal y es experimental: **no sirve** para jefes de proyectos separados. [docs](https://code.claude.com/docs/en/agent-teams) |

Comparativas generales: [nimbalyst](https://nimbalyst-website.pages.dev/blog/best-session-managers-for-claude-code-and-codex/),
[orcy](https://orcy-website.pages.dev/compare), [crystl](https://crystl.dev/blog/desktop-app-gui-multiple-claude-code-sessions/).

### Bandejas de entrada y "que no se pierda la pregunta"

- **LangChain Agent Inbox**: cada interrupción de un agente es una **tarjeta** con las respuestas que admite
  (aceptar, editar, responder o ignorar). La tarjeta no desaparece hasta que la persona actúa.
  [repo](https://github.com/langchain-ai/agent-inbox) · [explicación](https://www.agentmail.to/insights/langchain-agent-inbox)
- **"Agent inbox" de Pushary**: reduce todo a tres estados, **hecho · atascado · necesita respuesta**, y manda un
  aviso al móvil. [hilo](https://www.producthunt.com/p/pushary/how-do-you-stay-aware-of-what-your-ai-coding-agents-are-doing?page=3)
- **Issues de Claude Code** que piden justo esto: saber *qué* sesión de varias espera al usuario, y no avisar
  mientras siguen trabajando agentes en segundo plano.
  [#36885](https://github.com/anthropics/claude-code/issues/36885) · [#45781](https://github.com/anthropics/claude-code/issues/45781)
- **Lección de "estado frente a status"**: un "esperando respuesta" se puede perder si el proceso se para, y el
  resumen de una línea lo escribe un modelo. Hay que **guardar la pregunta en cuanto aparece**, en un sitio propio.
  [artículo](https://www.besthub.dev/articles/claude-code-agent-state-vs-status-why-waiting-for-answer-loses-context-after-an-hour-65d373403247)

### Avisos y señales automáticas

- **Hooks de Claude Code** (oficial): `Notification` se dispara con `permission_prompt`, `idle_prompt`,
  `agent_needs_input` o `agent_completed`, y `Stop` al acabar cada turno (trae `last_assistant_message`). Todos
  llevan `session_id` y `cwd`. [docs](https://code.claude.com/docs/en/hooks)
- Paneles hechos con hooks: **ccmon** escribe un archivo de estado por evento ([repo](https://github.com/appaquet/ccmon));
  **Band** pasa a "necesita atención" con `Stop` ([docs](https://band-docs.pages.dev/docs/agent-status));
  **AgentsRoom** defiende que los hooks son la señal más fiable ([blog](https://agentsroom.dev/blog/claude-code-hook-events-lifecycle)).
- **Avisos al móvil**: ntfy con hooks ([guía](https://dev.to/felipeelias/perfect-claude-code-notifications-setup-with-tailscale-and-ntfy-1ii1),
  [aprobar desde el móvil](https://dev.to/coa00/how-i-built-a-mobile-approval-system-for-claude-code-so-i-can-finally-leave-my-desk-1ida)).
  La vía oficial es **Remote Control**: "Push when actions required" avisa al móvil cuando una sesión tiene una
  pregunta o un permiso pendiente. [docs](https://code.claude.com/docs/en/remote-control#mobile-push-notifications)

### El consenso, resumido

1. **Pocos estados, siempre los mismos, con color**: trabajando · espera al usuario · fallo o bloqueo · terminado · parado.
2. **Lo que espera al usuario va arriba**, ordenado por antigüedad, y **enseña la pregunta literal**.
3. **La pregunta se guarda aparte** del flujo de mensajes y se cierra solo con una respuesta.
4. **Señales automáticas** (hooks, estado de la app) mejor que fiarse de que el agente se acuerde de avisar.
5. **Aviso al móvil** solo cuando hay que actuar, no por cada progreso.

---

## 2. Cómo estamos hoy

| Pieza | Qué funciona | Qué falla |
|---|---|---|
| **Protocolo de los jefes** | Llegan los avisos (la BITACORA de hoy tiene decenas) | El aviso trae la pregunta mezclada con el informe. No hay un campo `PREGUNTA`. Cuando el usuario contesta **directamente en el jefe** (hoy pasó varias veces: "ya le he dicho que las junte"), el Centro no lo sabe hasta el siguiente aviso. |
| **`estado/`** | Es la fuente de verdad, con decisiones citadas | Las preguntas abiertas están en `PENDIENTES.md` sin id ni fecha, mezcladas con tareas del usuario. En el chat, una pregunta del Centro queda **enterrada** en cuanto llega otro aviso. El REGISTRO tiene una fila rota (la de `centro-control`, mezclada con la de Percha). |
| **Dashboard** | Diseño v2 correcto, se ve bien en el móvil | **Lleva 4 días sin actualizarse** (última sincronización: 2026-10-29 23:20). Todavía dice "¿lanzo la Fase 3?" y "límite 4". Es del tipo Dashboard, que solo lee **archivos subidos**: cada cambio pide reescribir el JSON, subirlo y actualizar el dataset, **2-3 llamadas por archivo y 6 archivos**. Por eso no se mantiene. Tampoco avisa de que los datos son viejos. |
| **Regla de 3 proyectos** | Escrita en `CLAUDE.md` y en el REGISTRO | No se ve en ningún panel. El dashboard sigue contando **sesiones** sobre un máximo de 4. |

**Causa raíz:** la pregunta no tiene identidad ni sitio fijo, y el panel cuesta tanto de actualizar que no se actualiza.

---

## 3. Qué conviene copiar y qué no

| Idea | ¿Copiar? | Por qué |
|---|---|---|
| Bandeja de preguntas con id fijo, arriba siempre | ✅ **Sí, lo primero** | Es justo el fallo que describe el usuario. Agent view, ccm y Agent Inbox hacen lo mismo. Cuesta poco: es kit y formato, sin código. |
| Semáforo por jefe | ✅ Sí | Es el estándar (agent view). Se mira de un vistazo en el móvil. |
| Dashboard en vivo con `db` (escribir filas con `ArtifactData`) | ✅ Sí | Una sola llamada `batch` por cambio, en lugar de 12-18. La página se actualiza sola (`onSnapshot`). |
| Contador "3 de 3 proyectos" con cola | ✅ Sí | La regla es de hoy y no se ve en ninguna parte. |
| Aviso de "datos viejos" en el dashboard | ✅ Sí | Si el panel miente, se deja de mirar, que es lo que ha pasado. |
| Campo `PREGUNTA` / `RESPONDIDA` en el protocolo de jefes | ✅ Sí | Así se cierran las preguntas que el usuario contesta en el jefe. |
| Aviso al móvil (`PushNotification` del Centro) | 🟡 Sí, con límite | Solo cuando entra una pregunta nueva o un bloqueo, nunca por progreso. Necesita Remote Control conectado. |
| Responder desde el dashboard | 🟡 Probar primero | Hay una vía real (abajo, punto 5), pero hay que comprobar que funciona desde el móvil antes de apoyarse en ella. |
| Hook global `Notification`/`Stop` que escribe eventos en `estado/` | 🟡 Experimento opcional | Es la señal más fiable según la literatura. Pero cambia la configuración de todas las sesiones, no está claro que la app de escritorio lo respete igual y añade un archivo más. Solo si lo anterior no basta. |
| Agent teams, Claude Squad, ccmanager, Conductor, Vibe Kanban | ❌ No | Son de terminal o solo de Mac, experimentales u otra interfaz más que mantener. El usuario usa la app de escritorio en Windows y el móvil, y la app ya tiene sesiones, secciones y mensajes entre sesiones. |

---

## 4. Propuesta priorizada

### P1. Bandeja de preguntas (kit, sin código; ~1 h) — lo que más se nota

1. **Archivo nuevo `estado/PREGUNTAS.md`** (lo escribe el Centro). Una fila por pregunta:
   `id | proyecto | de (sesión) | pregunta | opciones | propongo | abierta desde | estado | respuesta literal | cerrada`.
   - `id` fijo y corto: `P-001`, `P-002`… nunca se reutiliza.
   - Estados: `abierta` · `respondida` (enviada al jefe) · `aplazada` (el usuario dijo "luego") · `caducada` (el jefe dice que ya no aplica).
   - `PENDIENTES.md` se queda para revisiones, cola y tareas del usuario. Las preguntas salen de allí.
2. **Cabecera fija en cada respuesta del Centro**, aunque hable de otra cosa (amplía el principio 5b):
   `📌 3 abiertas: P-014 Percha · ¿integro en main? | P-015 FightLab · F8 | P-016 NoticIA · MOSS`.
   Si no hay ninguna, no sale la línea.
3. **El cuadro de siguiente paso** (`wrap_up`) lleva siempre la **pregunta abierta más antigua**, con botones de
   respuesta (la propuesta del Centro primero). Así queda abajo, con botones, aunque el chat siga.
4. **Responder por id:** "P-14 sí", "14 sí, 15 no". El Centro cita la respuesta literal, la manda al jefe, la apunta
   en `PREGUNTAS.md` y en la BITACORA como `[DECISIÓN]`.
5. **Protocolo de los jefes, dos líneas nuevas** (en `protocolo-jefes.md` y en el bloque de `~/.claude/CLAUDE.md`):
   - `PREGUNTA: <texto literal> · OPCIONES: a / b` → el Centro abre un `P-nnn`.
   - `RESPONDIDA AQUÍ: <id o resumen> → <lo que dijo el usuario>` → el Centro cierra la pregunta. Así se cierran las que el usuario contesta directamente en el jefe.
6. **En B (estado) y L (buenos días)**, la sección "🟡 Te esperan" sale de `PREGUNTAS.md`, ordenada por antigüedad.
   Una pregunta con más de 24 h se marca ⏰.

### P2. Dashboard nuevo, en vivo (subsesión, ~2-3 h)

**Cambio de base:** de tipo Dashboard (archivos subidos) a **página HTML propia con `db` + `user`** (y `comments` si
se aprueba P4). El Centro escribe filas con **una sola** llamada `ArtifactData batch` cada vez que cambia algo, y la
página se actualiza sola.

⚠️ **La URL cambia:** un artifact del tipo Dashboard no se puede convertir en una página propia. El viejo se deja
(o se borra si el usuario lo pide) y la URL nueva va al REGISTRO. *Alternativa sin cambiar de URL:* seguir con el
tipo Dashboard pero juntar todo en **un único `centro.json`**. Son 2 llamadas por cambio en lugar de 12-18, pero
no se actualiza en vivo ni permite responder.

**Colecciones:**

| Colección | Un documento por | Campos |
|---|---|---|
| `jefes` | proyecto | `proyecto`, `emoji`, `jefe` (name), `estado`, `frase` (qué hace o qué espera), `hijas` (n.º trabajando), `actualizado` |
| `preguntas` | `P-nnn` abierta (las cerradas se borran o pasan a `estado: respondida`) | `proyecto`, `de`, `texto`, `opciones[]`, `propongo`, `desde` |
| `meta` | `centro` (uno) | `activos[]`, `limite: 3`, `cola[]`, `ultima_sync` |
| `revisar` | cosa que revisar | `proyecto`, `texto`, `rama` |
| `actividad` | evento (los 15 últimos) | `fecha`, `tipo`, `proyecto`, `texto` |

**Diseño (móvil primero, una columna):**

1. **Arriba: frescura.** "Actualizado hace 12 min". Si pasan más de 3 h, una franja ámbar: "datos de hace 2 días: pídele estado al Centro".
2. **Bandeja "Te esperan"**: tarjetas grandes con `P-nnn`, el proyecto, la pregunta literal, las opciones y "propongo: …". Ordenadas por antigüedad, con ⏰ si llevan más de 24 h.
3. **Semáforo de jefes**: una baldosa por proyecto con borde y punto de color:
   🟢 **trabajando** (el jefe o una hija en marcha) · 🟡 **te espera** (tiene un `P-nnn` abierto) ·
   🔴 **bloqueado** (fallo, clasificador o permiso dentro de la sesión: "abre la sesión") · ⏸️ **en pausa** (regla de 3) ·
   ✅ **terminado / para revisar** · ⚪ **inactivo** (más de 24 h sin nada). Una frase debajo y "hace X min".
   Prioridad del color (como ccm): 🔴 > 🟡 > 🟢 > ✅ > ⏸️ > ⚪. El jefe hereda el peor de sus hijas.
4. **Contador de la regla:** "Proyectos activos **3 de 3**" con tres huecos (rellenos con el emoji del proyecto) y "En cola: NoticIA, Entrenador IA".
5. Plegados: "Para revisar", "Actividad reciente". Fuera la hoja de ruta y las ramas (o plegadas al final): el usuario no las mira y encarecen la sincronización.

**Sincronización en la skill:** un paso fijo al final de D, E, F y B: "un `ArtifactData batch` con lo que cambió (`jefes`, `preguntas`, `meta`)". Nada de JSON ni de subir archivos.

### P3. Aviso al móvil (kit; 15 min)

- El Centro llama a `PushNotification` **solo** cuando: entra un `P-nnn` nuevo, un jefe pasa a 🔴, o una pregunta cumple 24 h.
  Texto: `P-017 Percha: ¿integro la fase 1 en main? → propongo sí`. Nunca por progreso ni por "terminado".
- Requiere Remote Control conectado en el Centro. Si no, la herramienta lo dice y no pasa nada.
- *Opcional, ajuste del usuario:* Remote Control por defecto en todas las sesiones con "Push when actions required"
  avisa también de los **permisos** dentro de cada jefe, que el Centro no puede ver ni contestar. Puede ser ruidoso
  con muchas hijas: que decida el usuario.

### P4. Responder desde el dashboard (probar en la fase 2)

- **Vía que despierta al Centro:** en cada tarjeta, un botón "Responder" abre el **comentario** del artifact
  (capability `comments`, `openComposer`). Si el comentario se manda "a Claude" y el Centro **vigila** el
  artifact (`ArtifactComments watch`, con las respuestas automáticas armadas), **despierta al Centro**, que lo trata
  como una respuesta a `P-nnn`.
  - Ojo: la vigilancia es de la sesión que **publica**. Debe publicar el propio Centro, o hacer `watch` después
    (la subsesión que haga P2 no sirve para esto). Y se pierde si el Centro se reinicia: hay que volver a armarla en A. Arranque.
  - Hay que probar que funciona desde la app del móvil.
- **Vía simple, sin despertar a nadie:** botones de opción que escriben en `respuestas/<P-nnn>` (`db`). El Centro
  las lee al empezar cada turno y en B. Sirve como "lo dejo contestado", pero el Centro no se entera hasta su
  siguiente turno.
- En cualquier caso, **la respuesta del dashboard es un dato**: el Centro la reenvía al jefe citándola, igual que
  si la hubiera escrito en el chat.

### P5. Experimento opcional: señales por hook

Un hook en `~/.claude/settings.json` para `Notification` (`permission_prompt`, `idle_prompt`) y `Stop` que añada una
línea a `estado/eventos.jsonl` (`session_id`, `cwd`, tipo, hora, 200 caracteres del último mensaje). El Centro lo
lee en B y detecta jefes 🔴 por permiso y turnos que el jefe olvidó avisar. **Solo con OK** (es configuración
persistente) y probándolo antes en una sesión, porque no está confirmado que la app de escritorio lance los hooks igual.

---

## 5. Orden sugerido y lo que pide al usuario

| # | Qué | Quién | Necesita del usuario |
|---|---|---|---|
| 1 | **P1 bandeja** (`PREGUNTAS.md`, cabecera 📌, `wrap_up`, protocolo con `PREGUNTA`/`RESPONDIDA AQUÍ`) | Esta subsesión (kit) + el Centro (lo pone en marcha con las preguntas de hoy) | OK al formato |
| 2 | **P2 dashboard en vivo** | Esta subsesión lo escribe; el Centro lo publica (por la vigilancia de P4) | Elegir: **URL nueva en vivo** (recomendado) o mantener la URL con un solo JSON |
| 3 | **P3 aviso al móvil** | Kit | OK; tener Remote Control conectado |
| 4 | **P4 responder desde el dashboard** | Con P2 | Probarlo una vez desde el móvil |
| 5 | P5 hook | Más adelante | OK explícito (cambia la configuración) |

**Para el Centro (fuera de esta tarea):** la fila de `centro-control` en `estado/REGISTRO.md` está rota (líneas 42-43,
mezclada con la de Percha).
