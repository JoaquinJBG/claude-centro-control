---
name: auditoria-integral
description: >-
  Use when the user asks for a review that is too big for review-mr or craft-review: auditing a
  whole project or repo, "auditoría integral", "revisión exhaustiva", "audita todo el repo",
  "revisa el proyecto entero", a very large PR/MR (many components, thousands of lines, or several
  repos at once). Works on any stack and on GitHub PRs or GitLab MRs. Builds an inventory, reviews
  each component, audits how they fit together, runs the checks, documents findings with stable
  IDs and writes a final report; publishes line-anchored threads on the PR/MR only after the
  user's OK. Read-only: it never fixes anything. Not for a normal PR/MR (that is review-mr), not
  for the user's own work before pushing (craft-review).
---

# Auditoría integral

Auditoría técnica independiente de un proyecto o de un cambio enorme, con la mirada de un senior,
un especialista en seguridad, un arquitecto y un QA. Crítica, verificable y basada en evidencias:
que funcione en el caso habitual no la da por buena. **Solo diagnostica**: no corrige, no toca
código de producción, no borra datos, no aprueba ni mergea, no cambia configuración. Lo que se
publique en la forja (GitHub, GitLab u otra) se publica **solo tras el OK explícito del usuario**.

Las listas de qué mirar por dimensión están en `checklists.md` (mismo directorio): léelo antes de
la fase 2.

## 0. Antes de empezar

1. **Datos del proyecto**, en este orden, sin dar nada por supuesto:
   1. La ficha del proyecto en `proyectos.md` del Centro de Control, si el brief la da (ruta,
      stack, rama base, forja, notas).
   2. El propio repo: `git remote -v` (si apunta a github.com → `gh`; si es GitLab → `glab` o el
      MCP de GitLab; otra forja → su CLI o API si la hay), la rama base (`git symbolic-ref
      refs/remotes/origin/HEAD`), `CLAUDE.md`, `README`, `CONTRIBUTING`, y los comandos de
      verificación en `package.json`, `Makefile`, `pyproject.toml`, `justfile`, `Taskfile` o la
      configuración de CI (`.github/workflows/`, `.gitlab-ci.yml`, etc.).
   3. Lo que siga sin estar claro, en **una sola pregunta** al usuario.
2. **Alcance**: pregunta lo que no esté claro (qué repos o carpetas, rama o commit, si hay PR/MR,
   qué se considera crítico, entornos que se pueden tocar). Una ronda, con opciones y tu
   recomendación. Puede ir en la misma pregunta del punto anterior.
3. **Registro de progreso fuera del repo**: una carpeta `auditorias/<proyecto>-<aaaa-mm-dd>/` en un
   directorio de documentación del usuario fuera del repo auditado (pregunta o confirma la ruta;
   nunca dentro del repo). Ahí van `inventario.md`, `hallazgos.md` y el informe. Los hallazgos
   llevan IDs estables para poder continuar en otra sesión sin repetir.
4. **Estado inicial intacto**: anota rama y commit de partida. Trabaja en un worktree aparte si hay
   que ejecutar algo. Cualquier modificación auxiliar imprescindible se documenta.
5. Si va a haber hilos, comprueba el acceso a la forja (`gh auth status`, `glab auth status` o el
   MCP correspondiente). Sin acceso, los comentarios se dejan preparados y se dice.

## 1. Reconocimiento e inventario

Estructura del repo, aplicaciones, módulos, servicios, puntos de entrada, configuración, variables
de entorno, dependencias y versiones, APIs, integraciones externas, persistencia, pruebas y
documentación (`CLAUDE.md`, ADR, README). **Contrasta la documentación con el código**: no se da
por actualizada. Identifica el stack real (lenguajes, frameworks, base de datos, infraestructura)
para elegir qué ejemplos de `checklists.md` aplican. Comprueba qué skills, comandos y herramientas
de auditoría hay ya disponibles y usa las que aporten (revisión, seguridad, arquitectura,
pruebas); no busques más por buscar.

`inventario.md`, una fila por componente: nombre · ubicación · responsabilidad · dependencias
relevantes · criticidad · **estado** (Revisado, Parcial, Pendiente, No verificable) · pruebas
ejecutadas · hallazgos · limitaciones. Demuestra qué se miró de verdad y qué queda.

## 2. Revisión por componente

Cada componente, no solo los que parecen problemáticos, con las seis dimensiones de
`checklists.md`: calidad y mantenibilidad, bugs funcionales, seguridad, usabilidad (si hay
interfaz), rendimiento, pruebas y fiabilidad. Adapta a la tecnología y salta lo que no aplique
diciéndolo en el inventario. Diferencia el defecto real de una preferencia de estilo. Reproduce el
fallo antes de llamarlo Confirmado.

