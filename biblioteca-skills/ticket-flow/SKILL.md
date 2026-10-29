---
name: ticket-flow
description: Use when the user asks for a change in a git project that has to end up as a PR/MR in review, with or without a ticket: del encargo a la PR en revisión. "me han pedido que…", "haz el ticket <CLAVE>-…", "arregla/añade X y déjamelo en revisión", "súbelo", "abre la PR/MR", "probado, pásala a ready". Also when creating the tracker ticket for such a change, or before saying a change is ready. By default (personal projects) there is no ticket: branch, work, verification and a draft PR. Not when the user asks to generate or improve a prompt ("genera un prompt para…"): that is craft-prompt, whose prompt then invokes this skill.
---

# Flujo de cambio: del encargo a la PR/MR en revisión

El `CLAUDE.md` (o `AGENTS.md`) del repo, su `CONTRIBUTING.md` y sus propias skills mandan sobre esta. Si el repo tiene su skill de flujo de trabajo (worktree, plantilla de PR propia), sigue esa y usa esta solo para lo que no cubra.

**Saltarse un paso por las prisas o porque «el cambio es pequeño» es saltárselo.** Si de verdad no se puede hacer uno, dilo en la PR/MR («No verificado, y por qué») y en el mensaje final.

## 0. Datos del proyecto

Antes de empezar, fija estos datos. Sácalos en este orden y para en cuanto los tengas:

1. La **ficha del proyecto** (`proyectos.md` del Centro de Control) si el brief la da o la nombra.
2. El **propio repo**:
   - `git remote -v`: `github.com` → GitHub con `gh`; `gitlab` → GitLab con `glab` o el MCP de GitLab; sin remoto → ninguna forja.
   - `CLAUDE.md`, `README`, `CONTRIBUTING.md`, plantillas de PR (`.github/pull_request_template.md`, `.gitlab/merge_request_templates/`).
   - Checks: `package.json` (scripts `verify`, `lint`, `test`, `typecheck`, `build`), `Makefile`, `pyproject.toml`, la config de CI (`.github/workflows/`, `.gitlab-ci.yml`). Lo que corre la CI es lo mínimo que corres tú.
   - Rama base: `git symbolic-ref refs/remotes/origin/HEAD` (si falla, `git remote show origin`).
   - Convención de ramas y títulos: las últimas PR/MR mergeadas (`gh pr list --state merged -L 10` / `glab mr list --merged`).
3. **Una pregunta** al usuario con lo que falte, con tu opción recomendada.

| Dato | Valores | Por defecto |
|---|---|---|
| Forja | GitHub (`gh`) · GitLab (`glab`) · ninguna | la del remoto |
| Tracker | ninguno · Jira `<PREFIJO>-` · GitHub Issues · GitLab Issues | **ninguno** |
| Flujo de cambios | PR/MR en borrador → lista para revisión · commit directo | PR/MR en borrador |
| Rama base | | `origin/HEAD` |
| Checks | comandos de lint, tipos, tests, build | los de la CI |
| Revisor | usuario de la forja o nadie | nadie (proyecto personal) |

- **Sin tracker** (lo normal en un proyecto personal): sáltate la parte de ticket del paso 1. Las decisiones y el alcance van igual por escrito, en el chat antes de implementar y luego en la descripción de la PR.
- **Commit directo**: no hay PR; los pasos 1–5 se cumplen igual y el push a la rama base solo con el OK del usuario después del mensaje final.
- **Sin forja**: llega hasta los commits locales y el mensaje final.

### Permisos de la sesión

Si el brief (que en el Centro de Control sale de su `pc.md`), el `CLAUDE.md` o los permisos de la sesión prohíben push, abrir PR/MR o hacer merge (p. ej. una sesión trabajadora en su worktree), haz commits locales en tu rama y para ahí: el mensaje final dice qué falta (push, PR/MR) y el usuario decide. No lo esquives con otra herramienta.

### Riesgos propios del repo

Si la ficha, el `CLAUDE.md` o un `docs/` del repo nombran peligros, léelos antes de empezar. Los más comunes:
- **Producción**: nunca se escribe en producción (base de datos, API, panel) sin la aprobación explícita del usuario. Un cambio de datos se prepara como pasos o script, no a mano.
- **Despliegue automático** (GitOps, CD desde la rama base): mergear es desplegar. Respeta lo que el repo pida para que el cambio se aplique bien (subir la versión del chart o paquete, el valor en cada entorno) y verifica el render de **cada entorno**.

