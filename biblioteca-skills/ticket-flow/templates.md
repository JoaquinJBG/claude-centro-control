# Plantillas

## Ticket (solo con tracker)

Si el proyecto tiene su plantilla o su skill de tickets, usa esa y no esta. No se escribe en el tracker sin el OK del usuario.

```markdown
## Contexto

<Por qué se pide, en lenguaje de producto. Quién lo pidió.>

## Reglas de negocio

- RN1: <regla que el código tiene que cumplir>

## Criterios de aceptación

- AC1: <comprobable: dado … cuando … entonces …>

## Anclajes

- `<fichero>:<línea>`: <qué hay ahí y por qué importa>

## Preguntas abiertas

- PQ1: <pregunta> · Recomendación: <opción>

## Decisiones

- <decisión tomada y por qué; «Pendiente de confirmar» si espera respuesta>

## Fuera de alcance

- <lo que no entra, con su ticket de seguimiento>
```

## PR/MR (descripción)

Si el repo tiene su plantilla (`.github/pull_request_template.md`, `.gitlab/merge_request_templates/default.md`), usa esa y no esta.

```markdown
## Qué cambia

<Una o dos frases en lenguaje de producto.>

<Con ticket: «Closes <CLAVE>» o «Closes #<n>» (o «Refs #<n>» si mergear no es desplegar). Sin ticket: quítalo.>

## Verificado de punta a punta

- <lo que se vio funcionar: comando, pantalla o E2E — no «compila»>

## No verificado, y por qué

- <honesto; si está vacío, o se verificó todo o no se ha pensado>

## Decisiones y supuestos

- <cada decisión, con su porqué; lo que se da por cierto sin haberlo comprobado>

## Fuera de alcance

- <lo que no entra, con su ticket o issue de seguimiento>

## Cómo probarlo

1. <happy path: qué levantar, con qué datos, qué hacer y qué se ve; un paso por criterio de aceptación>

## Notas para el revisor

- Autorevisión (paso 5.3): <qué revisión corrió, hallazgos arreglados, hallazgos que se quedan y por qué>
- <decisiones discutibles, desviaciones del encargo, limpieza de lint en commit aparte>
```

Si cambia un modelo de datos: nota de impacto en tests y migración. Si cambia el contrato de la API: la spec regenerada en el mismo commit.
