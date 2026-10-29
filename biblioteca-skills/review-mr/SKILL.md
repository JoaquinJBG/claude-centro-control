---
name: review-mr
description: >-
  Use when the user asks to review a pull request or merge request that already exists on GitHub
  or GitLab, usually written by someone else: "revisa la PR de <compañero>", "haz la CR de esta MR",
  "pásale una review a #123 / !393", a GitHub PR URL or a GitLab MR URL, "me han pedido revisión".
  Reads the PR/MR, its linked ticket or issues and its history, reviews it with an agent budget
  sized to the diff, tells the findings in chat first and posts inline comments only after the
  user's OK. Not for the user's own work before pushing (that is craft-review) and not when they
  want a single automatic comment posted at once.
---

# Revisar una PR/MR ajena

Una review que el autor pueda leer sin interpretar nada, con cada hallazgo anclado a una línea.
**No se publica ni se aprueba nada sin el OK explícito del usuario.** Se puede revisar una PR/MR
en borrador si él lo pide.

## 0. Antes de empezar

**Datos del proyecto.** No se dan por supuestos; se sacan en este orden:

1. La ficha del proyecto en `proyectos.md` del Centro de Control, si el encargo la da: `Forja`
   (GitHub `gh` | GitLab `glab` | ninguna), `Tracker` (ninguno | Jira `<PREFIJO>-` | GitHub
   Issues | GitLab Issues), `Flujo de cambios`, `Checks` y rama base.
2. El propio repo: la URL que te pasan o `git remote -v` (`github.com` → `gh`; GitLab, en
   gitlab.com o autoalojado → `glab` o el MCP de GitLab), el `CLAUDE.md`/`AGENTS.md`, `README` y
   `CONTRIBUTING` del repo, y para los checks `package.json`, `Makefile`, `pyproject.toml` o la
   configuración de CI.
3. Si aun así falta algo que importa, **una** pregunta al usuario.

Si la forja es "ninguna" no hay PR/MR que revisar: dilo y propón `craft-review` sobre la rama.

**Acceso y referencias.**

- GitHub: `gh auth status`. De la URL o de `#n` saca `<owner>/<repo>` y el número
  (`github.com/<owner>/<repo>/pull/<n>`). Rutas de API: `repos/<owner>/<repo>/pulls/<n>`.
- GitLab: `glab auth status` (o que el MCP de GitLab responda). De la URL o de `!n` saca el
  proyecto y el IID (`<host>/<grupo>/<proyecto>/-/merge_requests/<iid>`; el IID, no el id global).
  Ruta codificada para `glab api`: `projects/<grupo>%2F<proyecto>/merge_requests/<iid>`.
- Sin URL, el repo sale del directorio de trabajo (`git remote get-url origin`).
- Si la autenticación falla, para y dilo.

## 1. Contexto (ligero, antes de revisar)

1. Una consulta a la PR/MR: título, autor, rama origen y destino, borrador, estado, estado de la
   pipeline o de los checks, número de ficheros y líneas cambiadas, revisores y los SHA que
   necesitarás para publicar.
   - GitHub: `gh pr view <n> --repo <owner>/<repo> --json title,author,headRefName,baseRefName,isDraft,state,statusCheckRollup,changedFiles,additions,deletions,reviewRequests,headRefOid,body,closingIssuesReferences`.
   - GitLab: `glab api projects/.../merge_requests/<iid>` (`draft`, `state`, `head_pipeline`,
     `changes_count`, `reviewers` y `diff_refs`: `base_sha`, `start_sha`, `head_sha`).

   Si está cerrada o mergeada, dilo y para.
2. Sus comentarios y discusiones: qué se ha dicho ya, para no repetirlo y para no abrir un hilo
   sobre algo ya resuelto.
   - GitHub: `gh pr view <n> --comments` y `gh api repos/<owner>/<repo>/pulls/<n>/comments`
     (comentarios en línea) y `.../pulls/<n>/reviews`.
   - GitLab: `.../merge_requests/<iid>/discussions`.