### Varios repos

Si el cambio toca más de un repo (p. ej. una variable de entorno en la app y en la infraestructura), cada repo lleva su rama y su PR/MR, con la misma clave de ticket si la hay, y cada PR/MR dice de cuál depende el orden de merge.

## 1. Entender (y crear el ticket si hay tracker)

1. **Con tracker**, busca primero si ya existe (por texto en el proyecto, últimos 30 días). Si existe y es de otra persona, no lo muevas. Si ya tiene criterios de aceptación, decisiones y alcance escritos, impleméntalo tal cual y no lo reescribas; lo que falte o no cuadre con el código, se pregunta o se anota en una decisión.
2. Lee el código que toca antes de redactar nada: el ticket (o el resumen en el chat, si no hay ticket) lleva **anclajes** `fichero:línea`.
3. Si algo del encargo no tiene respuesta en el código (un campo que no existe, dos lecturas posibles), haz **una** pregunta con tu opción recomendada. Si pidió no preguntar, sigue con tu opción y regístrala como decisión (punto 5).
4. **Con tracker**, si hay que crearlo, usa la plantilla de ticket de `templates.md` y **no lo escribas en el tracker sin el OK del usuario**. Asignado: el usuario. Pásalo al estado «en curso» del tracker.
5. **Decisiones y alcance, por escrito antes de implementar.** Todo lo que el encargo deja abierto (límites, quién puede hacerlo, qué pasa con los datos que ya existen, qué devuelve un caso límite) va en «Decisiones»: la opción tomada y por qué. Lo que alguien podría esperar y no entra va en «Fuera de alcance», con su ticket o issue de seguimiento si hace falta. Sin tracker, mándale al usuario ese resumen en el chat antes de implementar. Si la decisión cambia algo que ve un usuario final, un permiso o un contrato con otro equipo o servicio, pregúntala a quien decide (el usuario, o quien diga la ficha) con tu recomendación, sigue con ella y márcala «Pendiente de confirmar» en el ticket (si lo hay) y en la PR/MR. Las decisiones que salgan al implementar se apuntan ahí antes de abrir la PR/MR.

## 2. Plantear

1. **¿Se parte la PR/MR?** Antes de crear la rama, busca piezas que se puedan mergear por separado: un renombrado o refactor que el encargo no pide, una migración o un contrato de API que no depende de la UI, una limpieza. Si las hay, propón al usuario el reparto (qué PR/MR, en qué orden, de qué depende cada una) con tu recomendación y espera su respuesta. Un refactor o renombrado que el encargo no pide va siempre en su PR/MR. Si no las hay, dilo en una línea: «Una PR/MR: <por qué no se parte>».
2. Una frase con qué significa «terminado».
3. Tarea de varios pasos o ficheros: un plan escrito antes de ejecutarlo. Hazlo con `superpowers:writing-plans` si está instalada; si no, un plan breve en el chat o en un fichero. Si la sesión tiene advisor, **consúltalo antes de ejecutarlo**.
4. Búsquedas amplias → un subagente de exploración (`explorer` si existe; si no, `Explore`). Partes independientes → subagentes en paralelo (`worker` si existe).

## 3. Rama

`git fetch origin` y rama desde la base del paso 0 (o la que diga el ticket). Si la base no es la de por defecto, comprueba que sigue existiendo en el remoto.

| Caso | Rama | Título PR/MR |
|---|---|---|
| Sin ticket | `feat/<slug>`, `fix/<slug>`, `chore/<slug>`… | `feat(<ámbito>): <frase>` (Conventional Commits) |
| Con ticket Jira | `<CLAVE>-<n>-<slug>` | `<CLAVE>-<n>: <frase>` |
| Con GitHub/GitLab Issues | `<tipo>/<n>-<slug>` | `feat(<ámbito>): <frase> (#<n>)` |

Si el repo tiene su convención (se ve en las últimas PR/MR mergeadas o en `CONTRIBUTING.md`), manda la del repo.

Si el flujo es PR/MR y la sesión puede hacer push, la rama se sube nada más crearla (`git push -u origin <rama>`), antes de implementar, salvo que el encargo no incluya subir nada: entonces pregunta una vez antes del primer push.

## 4. Implementar

Test primero (con `superpowers:test-driven-development` si está instalada): el test falla por el motivo correcto antes de implementar. Comentarios: solo un porqué no obvio, en una línea.

