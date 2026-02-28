---
name: mcp-builder
description: Guia para criar servidores MCP (Model Context Protocol) de alta qualidade que permitem que LLMs interajam com serviços externos através de ferramentas bem projetadas. Use ao construir servidores MCP para integrar APIs ou serviços externos, seja em Python (FastMCP) ou Node/TypeScript (MCP SDK).
license: Termos completos em LICENSE.txt
---

# Guia de Desenvolvimento de Servidor MCP

## Visão Geral

Crie servidores MCP (Model Context Protocol) que permitam que LLMs interajam com serviços externos através de ferramentas bem projetadas. A qualidade de um servidor MCP é medida por quão bem ele permite que LLMs realizem tarefas do mundo real.

---

# Processo

## 🚀 Fluxo de Trabalho de Alto Nível

Criar um servidor MCP de alta qualidade envolve quatro fases principais:

### Fase 1: Pesquisa Profunda e Planejamento

#### 1.1 Entenda o Design Moderno do MCP

**Cobertura de API vs. Ferramentas de Fluxo de Trabalho:**
Equilibre a cobertura abrangente de endpoint de API com ferramentas de fluxo de trabalho especializadas. Ferramentas de fluxo de trabalho podem ser mais convenientes para tarefas específicas, enquanto a cobertura abrangente dá aos agentes flexibilidade para compor operações. O desempenho varia de acordo com o cliente—alguns clientes se beneficiam da execução de código que combina ferramentas básicas, enquanto outros funcionam melhor com fluxos de trabalho de nível superior. Quando incerto, priorize a cobertura abrangente da API.

**Nomeação de Ferramentas e Descoberta:**
Nomes de ferramentas claros e descritivos ajudam os agentes a encontrar as ferramentas certas rapidamente. Use prefixos consistentes (por exemplo, `github_create_issue`, `github_list_repos`) e nomenclatura orientada para ação.

**Gerenciamento de Contexto:**
Os agentes se beneficiam de descrições concisas de ferramentas e da capacidade de filtrar/paginar resultados. Projete ferramentas que retornem dados focados e relevantes. Alguns clientes suportam execução de código, o que pode ajudar os agentes a filtrar e processar dados de forma eficiente.

**Mensagens de Erro Acionáveis:**
As mensagens de erro devem guiar os agentes em direção a soluções com sugestões e próximos passos específicos.

#### 1.2 Estude a Documentação do Protocolo MCP

**Navegue pela especificação MCP:**

Comece com o sitemap para encontrar páginas relevantes: `https://modelcontextprotocol.io/sitemap.xml`

Em seguida, busque páginas específicas com sufixo `.md` para formato markdown (por exemplo, `https://modelcontextprotocol.io/specification/draft.md`).

Páginas principais para revisar:

- Visão geral da especificação e arquitetura
- Mecanismos de transporte (HTTP streamable, stdio)
- Definições de ferramenta, recurso e prompt

#### 1.3 Estude a Documentação do Framework

**Stack recomendada:**

- **Linguagem**: TypeScript (suporte SDK de alta qualidade e boa compatibilidade em muitos ambientes de execução, por exemplo, MCPB. Além disso, os modelos de IA são bons em gerar código TypeScript, beneficiando-se de seu amplo uso, tipagem estática e boas ferramentas de linting)
- **Transporte**: HTTP streamable para servidores remotos, usando JSON sem estado (mais simples de escalar e manter, em oposição a sessões com monitoramento de estado e respostas de streaming). stdio para servidores locais.

**Carregar documentação do framework:**

- **Melhores Práticas MCP**: [📋 Ver Melhores Práticas](./reference/mcp_best_practices.md) - Diretrizes principais

**Para TypeScript (recomendado):**

- **TypeScript SDK**: Use WebFetch para carregar `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ Guia TypeScript](./reference/node_mcp_server.md) - Padrões e exemplos TypeScript

**Para Python:**

- **Python SDK**: Use WebFetch para carregar `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Guia Python](./reference/python_mcp_server.md) - Padrões e exemplos Python

#### 1.4 Planeje Sua Implementação

**Entenda a API:**
Revise a documentação da API do serviço para identificar pontos de extremidade chave, requisitos de autenticação e modelos de dados. Use pesquisa na web e WebFetch conforme necessário.

**Seleção de Ferramentas:**
Priorize a cobertura abrangente da API. Liste os pontos de extremidade a serem implementados, começando com as operações mais comuns.

---

### Fase 2: Implementação

#### 2.1 Configure a Estrutura do Projeto

Veja os guias específicos da linguagem para configuração do projeto:

- [⚡ Guia TypeScript](./reference/node_mcp_server.md) - Estrutura do projeto, package.json, tsconfig.json
- [🐍 Guia Python](./reference/python_mcp_server.md) - Organização do módulo, dependências

#### 2.2 Implemente a Infraestrutura Central

Crie utilitários compartilhados:

- Cliente API com autenticação
- Auxiliares de tratamento de erros
- Formatação de resposta (JSON/Markdown)
- Suporte a paginação

#### 2.3 Implemente Ferramentas

Para cada ferramenta:

**Esquema de Entrada:**

- Use Zod (TypeScript) ou Pydantic (Python)
- Inclua restrições e descrições claras
- Adicione exemplos nas descrições dos campos

**Esquema de Saída:**

