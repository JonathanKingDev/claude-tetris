# Guía de triage de issues (para Claude)

Este documento define cómo se debe analizar y etiquetar cada issue de `claude-tetris`. Es usado por el
workflow `.github/workflows/claude-issue-triage.yml`, que se dispara al abrir o editar un issue.

## Contexto del proyecto

Tetris clásico en JavaScript vanilla (ES6+), HTML5 Canvas y CSS. Sin dependencias ni build. Tres archivos:

- `index.html` — DOM: canvas `#board` (300×600 = `COLS×BLOCK` por `ROWS×BLOCK`), `#next-canvas` (120×120),
  HUD (`#score`, `#lines`, `#level`), `#overlay` (pausa / game over).
- `style.css` — estilo retro/arcade; `.overlay.hidden` alterna la visibilidad del overlay.
- `game.js` — toda la lógica en un único script global. Funciones/constantes clave:
  - `createBoard`, `PIECES`, `rotateCW`, `collide`, `tryRotate` (wall kicks: offsets `[0,-1,1,-2,2]`)
  - `lockPiece` → `merge` + `clearLines` + `spawn` (game over si el spawn colisiona)
  - `ghostY` (pieza fantasma), `draw`, `loop` (bucle con `requestAnimationFrame`, `dropAccum`)
  - Scoring: `LINE_SCORES = [0,100,300,500,800]` × `level`; `dropInterval = max(100, 1000-(level-1)*90)`
  - Listener único de `keydown` al final del archivo (flechas, Space, `P`); `restart-btn` → `init()`
  - Constantes: `COLS`, `ROWS`, `BLOCK`, `COLORS`

## Paso 1 — Leer el issue

```
gh issue view <N> --json title,body,labels,comments
```

## Paso 2 — Investigar el código

Lee `game.js` (y `index.html`/`style.css` si aplica) para localizar las funciones y líneas concretas
relacionadas con lo descrito. **No especules**: toda referencia a código debe citar `game.js:línea` real,
verificada leyendo el archivo en este checkout, no de memoria.

## Paso 3 — Etiquetar

Aplica exactamente:
- **1 label de tipo** (uno de los ya existentes en el repo): `bug`, `enhancement`, `documentation`,
  `question`, `accessibility`, `duplicate`, `invalid`, `wontfix`.
- **1 o más `area:*`**: `area:gameplay`, `area:rendering`, `area:input`, `area:scoring`, `area:ui`.
- **1 `priority:*`**: `priority:high` (rompe el juego / bloquea uso normal), `priority:medium` (molesto,
  con workaround), `priority:low` (cosmético o mejora menor).
- Opcionalmente `good first issue` / `help wanted` si aplica.
- Añade siempre `triaged` al terminar.

```
gh issue edit <N> --add-label "bug,area:gameplay,priority:high,triaged"
```

Si el issue duplica otro o es inválido, etiqueta `duplicate`/`invalid` + `triaged`, comenta la razón
brevemente (referenciando el issue original si es duplicado) y **detente** — no hace falta diagnóstico
técnico completo.

## Paso 4 — Normalizar el título

Si el título no lleva ya un prefijo `[area]`, renómbralo a `[área-principal] Título conciso en el idioma
original del issue`, manteniendo el sentido. Si ya tiene un prefijo de área correcto, no lo toques.

```
gh issue edit <N> --title "[gameplay] Título conciso"
```

## Paso 5 — Publicar el diagnóstico

Idioma: **el mismo en que está escrito el issue** (si el issue está en inglés, responde en inglés; si
está en español, en español).

Antes de comentar, comprueba si ya existe un comentario propio con el encabezado
`## 🤖 Diagnóstico automático` (caso de reejecución por edición del issue). Si existe, **edítalo**
(`gh issue comment <N> --edit-last`) en lugar de crear uno nuevo. Si no existe, créalo
(`gh issue comment <N> --body-file -` o `--body`).

Plantilla (traduce los encabezados si el issue está en otro idioma, pero conserva la estructura):

```markdown
## 🤖 Diagnóstico automático

**Clasificación:** <tipo> · <áreas> · <prioridad>

### Resumen
<1-2 frases reformulando el problema/petición en términos técnicos>

### Reproducción / comportamiento esperado
<pasos si es bug; criterio de "hecho" si es enhancement>

### Causa probable / puntos de intervención
- `game.js:NNN` — `nombreFunción()` — por qué es relevante
- ...

### Enfoque sugerido
<2-5 pasos concretos de implementación, sin escribir el código>

### Riesgos y efectos colaterales
<qué se puede romper: p.ej. cambiar COLS/ROWS/BLOCK obliga a actualizar el canvas de index.html>

### Criterios de aceptación
- [ ] ...

### Verificación manual
<qué probar en el navegador: movimiento, rotación, line clears, game over, pausa>

---
*Generado automáticamente por Claude a partir del código actual del repositorio. Puede contener errores — revisar antes de implementar.*
```

## Paso 6 — Información insuficiente

Si el issue no da datos suficientes para diagnosticar (falta repro, falta contexto, ambiguo), etiqueta
`question` + `priority:low` (+ áreas si son deducibles) + `triaged`, y en lugar del diagnóstico completo
publica solo la sección "Resumen" y una lista concreta de preguntas pendientes para el autor.

## Reglas generales

- No modifiques código ni abras PRs desde este workflow: solo etiquetas, título y comentario.
- No repitas contenido genérico ("revisar el código", "podría ser un bug") sin anclarlo a líneas/funciones
  reales de este repo.
- Sé conciso: el objetivo es que el diagnóstico sirva de punto de partida para implementar la solución
  después, no sustituir el PR.