3. **Qué debía hacer.** Si el proyecto tiene tracker, la clave del ticket sale del título, de la
   rama o de la descripción; léelo (criterios de aceptación, decisiones, fuera de alcance). Si no
   hay tracker, el objetivo sale de la descripción de la PR/MR y de las issues enlazadas
   (`closingIssuesReferences`, "Closes #n", "Closes !n"). Sin nada de eso, o si no aclara el
   objetivo, **pregúntalo** en una línea: una review que no sabe qué se pedía solo encuentra bugs
   de sintaxis.
4. Convenciones: `CLAUDE.md` o `AGENTS.md` del repo (raíz y directorios tocados), `CONTRIBUTING` y
   sus skills. Léelos a la versión de la rama de la PR/MR, no a la de la rama base.
5. Superficie: ficheros y líneas cambiadas (`gh pr diff <n>` / `gh pr diff <n> --name-only`, o
   `.../merge_requests/<iid>/changes` en GitLab). Con eso eliges el plan de la sección 2.

## 2. Plan de agentes (suelo, no cuota)

| Diff | Arranque | Agentes |
|---|---|---|
| menos de ~200 líneas, un área | review directa en la sesión | 0 |
| varias áreas, o ~200-800 líneas | un finder por dimensión **aplicable** | 2-4 |
| más de ~800 líneas, migración, seguridad o dinero | finders y un verificador adversarial **solo de los bloqueantes candidatos** | 4 + 1 por bloqueante |

Escala solo con una señal observada: un candidato a bloqueante pide un verificador de **ese**
hallazgo; superficie no prevista (auth, migración, dinero, concurrencia) añade el finder de esa
dimensión. Una dimensión sin superficie en el diff no lleva agente: se dice en el informe. Tope de
6 agentes; si lo alcanzas con áreas sin leer, informa de "cobertura parcial: <áreas>". Los finders
leen el diff y los ficheros de Scope, citan `archivo:línea` y **no** puntúan severidad; el
orquestador deduplica y decide. Modelo: finders mecánicos en Haiku 5.5, de criterio en Sonnet 5.5
`medium`, orquestador en Opus 5.5 `medium`, verificador de bloqueantes en Opus 5.5 `high`
(`xhigh` en seguridad, concurrencia, dinero o migraciones). Pon `model:` y `effort:` en cada agente.

Un diff de más de ~3000 líneas, varios componentes o varios repos a la vez ya no es una review de
PR/MR: usa la skill `auditoria-integral` (inventario, revisión por componente y auditoría
transversal).

## 3. Qué se mira

Cada dimensión, aterrizada en este cambio y no en general:

| Dimensión | Qué busca |
|---|---|
| Bugs y regresiones | casos borde, nulos, fechas y zonas horarias, doble ejecución o concurrencia, fallo a medias que deja datos inconsistentes, qué rompe el cambio en los caminos vecinos |
| Cumple la tarea | cada criterio del ticket o de la issue cumplido o no; nada a medias; nada fuera de alcance |
| Convenciones | `CLAUDE.md` y el código vecino: servicio, permisos, i18n, tipos, comentarios |
| Seguridad y datos | permisos y acceso a recursos ajenos, validación de entrada, secretos, migraciones sin reversa, consultas N+1 o sin acotar |
| Pruebas | que el test **falle sin el arreglo**, que cubra los casos borde, que no dependa del entorno |
| Contratos | si toca API, esquema o formato: que productores y consumidores sigan alineados (front, back, openapi, datos que ya hay) |

**Contexto de historia** (`git blame` y notas de PRs/MRs anteriores que tocaron los mismos
ficheros) solo si el diff modifica código antiguo o delicado.

**Checks.** Los comandos salen de la ficha (`Checks`) o del repo (scripts de `package.json`,
`Makefile`, `pyproject.toml`, configuración de CI). Si el entorno permite arrancar el repo (rama en
un worktree aparte, sin tocar el checkout del usuario; `gh pr checkout` o `git fetch` de la rama
dentro de ese worktree), corre los tests de lo tocado y el lint rápido. Si no se puede, o la
pipeline ya está en verde y no aportaría nada, **dilo**: «No ejecutado, <motivo>». Nunca afirmes
que algo pasó sin su salida.

