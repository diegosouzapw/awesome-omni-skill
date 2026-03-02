---
name: minimax-agent-creator
description: Use when creating, configuring, or extending AI agents with MiniMax M2.1 model - provides complete framework patterns, tool integration, memory systems, and MCP connectivity for building production-ready agents
license: MIT
---

# MiniMax Agent Creator

## Overview

Framework completo para crear agentes AI usando el modelo MiniMax M2.1 con API compatible con Anthropic. Integra herramientas del sistema, memoria de sesión, Claude Skills, y servidores MCP para construir agentes robustos y extensible.

## When to Use

**Actívame cuando el usuario quiera:**

- **Crear un nuevo agente** desde cero o desde template
- **Extender un agente existente** con nuevas herramientas
- **Configurar la conexión** con MiniMax API
- **Integrar herramientas MCP** (web search, memory, etc.)
- **Implementar memoria persistente** entre sesiones
- **Definir system prompts** personalizados
- **Crear herramientas personalizadas** (file, bash, API)
- **Debuggear agentes** MiniMax
- **Migrar agentes de Claude** a MiniMax

**No me uses para:**

- Tareas simples que no requieren un agente estructurado
- Consultas sobre modelos que no sean MiniMax
- Modificar la configuración global de Claude

## Quick Start

### Agente Basico en 5 Minutos

```python
from mini_agent import LLMClient, Agent
from mini_agent.tools import ReadTool, WriteTool, EditTool, BashTool

async def create_simple_agent():
    # 1. Configurar cliente LLM
    llm_client = LLMClient(
        api_key="tu-api-key",
        api_base="https://api.minimax.io",
        model="MiniMax-M2.1",
    )

    # 2. Definir herramientas
    tools = [
        ReadTool(workspace_dir="./workspace"),
        WriteTool(workspace_dir="./workspace"),
        EditTool(workspace_dir="./workspace"),
        BashTool(),
    ]

    # 3. Crear sistema de memoria
    from mini_agent.tools import SessionNoteTool, RecallNoteTool

    # 4. Crear agente
    agent = Agent(
        llm_client=llm_client,
        system_prompt="Eres un asistente útil y preciso.",
        tools=tools,
        max_steps=50,
        workspace_dir="./workspace",
    )

    # 5. Ejecutar tarea
    agent.add_user_message("Crea un archivo hello.py que imprima 'Hola Mundo'")
    result = await agent.run()
    return result
```

### Con Configuración YAML

```yaml
# config.yaml
api_key: "YOUR_API_KEY"
api_base: "https://api.minimax.io"
model: "MiniMax-M2.1"

agent:
  max_steps: 100
  workspace_dir: "./workspace"
  token_limit: 80000

tools:
  enable_file_tools: true
  enable_bash: true
  enable_mcp: true
  enable_skills: true
```

## Architecture Patterns

### 1. Single Agent Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                        Agent                                 │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ LLM Client  │  │ Tool System │  │ Memory System       │  │
│  │ (M2.1)      │  │ (File/Bash) │  │ (Session Notes)     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│                    Workspace Dir                             │
└─────────────────────────────────────────────────────────────┘
```

**Casos de uso:** Tareas autonomous simples, scripting, automatización.

### 2. Multi-Tool Agent Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                        Agent                                 │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ LLM Client  │  │   Tools     │  │ Memory System       │  │
│  │ M2.1        │──│─┬──────────┐ │  │                     │  │
│  │              │  │ │ File     │ │  │ ┌───────────────┐  │  │
│  │              │  │ │ Bash     │ │──│ │Session Notes  │  │  │
│  │              │  │ │ MCP      │ │  │ │ Recall        │  │  │
│  │              │  │ │ Skills   │ │  │ └───────────────┘  │  │
│  │              │  │ └──────────┘ │  └─────────────────────┘  │
│  └─────────────┘  └─────────────┘                            │
├─────────────────────────────────────────────────────────────┤
│              Context Management (Auto-Summarization)         │
└─────────────────────────────────────────────────────────────┘
```

**Casos de uso:** Investigación, desarrollo complejo, análisis de datos.

### 3. Workflow Agent Pattern

