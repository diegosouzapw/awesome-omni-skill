---
name: article-generator
description: Genera artículos bien estructurados y redactados a partir de notas y estructura definida en blog/draft/<name>/. Usa cuando el usuario pida crear o generar un artículo desde notas.
metadata:
  author: jondotsoy
  version: "1.0"
---

# Article Generator Skill

Este skill genera artículos estructurados y bien redactados a partir de notas e ideas en formato markdown.

## Estructura de Entrada

El skill espera encontrar en el directorio `blog/draft/<nombre-articulo>/`:

- **notes.md** (requerido): Archivo con las ideas principales, puntos clave y contenido del artículo
- **content-structure.md** (opcional): Define la estructura específica que debe seguir el artículo (secciones, orden, etc.)
- **assets/** (opcional): Carpeta con imágenes, archivos y recursos para el artículo

## Estructura de Salida

El skill generará:

- **article.md**: Artículo final con metadata YAML y contenido estructurado

## Formato de Metadata YAML

El archivo `article.md` debe comenzar con la siguiente metadata YAML:

```yaml
---
title: Título del artículo
description: Descripción breve del artículo (1-2 oraciones)
cover: ./assets/cover.png
lang: es
author:
  name: Jonathan Delgado
  email: hi@jon.soy
  website: https://jon.soy
  github: "@jondotsoy"
date: YYYY-MM-DD
publications:
  - url: https://...
    date: YYYY-MM-DD
---
```

### Campos de Metadata

- **title**: Título principal del artículo
- **description**: Resumen conciso del contenido (máximo 2-3 oraciones)
- **cover**: Ruta relativa a la imagen de portada (siempre en `./assets/`)
- **lang**: Código de idioma (es, en, etc.)
- **author**: Información del autor
  - **name**: Nombre completo
  - **email**: Correo electrónico
  - **website**: Sitio web personal
  - **github**: Usuario de GitHub con @
- **date**: Fecha de creación en formato YYYY-MM-DD
- **publications**: Lista de publicaciones (opcional inicialmente)
  - **url**: URL donde fue publicado
  - **date**: Fecha de publicación

## Formato del Contenido Markdown

El contenido del artículo debe seguir estas reglas de formato:

### Separadores de página

- **NO usar `---` como separador de página** en el contenido del artículo
- Los únicos `---` permitidos son los que delimitan el bloque de metadata YAML al inicio del archivo
- Las secciones se separan simplemente con espacio en blanco (línea vacía) entre ellas

### Tablas vs. Bullet Points

- **Preferir bullet points sobre tablas** para presentar información estructurada
- Las tablas solo deben usarse cuando la información es genuinamente tabular y requiere columnas relacionadas
- Para comparaciones, especificaciones o listas de características, usar bullet points con formato anidado

**Ejemplo correcto (bullet points):**

```markdown
**Plataformas soportadas:**

- **Claude**
  - URL base: `https://claude.ai/new`
  - Parámetro: `?q=`
  - Comportamiento: Precarga el prompt en la caja de texto

- **ChatGPT**
  - URL base: `https://chat.openai.com/?q=`
  - Parámetro: `?q=`
  - Comportamiento: Precarga el prompt y puede auto-enviar
```

**Ejemplo a evitar (tabla):**

```markdown
| Plataforma | URL base | Parámetro |
|------------|----------|-----------|
| Claude     | ...      | ?q=       |
```

### Otros elementos de formato

- Usar títulos jerárquicos apropiados (`##`, `###`, `####`)
- Usar bloques de código con syntax highlighting cuando sea relevante
- Usar **negritas** para enfatizar términos importantes
- Usar *cursivas* para referencias o expresiones
- Usar listas ordenadas solo cuando el orden sea relevante
- Usar citas (`>`) para destacar definiciones o conceptos clave

## Instrucciones de Ejecución

Cuando el usuario solicite generar un artículo:

1. **Leer el contexto**:
   - Lee `blog/draft/<nombre-articulo>/notes.md` para obtener las ideas principales
   - Si existe, lee `blog/draft/<nombre-articulo>/content-structure.md` para conocer la estructura deseada
   - Verifica los archivos disponibles en `blog/draft/<nombre-articulo>/assets/`

2. **Analizar y estructurar**:
   - Identifica los temas principales de las notas
   - Si hay `content-structure.md`, sigue esa estructura; de lo contrario, crea una estructura lógica basada en las notas
   - Organiza las ideas en secciones coherentes con títulos claros

3. **Generar el artículo**:
   - Crea la metadata YAML con la información del autor (usar datos por defecto de Jonathan Delgado)
   - Usa la fecha actual para el campo `date`
   - Deja `publications` como array vacío inicialmente
   - Redacta el contenido de manera clara, profesional y bien estructurada
   - Usa markdown apropiadamente (títulos, listas, código, citas, etc.)
   - Si hay imágenes en assets/, referéncialas correctamente con rutas relativas

4. **Estilo de redacción**:
   - Escribe en el idioma indicado en las notas o en español por defecto
   - Usa un tono profesional pero accesible
   - Incluye ejemplos prácticos cuando sea relevante
   - Divide el contenido en secciones bien definidas
   - Usa listas, código y otros elementos markdown para mejorar la legibilidad

5. **Validación**:
   - Asegúrate de que la metadata YAML sea válida
   - Verifica que todas las referencias a archivos en assets/ sean correctas
   - Confirma que el contenido sigue la estructura definida (si existe `content-structure.md`)

6. **Guardar**:
   - Escribe el archivo completo en `blog/draft/<nombre-articulo>/article.md`
   - Confirma al usuario que el artículo fue generado exitosamente

## Ejemplo de Uso

Usuario: "Genera el artículo para 'porque-usar-runbook'"

El skill:
1. Lee `blog/draft/porque-usar-runbook/notes.md`
2. Lee `blog/draft/porque-usar-runbook/content-structure.md` (si existe)
3. Verifica assets disponibles
4. Genera `blog/draft/porque-usar-runbook/article.md` con metadata completa y contenido estructurado

## Notas Importantes

- Siempre incluye la metadata YAML completa al inicio del archivo
- Las imágenes deben referenciarse con rutas relativas: `./assets/imagen.png`
- El campo `date` debe usar el formato ISO: YYYY-MM-DD
- Si no hay información específica del autor, usa los datos por defecto de Jonathan Delgado
- El contenido debe ser original y bien redactado, no una simple copia de las notas
- Respeta la estructura definida en `content-structure.md` si existe
- El artículo debe ser autocontenido y comprensible sin necesidad de leer las notas originales
- **NO usar `---` como separadores de página** en el contenido (solo en el frontmatter YAML)
- **Preferir bullet points en lugar de tablas** para presentar información estructurada
