---
name: craft-review
description: >-
  Prepara la revisión de código de la tarea en la que estás trabajando: por defecto devuelve un
  prompt de code review en inglés listo para pegar (con modelo, effort y ultracode recomendados),
  y ejecuta esa misma review aquí si se lo pides. Sirve para cualquier repo git: la rama base, el
  diff y los checks salen del propio proyecto.

  Usa este skill cuando el usuario pida revisar su trabajo antes de subirlo: "/craft-review",
  "créame una skill/prompt para codereview de esta tarea", "revísame esto antes de subir", "prompt
  de review para esta rama", "¿esto se puede subir?", "pásale una review a lo que he hecho". Aplica
  en español e inglés. NO lo uses para revisar una PR/MR ya abierta en el forge (eso es
  `review-mr`, o `/code-review-gitlab` si existe ese comando y quiere un único comentario publicado
  de una vez), ni para auditar un repo entero (`auditoria-integral`), ni cuando pida escribir
  código: entonces haz la tarea.
---

# Craft Review

Preparas la revisión del trabajo **antes de subirlo**. El objetivo es que ninguna tarea llegue a la
PR/MR con un fallo real dentro.

**Dos modos, mismo trabajo previo:**

- **Por defecto** → entregas un **prompt de code review en inglés** + modelo/effort/ultracode. No revisas.
- **Modo ejecutar** (solo si el usuario lo pide: "ejecútalo", "y revísalo tú", "hazla ahora") →
  corres esa misma review aquí y entregas el informe con veredicto.

Ante la duda de qué modo quiere, pregúntaselo en una línea antes de seguir.

## Cuándo SÍ / cuándo NO

- **SÍ**: revisar lo que el usuario acaba de programar, antes del push o de abrir la PR/MR.
- **NO**: una PR/MR ya abierta → `review-mr` (o `/code-review-gitlab`, solo si ese comando existe).
  Un proyecto entero o un cambio enorme multi-repo → `auditoria-integral`. Escribir o arreglar
  código → haz la tarea.

## De dónde salen los datos del proyecto

Rama base, forge, comandos de checks y convenciones **no se suponen**: salen del proyecto, en este
orden, y paras en cuanto tengas el dato:

1. **La ficha del proyecto** en `proyectos.md` del Centro de Control, si el encargo la trae (rama
   base, forge, comandos de verificación).