```python
# Agent con sub-agentes o pipeline de tareas
from mini_agent import Agent

class WorkflowAgent:
    def __init__(self):
        self.planner = self._create_agent("planner")
        self.executor = self._create_agent("executor")
        self.validator = self._create_agent("validator")

    async def run(self, task: str) -> dict:
        plan = await self.planner.run(f"Planifica: {task}")
        results = await self.executor.run(plan)
        validation = await self.validator.run(f"Valida: {results}")
        return validation
```

**Casos de uso:** Tareas multi-etapa, QA automation, pipelines de datos.

## Tool System

### Built-in Tools

| Herramienta | Descripción | Uso Principal |
|-------------|-------------|---------------|
| `ReadTool` | Lee archivos con contexto | Análisis de código, revisión |
| `WriteTool` | Crea/sobrescribe archivos | Generación de código, docs |
| `EditTool` | Reemplazo exacto de texto | Refactoring, patches |
| `BashTool` | Ejecuta comandos shell | Compilación, tests, git |
| `BashOutputTool` | Obtiene salida de procesos | Monitoreo async |
| `BashKillTool` | Termina procesos | Cleanup de procesos |
| `SessionNoteTool` | Memoria persistente | Contexto entre sesiones |
| `RecallNoteTool` | Recupera memoria | Acceso a contexto previo |
| `GetSkillTool` | Carga Claude Skills | Herramientas advanced |

### Custom Tool Pattern

```python
from mini_agent.tools.base import Tool, ToolResult
from typing import Dict, Any

class APITool(Tool):
    """Custom tool para llamadas a API."""

    @property
    def name(self) -> str:
        return "api_call"

    @property
    def description(self) -> str:
        return "Realiza llamadas HTTP a APIs REST con soporte para GET, POST, PUT, DELETE"

    @property
    def parameters(self) -> Dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "method": {
                    "type": "string",
                    "enum": ["GET", "POST", "PUT", "DELETE"],
                    "description": "HTTP method"
                },
                "url": {
                    "type": "string",
                    "description": "Endpoint URL"
                },
                "headers": {
                    "type": "object",
                    "description": "Request headers",
                    "default": {}
                },
                "json": {
                    "type": "object",
                    "description": "JSON body para POST/PUT"
                }
            },
            "required": ["method", "url"]
        }

    async def execute(self, method: str, url: str,
                      headers: Dict = None, json: Dict = None) -> ToolResult:
        try:
            import httpx
            async with httpx.AsyncClient() as client:
                response = await client.request(
                    method, url, headers=headers, json=json
                )
                return ToolResult(
                    success=response.status_code < 400,
                    content=response.text,
                    error=f"Status: {response.status_code}" if response.status_code >= 400 else None
                )
        except Exception as e:
            return ToolResult(success=False, error=str(e))

# Registro en agente
tools = [
    # ... otras herramientas
    APITool(),
]
```

### MCP Tool Integration

```python
# mini_agent/tools/mcp_loader.py
async def load_mcp_tools_async(config_path: str = "mcp.json") -> list[Tool]:
    """
    Carga herramientas desde servidores MCP.

    Formatos soportados:
    - STDIO: comando local (npm, python, etc.)
    - SSE: Server-Sent Events para streaming
    - HTTP: REST API endpoints
    """
```

**Configuración MCP (mcp.json):**

```json
{
  "mcpServers": {
    "web-search": {
      "description": "Búsqueda web y scraping",
      "command": "npx",
      "args": ["-y", "minimax-search"],
      "env": {"JINA_API_KEY": ""},
      "disabled": false
    },
    "memory": {
      "description": "Knowledge graph para memoria",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "disabled": false
    },
    "context7": {
      "description": "Documentación de librerías",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "disabled": false
    }
  }
}
```

### Claude Skills Integration

```python
from mini_agent.tools.skill_tool import GetSkillTool
from mini_agent.tools.skill_loader import create_skill_tools

# Cargar todas las skills disponibles
skill_tools, skill_loader = create_skill_tools("./skills")

# Usar skill específica
skill_tool = GetSkillTool()

# En el contexto del agente, usar:
# "Usa la skill pdf para generar un reporte"
```

## Memory Systems

### Session Notes (Persistente)