**Plan de agentes** (tope duro; ningún agente corrige nada):

- Un agente por componente o lote, finders que citan `archivo:línea` y **no** puntúan severidad;
  el orquestador deduplica y decide.
- **Máximo 6 agentes en total y 3 vivos a la vez**; hasta 10 solo si el inventario lo exige, y
  entonces se dice antes. Si hay más componentes que agentes, se hace por tandas con el estado en
  `inventario.md`, o en una sesión nueva por tanda.
- Modelos: Haiku para lo mecánico (inventario, listados, extraer datos); Sonnet `medium` para
  finders; Opus solo para integrar, decidir y verificar candidatos a Crítica o Alta (máximo 1-2,
  `high`; `xhigh` en seguridad, dinero o migraciones). Pon `model:` y `effort:` en cada agente.
- No relances los que acabaron bien: reintenta solo el que falló. Si el plan necesita más agentes
  del tope, para y pregunta.
- Tests largos y pruebas de interfaz con capturas van en un subagente que devuelve solo el
  veredicto, para no cargar el contexto principal.

## 3. Auditoría transversal (obligatoria)

Tan importante como la fase 2. No se da por terminada la auditoría tras leer los ficheros.
Contradicciones entre componentes, contratos y formatos de datos, arquitectura e integraciones, y
trazabilidad de extremo a extremo de los flujos críticos (ver `checklists.md`). Documenta la causa
raíz y todos los componentes afectados, no un hallazgo por síntoma.

## 4. Comprobaciones técnicas

Tests, linters, tipos, build, auditoría de dependencias (`npm audit`/`pnpm audit`, `pip-audit`,
`cargo audit`, `govulncheck`... según el stack), análisis estático y de seguridad, y los flujos
críticos, según lo que permita el entorno. Usa los comandos que define el propio proyecto (scripts
del gestor de paquetes, `Makefile`, CI) antes que inventar los tuyos. Registra comando, resultado,
errores y advertencias. **Nunca afirmes haber corrido algo que no corriste**; si falta credencial,
servicio o permiso, escribe «No ejecutado, <motivo>». Sin pruebas destructivas ni sobre datos
reales, y solo en entornos autorizados (nunca producción). Si la verificación falla solo por el
sistema operativo local (p. ej. Windows frente a una CI en Linux), prueba en WSL o en un
contenedor y dilo; no lo cuentes como fallo del proyecto sin comprobarlo. Una prueba de regresión
para un defecto confirmado solo se **propone** (o se escribe en un worktree aparte): la rama
auditada no se modifica.

## 5. Hallazgos

Descarta lo estético, lo que ya caza el linter en CI y lo que no tiene una línea de código que lo
respalde. Cada hallazgo:

- **ID estable** por categoría: `SEC-001`, `BUG-001`, `UX-001`, `PERF-001`, `ARCH-001`, `QA-001`,
  `CAL-001`. Sin duplicados: una causa que afecta a varios sitios es un hallazgo con todas sus
  ubicaciones.
- **Severidad**: Crítica (consecuencias graves e inmediatas), Alta (compromete seguridad,
  integridad de datos o una función crítica), Media (relevante, con impacto limitado o
  condicionado), Baja. Ajústala al contexto real del proyecto.
- **Confianza**: Confirmado (reproducido o demostrado), Probable (evidencia técnica sólida sin
  confirmación completa), Hipótesis (requiere investigar). Una hipótesis nunca se redacta como
  seguro. Un problema de rendimiento sin medición es Hipótesis, con la medición que haría falta.
- Componente, archivo y línea exacta, descripción técnica, evidencia, impacto, condiciones de
  reproducción o explotación, corrección recomendada (localizada; no propongas reescrituras si un
  arreglo local resuelve), prueba que la verificaría, estado y estado del hilo.
- **Sin secretos ni datos personales** en hallazgos, informe o comentarios.

## 6. Informe en el chat y parar

Antes de publicar nada: resumen ejecutivo (estado real, riesgos principales, seguridad, pruebas,
lo que pide atención inmediata; sin suavizar), cobertura (Revisado, Parcial, Pendiente, No
verificable; una revisión parcial nunca se presenta como completa), esquema de hallazgos por
severidad y, por cada uno, el **comentario exacto** que se publicaría. Pregunta qué se publica y
espera.

## 7. Hilos en la PR/MR (solo tras el OK)

Un hilo por hallazgo relevante, anclado a la línea donde empieza el problema, sin agrupar
problemas distintos ni duplicar. Si el mismo hallazgo toca varios ficheros, un hilo por sitio con
el mismo ID. Formato, en el idioma de la PR/MR (o el del usuario si no está claro), directo y sin
emojis:

