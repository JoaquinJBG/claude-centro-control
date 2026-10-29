# Este PC

El Centro de Control lee este archivo en cada arranque. Fija el **ámbito** y las reglas que cambian de un PC
a otro. Es de este PC: está en `.gitignore` y no se sube. En un PC nuevo, el arranque lo copia desde
`plantillas/pc.md` y le pregunta al usuario los huecos.

## Ámbito
- **Rol:** personal | trabajo
- **Orquesta:** <qué proyectos de `proyectos.md` lleva este centro; p. ej. "todos" o "los de trabajo y, si se lo pido, los personales">
- **Fuera de ámbito (nunca los toca, ni lanza ni mensajea sesiones allí):** <rutas o nombres; en el PC personal, todo lo del trabajo>

## Push, PR/MR y merge
- **Las trabajadoras pueden hacer push y abrir PR/MR:** no | sí, en borrador, siguiendo la skill del flujo del proyecto (`ticket-flow`)
- **Marcar la PR/MR como lista para revisión:** solo con OK del usuario
- **Merge:** nunca (lo hace el usuario)

## Skills
- **Skills propias de este PC** (en `~/.claude/skills`) que mandan sobre las de `biblioteca-skills/` con el mismo nombre:
  <p. ej. "ticket-flow, review-mr (versiones del trabajo)"; "ninguna">

## Jefes de ciclo
Un jefe de ciclo es una sesión que hace de jefe de **varios** proyectos durante un periodo fijo (un sprint, por
ejemplo). Al acabar el ciclo se cierra y se abre otro jefe para el ciclo siguiente.

| ciclo | jefe (name) | id | proyectos que cubre | desde | hasta | cómo se renueva |
|---|---|---|---|---|---|---|
<!-- | Sprint 22/10–05/11 | sprint | local_… | api, web, móvil | 2026-10-22 | 2026-11-05 | cada 2 semanas (jueves): el centro propone cerrar y lanzar el siguiente | -->
