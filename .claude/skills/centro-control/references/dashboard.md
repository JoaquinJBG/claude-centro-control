# Especificación del dashboard (versión 3, 2026-11-03)

Es una página privada en claude.ai con lo que el usuario tiene que ver de un vistazo, también desde el móvil:
qué le espera, cómo va cada proyecto y cuántos huecos quedan de la regla de 3. **`estado/` es la fuente de
verdad. El dashboard es una vista.** Si no coinciden, manda `estado/`.

Desde la versión 3 es una **página HTML propia con base de datos** (capabilities `db`, `user` y `comments`). Ya no
usa el tipo Dashboard, que obligaba a subir un JSON por cada cambio y por eso dejó de actualizarse. Ahora cada
cambio es **una sola llamada** `ArtifactData batch`, y la página se actualiza sola en el móvil.

- URL: la de la cabecera de `estado/REGISTRO.md` (`Dashboard: <url>`). Cada PC tiene la suya.
- Página: `estado/dashboard/centro.html` (en un PC nuevo, se copia de `plantillas/estado/dashboard/centro.html`).
- El dashboard antiguo (tipo Dashboard, `index.html` y los `*.json`) ya no se actualiza. Se puede borrar cuando el usuario lo diga.

## Crear o volver a publicar

Lo hace **el propio Centro**, no una subsesión: la sesión que publica es la que **vigila** el artifact y recibe las
respuestas enviadas desde el panel (ver "Responder desde el panel").

- **PC nuevo:** `Artifact` con `file_path` = `estado/dashboard/centro.html`, `icon: "radar"` y las capabilities de
  abajo. Guarda la URL en el REGISTRO y siembra los datos con un `batch`.
- **Ya existe:** `Artifact` `read` con la `url` y luego `Artifact` publish con `url` y el mismo `file_path`, **sin**
  `capabilities` (se conservan). Publicar arma la vigilancia de esta sesión.

```json
{"db": {"rules": [
  {"path": "jefes", "read": "view", "write": "owner"},
  {"path": "preguntas", "read": "view", "write": "owner"},
  {"path": "meta", "read": "view", "write": "owner"},
  {"path": "revisar", "read": "view", "write": "owner"},
  {"path": "actividad", "read": "view", "write": "owner"},
  {"path": "respuestas", "read": "view", "write": "owner"}
]}, "user": {}, "comments": {}}
```

## Datos (colecciones)

| Colección / documento | Un documento por | Campos |
|---|---|---|
| `meta/centro` | — (uno solo) | `limite` (3), `activos` (nombres de proyecto, como en `jefes.proyecto`), `cola` (ídem), `ultima_sync` (ISO). **Se escribe en cada sincronización**: la página avisa si tiene más de 3 h |
| `jefes/<slug>` | proyecto | `orden`, `proyecto`, `emoji`, `jefe` (name), `estado`, `frase` (qué hace o qué espera, una frase), `hijas` (n.º trabajando), `actualizado` (ISO) |
| `preguntas/<P-nnn>` | pregunta **abierta** de `estado/PREGUNTAS.md` | `proyecto`, `de` (name), `texto` (literal), `opciones` (array; puede ir vacío), `propongo` (si coincide con una opción, sale resaltada), `desde` (ISO), `estado` (`abierta` o `aplazada`) |
| `revisar/<id>` | cosa que hacer o revisar (de `PENDIENTES.md`) | `orden`, `proyecto`, `tipo` (`tarea` · `revisar` · `sin prisa` · `esperar`), `texto` |
| `actividad/<id>` | evento importante de la BITACORA (las **10 últimas**) | `fecha` (ISO), `tipo` (`decision` · `aviso` · `accion`), `proyecto`, `texto` |
| `respuestas/<P-nnn>` | **la escribe la página** cuando el usuario responde | `texto`, `en` (ISO), `avisado` (si llegó como comentario a Claude), `pregunta`, `proyecto` |