```python
from mini_agent.tools import SessionNoteTool, RecallNoteTool

# Grabar información importante
record_tool = SessionNoteTool(memory_file="./.agent_memory.json")

await record_tool.execute(
    content="Usuario es desarrollador Python senior, prefiere type hints",
    category="user_preferences"
)

# Recuperar en sesión nueva
recall_tool = RecallNoteTool(memory_file="./.agent_memory.json")
result = await recall_tool.execute(category="user_preferences")
```

### Context Management (Auto-Summarization)

```python
agent = Agent(
    llm_client=llm_client,
    system_prompt=system_prompt,
    tools=tools,
    max_steps=100,
    workspace_dir="./workspace",
    token_limit=80000,  # Auto-sumariza cuando se exceede
)

# El agente automáticamente:
# 1. Detecta cuando tokens > token_limit
# 2. Resume mensajes antiguos
# 3. Mantiene contexto relevante
# 4. Continúa ejecución sin pérdida de información
```

## Model Configuration

### API Compatible Anthropic

```python
from mini_agent import LLMClient, LLMProvider

# Cliente Anthropic (recomendado para MiniMax)
client = LLMClient(
    api_key="tu-api-key",
    provider=LLMProvider.ANTHROPIC,
    api_base="https://api.minimax.io/anthropic",  # /anthropic suffix
    model="MiniMax-M2.1",
)

# Cliente OpenAI
client = LLMClient(
    api_key="tu-api-key",
    provider=LLMProvider.OPENAI,
    api_base="https://api.minimax.io/v1",  # /v1 suffix
    model="MiniMax-M2.1",
)
```

### Parameters

| Parámetro | Rango | Default | Descripción |
|-----------|-------|---------|-------------|
| `temperature` | 0.0 - 2.0 | 1.0 | Creatividad (1.0 = balanceado) |
| `max_tokens` | 1 - 8192 | 4096 | Máximo tokens de output |
| `top_p` | 0.0 - 1.0 | 0.9 | Nucleus sampling |
| `stream` | boolean | true | Streaming de respuesta |

### Configuración Global (settings.json)

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.minimax.io/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "tu-api-key",
    "ANTHROPIC_MODEL": "MiniMax-M2.1",
    "API_TIMEOUT_MS": "3000000"
  }
}
```

## System Prompts

### Template Base

```markdown
Eres {nombre_agente}, un agente de IA especializado en {dominio}.

## Tu Rol
- {descripcion_rol}
- Principalmente interactúas mediante herramientas y código

## Principios
1. {principio_1}
2. {principio_2}
3. {principio_3}

## Restricciones
- {restriccion_1}
- {restriccion_2}

## when trabajar con código:
- Siempre verifica antes de modificar
- Usa type hints en Python
- Maneja errores explícitamente
- Documenta funciones complejas

## when no sepas algo:
- Indica claramente tu incertidumbre
- Sugiere cómo investigar o verificar
- No inventes información
```

### Prompt Optimizado para M2.1

```markdown
Eres un asistente de IA avanzado con capacidades de razonamiento extensivo.

## Capacidades
- Razonamiento paso a paso para problemas complejos
- Uso de herramientas para ejecutar código y acceder archivos
- Memoria de sesión para mantener contexto

## Directivas
1. Piensa en voz alta (thinking blocks) para problemas complejos
2. Usa herramientas cuando necesites verificar información
3. Mantén tus respuestas precisas y basadas en hechos
4. Si no puedes completar una tarea, explica claramente por qué

## Formato de Respuesta
- Cuando uses herramientas, reporta resultados completos
- Para código, incluye contexto y explicaciones
- Para decisiones, muestra el razonamiento
```

## Templates de Agentes

### Template 1: Research Agent

Ver: `templates/research_agent.py`

```python
"""Agente especializado en investigación y análisis."""
from mini_agent import LLMClient, Agent
from mini_agent.tools import (
    ReadTool, WriteTool, BashTool,
    SessionNoteTool, RecallNoteTool
)

