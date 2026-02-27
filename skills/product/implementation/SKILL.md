---
name: implementation
description: Converte especificações de produto ou tecnologia em tarefas concretas que o Claude Code pode implementar. Decompõe specs em planos detalhados com tarefas claras, critérios de aceitação e acompanhamento de progresso.
---

# Da Especificação à Implementação

Transforma especificações em planos de implementação acionáveis com acompanhamento de progresso usando ferramentas nativas do Claude Code.

## Início Rápido

Quando for solicitado implementar uma especificação:

1. **Encontrar a spec**: use `Glob` e `Grep` para localizar o arquivo de especificação
2. **Ler a spec**: use `Read` para ler o conteúdo da especificação
3. **Extrair requisitos**: analise e estruture requisitos a partir da spec
4. **Criar plano**: use `Write` para criar o plano de implementação em markdown
5. **Criar tarefas**: use `TodoWrite` para criar as tarefas de implementação
6. **Acompanhar progresso**: use `TodoWrite` para atualizar status das tarefas

## Fluxo de Implementação

### Etapa 1: Encontrar a especificação

```
1. Buscar especificação:
   - Use Glob para encontrar arquivos de spec (*.md, *spec*, *prd*, *requisitos*)
   - Use Grep para buscar por palavras-chave no conteúdo
   - Procure em: docs/, specs/, requirements/, ou raiz do projeto
   - Se não encontrar, peça ao usuário o caminho do arquivo

Exemplos de busca:
- Glob: "**/*spec*.md", "**/docs/**/*.md"
- Grep: "requisitos", "funcionalidade", "objetivo"
```

### Etapa 2: Ler e analisar a especificação

```
1. Ler arquivo da especificação:
   - Use Read para ler todo o conteúdo
   - Leia requisitos, design e restrições

2. Analisar especificação:
   - Identificar requisitos funcionais
   - Anotar requisitos não funcionais (desempenho, segurança, etc.)
   - Extrair critérios de aceitação
   - Identificar dependências e bloqueios
```

Veja [reference/spec-parsing.md](reference/spec-parsing.md) para padrões de parsing.

### Etapa 3: Criar plano de implementação

```
1. Dividir em fases/marcos
2. Identificar abordagem técnica
3. Listar tarefas necessárias
4. Identificar riscos

Use o template de plano de implementação:
- [reference/standard-implementation-plan.md](reference/standard-implementation-plan.md)
- [reference/quick-implementation-plan.md](reference/quick-implementation-plan.md)
```

### Etapa 4: Salvar o plano de implementação

```
Use Write para criar arquivo markdown:
- Caminho sugerido: docs/planos/[nome-feature].md ou IMPLEMENTATION.md
- Conteúdo: Plano estruturado com fases, tarefas e abordagem
- Link de volta para a especificação original
```

### Etapa 5: Criar tarefas de implementação

```
Use TodoWrite para criar as tarefas:

Para cada tarefa no plano:
1. Definir content (forma imperativa): "Implementar autenticação"
2. Definir activeForm (forma contínua): "Implementando autenticação"
3. Status inicial: "pending"

Exemplo de estrutura:
[
  { "content": "Criar schema do banco", "activeForm": "Criando schema do banco", "status": "pending" },
  { "content": "Implementar endpoints de API", "activeForm": "Implementando endpoints de API", "status": "pending" },
  { "content": "Criar componentes de frontend", "activeForm": "Criando componentes de frontend", "status": "pending" },
  { "content": "Escrever testes", "activeForm": "Escrevendo testes", "status": "pending" }
]
```

### Etapa 6: Executar implementação

```
1. Marcar tarefa atual como "in_progress" no TodoWrite
2. Implementar usando as ferramentas de código (Read, Edit, Write, Bash)
3. Marcar como "completed" ao finalizar
4. Passar para próxima tarefa
```

### Etapa 7: Acompanhar progresso