2. **El propio repo**: `git remote -v` (forge: GitHub, GitLab, Gitea…), `CLAUDE.md`, `README`,
   `CONTRIBUTING`, y los ficheros que definen comandos: `package.json` (scripts), `Makefile`,
   `pyproject.toml`/`tox.ini`/`noxfile.py`, `Cargo.toml`, `go.mod`, `composer.json`, y la config de
   CI (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`…), que suele decir exactamente qué
   checks corren antes de mergear.
3. **Una pregunta al usuario**, si tras lo anterior sigue faltando algo que cambie la review.

## Paso 1 — Fijar el objetivo de la review (siempre, en ambos modos)

Contexto **ligero**, no exploración profunda. En modo prompt el código lo lee a fondo la sesión que
ejecute el prompt; en modo ejecutar lo lees tú después, no ahora.

1. **Rama base**: la de la ficha o la que diga la documentación del repo; si no la hay,
   `git symbolic-ref --short refs/remotes/origin/HEAD` y, si falla, prueba `main` → `master` →
   `develop`. Si el repo trabaja con una rama de integración distinta de la por defecto (lo dicen
   CONTRIBUTING o CLAUDE.md), manda esa. El diff a revisar es
   `git diff $(git merge-base <base> HEAD)...HEAD`.
2. **Superficie**: `git diff --stat` contra esa base + `git log --oneline` de la rama. Con eso ya
   sabes tamaño, archivos y áreas tocadas.
3. **Convenciones**: `CLAUDE.md` (raíz y de los directorios tocados) y `README`. Solo rutas y
   contenido de los que apliquen.
4. **Qué debía hacer la tarea**: si no está claro por los commits, el ticket enlazado o la
   conversación, **pregúntaselo al usuario** con `AskUserQuestion`. Sin eso no se puede juzgar si
   el cambio cumple, y una review que no sabe qué se pedía solo encuentra bugs de sintaxis.

Pregunta antes de suponer cualquier dato que cambie la review materialmente (alcance, qué queda
fuera, criterio de "hecho", si hay partes intencionalmente a medias). Sin límite de preguntas:
encadena tandas de `AskUserQuestion` si hacen falta. Las convenciones obvias del repo no son
huecos: se siguen sin preguntar.

## Las 4 dimensiones

Todo prompt de review (y toda review ejecutada) cubre estas cuatro, cada una como bloque propio:

| Dimensión | Qué busca |
|-----------|-----------|
| **Bugs y regresiones** | Casos borde, null/undefined, off-by-one, promesas sin await, estado compartido, y qué funcionalidad existente rompe el cambio |
| **Cumple la tarea** | Hace lo que se pedía, no queda nada a medias (TODO, rama muerta, feature flag olvidada), no mete nada fuera de alcance |
| **Convenciones del repo** | CLAUDE.md y patrones reales del código vecino: naming, estructura, manejo de errores, cómo se hacen ahí las cosas parecidas |
| **Seguridad y datos** | Permisos y autorización, validación de input, secretos en el diff, migraciones destructivas o sin reversa, N+1, XSS/inyección |

## Verificación: checks si existen

Los checks son los que el proyecto ya define (ficha, CONTRIBUTING, scripts, Makefile, CI); no los
inventas. Córrelos **si existen y son rápidos**. Ejemplos de dónde suelen estar:

- Node/JS/TS: scripts de `package.json` (`check`, `lint`, `typecheck`, `test`).
- Python: `pytest`, `ruff`/`flake8`, `mypy`; en Django, además `python manage.py check` y
  `makemigrations --check --dry-run`.
- Otros: `make test`/`make lint`, `cargo test`/`cargo clippy`, `go test ./...`/`go vet ./...`.
- Si hay CI, los jobs que bloquean el merge son la lista de referencia.

Si no hay checks, o tardan demasiado, **dilo explícitamente** en el informe (`Checks: no hay` /
`Checks: omitidos, <razón>`). No los inventes ni los des por pasados. La salida real del comando es
la evidencia; una afirmación sin salida no vale.

## Anatomía del prompt de review (en inglés)

Incluye solo lo que aporte. Un diff pequeño puede ser 5-8 líneas.

- **Context** — 1-3 frases: qué es la app, qué hacía esta tarea, contra qué rama va.
- **Scope** — el comando de diff exacto y los archivos tocados (`@ruta`). Qué queda fuera.
- **Review for** — las 4 dimensiones, una línea concreta cada una, aterrizadas en este cambio
  ("check the new database migration is reversible", no "check security").
- **Checks to run** — los comandos detectados en el proyecto, y que reporte su salida real.
- **Agents & budget** — **obligatorio siempre**, incluso cuando el plan es "sin subagentes". Tope de
  agentes, qué modelo corre cada rol y qué NO hay que leer. Bloque literal justo debajo.
- **False positives to ignore** — la lista de abajo, recortada a lo que aplique.
- **Output** — el contrato del informe: hallazgos con severidad, archivo:línea, y veredicto final.
  Y que la única parada sea ese informe: Opus 5.5 tiende a terminar el turno con un resumen de
  progreso a mitad de tarea; en una review con subagentes, dilo en una línea ("Don't end the turn
  to report partial progress; the only stop is the final report").

### Bloque `Agents & budget` (va literal en el prompt, ajustando números y modelos)

```
## Agents & budget
- Start with <N> subagents: <dimensiones de arranque>. That is a starting plan, not a quota
  to fill.
- Escalate one step at a time, and only on a trigger you actually observed:
  - a finder reports a candidate blocking finding -> spawn one adversarial verifier for
    that finding only, not for the rest;
  - reading the diff turns up surface you weren't told about (auth, migrations, money,
    concurrency) -> add that dimension's finder.
