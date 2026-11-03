# 🛰️ Centro de Control

Una sesión de Claude Code que hace de **orquestador** de todas las demás. Tú le hablas a ella y ella:

- **supervisa** qué hace cada sesión y te da el estado en un vistazo;
- **lanza** sesiones nuevas con un brief bien escrito, en el proyecto, el modelo y el worktree adecuados;
- **desbloquea**: te dice quién te espera y te propone la respuesta;
- **coordina**: pasa información de una sesión a otra;
- **mantiene** todo en orden: grupos en la barra lateral, títulos y archivado de lo terminado;
- **recuerda** todo en archivos (`estado/`) y lo muestra en un dashboard que puedes ver en el móvil.

```
               Tú (PC o móvil)
                     │
            ┌────────▼────────┐        estado/  (fuente de verdad)
            │ Centro de Control│◄──────►REGISTRO · PENDIENTES · BITÁCORA
            │    (Opus)       │──────► Dashboard (claude.ai, en vivo)  
            └─┬──────┬──────┬─┘
     lanza/   │      │      │   avisos automáticos al terminar o bloquearse
    mensajes  ▼      ▼      ▼   (solo de las sesiones que lanzó él)
        fightlab-x  coach-y  android-z    ← trabajadoras, cada una en su worktree
```

## Estructura

| Ruta | Qué es |
|---|---|
| `CLAUDE.md` | **El prompt de la sesión.** Se carga solo en cada conversación: identidad, autonomía, reglas y herramientas |
| `.claude/skills/centro-control/SKILL.md` | **La skill:** procedimientos paso a paso (arranque, estado, lanzar, avisos, desbloquear, mantenimiento…) |
| `.claude/skills/centro-control/references/brief-trabajador.md` | Plantilla de brief y formato de informe de las trabajadoras |
| `.claude/skills/centro-control/references/dashboard.md` | Especificación del dashboard |
| `INICIO.md` | Primer mensaje para pegar al crear la sesión |
| `estado/` | Registro de sesiones, pendientes y bitácora de acciones y decisiones |
| `pc.md` | **Ámbito de este PC:** rol (personal/trabajo), qué queda fuera, si las trabajadoras pueden abrir PR/MR, skills propias que mandan y jefes de ciclo (sprints) |
| `proyectos.md` | Catálogo de proyectos (ruta, git, stack) del que salen las carpetas de lanzamiento |
| `biblioteca-skills/` | Skills que le vayas dando para asignarlas a las trabajadoras (se comparten entre PCs) |
| `plantillas/` | `estado/`, `proyectos.md` y la página del dashboard vacíos, para un PC nuevo |

`estado/`, `pc.md` y `proyectos.md` **no se suben** (`.gitignore`): son de cada PC. Así el mismo kit sirve
para un centro en el PC del trabajo (proyectos del trabajo, con PR/MR) y otro en el personal.

## Instalación en un PC

1. Clona el repo **fuera de OneDrive** (y de cualquier carpeta sincronizada). OneDrive sincroniza `.git`
   archivo a archivo: bloquea archivos a mitad de un commit, crea duplicados y mezcla el `estado/` de
   varios PCs.

   ```bash
   gh repo clone JoaquinJBG/claude-centro-control "$HOME/proyectos/centro-control"
   ```

2. En el Code tab, crea una **sesión nueva** con esa carpeta.
3. **Modelo: Opus.** Las sesiones que lance no pueden usar un modelo superior al suyo.
4. **Modo de permisos: Auto** (recomendado). Las trabajadoras heredan como máximo el modo del centro;
   en *default* te llegarían muchas peticiones de permiso desde todas ellas. Las reglas de "pide OK antes
   de…" están en `CLAUDE.md` y se aplican igual en cualquier modo.
5. Pega el mensaje de `INICIO.md`. Creará el `estado/`, el `pc.md` y el `proyectos.md` de ese PC a partir de
   `plantillas/` (te preguntará el rol del PC y qué queda fuera de ámbito), hará el inventario, creará los grupos y el dashboard, y te dará el estado.
6. Cuando cree el dashboard, fíjalo (*pin*) en la barra lateral de claude.ai si quieres tenerlo a mano en el móvil.

## Uso diario

