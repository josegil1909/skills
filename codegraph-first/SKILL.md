---
title: CodeGraph MCP First
description: |
  Este proyecto tiene un índice de código CodeGraph (`.codegraph/`).  
  Antes de hacer búsquedas manuales con `find`, `grep`, `rg` o leer archivos
  sueltos con `read`, DEBE usarse la herramienta MCP `codegraph`.
trigger: |
  Cualquier tarea de exploración, navegación, búsqueda de símbolos,
  refactorización, o análisis de impacto en el codebase del proyecto.
---

# CodeGraph MCP First

## Regla de oro

> Si necesitás encontrar un componente, función, tipo, ruta, traducción,
> o entender relaciones entre símbolos, **usá CodeGraph antes que cualquier
> otra herramienta de exploración**.

No vale hacer `find . -name "*.vue"`, `grep -r "dropdown"` o leer
archivos uno por uno hasta dar con el objetivo.

## Flujo correcto

### 1. Encontrar símbolos por nombre
```
mcp({ server: "codegraph", tool: "codegraph_codegraph_search",
      args: '{"query": "NombreDelComponente o término"}' })
```

### 2. Explorar intencionalmente (contexto amplio)
```
mcp({ server: "codegraph", tool: "codegraph_codegraph_context",
      args: '{"task": "Describe qué necesitás encontrar o hacer",
            "maxNodes": 30, "includeCode": true}' })
```

### 3. Ver relaciones entre símbolos conocidos
```
mcp({ server: "codegraph", tool: "codegraph_codegraph_explore",
      args: '{"query": "SymbolA SymbolB relación", "includeCode": true}' })
```

### 4. Impacto de un cambio
```
mcp({ server: "codegraph", tool: "codegraph_codegraph_impact",
      args: '{"symbol": "nombre.exacto"}' })
```

### 5. Navegación de código (LSP-like)
```
mcp({ server: "codegraph", tool: "codegraph_codegraph_callers",
      args: '{"symbol": "nombreFuncion"}' })
mcp({ server: "codegraph", tool: "codegraph_codegraph_callees",
      args: '{"symbol": "nombreFuncion"}' })
```

## Parámetros clave por herramienta

| Herramienta | Parámetro principal | Tipo |
|-------------|---------------------|------|
| `codegraph_search` | `query` (string, **obligatorio**) | palabras clave |
| `codegraph_context` | `task` (string, **obligatorio**) | descripción de la tarea |
| `codegraph_explore` | `query` (string, **obligatorio**) | símbolos o términos |
| `codegraph_impact` | `symbol` (string, **obligatorio**) | nombre exacto |
| `codegraph_callers` | `symbol` (string, **obligatorio**) | nombre exacto |
| `codegraph_callees` | `symbol` (string, **obligatorio**) | nombre exacto |
| `codegraph_node` | `symbol` (string, **obligatorio**) | nombre exacto |
| `codegraph_trace` | `from` y `to` (string, **obligatorio**) | dos símbolos |
| `codegraph_files` | `path` (string, **obligatorio**) | ruta de archivo o carpeta |

**Error común:** usar `query` en `codegraph_context` (usa `task`) o `task` en `codegraph_search` (usa `query`). Leer la firma de la herramienta con `mcp({ server: "codegraph", tool: "NOMBRE" })` antes de ejecutar si hay dudas.

## Cuándo está justificado saltear CodeGraph

- Modificación puntual de un archivo YA conocido (typo, rename).
- `git status`, `git diff`, `git log`.
- Comandos de build/test que no requieren exploración.
- Lectura de archivos de configuración ya identificados.

Cualquier otra situación → CodeGraph primero.
