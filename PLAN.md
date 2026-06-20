# Plan: Presentación estilo archivo migratorio — Markdown → Slides

## Contexto

El objetivo es construir una web tipo presentación, navegable slide por slide, desplegable en GitHub Pages sin infraestructura adicional. El contenido vive en un `content.md` que se parsea en el browser al cargar. La estética base ya está definida en `fichas-migratorias.html`: papel envejecido, tipografías victorianas y mapas Leaflet con tratamiento sepia.

## Estructura de archivos

```text
/
├── index.html
├── content.md
└── public/
```

- `index.html`: app completa con shell HTML, CSS y JS.
- `content.md`: contenido editable de la presentación.
- `public/`: assets opcionales, como retratos o imágenes auxiliares.

## Formato del Markdown

El contenido se organiza por niveles de heading:

```markdown
# Archivos del Sur
## Habitantes del Perilago

### Juan Carlos Chabon
- Nombre: Juan Carlos Chabon
- Nacimiento: 14 mar. 1891
- Registro: 9 nov. 1923
- Padre: José Chabon
- Madre: Rosa López de Chabon
- Residencia: 12
- Profesión: Pescador
- Origen: -38.115, -63.362, Bahía Blanca
- Destino: -34.603, -58.381, Buenos Aires
- Observaciones: Llegó con documentación en regla. Sin antecedentes.
```

- `#`: portada.
- `##`: separador de sección.
- `###`: ficha individual.
- `- Clave: Valor`: campos de la ficha.

Campos opcionales ausentes deben renderizarse como `—`. `Origen` y `Destino` se parsean con formato `lat, lng, Nombre lugar`.

## Tipos de slide

| Heading | Tipo | Render |
|---------|------|--------|
| `#` | `title` | Portada centrada |
| `##` | `section` | Separador de capítulo |
| `###` + bullets | `card` | Ficha migratoria |

## Parser en cliente

Implementación sugerida:

1. Hacer `fetch('content.md')` al cargar la página.
2. Iterar línea por línea.
3. Al encontrar `# texto`, crear slide `{ type: 'title', text }`.
4. Al encontrar `## texto`, crear slide `{ type: 'section', text, index }` y actualizar la sección actual.
5. Al encontrar `### texto`, iniciar una ficha `{ type: 'card', heading, fields, sectionLabel, sectionIndex }`.
6. Al encontrar `- Clave: Valor`, agregar el campo a la ficha activa.
7. Al encontrar línea vacía o nuevo heading, cerrar la ficha activa.

Para coordenadas:

```js
/^(-?\d+\.?\d*),\s*(-?\d+\.?\d*),\s*(.+)$/
```

Debe devolver `{ lat, lng, label }`.

## Shell HTML

Estructura base:

```html
<div class="archive-header"></div>
<div class="stage" id="stage"></div>
<div class="nav">
  <button id="prev">← Anterior</button>
  <span id="counter">1 / N</span>
  <button id="next">Siguiente →</button>
</div>
```

El `archive-header` funciona además como breadcrumb de sección cuando corresponda.

## CSS

Reutilizar y limpiar la base visual de `fichas-migratorias.html`:

- variables CSS (`--ink`, `--ocre`, `--border`, etc.)
- layout `.card`
- header, body, footer de ficha
- retrato enmarcado
- caja de mapa
- transiciones entre slides

Agregar variantes específicas:

- `.card-title`: portada
- `.card-section`: slide de sección

## Mapas Leaflet

- Reutilizar la lógica de `initMap()` de la maqueta.
- Inicializar mapas en forma lazy.
- Solo las slides `card` llevan mapa.
- Guardar referencias en un array `leafletMaps[]`.

## Navegación

- Botones `Anterior` y `Siguiente`
- Flechas de teclado
- `goTo(index, dir)` para controlar transición y estado
- breadcrumb clickeable para volver a la slide de sección

## Ejemplo de contenido a generar

El `content.md` inicial debería incluir:

- 1 slide `#`
- 2 secciones `##`
- 2 o 3 personas por sección
- coordenadas reales para mapas
- al menos un campo faltante para probar empty state

## Verificación

1. Servir el proyecto localmente.
2. Comprobar portada, secciones y fichas.
3. Verificar mapas y navegación.
4. Editar `content.md` y recargar para validar que el contenido cambia sin tocar HTML.
5. Confirmar que el breadcrumb vuelve a la sección correcta.
6. Subir a GitHub Pages desde la rama principal y directorio raíz.