| Le dices | Hace |
|---|---|
| `estado` | Resumen: 🟢 trabajando · 🟡 te esperan · ✅ para revisar |
| `¿qué hace fightlab-auth?` | Detalle de una sesión en ≤8 líneas |
| `lanza en FightLab: rotación de refresh tokens con tests` | Escribe el brief, elige el modelo, lo lanza en worktree y lo registra |
| `quiero X en el coach` (algo grande) | Lo trocea en piezas independientes y te pregunta "¿Lanzo?" |
| `desbloquea` | Las preguntas abiertas con su id (`P-014`…) y la respuesta propuesta; contestas "14 sí, 15 no…", aquí o desde el dashboard |
| `dile a X que…` / `pásale a Y lo de X` | Redacta el mensaje, te lo enseña y lo envía con tu OK |
| `limpia` | Ordena los grupos y te propone qué archivar |
| `guarda esta skill: …` | La revisa, la guarda en la biblioteca y la cataloga |
| `instala las skills` | Copia las de la biblioteca a `~/.claude/skills` para que las uses tú con `/nombre` (sin pisar las propias del PC) |
| `cierra el sprint` | Cierra el ciclo del jefe de sprint, propone qué pasa al siguiente y lanza el nuevo jefe |
| `cierre` | Resumen del día en la bitácora y propuesta para mañana |

**No hace falta vigilar:** cuando una sesión que lanzó él termina o se bloquea, la app lo despierta
solo y te avisa.

## Autonomía (modo mixto)

- **Hace solo:** mirar, leer, resumir, apuntar, ordenar la barra lateral, proponer.
- **Te pide OK:** lanzar (decirle "lanza X" ya es el OK), enviar mensajes, parar, archivar, cambiar modelos o permisos.
- **Nunca:** borrar sesiones, hacer merge, hacer push o abrir PR/MR él mismo, aprobar en tu nombre, tocar lo que `pc.md` pone fuera de ámbito.
- **Push y PR/MR de las trabajadoras:** según `pc.md`. Si se permiten, en borrador y con `ticket-flow`; marcarlas listas, con tu OK.

## Añadir cosas

- **Un proyecto:** abre una sesión en esa carpeta (el centro lo detecta en el siguiente `estado`) o díselo:
  "añade el proyecto X en ruta Y". Completa su ficha en `proyectos.md`.
- **Una skill para las trabajadoras:** "guarda esta skill" y pégala o dale la ruta. Se usa automáticamente en
  los briefs según su "cuándo asignarla" del catálogo.
- **Skills desde otro PC** (por ejemplo, el del trabajo), aunque allí no uses el centro:

  ```bash
  gh repo clone JoaquinJBG/claude-centro-control "$HOME/proyectos/centro-control"
  ```

  Copia cada skill a `biblioteca-skills/<nombre>/` (con su `SKILL.md` y sus archivos de apoyo), y luego:

  ```bash
  git -C "$HOME/proyectos/centro-control" add biblioteca-skills
  ```

  ```bash
  git -C "$HOME/proyectos/centro-control" commit -m "skills: añadir <nombre>"
  ```

  ```bash
  git -C "$HOME/proyectos/centro-control" push
  ```

  En el PC personal, dile al centro "baja las skills": hará `git pull`, las revisará y las añadirá al catálogo.
  Luego "instala las skills" para poder usarlas tú directamente.
  Antes de subirlas, quita cualquier dato del trabajo (clientes, URLs internas, tokens), aunque el repo sea privado.
- **Una decisión permanente:** "apunta como decisión: en FightLab siempre…". Queda como `[DECISIÓN]` en la
  bitácora y la usará para proponerte respuestas.

## Límites conocidos

- **Las sesiones que abras tú a mano no avisan.** El centro las ve cuando le pides `estado` o al arrancar.
  Si quieres que algo esté vigilado, pídeselo a él.
- **No puede pulsar botones de permiso ni aprobar planes dentro de otra sesión.** Te da el enlace para que lo hagas tú.
- **Las carpetas que no son repo git no admiten worktrees.** El centro no pondrá a dos sesiones a escribir
  a la vez en una de ellas (va marcado en `proyectos.md` de cada PC).
- **Cada PC tiene su propio estado y su propio dashboard.** Las sesiones son locales de cada máquina. Lo que se
  comparte es el kit: el prompt, la skill, las plantillas y la biblioteca de skills.
- **Coste:** como máximo 4 trabajadoras a la vez, por defecto. Cámbialo diciéndoselo.
- **Nada de OneDrive:** el repo va en una carpeta local (`~/proyectos/centro-control`). GitHub ya hace de
  copia de seguridad del kit, y el `estado/` es de cada PC.

## Ideas para más adelante

- **Briefing diario:** una tarea programada que cada mañana te deje un resumen de lo que hay en curso y lo pendiente.
- **Ronda periódica** (`/loop 30m estado`) si empiezas a trabajar mucho con sesiones abiertas a mano.
- **Plantillas de tareas por proyecto** (p. ej. "endpoint nuevo en FightLab") dentro de `biblioteca-skills/`.