- De-escalate freely: a dimension with no surface in this diff gets no agent. Say so in the
  report instead of spawning one to confirm the obvious.
- Hard cap <M> agents. If you hit it with areas still unread, stop and report
  "partial coverage: <areas>" rather than going over.
- Finders run on <modelo finder>: they read the diff and the files named in Scope. They
  report findings with file:line; they don't rank severity.
- You orchestrate on <modelo>: dedupe, decide severity, write the verdict. Don't re-read a
  file a finder already reported on.
- Read the Scope files and their direct callers. Don't sweep the repo.
```

Cuando el arranque es sin subagentes el bloque se reduce, pero sigue estando — y sigue llevando su
trigger de escalado, porque cero agentes también es un punto de partida:

```
## Agents & budget
- No subagents to start: review this diff directly. Read the Scope files and their direct
  callers, nothing else.
- Only exception: if you hit a candidate blocking finding you can't settle from the code in
  front of you, spawn one verifier for that finding and say in the report that you did.
```

Reglas de redacción: nombra archivos y comandos concretos; el prompt más corto que siga siendo
inequívoco gana; nada de "CRITICAL/YOU MUST" ni "be extremely thorough" (inflan exploración sin
mejorar hallazgos). Añade siempre el ancla anti-alucinación: *"Cite the actual code for every
finding. If you can't point to a line, don't report it."*

**Add-ons del modelo destino**, si aplican a esta review: están en la carpeta de la skill
`craft-prompt` (junto a esta en la biblioteca): Fable → `fable-best-practices.md`; Opus → `opus-best-practices.md`;
Sonnet → `sonnet-best-practices.md`; Haiku → `haiku-best-practices.md`. Si no están disponibles, sigue sin ellos. Lee solo el del modelo destino y injerta solo los bloques cuya
condición aplique. Con Opus 5.5 como orquestador, dos reglas de su guía pesan aquí: no añadas
"think carefully" (la palanca es el effort) y **no** le pongas la señal de tiempo ("Time matters
here…"): bajo presión de tiempo verifica menos, justo lo contrario de lo que quiere una review.

## Falsos positivos que hay que descartar

- Problemas **preexistentes**, en líneas que la tarea no tocó.
- Lo que pilla el linter, el typechecker o el compilador (imports, tipos, formato) — ya corre en CI.
- Nitpicks que un senior no comentaría; preferencias de estilo no escritas en CLAUDE.md.
- Cambios de comportamiento que son claramente el objetivo de la tarea.
- Falta de tests o de documentación en general, salvo que CLAUDE.md o CONTRIBUTING lo exijan.
- Cosas silenciadas a propósito en el código (lint-ignore, comentario explicativo).

Un hallazgo sin línea de código que lo respalde no es un hallazgo.

## Rúbricas

Dos decisiones, en este orden: **con cuántos agentes se arranca** y después **qué modelo y effort
corre cada rol**. El coste de una review se dispara por el número de agentes mucho antes que por el
modelo. En modo ejecutar aplicas este mismo plan aquí: mismo arranque, mismos triggers, mismo tope.

**Plan de arranque** — es un suelo, no una cuota. Ultracode se usa siempre que haga falta; lo que no
se hace es reservar agentes por adelantado "por si acaso":

| Diff | Arranque | Agentes |
|------|----------|---------|
| < ~200 líneas, un área | Sin ultracode, review directa en sesión | 0 |
| Varias áreas, o ~200-800 líneas | Ultracode: un finder por dimensión **aplicable**, veredicto en el orquestador | 2-4 |
| > ~800 líneas, migración end-to-end, seguridad o dinero | Finders + verificador adversarial **solo de los bloqueantes candidatos** | 4 + 1 por bloqueante |

"Dimensión aplicable" es la bisagra: si el diff no toca permisos, input de usuario ni migraciones, no
hay finder de seguridad. Cuatro finders fijos en un diff de dos archivos es exactamente lo que hace
que ultracode se vaya de las manos.

**Escalado automático** — a partir del arranque el plan se mueve solo con señales observadas, nunca
con una corazonada:

| Señal | Movimiento |
|-------|------------|
| Un finder marca un candidato a BLOQUEANTE | **Sube**: un verificador adversarial para **ese** hallazgo, no para los demás |
| Al leer el diff aparece superficie no prevista (auth, migración, dinero, concurrencia) | **Sube**: añade el finder de esa dimensión |
| El verificador tumba el hallazgo | **Para**: no se escala más por esa línea |
| Una dimensión no tiene superficie en el diff | **Baja**: no se lanza su finder; se dice en el informe |
| Dos finders reportan lo mismo | **Baja**: dedupe en el orquestador, no un tercer agente |
| Llegas al tope con áreas sin leer | **Para**: informe con "cobertura parcial: <áreas>", no te lo saltes |

El tope por defecto es 6 agentes; súbelo solo si el usuario lo pide. Cada agente por encima del
arranque necesita una fila de esta tabla detrás: si no puedes nombrar el trigger, no lo lances.

**Modelo y effort por rol:**

| Rol | Modelo | Effort |
|-----|--------|--------|
| Finder mecánico: secretos en el diff, TODO y ramas muertas, naming y convenciones contra CLAUDE.md | Haiku 4.5 | — |
| Finder de criterio: bugs y regresiones, "cumple la tarea", seguridad y datos | Sonnet 5 | `medium` |
| Orquestador: dedupe, severidad y veredicto | Opus 5.5 | `medium` (su default; equivale a `high` de Opus 5) |
| Verificador adversarial de bloqueantes | Opus 5.5 | `high` · `xhigh` si es seguridad, concurrencia, dinero o migración (de datos o de esquema) |
| Diff enorme o auditoría multi-área | Opus 5.5 como orquestador (sostiene auditorías de horas con subagentes) | `high` · Fable 5 solo si el usuario lo pide o es multi-repo |

Haiku sí entra, pero solo en el rol mecánico: buscar un patrón y citar `archivo:línea`. Nunca decide
severidad, nunca escribe el veredicto y nunca juzga si el cambio cumple la tarea — para eso hace
falta criterio, y ahí Haiku genera ruido que luego cuesta más filtrar de lo que ahorró.

Sin subagentes el orquestador es el único rol que hay: ese es el modelo y el effort del prompt, salvo
que sea un diff < ~200 líneas en un área conocida, donde basta Sonnet 5 con `medium`. `max` se
reserva a un incidente sobre datos reales. El mínimo que encaje; ante la duda manda el coste de
equivocarse, no el tamaño del diff. Los nombres de nivel no equivalen entre modelos: no arrastres a
Opus 5.5 el `high` que usabas en Opus 5 — piensa más por turno a igual nivel, sobre todo en
`xhigh`/`max`.

## Formato de salida — modo prompt (lo único que entregas)

1. Una frase en español: qué cambio has entendido que se revisa (rama, base, nº de archivos).
2. El **prompt en inglés** en un bloque de código copiable.
3. Línea **"Cómo lanzarlo"**: `Modelo: <orquestador> · Effort: <nivel> · Ultracode: <plan>` + una
   frase de razón. El plan nunca es un "sí" a secas: dice con qué arranca, con qué trigger sube y
   dónde está el tope — p. ej. `Ultracode: sí — arranca con 2 finders en Sonnet 5 (bugs, cumple la
   tarea) + 1 en Haiku 4.5 (secretos y convenciones), veredicto en Opus 5.5; sube a verificador solo
   si sale un candidato a bloqueante; tope 6`. Si es sí, recuerda anteponer `ultracode` al prompt.
4. Una línea: "Dime «ejecútalo» y la corro yo aquí."

No revisas nada en este modo.

## Formato de salida — modo ejecutar

```
### Review — <rama> vs <base>