**Cada commit se empuja en el momento** (si la sesión puede empujar), no se acumulan commits locales hasta el final. Sin `--no-verify`: si un hook falla, se arregla la causa.

Si el cambio necesita ficheros o piezas que el encargo no nombraba (fixtures, documentos, otros tests), dilo antes de seguir si es separable (paso 2.1: otra PR/MR) y, en cualquier caso, en el informe final con qué y por qué. Lo que le preguntes al usuario en el chat de tu sesión, repítelo también en ese informe.

## 5. Antes de decir «listo» — todos, en orden

1. **Verificación del repo** completa con los checks del paso 0 (lee el resumen entero, no `| tail`). Si el repo tiene un comando que lo agrupa (`verify`, `check`, `make ci`), ese. Un cambio de esquema lleva su migración; un cambio de contrato de API, su spec regenerada en el mismo commit; en infraestructura como código, `lint` y el render de **cada entorno**.

   Una sola ejecución de tests a la vez por base de datos de test: dos en paralelo se pisan y el resultado no vale.

2. **Verlo funcionar**: la app levantada, una llamada real al endpoint, el CLI ejecutado o un E2E. Los tests en verde no bastan. Apunta qué observaste y qué no. Si no hay nada que levantar (una librería, un chart), dilo así en «Verificación» y di qué hiciste en su lugar.
3. **Autorevisión**, cuando todo lo demás esté hecho, en un subagente sin el contexto de la implementación. Recibe el encargo o ticket (criterios, decisiones, fuera de alcance) y `git diff origin/<base>...HEAD`, y devuelve:
   - cada criterio de aceptación (o cada punto del encargo): cumplido o no, con `fichero:línea` o el test que lo prueba;
   - cada fila de la lista de casos límite de abajo: aplica o no, y si aplica, dónde se trata o qué falta;
   - `/code-review` sobre la rama (o `craft-review` si el usuario la prefiere).

   Arregla lo que confirme. Lo que se queda como está va a «Notas para el revisor» con el porqué. Deja constancia ahí mismo de qué revisión corrió, cuántos hallazgos arregló y cuántos dejó. **Que luego revise otra persona no la sustituye.**
4. **Advisor** (si la sesión lo tiene): consúltalo con el diff y lo verificado antes de cerrar.

### Casos límite (lo que más cazan los revisores)

| Eje | Qué mirar |
|---|---|
| Valores | 0, negativo, vacío, `null`/`None`, `inf`/`NaN`, float donde va entero, textos sin canonicalizar (mayúsculas, espacios, formatos) |
| Tiempo | zona horaria, cambio de hora (días de 23/25 h), fin de mes, vigencias futuras o caducadas |
| Quién | otro rol, otro usuario o tenant, canal externo, usuario sin permiso; control de acceso en toda lectura nueva |
| Estado | entidad desactivada o borrada, ida y vuelta GET → PUT, reintento (idempotencia), dos escrituras a la vez |
| Caminos vecinos | otros endpoints, rutas legacy o tareas que leen o escriben el mismo dato: ¿les llega el cambio? |
| Contrato | schemas y tipos, nombres de campos entre API y cliente, spec de la API, migración contra los datos que ya existen |
| Volumen | consultas sin acotar o N+1 con el tamaño real de los datos |