```
[ID] [SEVERIDAD] Título
Problema: ...
Evidencia: archivo:línea y qué hace el código.
Impacto: ...
Corrección recomendada: ...
Verificación: ...
```

Con PR/MR: usa el mecanismo de publicación de `review-mr` si está disponible. Si no:

- **GitHub**: una review con comentarios en línea (`gh api repos/{owner}/{repo}/pulls/{n}/reviews`
  con `event: "COMMENT"` y `comments[]` con `path`, `line` y `side`, o
  `.../pulls/{n}/comments` con `commit_id` = el `head.sha` leído al momento). Nunca `APPROVE` ni
  `REQUEST_CHANGES` salvo que el usuario lo pida.
- **GitLab**: `glab api projects/:id/merge_requests/:iid/discussions` (o el MCP de GitLab) con
  `position[...]` y los `diff_refs` (`base_sha`, `start_sha`, `head_sha`) leídos al momento.

Una línea solo se puede anclar si está en el diff de la PR/MR; si no lo está, el comentario va
como comentario general citando `archivo:línea`. Sin PR/MR no hay dónde anclar una línea: los
comentarios quedan preparados en `hallazgos.md` y se pregunta al usuario qué hacer (no se abre una
PR/MR, issue ni nada sin su autorización). Comprueba la respuesta de cada comentario: si la forja
lo rechaza, relee el commit de cabeza o los `diff_refs`, reintenta una vez y si falla déjalo
preparado con el motivo. **Nunca digas que algo está publicado sin haber verificado que se
aceptó.** No aprobar, no mergear, no resolver hilos sin evidencia y sin que el usuario lo pida.

## 8. Informe final

Archivo `informe.md` en la carpeta de la auditoría, y resumen en el chat. Contiene:

1. Resumen ejecutivo.
2. Inventario y cobertura.
3. Resultados por componente: qué se revisó, qué pruebas, problemas, riesgos, mejoras, limitaciones.
4. Resultados transversales: contradicciones y errores de integración, con causa raíz y componentes.
5. Tabla de hallazgos: `ID | Componente | Hallazgo | Severidad | Confianza | Estado | Hilo`
   (enlace verificable cuando exista, o el motivo de que no).
6. Comprobaciones: comandos, resultados, fallidas, no ejecutadas y limitaciones del entorno.
7. Deuda técnica y mejoras opcionales, separadas de los defectos que sí hay que corregir.
8. Plan priorizado: críticos, altos, bugs funcionales, inconsistencias e integración, rendimiento,
   calidad, usabilidad, deuda; con dependencias entre tareas.
9. Riesgos residuales y pruebas pendientes.
10. Conclusión: si está listo para usarse o desplegarse según su contexto, separando Confirmado,
    Probable, Hipótesis y Mejoras recomendadas.

Cuenta aparte hallazgos y comentarios (un hallazgo transversal puede tener varios): total de
hallazgos, comentarios publicados, pendientes, rechazados con su motivo y enlace a la PR/MR.

Si el usuario quiere después corregir los hallazgos, eso es otra fase y otra autorización: cada
arreglo sigue el flujo normal del proyecto (`ticket-flow` si el proyecto trabaja con tickets y
PR/MR, `craft-prompt` para preparar las sesiones que lo hagan, `conventional-commits` para los
commits).

## Cuándo se da por terminada

Inventario hecho; cada componente con resultado o limitación documentada; auditoría transversal
hecha; comprobaciones viables ejecutadas; hallazgos priorizados y con evidencia; publicación
intentada tras el OK y verificada; informe y plan hechos; riesgos residuales escritos. Si no se
puede acabar, se entrega un **informe parcial** marcado como tal, con el progreso real y lo que
queda, sin recortar profundidad para parecer completo.

## Excusas que no valen

| Excusa | Realidad |
|---|---|
| «Ya he leído los ficheros importantes» | Cada componente del inventario tiene resultado o limitación escrita. |
| «Cada módulo está bien, no hace falta mirar cómo encajan» | La fase transversal es obligatoria; ahí viven las contradicciones. |
| «Seguro que los tests pasan» | «No ejecutado, <motivo>». |
| «Lo publico y luego se lo cuento» | Primero el chat, después el OK, después la forja. |
| «Es probable, lo redacto como seguro» | Si no está confirmado, es Probable o Hipótesis. |
| «Ya que estoy, lo arreglo» | Esto es diagnóstico; el arreglo es otra fase y otra autorización. |
| «El repo es GitHub/GitLab, seguro que es como siempre» | Forja, rama base y comandos salen de la ficha o del repo, no de memoria. |
