# Plantilla de brief para sesiones trabajadoras

Rellena cada bloque con datos reales: rutas, nombres de funciones, comandos. Borra los bloques que no apliquen.
El `prompt` es lo que la trabajadora tiene que hacer. El `background` es el contexto para alguien que no ha
visto nada: debe tener ≥80 caracteres.

## prompt

```
# Tarea: <título corto>

## Objetivo
<Qué tiene que existir o funcionar al terminar, en 1-3 frases.>

## Dónde
- Repo: `<ruta>` (rama base `<rama>`)
- Archivos o zonas a tocar: `<ruta/a/archivo>`, `<carpeta/>`
- Puntos de entrada útiles: `<archivo:línea>`, `<comando para arrancar>`

## Fuera de alcance
- <Lo que NO debe tocar, sobre todo lo que estén tocando otras sesiones.>

## Hecho cuando
- [ ] <Criterio verificable 1, p. ej. "el endpoint POST /x devuelve 201 con …">
- [ ] <Criterio 2, p. ej. "los tests de `backend/tests/test_x.py` pasan">
- [ ] Hay commits en tu rama con mensajes claros

## Skills
<"Antes de empezar, lee entero y sigue `…\biblioteca-skills\<nombre>\SKILL.md`", o "ninguna".>

## Reglas
- Trabajas en tu propio worktree. Puedes hacer commits. **No hagas push, ni abras PR, ni hagas merge.**
- No toques nada fuera de "Dónde" sin preguntar.
- Si te falta una decisión que no puedes deducir del código, **pregunta y para**. No lo resuelvas
  por tu cuenta en temas de producto, datos o seguridad.
- No instales dependencias nuevas sin explicarlo en el informe.

## Informe final (obligatorio, con este formato y en este orden)
RESUMEN: <1-2 frases, lo primero que se lee>
ESTADO: hecho | parcial | bloqueado
RAMA: <nombre completo de la rama> · COMMITS: <n>
CAMBIOS: <lista de archivos y qué cambió>
PRUEBAS: <qué ejecutaste y el resultado>
PENDIENTE: <lo que queda, si queda algo>
PREGUNTAS: <las decisiones que necesitas, si hay alguna>
```

## background

```
Proyecto <nombre>: <qué es en una frase, del catálogo proyectos.md>. Esta tarea forma parte de
<objetivo mayor del usuario>. Decisiones ya tomadas: <[DECISIÓN] relevantes de la BITACORA>.
Otras sesiones activas en este repo: <name: zona que tocan>, así que no toques <zona>.
Te ha lanzado el Centro de Control; tu informe final lo leerá él primero.
```

## Por qué el informe tiene ese formato

El aviso que recibe el Centro de Control solo cita los primeros 400 caracteres del informe.
Con `RESUMEN` y `ESTADO` delante, esos 400 caracteres bastan para clasificar el aviso sin leer el transcript entero.