class ResearchAgent:
    def __init__(self, api_key: str, workspace_dir: str = "./workspace"):
        self.workspace_dir = workspace_dir
        self.llm_client = LLMClient(
            api_key=api_key,
            api_base="https://api.minimax.io/anthropic",
            model="MiniMax-M2.1",
        )
        self.agent = self._create_agent()

    def _create_agent(self) -> Agent:
        tools = [
            ReadTool(workspace_dir=self.workspace_dir),
            WriteTool(workspace_dir=self.workspace_dir),
            BashTool(),
            SessionNoteTool(memory_file=f"{self.workspace_dir}/.research_memory.json"),
            RecallNoteTool(memory_file=f"{self.workspace_dir}/.research_memory.json"),
        ]
        return Agent(
            llm_client=self.llm_client,
            system_prompt=self._get_system_prompt(),
            tools=tools,
            max_steps=100,
            workspace_dir=self.workspace_dir,
        )

    def _get_system_prompt(self) -> str:
        return """Eres un agente de investigación especializado.

## Tu Objetivo
Analizar temas complejos, buscar información relevante, y sintetizar hallazgos.

## Metodología
1. Identificar fuentes confiables
2. Verificar información cruzada
3. Documentar hallazgos
4. Proporcionar conclusiones basadas en evidencia

## Restricciones
- Solo usa fuentes verificables
- Indica incertidumbre claramente
- Cita tus fuentes cuando sea posible"""
```

### Template 2: Code Agent

Ver: `templates/code_agent.py`

```python
"""Agente especializado en desarrollo de código."""
from mini_agent import LLMClient, Agent
from mini_agent.tools import (
    ReadTool, WriteTool, EditTool, BashTool
)

class CodeAgent:
    def __init__(self, api_key: str, workspace_dir: str = "./workspace"):
        self.workspace_dir = workspace_dir
        self.llm_client = LLMClient(
            api_key=api_key,
            api_base="https://api.minimax.io/anthropic",
            model="MiniMax-M2.1",
        )
        self.agent = self._create_agent()

    def _create_agent(self) -> Agent:
        tools = [
            ReadTool(workspace_dir=self.workspace_dir),
            WriteTool(workspace_dir=self.workspace_dir),
            EditTool(workspace_dir=self.workspace_dir),
            BashTool(),
        ]
        return Agent(
            llm_client=self.llm_client,
            system_prompt=self._get_system_prompt(),
            tools=tools,
            max_steps=50,
            workspace_dir=self.workspace_dir,
        )

    def _get_system_prompt(self) -> str:
        return """Eres un desarrollador senior experto en múltiples lenguajes.

## Principios
1. Escribe código limpio y mantenible
2. Usa type hints y documentación
3. Maneja errores explícitamente
4. Prefiere soluciones simples sobre complejas

## Stack Preferido
- Python con type hints
- JavaScript/TypeScript moderno
- SQL para bases de datos
- Bash para automatización

## Patrones
- SOLID para OOP
- Functional programming cuando aplique
- TDD cuando sea factible"""
```

### Template 3: Web Scraping Agent

Ver: `templates/web_agent.py`

```python
"""Agente especializado en scraping y extracción web."""
from mini_agent import LLMClient, Agent
from mini_agent.tools import (
    ReadTool, WriteTool, BashTool
)

class WebAgent:
    def __init__(self, api_key: str, workspace_dir: str = "./workspace"):
        self.workspace_dir = workspace_dir
        self.llm_client = LLMClient(
            api_key=api_key,
            api_base="https://api.minimax.io/anthropic",
            model="MiniMax-M2.1",
        )
        self.agent = self._create_agent()

    def _create_agent(self) -> Agent:
        tools = [
            ReadTool(workspace_dir=self.workspace_dir),
            WriteTool(workspace_dir=self.workspace_dir),
            BashTool(),
            # MCP tools would be added here
        ]
        return Agent(
            llm_client=self.llm_client,
            system_prompt=self._get_system_prompt(),
            tools=tools,
            max_steps=30,
            workspace_dir=self.workspace_dir,
        )

    def _get_system_prompt(self) -> str:
        return """Eres un agente de extracción web especializado.

## Capacidades
- Scraping de páginas web
- Extracción de datos estructurados
- Búsqueda de información en línea
- Resumen de contenido web

## Restricciones
- Respeta robots.txt
- No sobrecargar servidores
- Usa rate limiting apropiado
- Almacena datos de forma organizada

## Formato de Salida
- JSON estructurado cuando aplique
- Markdown para resúmenes
- CSV para datos tabulares"""
```

## MCP Servers Disponibles

### En tu Configuración

| Servidor | Tipo | Descripción |
|----------|------|-------------|
| `polymarket` | Python | Datos de mercados de predicción |
| `mermaid` | npx | Diagramas Mermaid |
| `playwright` | npx | Automatización de browser |
| `context7` | npx | Documentación de librerías |
| `greptile` | HTTP | Code review automatizado |
| `supabase` | HTTP | Base de datos backend |
| `github` | HTTP | Integración GitHub |
| `stripe` | HTTP | Pagos y fintech |
| `linear` | HTTP | Project management |

### Integración MCP en Agente

```python
from mini_agent.tools.mcp_loader import load_mcp_tools_async