- Defina `outputSchema` sempre que possível para dados estruturados
- Use `structuredContent` nas respostas da ferramenta (recurso TypeScript SDK)
- Ajuda os clientes a entender e processar as saídas da ferramenta

**Descrição da Ferramenta:**

- Resumo conciso da funcionalidade
- Descrições de parâmetros
- Esquema do tipo de retorno

**Implementação:**

- Async/await para operações de E/S
- Tratamento de erros adequado com mensagens acionáveis
- Suporte a paginação onde aplicável
- Retorne tanto conteúdo de texto quanto dados estruturados ao usar SDKs modernos

**Anotações:**

- `readOnlyHint`: true/false
- `destructiveHint`: true/false
- `idempotentHint`: true/false
- `openWorldHint`: true/false

---

### Fase 3: Revisão e Teste

#### 3.1 Qualidade do Código

Revise para:

- Sem código duplicado (princípio DRY)
- Tratamento de erros consistente
- Cobertura de tipo completa
- Descrições de ferramentas claras

#### 3.2 Construir e Testar

**TypeScript:**

- Execute `npm run build` para verificar a compilação
- Teste com MCP Inspector: `npx @modelcontextprotocol/inspector`

**Python:**

- Verifique a sintaxe: `python -m py_compile your_server.py`
- Teste com MCP Inspector

Veja os guias específicos da linguagem para abordagens de teste detalhadas e listas de verificação de qualidade.

---

### Fase 4: Criar Avaliações

Após implementar seu servidor MCP, crie avaliações abrangentes para testar sua eficácia.

**Carregue [✅ Guia de Avaliação](./reference/evaluation.md) para diretrizes de avaliação completas.**

#### 4.1 Entenda o Objetivo da Avaliação

Use avaliações para testar se os LLMs podem usar efetivamente seu servidor MCP para responder a perguntas realistas e complexas.

#### 4.2 Crie 10 Perguntas de Avaliação

Para criar avaliações eficazes, siga o processo descrito no guia de avaliação:

1. **Inspeção de Ferramentas**: Liste as ferramentas disponíveis e entenda suas capacidades
2. **Exploração de Conteúdo**: Use operações SOMENTE LEITURA para explorar os dados disponíveis
3. **Geração de Perguntas**: Crie 10 perguntas complexas e realistas
4. **Verificação de Resposta**: Resolva cada pergunta você mesmo para verificar as respostas

#### 4.3 Requisitos de Avaliação

Certifique-se de que cada pergunta seja:

- **Independente**: Não dependente de outras perguntas
- **Somente leitura**: Apenas operações não destrutivas necessárias
- **Complexa**: Exigindo múltiplas chamadas de ferramenta e exploração profunda
- **Realista**: Baseada em casos de uso reais com os quais os humanos se importariam
- **Verificável**: Resposta única e clara que pode ser verificada por comparação de strings
- **Estável**: A resposta não mudará com o tempo

#### 4.4 Formato de Saída

Crie um arquivo XML com esta estrutura:

```xml
<evaluation>
  <qa_pair>
    <question>Encontre discussões sobre lançamentos de modelos de IA com codinomes de animais. Um modelo precisava de uma designação de segurança específica que usa o formato ASL-X. Qual número X estava sendo determinado para o modelo nomeado após um gato selvagem malhado?</question>
    <answer>3</answer>
  </qa_pair>
<!-- Mais qa_pairs... -->
</evaluation>
```

---

# Arquivos de Referência

## 📚 Biblioteca de Documentação

Carregue esses recursos conforme necessário durante o desenvolvimento:

### Documentação Principal do MCP (Carregar Primeiro)

- **Protocolo MCP**: Comece com o sitemap em `https://modelcontextprotocol.io/sitemap.xml`, depois busque páginas específicas com sufixo `.md`
- [📋 Melhores Práticas MCP](./reference/mcp_best_practices.md) - Diretrizes universais MCP, incluindo:
  - Convenções de nomenclatura de servidor e ferramenta
  - Diretrizes de formato de resposta (JSON vs Markdown)
  - Melhores práticas de paginação
  - Seleção de transporte (HTTP streamable vs stdio)
  - Padrões de segurança e tratamento de erros

### Documentação SDK (Carregar Durante a Fase 1/2)

- **Python SDK**: Busque em `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- **TypeScript SDK**: Busque em `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`

### Guias de Implementação Específicos da Linguagem (Carregar Durante a Fase 2)

- [🐍 Guia de Implementação Python](./reference/python_mcp_server.md) - Guia completo Python/FastMCP com:
  - Padrões de inicialização de servidor
  - Exemplos de modelo Pydantic
  - Registro de ferramenta com `@mcp.tool`
  - Exemplos de trabalho completos
  - Lista de verificação de qualidade

- [⚡ Guia de Implementação TypeScript](./reference/node_mcp_server.md) - Guia completo TypeScript com:
  - Estrutura do projeto
  - Padrões de esquema Zod
  - Registro de ferramenta com `server.registerTool`
  - Exemplos de trabalho completos
  - Lista de verificação de qualidade

### Guia de Avaliação (Carregar Durante a Fase 4)

- [✅ Guia de Avaliação](./reference/evaluation.md) - Guia completo de criação de avaliação com:
  - Diretrizes de criação de perguntas
  - Estratégias de verificação de resposta
  - Especificações de formato XML
  - Perguntas e respostas de exemplo
  - Executando uma avaliação com os scripts fornecidos