**Estados del semáforo** (`jefes.estado`). El color va de más a menos urgente, y la página ordena las baldosas así:

| Estado | Color | Cuándo |
|---|---|---|
| `bloqueado` | 🔴 | Falló, lo paró el clasificador o tiene un **permiso** pendiente dentro de su sesión (el Centro no puede contestarlo: la frase dice "abre la sesión") |
| `espera` | 🟡 | El proyecto está **parado** hasta que el usuario conteste |
| `trabajando` | 🟢 | El jefe o alguna hija está en marcha |
| `terminado` | ✅ | Hecho o listo para revisar, sin nada en marcha |
| `pausa` | ⏸️ | En pausa por la regla de 3 o porque lo dijo el usuario |
| `inactivo` | ⚪ | Más de 24 h sin actividad y sin nada pendiente |

El jefe hereda lo peor de sus hijas (una hija bloqueada pone al jefe en 🔴). Un proyecto que trabaja y además tiene
preguntas abiertas sigue en 🟢: la baldosa enseña "N preguntas" aparte.

## Sincronizar

Al final de **A, B, D, E, F, I y L**, siempre que haya cambiado algo, haz **una sola** llamada:

`ArtifactData` `batch` (máximo 50 escrituras) con:
- `set` de los `jefes/*` y `preguntas/*` que cambiaron;
- `delete` de las `preguntas/*` cerradas (respondidas o caducadas) y de sus `respuestas/*`, si las hay;
- `set` de `meta/centro` con el `ultima_sync` nuevo (**siempre**, aunque no cambie nada más);
- si hace falta, `revisar/*` y `actividad/*` (borra la actividad más antigua para quedarte con 10).

Cada escritura sobre un documento que ya existe lleva su `if_version`. La versión la da la última lectura o el
resultado del `batch` anterior. Si no la tienes, haz un `list` de esa colección primero.

## Responder desde el panel

En cada tarjeta, el usuario pulsa una opción o escribe otra respuesta. La página:

1. **Si alguna sesión vigila el artifact**, publica un comentario "Respuesta a P-nnn (proyecto): …" con *Send to
   Claude*. Eso **despierta al Centro**. Para que funcione, el Centro tiene que haber publicado la página en esta
   conversación o haber hecho `ArtifactComments watch` con la `url` (y tenerla armada para responder). Haz esto en
   **A. Arranque**: después de un reinicio, la vigilancia se pierde.
2. **Siempre** guarda `respuestas/<P-nnn>`. Si no había vigilancia, el Centro la recoge al principio de su siguiente
   turno: en A, B, L, o cuando el usuario diga "mira el panel" (`ArtifactData list` de `respuestas`).

Cuando llega una respuesta del panel: es un dato del usuario, como si la hubiera escrito en el chat. Ciérrala con
**M. Bandeja de preguntas** (cítala literal, envíala al jefe con OK si hace falta, apúntala) y, si llegó como comentario,
contesta en el hilo con lo que hiciste y márcalo resuelto (`ArtifactComments reply` + `resolve`).

## Diseño (para no perderlo si se rehace)

Una columna, para el móvil primero, en tema claro y oscuro:

1. Cabecera: "Actualizado hace X". Si pasan más de 3 h, una franja ámbar con "pídele «estado» al Centro". Después, las píldoras de preguntas abiertas, bloqueados y trabajando.
2. **Te esperan**: tarjetas `P-nnn` de la más antigua a la más nueva, con el proyecto, quién pregunta, hace cuánto (⏰ y en rojo si pasa de 24 h), la pregunta literal, "El Centro propone", y botones de opción y "Otra respuesta".
3. **Proyectos**: una baldosa por jefe con su color de estado. El punto late en 🟡 y 🔴, salvo con movimiento reducido.
4. **Proyectos trabajando N de 3**: tres huecos y la cola.
5. Plegadas: "Para ti" (`revisar`) y "Actividad reciente".
