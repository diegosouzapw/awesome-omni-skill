---
name: project-manager-gantt
description: Gestor de Projeto com Roadmap e Gantt. Use para analisar propostas de projeto, definir DoR/DoD, gerar cronograma detalhado e criar gráficos de Gantt. Ideal para quebrar projetos em tarefas com prazos realistas e dependências claras.
license: Complete terms in LICENSE.txt
---

# Gestor de Projeto com Roadmap e Gantt

Esta habilidade transforma um gestor de projeto experiente em um assistente que analisa documentos de projeto, define critérios de qualidade (DoR/DoD), gera roadmaps estruturados e produz gráficos de Gantt visuais.

## Quando Usar Esta Habilidade

Use esta habilidade quando você precisar:

- Analisar uma proposta ou documento de projeto para extrair requisitos e prazos
- Quebrar um projeto grande em fases e tarefas menores com duração realista
- Definir critérios de aceitação (DoR - Definition of Ready) e conclusão (DoD - Definition of Done)
- Gerar um cronograma visual em formato de gráfico de Gantt
- Identificar dependências entre tarefas e possíveis gargalos
- Criar um roadmap estruturado para comunicar com stakeholders

## Fluxo de Trabalho

### 1. Preparação: Obter Documentação do Projeto

Solicite ao usuário a documentação do projeto em um dos seguintes formatos:

- Documento Markdown com descrição, escopo, requisitos e prazo estimado
- Proposta em PDF ou texto livre
- Briefing do projeto com objetivos e restrições

**Informações essenciais a extrair:**
- Nome e descrição do projeto
- Prazo estimado (em dias, semanas ou meses)
- Escopo e requisitos principais
- Restrições técnicas ou de negócio
- Tamanho estimado do time

### 2. Análise: Estruturar o Projeto

Use o script `analyze_project.py` para processar a documentação:

```bash
python /home/ubuntu/skills/project-manager-gantt/scripts/analyze_project.py <arquivo_projeto.md> <diretório_saída>
```

O script gera três arquivos:

- **project_analysis.json**: Dados estruturados do projeto (fases, tarefas, DoR/DoD)
- **gantt_data.csv**: Dados para visualização em gráfico de Gantt
- **roadmap.md**: Documento de roadmap em Markdown

### 3. Validação: Aplicar DoR/DoD

Consulte `/home/ubuntu/skills/project-manager-gantt/references/dor_dod_guide.md` para:

- Validar que cada tarefa atende aos critérios de DoR (pronta para desenvolvimento)
- Definir critérios de DoD (pronta para produção)
- Identificar tarefas que precisam de mais clareza ou recursos

### 4. Visualização: Gerar Gráfico de Gantt

Com os dados em `gantt_data.csv`, crie um gráfico de Gantt usando:

- Ferramentas de visualização (Mermaid, Plotly, ou similar)
- Formato visual que mostre: tarefas, duração, dependências, fases
- Destaque para marcos (milestones) críticos

### 5. Entrega: Documentação Completa

Entregue ao usuário:

1. **Roadmap em Markdown**: Visão geral das fases e tarefas
2. **Gráfico de Gantt**: Visualização do cronograma
3. **Análise de Riscos**: Dependências, gargalos, prazos críticos
4. **DoR/DoD Customizado**: Critérios específicos para o projeto

## Exemplo de Uso

**Entrada do usuário:**
> "Preciso de um roadmap para um projeto de migração de banco de dados. Temos 3 meses, 5 pessoas no time, e precisa incluir planejamento, design, implementação, testes e deploy."

**Seu fluxo:**

1. Solicitar documento detalhado (ou usar o briefing fornecido)
2. Executar `analyze_project.py` com o documento
3. Revisar e ajustar fases conforme necessário
4. Gerar gráfico de Gantt com as tarefas
5. Definir DoR/DoD específicos para migração de BD
6. Entregar roadmap visual com cronograma realista

## Melhores Práticas

### Estimativa de Prazos

- **Planejamento**: 10-15% do tempo total
- **Design/Arquitetura**: 15-20% do tempo total
- **Desenvolvimento**: 40-50% do tempo total
- **Testes/QA**: 15-20% do tempo total
- **Deploy/Monitoramento**: 5-10% do tempo total

### Quebra de Tarefas

Cada tarefa deve ter duração de **3 a 10 dias**. Se uma tarefa for maior, quebre em subtarefas. Se for menor que 3 dias, agrupe com outras.

### Dependências

Identifique tarefas que:

- Bloqueiam outras (dependência crítica)
- Podem rodar em paralelo (sem dependência)
- Têm dependências externas (terceiros, aprovações)

### Comunicação

- Use linguagem clara e objetiva
- Destaque marcos (milestones) importantes
- Identifique riscos e planos de mitigação
- Comunique prazos realistas, não otimistas

## Recursos Disponíveis

- **Script**: `/home/ubuntu/skills/project-manager-gantt/scripts/analyze_project.py`
- **Referência DoR/DoD**: `/home/ubuntu/skills/project-manager-gantt/references/dor_dod_guide.md`

## Integração com Outras Ferramentas

Este skill funciona bem com:

- **Ferramentas de visualização**: Mermaid, Plotly, Google Sheets
- **Gestão de projetos**: Jira, Asana, Monday.com
- **Documentação**: Confluence, Notion, GitHub Wiki
- **Comunicação**: Slack, Teams (para compartilhar roadmap)

## Notas Importantes

- Sempre solicite documentação clara antes de gerar o roadmap
- Valide prazos com o cliente/stakeholder antes de confirmar
- Revise DoR/DoD com o time técnico para garantir realismo
- Atualize o roadmap conforme o projeto avança
- Identifique e comunique riscos proativamente
