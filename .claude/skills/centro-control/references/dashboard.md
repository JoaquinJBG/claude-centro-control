# Especificación del dashboard

Es una página privada en claude.ai con el estado de todas las sesiones, pensada para verla de un vistazo
también desde el móvil. **Los archivos de `estado/` son la fuente de verdad. El dashboard es solo una vista.**
Si los dos no coinciden, manda `estado/`.

## Creación (una sola vez)

1. Llama a `Artifact` con `action: "quickstart"` e `intent: "other"`, y sigue lo que indique: si propone
   el tipo Dashboard, úsalo; si no, haz una página HTML con la skill `artifact-design`.
2. Carga la skill `artifact-capabilities` y da a la página la base de datos compartida (`db`). Así los
   datos se actualizan escribiendo filas con `ArtifactData`, sin volver a publicar la página.
3. Título: `Centro de Control`. Icono: `radar` (o `dashboard`).
4. Guarda la URL en la cabecera de `estado/REGISTRO.md` (`Dashboard: <url>`).

## Datos (colecciones)

**`sesiones`**: un documento por sesión, con `doc_id` = sessionId.

| Campo | Ejemplo |
|---|---|
| `name` | `fightlab-auth` |
| `titulo` | `FightLab · refresh tokens` |
| `proyecto` | `FightLab PRO` (las hijas llevan el proyecto de su jefe, para que salgan en su árbol) |
| `jefe` | sessionId de la sesión padre; vacío si la sesión es el jefe |
| `objetivo` | `Rotación de refresh tokens JWT` |
| `estado` | `trabajando` · `te_espera` · `revisar` · `inactiva` · `archivada` |
| `espera` | `¿Uso refresh tokens rotativos?` (solo si `te_espera`) |
| `modelo` | `sonnet` |
| `rama` | `refs/heads/…` |
| `origen` | `centro` · `externa` |
| `actualizado` | ISO 8601 |

**`pendientes`**: un documento por cosa que espera al usuario: `tipo` (`responder` · `revisar` · `en_cola`),
`texto`, `sesion` (sessionId o vacío) y `creado`.

**`meta`**: un único documento `resumen`, con los recuentos por estado y `ultima_sincronizacion`.

## Diseño

- Arriba, una fila de contadores: 🟢 trabajando · 🟡 te esperan · ✅ para revisar.
- Después, "Te esperan", la sección más importante: tarjetas con la pregunta pendiente.
- Luego, las sesiones agrupadas por proyecto, con un chip de estado, el objetivo, el modelo y la hora de actualización.
  Dentro de cada proyecto, en árbol: el jefe arriba, con la etiqueta "Jefe", y sus hijas sangradas debajo (y las nietas debajo de ellas).
- Las archivadas, plegadas al final.
- En móvil, una sola columna y sin tablas anchas. Debe verse bien en tema claro y oscuro.
- Solo lectura: la página no envía nada a ninguna sesión.

## Sincronización

Cada vez que cambie `estado/REGISTRO.md` o `estado/PENDIENTES.md`, haz **una sola** llamada a
`ArtifactData` con `action: "batch"`: escribe (`set`) los documentos que cambiaron, borra (`delete`)
los pendientes ya resueltos y actualiza `meta/resumen`.

---

## Cómo quedó implementado (2026-10-29) — esto manda sobre lo anterior

El quickstart propuso el tipo **Dashboard**, y ese tipo no lee colecciones propias: lee **datasets**
(archivos adjuntos o consultas). Por eso las colecciones `sesiones`, `pendientes` y `meta` de arriba
se convirtieron en:

| Antes | Ahora |
|---|---|
| colección `sesiones` | archivo `estado/dashboard/sesiones.json` (array de filas con los campos de la tabla de arriba + `id`) → dataset `datasets/sesiones` |
| colección `pendientes` | archivo `estado/dashboard/pendientes.json` (filas con `id`, `tipo`, `texto`, `sesion`, `creado`; las `responder` primero, porque la primera fila es el "Siguiente paso") → dataset `datasets/pendientes` |
| `meta/resumen` | lo calcula la propia página (`dash.calc('resumen')`); no hay que escribirlo |

**Fuentes añadidas el 2026-10-29**, a petición del usuario, para enseñar lo que la barra lateral no enseña:

| Archivo → dataset | Filas | Cuándo se actualiza |
|---|---|---|
| `proyectos.json` → `proyectos` | `id`, `proyecto`, `jefe` (name), `estado` (como el de la sesión), `trabajando` (n.º de sesiones trabajando), `siguiente` (siguiente paso en una frase), `actualizado` | Cada vez que cambie el estado de un proyecto o su siguiente paso |
| `fases.json` → `fases` | `id`, `proyecto`, `orden`, `fase` (etiqueta corta: `0`, `1`, `E`…), `nombre`, `estado` (`hecha` · `en_curso` · `pendiente` · `opcional`) | Al empezar o terminar una fase, o cuando un plan nuevo defina las suyas |
| `ramas.json` → `ramas` | `id`, `proyecto`, `rama`, `commits`, `estado` (`sin_subir` · `sin_remoto` · `en_curso` · `superada`), `nota` | Al verificar commits (D) o cuando el usuario sube o borra una rama (sale de la lista) |
| `actividad.json` → `actividad` | `id`, `fecha` (ISO), `tipo` (`decision` · `aviso` · `accion`), `proyecto`, `texto` (una frase) | Con cada entrada importante de la BITACORA. Se guardan las **10 últimas** |

**Diseño (versión 2):** contadores (trabajando de 4, te esperan, para revisar y ramas sin subir) · "Te esperan" ·
"Proyectos": una línea por proyecto con el siguiente paso, y sus sesiones plegadas en árbol jefe → hijas · "Hoja de ruta"
con chips por fase · "Ramas" · "Para revisar" (incluye lo que está en cola) · "Actividad reciente" · "Archivadas", plegada.
El usuario pidió que "Sesiones por proyecto" no ocupe tanto, porque la barra lateral ya lo enseña.

- URL: la de la cabecera de `estado/REGISTRO.md`. Cada PC tiene su propio dashboard.
- **PC nuevo:** la página base es `plantillas/estado/dashboard/index.html`, que en el arranque se copia a
  `estado/dashboard/`. Crea el dashboard igual que en el original (tipo Dashboard, datasets `sesiones` y
  `pendientes`) y publica esa página en lugar de diseñar una nueva.
- La página está en `estado/dashboard/index.html` y se guarda en el documento `files/index.html` como `{text}`.
  Si se cambia, se vuelve a escribir entera (antes, léela con `ArtifactData get` por si el usuario la editó).

**Sincronizar** (cada vez que cambien `REGISTRO.md` o `PENDIENTES.md`):

1. Reescribe los JSON de `estado/dashboard/` que hayan cambiado: `sesiones`, `pendientes`, `proyectos`, `fases`, `ramas` o `actividad`.
2. Súbelos con `Artifact` (`url` del dashboard, `asset: true`, `file_path`): un archivo JSON por llamada.
   Cada subida devuelve una URL nueva `/_blob/<id>`.
3. Una sola llamada a `ArtifactData` `batch` con `update` de los `datasets/<id>` que cambiaron:
   `source.url` = la URL nueva y `updated: {at: <ISO>, by: "Centro de Control"}` (con `if_version`; léelos antes con `get`).
4. Opcional: borra el asset viejo (`Artifact` `delete` con `url` y `path` = id del asset anterior).