BLOQUEANTE (n)
1. <qué falla> — `archivo:línea`
   Por qué es real: <evidencia, código citado>
   Cómo arreglarlo: <una línea>

MENOR (n)
1. <igual, pero no impide subir>

Checks: <salida real de cada comando, o "no hay" / "omitidos: razón">

Veredicto: SUBIR | NO SUBIR
```

**BLOQUEANTE** = rompe funcionalidad, incumple lo que pedía la tarea, o es un riesgo de seguridad o
de datos. Todo lo demás es MENOR. El veredicto lo decide el contador: 0 bloqueantes → SUBIR.
Si no hay hallazgos, di "Sin hallazgos" y el veredicto igualmente.

Si el siguiente paso es abrir la PR/MR y el proyecto usa el flujo de `ticket-flow`, esa skill se
encarga; esta solo entrega el veredicto.

## Errores comunes

- Ejecutar la review cuando solo se pedía el prompt (o al revés). → Ante la duda, pregunta en una línea.
- Revisar el working tree o el último commit en vez de la rama entera contra su base. → Siempre `merge-base`.
- Suponer que la base es `main` sin mirarlo. → Ficha del proyecto, docs del repo, `origin/HEAD`; si sigue dudoso, pregunta.
- Revisar sin saber qué pedía la tarea. → Pregúntalo; la dimensión "cumple la tarea" depende de ello.
- Explorar el repo a fondo en modo prompt. → Contexto ligero; el código lo lee la sesión ejecutora.
- Inventar los comandos de checks por el lenguaje del repo. → Salen de la ficha, los scripts, el Makefile o la CI; si no hay, se dice.
- Dar por pasados unos checks que no corriste. → Salida real o "no hay". Nunca un "todo OK" sin evidencia.
- Reportar problemas preexistentes o nitpicks. → Filtra con la lista de falsos positivos.
- Enterrar el veredicto entre párrafos. → El informe sigue el formato de arriba, veredicto al final y solo, o SUBIR o NO SUBIR.
- Escribir el prompt en español. → El prompt SIEMPRE en inglés; la explicación, en español.
- Marcar todo como bloqueante "por si acaso". → Bloqueante es rompe / incumple / arriesga. Lo demás, menor.
- Lanzar los cuatro finders en un diff de un archivo. → Un finder por dimensión que el diff **pueda** tocar; muchas veces son uno o dos.
- Gastar Opus en leer archivos. → El escaneo mecánico es Haiku, la lectura con criterio es Sonnet; Opus solo decide severidad y veredicto.
- Poner a Haiku a juzgar si el cambio cumple la tarea o a marcar severidad. → Haiku busca patrones y cita líneas; el criterio no es su rol.
- Verificar hallazgos menores con un agente adversarial. → El verificador solo entra en los candidatos a BLOQUEANTE.
- Entregar el prompt sin bloque `Agents & budget`. → Va siempre, aunque sea la línea de "no subagents".
- Congelar el plan de agentes al escribir el prompt. → La tabla da el arranque; el prompt lleva los triggers para que la sesión ejecutora suba o baje sola.
- Escalar "por si acaso", sin trigger. → Cada agente por encima del arranque necesita una señal de la tabla de escalado; si no la puedes nombrar, no lo lances.
- Pasarse del tope en silencio. → Se para y se informa "cobertura parcial: <áreas>".
- Arrastrar a Opus 5.5 el effort de Opus 5, o subir a Fable por tamaño de diff. → Orquestador en `medium`; Fable solo si el usuario lo pide o es multi-repo.
- Meter "Time matters here…" o "think carefully" en el prompt de review. → La primera hace verificar menos; la segunda no hace nada: la palanca es el effort.