async def create_agent_with_mcp():
    llm_client = LLMClient(api_key="...", model="MiniMax-M2.1")

    # Cargar herramientas base
    tools = [
        ReadTool(workspace_dir="./workspace"),
        WriteTool(workspace_dir="./workspace"),
        BashTool(),
    ]

    # Cargar herramientas MCP
    mcp_tools = await load_mcp_tools_async("mcp.json")
    tools.extend(mcp_tools)

    agent = Agent(
        llm_client=llm_client,
        system_prompt="Eres un agente con acceso a herramientas MCP.",
        tools=tools,
        max_steps=50,
        workspace_dir="./workspace",
    )

    return agent
```

## Ejemplos Completos

### Ejemplo 1: Pipeline de Análisis

Ver: `examples/analysis_pipeline.py`

```python
"""
Pipeline completo de análisis con múltiples etapas.
"""
import asyncio
from pathlib import Path
from mini_agent import LLMClient, Agent
from mini_agent.tools import ReadTool, WriteTool, BashTool

class AnalysisPipeline:
    def __init__(self, api_key: str, workspace_dir: str = "./workspace"):
        self.workspace_dir = Path(workspace_dir)
        self.workspace_dir.mkdir(parents=True, exist_ok=True)
        self.llm_client = LLMClient(
            api_key=api_key,
            api_base="https://api.minimax.io/anthropic",
            model="MiniMax-M2.1",
        )

    async def run(self, task: str) -> dict:
        """Ejecuta el pipeline completo."""
        # Fase 1: Investigación
        research_result = await self._research_phase(task)

        # Fase 2: Análisis
        analysis_result = await self._analysis_phase(research_result)

        # Fase 3: Síntesis
        final_result = await self._synthesis_phase(analysis_result)

        return final_result

    async def _research_phase(self, query: str) -> str:
        """Fase de investigación."""
        agent = self._create_agent(
            "Investiga sobre: " + query,
            "Eres un investigador exhaustivo. Busca información detallada."
        )
        return await agent.run()

    async def _analysis_phase(self, context: str) -> str:
        """Fase de análisis."""
        agent = self._create_agent(
            f"Analiza esta información:\n\n{context}",
            "Eres un analista riguroso. Identifica patrones, inconsistencias y insights."
        )
        return await agent.run()

    async def _synthesis_phase(self, analysis: str) -> dict:
        """Fase de síntesis."""
        agent = self._create_agent(
            f"Sintetiza el análisis en un reporte estructurado:\n\n{analysis}",
            "Eres un redactor técnico. Produce reportes claros y estructurados."
        )
        result = await agent.run()

        # Guardar resultado
        output_file = self.workspace_dir / "analysis_report.md"
        await WriteTool(workspace_dir=str(self.workspace_dir)).execute(
            path=str(output_file), content=result
        )

        return {"report": result, "output_file": str(output_file)}

    def _create_agent(self, task: str, system_prompt: str) -> Agent:
        tools = [
            ReadTool(workspace_dir=str(self.workspace_dir)),
            WriteTool(workspace_dir=str(self.workspace_dir)),
            BashTool(),
        ]
        return Agent(
            llm_client=self.llm_client,
            system_prompt=system_prompt,
            tools=tools,
            max_steps=50,
            workspace_dir=str(self.workspace_dir),
        )

