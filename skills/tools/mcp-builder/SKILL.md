---
name: mcp-builder
description: Princípios de construção de servidores MCP (Model Context Protocol). Design de ferramentas, padrões de recursos e melhores práticas.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Construtor de MCP

> Princípios para construção de servidores MCP.

---

## 1. Visão Geral do MCP

### O que é MCP?

Model Context Protocol - um padrão para conectar sistemas de IA com ferramentas externas e fontes de dados.

### Conceitos Centrais

| Conceito | Propósito |
| :--- | :--- |
| **Ferramentas (Tools)** | Funções que a IA pode chamar |
| **Recursos (Resources)** | Dados que a IA pode ler |
| **Prompts** | Templates de prompt pré-definidos |

---

## 2. Arquitetura do Servidor

### Estrutura do Projeto

```
meu-servidor-mcp/
├── src/
│   └── index.ts      # Entrada principal
├── package.json
└── tsconfig.json
```

### Tipos de Transporte

| Tipo | Uso |
| :--- | :--- |
| **Stdio** | Local, baseado em CLI |
| **SSE** | Baseado na web, streaming |
| **WebSocket** | Tempo real, bidirecional |

---

## 3. Princípios de Design de Ferramentas

### Bom Design de Ferramenta

| Princípio | Descrição |
| :--- | :--- |
| Nome claro | Orientado à ação (get_weather, create_user) |
| Propósito único | Faz uma coisa bem feita |
| Entrada validada | Schema com tipos e descrições |
| Saída estruturada | Formato de resposta previsível |

### Design de Schema de Entrada

| Campo | Obrigatório? |
| :--- | :--- |
| Tipo (Type) | Sim - object |
| Propriedades | Define cada parâmetro |
| Obrigatório | Lista parâmetros mandatórios |
| Descrição | Legível para humanos |

---

## 4. Padrões de Recursos

### Tipos de Recurso

| Tipo | Uso |
| :--- | :--- |
| Estático | Dados fixos (config, docs) |
| Dinâmico | Gerado sob demanda |
| Template | URI com parâmetros |

### Padrões de URI

| Padrão | Exemplo |
| :--- | :--- |
| Fixo | `docs://readme` |
| Parametrizado | `users://{userId}` |
| Coleção | `files://projeto/*` |

---

## 5. Tratamento de Erros

### Tipos de Erro

| Situação | Resposta |
| :--- | :--- |
| Parâmetros inválidos | Mensagem de erro de validação |
| Não encontrado | Mensagem clara de "não encontrado" |
| Erro do servidor | Erro genérico, logar detalhes |

### Melhores Práticas

- Retorne erros estruturados.
- Não exponha detalhes internos.
- Logue para fins de depuração.
- Forneça mensagens acionáveis.

---

## 6. Tratamento Multimodal

### Tipos Suportados

| Tipo | Codificação |
| :--- | :--- |
| Texto | Texto simples (Plain text) |
| Imagens | Base64 + tipo MIME |
| Arquivos | Base64 + tipo MIME |

---

## 7. Princípios de Segurança

### Validação de Entrada

- Valide todas as entradas de ferramentas.
- Higienize (sanitize) os dados fornecidos pelo usuário.
- Limite o acesso aos recursos.

### Chaves de API

- Use variáveis de ambiente.
- Não logue segredos.
- Valide permissões.

---

## 8. Configuração

### Configuração do Claude Desktop

| Campo | Propósito |
| :--- | :--- |
| command | Executável a ser rodado |
| args | Argumentos do comando |
| env | Variáveis de ambiente |

---

## 9. Testes

### Categorias de Teste

| Tipo | Foco |
| :--- | :--- |
| Unitário | Lógica da ferramenta |
| Integração | Servidor completo |
| Contrato | Validação de schema |

---

## 10. Checklist de Melhores Práticas

- [ ] Nomes de ferramentas claros e orientados à ação.
- [ ] Schemas de entrada completos com descrições.
- [ ] Saída JSON estruturada.
- [ ] Tratamento de erros para todos os casos.
- [ ] Validação de entradas.
- [ ] Configuração baseada em ambiente.
- [ ] Logging para depuração.

---

> **Lembre-se:** As ferramentas MCP devem ser simples, focadas e bem documentadas. A IA depende das descrições para usá-las corretamente.
