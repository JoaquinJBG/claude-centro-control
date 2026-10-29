# Catálogo de skills para trabajadoras

Son las skills que el usuario le da al Centro de Control para que las asigne a las sesiones que lanza.
Cada una vive en `biblioteca-skills/<nombre>/SKILL.md`. Para dar una nueva, basta con decirle al
Centro de Control "guarda esta skill" y pegarla o darle la ruta.

Son **genéricas**: valen para cualquier repo git. Lo propio de cada proyecto (forja GitHub/GitLab, tracker,
rama base, checks) lo leen de su ficha en `proyectos.md` o del propio repo. Si un PC tiene su propia versión
de una skill con el mismo nombre (`pc.md` → Skills), allí manda la propia. Para usarlas tú con `/nombre`,
pide al centro "instala las skills" (procedimiento K).

| Nombre | Para qué | Cuándo asignarla | Proyectos | Modelo mín. |
|---|---|---|---|---|
| ticket-flow | Del encargo a la PR/MR en revisión: rama, trabajo, verificación, PR/MR en borrador y lista solo con OK; ticket opcional si el proyecto tiene tracker | Toda trabajadora que escribe código y cuyo trabajo debe acabar en una PR/MR (si `pc.md` permite push y PR/MR) | todos | Sonnet |
| craft-prompt | Convierte un encargo en estrategia de ejecución (una sesión, paralelo por worktrees, fases, Fable o ultracode, verificación, dónde entra el usuario) y en los prompts en inglés con modelo, effort y ultracode por pieza | Antes de lanzar una tarea grande o difusa: la trabajadora la invoca primero con el contexto y sigue el prompt resultante | todos | Opus |
| craft-review | Revisar el trabajo propio antes de subirlo: prompt de code review en inglés (modelo, effort, ultracode) o la review ejecutada con veredicto | Cuando una trabajadora termina un cambio, antes del push o de marcar la PR/MR lista | todos | Sonnet (Opus para orquestar diffs grandes) |
| review-mr | Revisar una PR/MR ya abierta (GitHub o GitLab), normalmente ajena, con su ticket o issues e historia; hallazgos en el chat y comentarios en línea solo tras el OK | Cuando el usuario pide revisar o hacer la CR de una PR/MR existente (URL, #n o !n) | todos | Sonnet (orquestador Opus) |
| auditoria-integral | Auditoría de solo lectura de un repo entero o una PR/MR enorme: inventario, revisión por componente, auditoría transversal, checks, hallazgos con ID e informe; hilos solo tras el OK | "Auditoría integral", "revisa el proyecto entero", o una PR/MR demasiado grande para review-mr | todos | Opus (finders en Sonnet) |
| conventional-commits | Formato Conventional Commits 1.0.0 para los mensajes de commit | Toda trabajadora que hace commits, salvo que el repo tenga otra convención | todos | Haiku |
