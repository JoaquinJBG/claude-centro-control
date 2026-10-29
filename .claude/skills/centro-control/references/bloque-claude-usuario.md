# Bloque del Centro en el `CLAUDE.md` de usuario

El protocolo de `protocolo-jefes.md` se manda por mensaje, pero un mensaje se puede perder cuando la conversación
de un jefe se compacta. Para que **no se olvide nunca**, el Centro mantiene este bloque en el `CLAUDE.md` de
usuario de cada PC (`~/.claude/CLAUDE.md`, en Windows `C:\Users\<usuario>\.claude\CLAUDE.md`). Claude Code
lo carga en todas las sesiones y lo vuelve a leer tras cada compactación.

Es la **única excepción** a "solo escribes en tu carpeta". Dentro de ese archivo, el Centro solo toca lo que hay
entre las marcas `centro-control:inicio` y `centro-control:fin`. El resto del archivo es del usuario y no se toca.

## Cuándo

- **En el arranque (A.1):** comprueba que el bloque existe y que lleva el id actual del Centro (`get_session`
  con `"self"`). Si falta, añádelo al final del archivo (o créalo si no existe). Si el id o el ámbito cambiaron,
  reescribe solo el bloque. Apúntalo en la BITACORA.
- **Al cambiar `pc.md`:** si cambian los proyectos fuera de ámbito, actualiza esa línea.

## Plantilla

Sustituye `<ID_CENTRO>`, `<CARPETA_CENTRO>` y `<CARPETAS_FUERA_DE_AMBITO…>` (sale de `pc.md`).

<!-- centro-control:inicio · Lo mantiene la sesión «🛰 Centro de Control». Si cambia su id, lo actualiza ella. -->
## Aviso al Centro de Control

En este PC hay una sesión **🛰 Centro de Control** que coordina todas las sesiones de Claude Code del usuario.
Su id es `<ID_CENTRO>`. El usuario quiere que se entere al momento de lo que pasa
en cada proyecto.

**A quién se aplica.** Solo a la sesión **jefe** de un proyecto: la sesión con la que habla el usuario sobre ese
proyecto y que crea las hijas. Eres jefe si trabajas en un proyecto y nadie te inició (tus instrucciones no dicen
que te inició otra sesión), o si te inició el propio Centro de Control. **No se aplica:**
- al Centro de Control (carpeta `<CARPETA_CENTRO>`);
- a las sesiones que inició otra sesión que no es el Centro: esas informan a su jefe como siempre;
- a las conversaciones sueltas que no trabajan en ningún proyecto;
- a los proyectos fuera del ámbito del Centro en este PC: <CARPETAS_FUERA_DE_AMBITO o «ninguno»>.

Si te inició el Centro y sigues anidada bajo él, tus propios turnos ya le llegan solos: avísale solo de lo
que hagan tus hijas. Si el Centro te dice que te ha desanclado (lo hace para ordenar la barra lateral), avísale
también de tus propios turnos: cuando termines, te bloquees o tengas una pregunta para el usuario.

**Qué hacer.** Manda un mensaje con `SendMessage` (`to`: el id de arriba; si la herramienta no está cargada,
cárgala con `ToolSearch` y `select:SendMessage`) cada vez que:
- termines una tarea o una fase (con rama y commits);
- una de tus hijas termine, se bloquee o pregunte algo;
- necesites una decisión del usuario;
- crees una hija o la dejes como tarjeta para que el usuario la pulse (nombre e id si ya lo tiene);
- algo falle o se bloquee.

Formato (máximo unas 10 líneas):
```
[PROYECTO] EVENTO: resumen en una frase
ESTADO: hecho | en marcha | bloqueado | espera al usuario
RAMA/COMMITS: …
NECESITA DEL USUARIO: … (o "nada")
SESIÓN: name e id, si el evento es de una hija
```

No avises en turnos sin novedades ni cuando el usuario solo está charlando contigo. No esperes respuesta: el
usuario sigue hablando contigo como siempre, y el Centro solo lleva el registro y el panel. Si el Centro te pide
algo, sigue tus normas de siempre. Si el envío falla porque ese id ya no existe, díselo al usuario una vez y sigue.
<!-- centro-control:fin -->
