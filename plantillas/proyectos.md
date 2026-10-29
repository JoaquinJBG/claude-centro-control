# Catálogo de proyectos

El Centro de Control saca de aquí la `cwd` de cada lanzamiento. Cuando aparezca una carpeta nueva en una
sesión, el Centro de Control la añade como `(por completar)` y el usuario la completa o se la dicta.
Aquí van los proyectos del ámbito de este PC (`pc.md`). Las skills genéricas (`ticket-flow`, `review-mr`,
`craft-review`…) leen de la ficha la forja, el tracker, el flujo de cambios y los checks.

Este archivo es de cada PC (está en `.gitignore`), porque las rutas cambian de un PC a otro.

---

<!-- Proyectos de este PC: el primer arranque los detecta a partir de las sesiones -->

---

## Plantilla
```
## <Nombre>
- **Ruta:** `<ruta absoluta>`
- **Git:** sí/no · rama base `<rama>` · origin `<url>`
- **Forja:** GitHub (`gh`) | GitLab (`glab` o MCP) | ninguna
- **Tracker:** ninguno | Jira (`<PREFIJO>-`) | GitHub Issues | GitLab Issues
- **Flujo de cambios:** PR/MR en borrador → lista para revisión con OK | commit directo
- **Qué es:** <una frase>
- **Stack:** <…>
- **Zonas:** <carpetas principales>
- **Checks:** <comandos de test y lint>
- **Notas:** <convenciones, cosas a no tocar>
```
