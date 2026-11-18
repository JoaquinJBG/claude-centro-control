---
name: explicame-el-cambio
description: Use when a task, ticket or PR/MR is done and the user must review and understand it before closing it, when ticket-flow reaches its final message, or when the user says "explícame el cambio", "no lo entiendo", "espera, ¿qué?". Explains what changed and why at the user's pace and waits for their «entendido».
---

# Explícame el cambio

Objetivo: que el usuario entienda el cambio antes de cerrarlo. Cada tarea se revisa y se entiende antes de darla por cerrada. Aquí no cambias código: explicas.

## Primera explicación

1. Lee el ticket o el encargo (reglas, criterios, decisiones) y `git diff origin/<base>...HEAD --stat`. Usa las palabras del ticket y del código; no inventes nombres.
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