# Uso
async def main():
    pipeline = AnalysisPipeline(api_key="tu-api-key")
    result = await pipeline.run("Tendencias en inteligencia artificial 2024")
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

### Ejemplo 2: Agent con Retry

Ver: `examples/agent_with_retry.py`

```python
"""
Agente con manejo de errores y reintentos.
"""
import asyncio
from mini_agent import LLMClient, Agent
from mini_agent.tools import ReadTool, WriteTool, BashTool
from mini_agent.retry import async_retry, RetryConfig

class ResilientAgent:
    def __init__(self, api_key: str):
        self.llm_client = LLMClient(
            api_key=api_key,
            api_base="https://api.minimax.io/anthropic",
            model="MiniMax-M2.1",
        )

    @async_retry(RetryConfig(max_retries=3, initial_delay=2.0))
    async def run_with_retry(self, task: str, max_steps: int = 50) -> str:
        """Ejecuta tarea con reintentos automáticos."""
        agent = self._create_agent()
        agent.add_user_message(task)
        return await agent.run()

    def _create_agent(self) -> Agent:
        tools = [
            ReadTool(workspace_dir="./workspace"),
            WriteTool(workspace_dir="./workspace"),
            BashTool(),
        ]
        return Agent(
            llm_client=self.llm_client,
            system_prompt="Eres un asistente preciso y cuidadoso.",
            tools=tools,
            max_steps=50,
            workspace_dir="./workspace",
        )
```

## Logging y Debugging

### Sistema de Logging

```python
from mini_agent.logger import get_logger

logger = get_logger(__name__)

# Logs automáticos para:
# - Requests LLM
# - Responses LLM
# - Tool executions
# - Errors y retries

# Ver logs en: ~/.mini-agent/logs/
```

### Debugging Tips

1. **Verbose mode:**
```python
agent = Agent(
    llm_client=llm_client,
    tools=tools,
    max_steps=100,
    # Los logs se guardan automáticamente
)
```

2. **Revisar logs:**
```bash
cat ~/.mini-agent/logs/*.log | tail -100
```

3. **Errores comunes:**
- SSL: `verify=False` en desarrollo
- API key: Verificar formato y permisos
- Timeout: Aumentar `API_TIMEOUT_MS`

## Common Mistakes

| Error | Causa | Solución |
|-------|-------|----------|
| `SSL CERTIFICATE_VERIFY_FAILED` | Certificados过期 | `pip install --upgrade certifi` |
| `ModuleNotFoundError` | No estás en el directorio | `cd Mini-Agent && python -m ...` |
| `401 Unauthorized` | API key inválida | Verificar credenciales |
| `RateLimitError` | Demasiadas requests | Implementar backoff |
| `Context overflow` | Token limit excedido | Reducir contexto o aumentar `token_limit` |

## Resources

### Archivos del Skill

- `templates/` - Templates de agentes listos para usar
- `scripts/` - Herramientas de utilidad
- `examples/` - Ejemplos completos
- `references/` - Documentación detallada

### Documentación Externa

- [MiniMax Platform](https://platform.minimax.io)
- [Mini-Agent Repository](https://github.com/MiniMax-AI/Mini-Agent)
- [MiniMax API Reference](https://platform.minimaxi.com/document)
- [MCP Protocol](https://github.com/modelcontextprotocol)
- [Anthropic API](https://docs.anthropic.com)

### Dependencias

```txt
# pyproject.toml
httpx>=0.25.0      # HTTP client async
pydantic>=2.0.0    # Data validation
pyyaml>=6.0.0      # Config parsing
tiktoken>=0.5.0    # Token counting
requests>=2.31.0   # HTTP sync (fallback)
prompt-toolkit>=3.0.0  # CLI interface
```

## Production Checklist

Antes de desplegar tu agente:

- [ ] API key configurada y rotada periódicamente
- [ ] Límites de rate implementados
- [ ] Manejo de errores robusto
- [ ] Logging configurado
- [ ] Memory system persistente
- [ ] Tests unitarios para herramientas custom
- [ ] Documentación de uso
- [ ] Configuración via variables de entorno
- [ ] Monitoreo de costos

## License

MIT License - Ver archivo LICENSE para detalles.