## 4. Hallazgos

Descarta: lo preexistente en líneas que la PR/MR no toca (como mucho, una nota «Info» aparte), lo
que cazan el linter o el compilador en CI, los nitpicks y las preferencias de estilo no escritas,
lo que es claramente el objetivo de la tarea y lo silenciado a propósito. Un hallazgo sin línea de
código que lo respalde no es un hallazgo.

Cada hallazgo lleva:

- **ID** estable (`R-1`, `R-2`…), **severidad** (Crítica, Alta, Media, Baja o Info) y **confianza**
  (Confirmado, Probable o Hipótesis). Una hipótesis nunca se redacta como un fallo seguro.
- **Evidencia**: `fichero:línea` y qué hace el código.
- **Impacto**, **corrección recomendada** y **cómo verificarla**.
- **El comentario exacto** que se publicaría, en el idioma de la PR/MR (el de su descripción y sus
  comentarios), directo, sin adornos y sin emojis.

## 5. Informe en el chat, y parar

Cuenta primero lo que has revisado y lo que no (checks no ejecutados, áreas sin leer), después
un esquema por severidad y por último cada hallazgo con su comentario propuesto. Si no hay
hallazgos, dilo con lo que comprobaste. Termina preguntando qué se publica, y espera.

## 6. Publicar (solo tras el OK)

Un hilo por hallazgo, en la línea donde empieza el problema, sin agrupar problemas distintos.

GitHub (comentario en línea sobre el código nuevo; `commit_id` es el `headRefOid`):

```bash
gh api --method POST "repos/<owner>/<repo>/pulls/<n>/comments" \
  -f body="<comentario>" \
  -f commit_id="<head_sha>" \
  -f path="<fichero>" -F line=<línea> -f side=RIGHT
```

Para un rango, añade `-F start_line=<inicio> -f start_side=RIGHT`. Para una línea borrada, usa
`side=LEFT` con el número de línea de la versión base.

GitLab:

```bash
glab api --method POST "projects/<grupo>%2F<proyecto>/merge_requests/<iid>/discussions" \
  -f body="<comentario>" \
  -f "position[position_type]=text" \
  -f "position[base_sha]=<base_sha>" -f "position[start_sha]=<start_sha>" -f "position[head_sha]=<head_sha>" \
  -f "position[new_path]=<fichero>" -F "position[new_line]=<línea>"
```

(O la herramienta equivalente del MCP de GitLab, con la misma posición.)

Comprueba la respuesta de cada uno. Si la forja lo rechaza (la línea no está en el diff, SHA
distinto porque llegó un push nuevo), vuelve a leer el SHA de cabeza (`headRefOid` o `diff_refs`),
reintenta una vez y, si sigue fallando, deja ese comentario preparado y dilo. Informa de cuáles se
aceptaron y cuáles no, con el enlace a la PR/MR.

No apruebes, no pidas cambios con una review formal, no mergees, no resuelvas hilos ni cambies la
PR/MR. No pongas pies de «generado por…». No incluyas secretos ni datos personales en un
comentario.

## 7. Cuando el autor responde

Si el usuario pide volver a mirar: relee las discusiones, comprueba contra el nuevo diff que cada
hallazgo está de verdad corregido y dile cuáles. **Un hilo se resuelve solo si hay evidencia y él
lo pide.**

## Excusas que no valen

| Excusa | Realidad |
|---|---|
| «Es pequeña, la miro por encima» | El plan de agentes ya es 0 para un diff pequeño; lo que no se salta es el objetivo de la tarea y los casos borde. |
| «La pipeline está en verde» | No dice si el test falla sin el arreglo ni si cumple la tarea. |
| «Publico y luego se lo cuento» | Primero el chat, después el OK, después la forja. |
| «Es probable, lo redacto como seguro» | Si no está confirmado, se llama Probable o Hipótesis. |
| «No pude ejecutar los tests, pero seguro que pasan» | «No ejecutado, <motivo>». |
| «No hay ticket, reviso solo el código» | Sin ticket, el objetivo sale de la descripción y las issues enlazadas; si tampoco está, se pregunta. |