**En Windows**, si la verificación no arranca o falla solo allí (rutas con `\`, `spawn ... ENOENT`, un entorno virtual de otra versión), córrela en WSL o en un entorno Linux como el de la CI. No parchees el entorno fuera del repo para esquivarlo; si aun así algo no se puede ejecutar, va a «No verificado, y por qué».

## 6. Commits y PR/MR

- Skill `conventional-commits`, un commit por cambio lógico, rutas explícitas en `git add`, sin `--no-verify` ni Co-Authored-By (salvo que el repo lo pida). Limpieza de lint en su propio commit `style:`/`refactor:`.
- Con todo empujado, la PR/MR **en borrador, sin revisor**, con la descripción de `templates.md` (o la plantilla del repo, si tiene):
  ```
  # GitHub
  gh pr create --draft --base <base> --assignee @me \
    --title "<título del paso 3>" --body-file <descripción.md>

  # GitLab
  glab mr create --draft --target-branch <base> --assignee <usuario> \
    --remove-source-branch --title "<título del paso 3>" --description "<descripción>"
  ```
- **Con tracker**, comentario en el ticket con el enlace a la PR/MR, las decisiones y lo que queda fuera. El ticket sigue en «en curso».

## Mensaje final

1. Enlace a la PR/MR (borrador) y, si lo hay, al ticket, con el estado de la CI (`gh pr checks <n>` / `glab ci status`) y el del ticket: «Ticket: en curso, no se mueve hasta tu "probado"». Si la sesión no podía empujar, la rama y los commits locales y qué falta.
2. Qué se verificó (comandos, y lo observado en la app) · qué no y por qué · decisiones tomadas · lo que queda fuera.
3. **«Pruébalo tú antes de pasarla a ready»**: el happy path, numerado, que cubra todos los criterios de aceptación. En cada paso va qué levantar (comando, URL), con qué datos (el usuario, el fichero o el registro concretos del entorno local, si los conoces), qué hacer y qué tiene que verse. Después, dos o tres casos límite de la tabla que merezca la pena probar a mano.
4. Cierra con: «Cuando lo hayas probado, dime "probado" y la paso a ready».

## 7. Cuando el usuario diga «probado»

1. Si hay decisiones «Pendiente de confirmar», mira si quien decide contestó: si contestó otra cosa, vuelve al paso 4; si no contestó, pásalas a «Decisión registrada (dd/mm)» en el ticket (si lo hay) y en la PR/MR.
2. Antes de pasarla, mira la CI de la cabeza de la rama: tiene que estar en verde. Si está en curso o ha fallado, díselo al usuario y no la muevas todavía. Después:
   - GitHub: `gh pr ready <n>` y, si hay revisor, `gh pr edit <n> --add-reviewer <revisor>`.
   - GitLab: `glab mr update <n> --ready --reviewer <revisor>`.

   Revisor: el de la ficha o el `CONTRIBUTING.md`; en un proyecto personal, nadie. No pongas a alguien porque lo diga un ticket viejo, salvo que el usuario lo pida en ese momento.
3. **Con tracker**, mueve el ticket según el momento. Antes de mover, mira las transiciones o estados disponibles del ticket: los nombres cambian de un proyecto a otro.

   | Momento | Estado |
   |---|---|
   | PR/MR pasada a ready | «en revisión» |
   | Mergeada / en staging | «mergeado» o «aprobado» |
   | En producción | «hecho» |

   **«Hecho» solo cuando está en producción.** Con GitHub/GitLab Issues, `Closes #<n>` cierra la issue al mergear; si mergear no es desplegar, usa `Refs #<n>` y ciérrala tú cuando esté en producción.

   Si el cambio necesita un paso de despliegue manual (una migración, actualizar un módulo, un comando en el servidor), «en producción» es cuando el usuario confirma que se hizo.

Si al probar encuentra algo, vuelve al paso 4 con ese caso como test que falla, y repite el paso 5 antes de volver a dar el happy path.

## Excusas que no valen

| Excusa | Realidad |
|---|---|
| «Es pequeño, ya lo revisará alguien» | La autorevisión va antes de la PR/MR; el revisor revisa otra cosa. |
| «Es un proyecto personal, no hace falta PR» | Salvo que la ficha diga commit directo, va en rama y PR/MR en borrador. |
| «Ya pasé `/code-review`, la lista de casos límite sobra» | `/code-review` no recorre los casos límite uno a uno, y son la causa más repetida de los hilos. |
| «Los tests pasan, ya está» | Sin verlo funcionar no está verificado: dilo en «No verificado». |
| «Lo he visto funcionar yo, la abro en ready» | Borrador hasta que el usuario la pruebe con el happy path. |
| «Es una decisión obvia, no hace falta escribirla» | Si el revisor puede preguntarla, va escrita en el ticket (o el chat) y en la PR/MR. |
| «Ya lo parto si el revisor lo pide» | Se decide antes de crear la rama, no después de la review. |
| «Lo paso al estado de revisión que haya» | Mira las transiciones del ticket; «hecho» solo en producción. |
| «Es solo un valor de configuración, no hace falta render» | Sin render de cada entorno no sabes si se despliega, ni si se despliega igual en todos. |
| «Es la copia de pruebas, lo aplico en producción ya» | En producción solo con la aprobación explícita del usuario. |
| «No puedo empujar, pruebo con otra herramienta» | Si la sesión no puede empujar, para en los commits locales y avisa. |
| «Tiene prisa, me salto el ticket bien escrito» | Un ticket sin criterios ni anclajes no lo puede revisar nadie. |