```
Atualizações via TodoWrite:
- Atualizar status das tarefas (pending -> in_progress -> completed)
- Adicionar novas tarefas descobertas durante implementação
- Remover tarefas que não são mais necessárias

Atualizações no plano (opcional):
- Use Edit para atualizar checkboxes no arquivo de plano
- Adicionar notas de progresso e decisões tomadas
```

## Padrões de Análise de Especificação

**Requisitos funcionais**: histórias de usuário, descrições de funcionalidade, fluxos de trabalho, requisitos de dados, pontos de integração

**Requisitos não funcionais**: metas de desempenho, requisitos de segurança, necessidades de escalabilidade, disponibilidade, conformidade

**Critérios de aceitação**: condições testáveis, pontos de validação do usuário, benchmarks de desempenho, definições de conclusão

Veja [reference/spec-parsing.md](reference/spec-parsing.md) para técnicas detalhadas.

## Estrutura do Plano de Implementação

```markdown
# Plano de Implementação: [Nome da Feature]

## Visão Geral
[Resumo do que será implementado]

## Especificação
- Link: [caminho/para/spec.md]

## Requisitos
### Funcionais
- [ ] Requisito 1
- [ ] Requisito 2

### Não Funcionais
- [ ] Performance: ...
- [ ] Segurança: ...

## Abordagem Técnica
[Descrição da solução técnica]

## Fases de Implementação

### Fase 1: [Nome]
- [ ] Tarefa 1.1
- [ ] Tarefa 1.2

### Fase 2: [Nome]
- [ ] Tarefa 2.1
- [ ] Tarefa 2.2

## Dependências
- [Lista de dependências externas ou bloqueios]

## Riscos e Mitigações
| Risco | Mitigação |
|-------|-----------|
| ... | ... |

## Critérios de Sucesso
- [ ] Critério 1
- [ ] Critério 2
```

Veja [reference/standard-implementation-plan.md](reference/standard-implementation-plan.md) para template completo.

## Padrões de Quebra de Tarefas

**Por componente**: Banco de dados, endpoints de API, componentes de frontend, integração, testes

**Por fatia de funcionalidade**: fatias verticais (fluxo de autenticação, entrada de dados, geração de relatórios)

**Por prioridade**: P0 (imprescindível), P1 (importante), P2 (bom ter)

## Boas Práticas

1. **Sempre vincular spec e implementação**: manter referências entre arquivos
2. **Quebrar em tarefas pequenas**: cada tarefa deve ser concluída em uma sessão
3. **Extrair critérios de aceitação claros**: saber quando "pronto" está pronto
4. **Identificar dependências cedo**: anotar bloqueios no plano
5. **Atualizar progresso via TodoWrite**: manter tarefas atualizadas
6. **Usar checklists no plano**: indicadores visuais de progresso
7. **Commitar incrementalmente**: commits pequenos e frequentes

## Ferramentas Nativas Utilizadas

| Ferramenta | Uso |
|------------|-----|
| `Glob` | Encontrar arquivos de spec e código |
| `Grep` | Buscar conteúdo específico |
| `Read` | Ler specs, código existente |
| `Write` | Criar planos, novos arquivos |
| `Edit` | Modificar código e planos |
| `TodoWrite` | Gerenciar tarefas de implementação |
| `Bash` | Executar comandos (build, test, git) |
| `Task` | Explorar codebase com agentes |

## Problemas Comuns

**"Não encontra a especificação"**: use Glob com padrões amplos, ou peça o caminho ao usuário

**"Spec pouco clara"**: registre ambiguidades no plano, pergunte ao usuário

**"Requisitos conflitantes"**: documente conflitos, peça decisão ao usuário

**"Escopo grande demais"**: quebre em fases menores, priorize P0

**"Tarefa muito complexa"**: subdivida em tarefas menores no TodoWrite

## Exemplos

Veja [examples/](examples/) para fluxos completos:
- [examples/api-feature.md](examples/api-feature.md) - Implementação de API
- [examples/ui-component.md](examples/ui-component.md) - Componente de frontend
- [examples/database-migration.md](examples/database-migration.md) - Migração de banco
