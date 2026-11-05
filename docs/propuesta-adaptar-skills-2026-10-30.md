# Adaptar 3 ideas de Matt Pocock y pstack a tus skills

Propuesta del 2026-10-30 (versión sin datos del trabajo, para el repo del Centro). **No se ha editado nada**: ni skills, ni `CLAUDE.md`, ni repos. Tú eliges qué se aplica.
Viene de una investigación previa: no instalar los packs y quedarse con 3 ideas.

## Conclusión

1. **Explicar el cambio → skill nueva y corta `explicame-el-cambio`** + 3 enganches en `ticket-flow`. Hace cumplir tu regla de hoy: ninguna tarea se cierra sin tu «entendido», y en front va archivo a archivo con la pantalla probada en local.
2. **Pasos que no se saltan → `ticket-flow` copia sus 12 pasos a la lista de tareas** al empezar. Un paso que no se hace se queda como `saltado: <motivo>` y aparece en la MR y en el mensaje final.
3. **Principios → 5 frases en `~/.claude/CLAUDE.md`** (≈300 tokens por turno), encima del bloque del Centro. El resto de los 24 ya lo cubren superpowers o `ticket-flow`, o chocan con tu forma de trabajar.
4. **Los plugins del equipo no se tocan**: tu regla global ya hace que los prompts de `craft-prompt` del trabajo digan «Follow the ticket-flow skill», así que los cambios llegan solos.

Coste total: ≈95 tokens fijos por turno (descripción de la skill) + ≈300 de principios. Lo demás solo se carga al usar la skill.

---

## Idea 1 · Explicar cada cambio hasta que lo entiendas

**De dónde sale**
- `wait-what` de Matt Pocock ([skills/productivity/wait-what](https://github.com/mattpocock/skills/tree/main/skills/productivity/wait-what), MIT). Son 3 líneas. La idea: cuando no entiendes, no se pide «sé breve» (eso da frases cortadas), sino que se dice dónde te has perdido; el agente vuelve atrás, añade el contexto que faltaba y usa el vocabulario del proyecto.
- `teach` de pstack ([cursor/plugins/pstack/skills/teach](https://github.com/cursor/plugins/tree/main/pstack/skills/teach), Lauren Tan, MIT). Es la que más se parece a lo que buscas: explicar un cambio para que se entienda, lo más corto y completo primero, conversación y no clase magistral, sin cuestionarios, diagramas que se construyen pieza a pieza.
- El `teach` de **Matt no sirve aquí**: monta un espacio de aprendizaje de varias sesiones (MISSION.md, lecciones HTML, fichas). Es para aprender un tema, no para revisar un cambio.

**Dónde va y por qué: skill nueva + enganche en `ticket-flow`**

| Opción | Por qué no / por qué sí |
|---|---|
| Solo dentro de `craft-review` | ❌ Su salida es un prompt de review en inglés o un veredicto SUBIR/NO SUBIR. Busca fallos; esto busca que tú entiendas. Otro objetivo y otro momento. |
| Solo como paso de `ticket-flow` | ❌ `ticket-flow` solo cubre los repos de trabajo con MR. Tu regla vale para **toda** tarea que se cierra (cambios sin MR, tareas del Centro, proyectos personales). |
| **Skill nueva + 3 enganches en `ticket-flow`** | ✅ La invocas tú cuando quieras («explícame el cambio», «no lo entiendo») y `ticket-flow` la llama sola antes de pasar a ready. |

Ojo: la regla está en `pc.md`, que es del Centro. Las sesiones trabajadoras no lo leen. Por eso la barrera tiene que estar escrita en la skill y en `ticket-flow`, no solo en el Centro.

### 1a. Archivo nuevo: `~/.claude/skills\explicame-el-cambio\SKILL.md`

```markdown
---
name: explicame-el-cambio
description: Use when a task, ticket or MR is done and el usuario must review and understand it before closing it, when ticket-flow reaches its final message, or when he says "explícame el cambio", "no lo entiendo", "espera, ¿qué?". Explains what changed and why at his pace and waits for his «entendido».
---

# Explícame el cambio

Objetivo: que el usuario entienda el cambio antes de cerrarlo. Cada tarea se revisa y se entiende antes de darla por cerrada. Aquí no cambias código: explicas.

## Primera explicación

1. Lee el ticket (RN, AC, decisiones) y `git diff origin/<base>...HEAD --stat`. Usa las palabras del ticket y del código; no inventes nombres.
2. Empieza por lo que nota quien usa el cambio (el cliente, otra pantalla, otro servicio) y por lo que hereda quien lo mantenga. Después, el mecanismo.
3. Lo más corto que esté completo: 3 a 5 frases. Luego los archivos clave, uno por línea, con `fichero:línea`: qué hace ahí el cambio y por qué.
4. Con 3 o más piezas que se mueven, un diagrama por paso: primero A → B, luego el mismo con C añadido. No uno con todo al final.
5. Decisiones tomadas y lo que queda fuera, una línea cada una.

## Front (repos web)

- El diff archivo a archivo, en el orden en que viaja el dato (API → estado → componente → pantalla). Tras cada archivo, para y deja que pregunte.
- La pantalla, para probarla en local: comando, URL, usuario o datos, qué hacer y qué tiene que verse.
- Antes de aceptar «probado», pregúntale si abrió la pantalla en local. Si no, todavía no está probado.

## Si dice «no lo entiendo» o «espera, ¿qué?»

No resumas más. Vuelve atrás hasta donde se perdió, dale el contexto que le faltaba y cuéntalo otra vez con frases simples y las palabras del proyecto. Más claro, no más corto. Si lo pide dos veces seguidas, no te vuelvas telegráfico.

## Cierre

Una sola pregunta abierta: «¿Algo que no cuadre? Cuando lo tengas claro, dime "entendido"». Sin cuestionarios ni «repítemelo». Hasta su «entendido», la tarea no está cerrada y no se abre otra.

Ideas adaptadas de `teach` de pstack (Lauren Tan, MIT) y `wait-what` de mattpocock/skills (Matt Pocock, MIT).
```

Decisiones de diseño:
- **Sin cuestionario.** El `teach` de pstack los prohíbe y Matt insiste en que una skill contra la verbosidad que se alarga fracasa. Basta con una pregunta abierta y tu «entendido».
- **Sin `GLOSSARY.md`.** Matt usa uno; tú no lo tienes. Hace su papel el vocabulario del ticket (RN, AC, decisiones) y del código.
- **Tiene que poder invocarla el modelo** (sin `disable-model-invocation`) para que `ticket-flow` la llame. Precio: su descripción está en todos los turnos de todas las sesiones.

### 1b. Enganches en `~/.claude/skills\ticket-flow\SKILL.md`

Mensaje final (líneas 104-109):

```diff
 ## Mensaje final
 
 1. Enlace a la MR (Draft) y al ticket, con el estado de la pipeline y el de Jira: «Jira: En curso, no se mueve hasta tu "probado"».
 2. Qué se verificó (comandos, y lo observado en la app) · qué no y por qué · decisiones tomadas · lo que queda fuera.
+3. **Qué cambió y por qué**, con la skill `explicame-el-cambio` (en los repos de front, archivo a archivo).
-3. **«Pruébalo tú antes de pasarla a ready»**: el happy path, numerado, que cubra todos los criterios de aceptación. En cada paso va qué levantar (comando, URL), con qué datos (la instalación, el usuario o el fichero concretos del entorno local, si los conoces), qué hacer y qué tiene que verse. Después, dos o tres casos límite de la tabla que merezca la pena probar a mano.
-4. Cierra con: «Cuando lo hayas probado, dime "probado" y la paso a ready».
+4. **«Pruébalo tú antes de pasarla a ready»**: el happy path, numerado, que cubra todos los criterios de aceptación. En cada paso va qué levantar (comando, URL), con qué datos (la instalación, el usuario o el fichero concretos del entorno local, si los conoces), qué hacer y qué tiene que verse. Después, dos o tres casos límite de la tabla que merezca la pena probar a mano.
+5. Cierra con: «Cuando lo tengas claro y probado, dime "entendido" y "probado" y la paso a ready».
```

(El texto del happy path no cambia; solo pasa de 3 a 4.)

Paso 7 (líneas 111-114):

```diff
-## 7. Cuando el usuario diga «probado»
+## 7. Cuando el usuario diga «entendido» y «probado»
 
+1. Si todavía no ha dicho «entendido», vuelve a `explicame-el-cambio` antes de nada. En los repos de front, pregúntale si abrió la pantalla en local: si no, todavía no es «probado».
-1. Mira en Jira si el revisor contestó a las decisiones pendientes: …
+2. Mira en Jira si el revisor contestó a las decisiones pendientes: …
-2. Antes de pasarla, mira la pipeline de la cabeza de la rama: …
+3. Antes de pasarla, mira la pipeline de la cabeza de la rama: …
-3. Jira, según el momento. El flujo depende del proyecto de la **clave del ticket**, no del repo:
+4. Jira, según el momento. El flujo depende del proyecto de la **clave del ticket**, no del repo:
```

(En los puntos 1 y 2 antiguos solo cambia el número; `…` es el resto de la línea, igual que hoy.)

Tabla «Excusas que no valen» (tras la línea 138):

```diff
 | «Lo he visto funcionar yo, la abro en ready» | Draft hasta que el usuario la pruebe con el happy path. |
+| «Ya le di el resumen, lo habrá entendido» | Sin su «entendido» no pasa a ready ni se abre otro ticket. |
```

**Qué ganas**
- Tu regla de hoy deja de depender de que te acuerdes: la MR no pasa a ready sin tu «entendido» y, en front, sin la pantalla probada en local.
- «No lo entiendo» tiene una respuesta fija: más contexto, no menos palabras.
- Al día siguiente sabes qué hiciste, que es lo que busca el límite de 2 tickets.

**Coste**
- Fijo: ≈95 tokens por turno (descripción de ~340 caracteres).
- Al usarla: ≈640 tokens (la skill entera, ~2.200 caracteres).
- `ticket-flow` crece ≈80 tokens.
- En tiempo: unos minutos tuyos por ticket. Es justo lo que pide la regla.

---

## Idea 2 · Pasos que no se saltan

**De dónde sale.** `poteto-mode` de pstack ([SKILL.md](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/SKILL.md), sección Playbooks, MIT). Son dos frases: los pasos del playbook se copian **tal cual** como primeras tareas de la lista, antes de las propias; y un paso que no se hace **se queda** en la lista con `skip: <motivo>` en una línea.

**Por qué gana fiabilidad `ticket-flow`.** Hoy la línea 16 ya dice que saltarse un paso es saltárselo, pero solo es una frase. En una sesión larga, el agente olvida lo que no tiene delante. Con la lista:
- los pasos están en la lista que ves bajo cada mensaje y que el agente marca uno a uno;
- un salto se ve (`saltado: …`), no desaparece en silencio;
- tú, o el Centro, veis al momento en qué paso va.

**Cambio en `ticket-flow/SKILL.md`, línea 16**

```diff
-**Saltarse un paso por las prisas o porque «el cambio es pequeño» es saltárselo.** Si de verdad no se puede hacer uno, dilo en la MR («No verificado, y por qué») y en el mensaje final.
+**Al empezar, copia esta lista tal cual (sin los números) en tu lista de tareas** (`update_plan` en la app, `TaskCreate` en la CLI), antes de las tareas propias del ticket:
+
+1. Ticket: buscar, leer código, decisiones
+2. Plantear: ¿se parte la MR?
+3. Rama creada y empujada
+4. Implementar con TDD, push por commit
+5. Verificación completa del repo
+6. Verlo funcionar en la app
+7. Autorevisión en subagente
+8. Advisor con diff y verificación
+9. MR en Draft y comentario en Jira
+10. Mensaje final y explicar el cambio
+11. Esperar «entendido» y «probado»
+12. Ready, revisor y Jira
+
+Ningún paso se borra. Si no lo haces, cambia su texto a `saltado: <motivo en una línea>` y márcalo hecho. **Las prisas o «el cambio es pequeño» no son un motivo.** Cada `saltado` va también a la MR («No verificado, y por qué») y al mensaje final.
```

Notas:
- Son 12 pasos porque `update_plan` admite como mucho 12 y unas 7 palabras por paso. El paso 5 de la skill se abre en sus 4 subpasos (5 a 8), que es donde más se recorta.
- Los pasos 10 y 11 dependen de la idea 1. Si no aplicas la idea 1, quedan en 11 pasos: «Mensaje final con happy path» y «Esperar «probado»».
- En los repos con su propio flujo de tickets manda ese flujo. Si quieres lo mismo allí, va en ese repo (ver «Fuera de este cambio»).

**Qué ganas.** Menos pasos olvidados en sesiones largas, sobre todo la autorevisión y «verlo funcionar», que son los que más se recortan. Y un saltado siempre lleva su motivo.

**Coste.** ≈150 tokens más en `ticket-flow`, solo cuando se usa. Cero fijo.

**Descarto** `check-plan.mjs`, el script de Node con el que pstack valida que los planes tengan sus casillas. Es demasiado pesado para lo que aporta aquí.

---

## Idea 3 · Principios cortos

**De dónde sale.** Los 24 principios de pstack ([guía](https://github.com/cursor/plugins/blob/main/pstack/docs/guide/08-principles.md) y carpetas `skills/principle-*`, MIT). Su mecanismo: cada principio tiene un nombre corto; el agente dice qué principio cambió qué decisión, y tú lo corriges citando el nombre («aplica lo mínimo»).

**Cuáles me quedo (5) y por qué no los cubre ya otra cosa**

| Principio (nombre aquí) | Original | Qué aporta |
|---|---|---|
| Lo mínimo | Laziness Protocol + Subtract Before You Add | Nada en tus reglas pide el diff más pequeño ni prohíbe guardas «por si acaso». |
| Fácil de leer | Minimize Reader Load | Va con tu regla de entender cada cambio: menos capas y menos estado que seguir. |
| Guarda solo si el estado es legítimo | La parte no cubierta de Fix Root Causes | `systematic-debugging` pide la causa raíz, pero no esta prueba concreta ni «arregla el patrón en todos los sitios». |
| Valida en el borde | Boundary Discipline | Encaja con schemas de Ninja y Next: validar al entrar y no repetir comprobaciones de `None` por dentro. |
| Lección repetida, a estructura | Encode Lessons in Structure | Tú vas añadiendo reglas y skills. Esta dice cuándo una regla debe ser un lint o un test en vez de más texto. |

**Dónde: `~/.claude/CLAUDE.md`, entre la línea 6 y la marca `<!-- centro-control:inicio … -->` de la línea 8** (fuera del bloque del Centro)

```diff
 - Keep code comments short and rare: only explain a non-obvious *why*. No comments narrating what the code does, what changed, or the ticket/task history.
 
+## Principios (adaptados de pstack, MIT)
+
+Cuando uno cambie una decisión, nómbralo junto a la decisión que cambió: un nombre sin decisión no es aplicarlo. El usuario puede corregirte citando el nombre.
+
+- **Lo mínimo.** El diff más pequeño que resuelve. Primero quita lo que sobra, luego añade. Sin validaciones, guardas ni capas «por si acaso».
+- **Fácil de leer.** Quita envoltorios con un solo llamador y capas que solo pasan argumentos. Prueba: «¿de dónde sale X y qué lo cambia?» en 30 segundos.
+- **Guarda solo si el estado es legítimo.** Si el estado no debería existir, busca quién lo produce; si puede existir, trátalo donde rompió. Arregla el patrón en todos los sitios, no solo el caso.
+- **Valida en el borde.** Valida y convierte al entrar (schemas, parámetros, APIs externas). Dentro, confía en los tipos y no vuelvas a comprobar.
+- **Lección repetida, a estructura.** Si corriges lo mismo por segunda vez, propón un lint, un test, un script o una línea en la skill, no otro recordatorio.
+
 <!-- centro-control:inicio · Lo mantiene la sesión «🛰 Centro de Control». Si cambia su id, lo actualiza ella. -->
```

**Global o en `ticket-flow`**

| Dónde | A favor | En contra |
|---|---|---|
| **`~/.claude/CLAUDE.md`** (recomendado) | Vale en todo: trabajo, Centro, proyectos personales, reviews. | ≈300 tokens (~1.030 caracteres) en cada turno de cada sesión. |
| `ticket-flow` | Gratis fuera de los tickets. | No está en reviews, charts sin MR ni proyectos personales. |

«Valida en el borde» no choca con la fila «Valores» de los casos límite de `ticket-flow` (`null`, vacío…): esa fila dice qué casos tratar y el principio dice dónde tratarlos.

Recomiendo el global porque se queda en el límite de ~300 tokens. Si algún día crece, la regla «Lección repetida, a estructura» dice qué hacer: pasar lo que se pueda a lint o test y sacarlo de aquí.

**Qué ganas.** Menos código de más (guardas, capas, validaciones repetidas), que es lo que alarga tus diffs y tus explicaciones. Y una forma rápida de corregir: «lo mínimo» en vez de un párrafo.

---

## Lo que descarto

| Principio de pstack | Por qué no |
|---|---|
| Fix Root Causes (núcleo) | Ya está en `superpowers:systematic-debugging`. Solo me quedo la parte de la guarda legítima. |
| Attack the Premise | `systematic-debugging` (tras 3 arreglos fallidos, cuestionar la arquitectura) y tu regla global de consultar al advisor si el error se repite. |
| Test Behavior, Not Implementation | `superpowers:test-driven-development`, referencia `writing-good-tests.md` (los mocks no se comprueban, se prueba el comportamiento). |
| Prove It Works | `superpowers:verification-before-completion` y el paso 5.2 de `ticket-flow` («Verlo funcionar»). |
| Make Operations Idempotent | Fila «Estado» de la tabla de casos límite de `ticket-flow` (reintento, idempotencia). |
| Guard the Context Window | Paso 2.4 de `ticket-flow` (búsquedas amplias a `explorer`). |
| Sequence Verifiable Units | TDD de superpowers y el paso 4 de `ticket-flow` (push por commit). |
| Never Block on the Human | **Choca** con tu forma de trabajar: una pregunta con recomendación y OK antes de lo que importa. |
| Migrate Callers Then Delete Legacy APIs | **Choca** con `ticket-flow` 2.1: un refactor que no pide el ticket va en su propia MR. |
| Explain the Number | Para benchmarks; no es tu día a día. |
| Model the Domain, Type System Discipline, Foundational Thinking, Redesign from First Principles, Outcome-Oriented Execution, Experience First, Exhaust the Design Space, Build the Lever, Separate Before Serializing Shared State | Buenos pero genéricos. Con 5 bien elegidos se pueden recordar. Con 15, ninguno. |

Tampoco adapto:
- **El `teach` de Matt** (espacio de aprendizaje con lecciones HTML): otro objetivo.
- **El router `/poteto-mode`, las reglas de redacción de pstack, sus subagentes y modelos**: chocan con `ticket-flow` y `craft-prompt` y están pensados para Cursor.

---

## ⚠️ Plugins del equipo

- Ningún cambio de esta propuesta toca los plugins compartidos del equipo. Si algún día `explicame-el-cambio` debe ser del equipo, se mueve a su plugin **por MR**, no a mano.

## Fuera de este cambio (por si lo quieres después)

- **Repos con su propio flujo de tickets** (p. ej. un `ticket-workflow` propio): mandan sobre `ticket-flow`. Los enganches de las ideas 1 y 2 allí van por MR en ese repo, con su propio ticket.
- **El Centro**: podría citar `explicame-el-cambio` cuando una tarea espera tu revisión. Eso es un cambio del kit del Centro, no de esta propuesta.

## Orden sugerido para aplicarlo

1. Idea 1 (skill + enganches). Es la que pide tu regla de hoy.
2. Idea 2 (lista en `ticket-flow`). Va junto a la 1: comparten los pasos 10 y 11.
3. Idea 3 (principios). Es independiente; se puede probar una semana y quitar si no aporta.

## Fuentes (MIT las dos)

- Matt Pocock, `mattpocock/skills`: [wait-what](https://github.com/mattpocock/skills/blob/main/skills/productivity/wait-what/SKILL.md), [su explicación](https://github.com/mattpocock/skills/blob/main/docs/productivity/wait-what.md), [teach](https://github.com/mattpocock/skills/blob/main/skills/productivity/teach/SKILL.md).
- Lauren Tan (poteto), `cursor/plugins/pstack`: [teach](https://github.com/cursor/plugins/blob/main/pstack/skills/teach/SKILL.md), [poteto-mode](https://github.com/cursor/plugins/blob/main/pstack/skills/poteto-mode/SKILL.md), [guía](https://github.com/cursor/plugins/blob/main/pstack/docs/guide/README.md), [principios](https://github.com/cursor/plugins/blob/main/pstack/docs/guide/08-principles.md).
- Todo está escrito con mis palabras; no hay bloques copiados.
