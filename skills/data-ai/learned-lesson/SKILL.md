---
name: learned-lesson
description: Captura e armazena licoes tecnicas de bug fixes ou decisoes arquiteturais
triggers:
  - "licao"
  - "aprendi"
  - "memorizar"
  - "learned"
  - "concluimos"
  - "padrao"
  - "sucesso"
  - "excelencia"
  - "decisao"
globs:
  - ".aidev/memory/kb/*.md"
  - "docs/lessons/*.md"
steps: 4
checkpoints:
  - context_captured
  - root_cause_identified
  - solution_documented
  - lesson_stored
artifact: ".aidev/memory/kb/YYYY-MM-DD-<topic>.md"
previous_skill: systematic-debugging
---

# Learned Lesson Skill

## Metadata
- **Total de Steps**: 4
- **Tempo Estimado**: 10-15 minutos
- **Prerequisito**: Conclusao de tarefa complexa, bug fix ou decisao arquitetural
- **Artefato Final**: `.aidev/memory/kb/YYYY-MM-DD-<topic>.md`

## Quando Usar
Ativa apos a conclusao de uma tarefa complexa, correcao de bug, **implementacao de um padrao de sucesso** ou decisao arquitetural importante. O objetivo e economizar tokens e tempo em sessoes futuras ao reutilizar solucoes de excelencia ou evitar erros repetidos.

## Proposito
Capturar conhecimento tacito (como resolvemos algo ou por que algo funcionou tao bem) e transforma-lo em conhecimento explicito reutilizavel.

---

## Step 1: Capturar Contexto
**Checkpoint**: `context_captured`

### Acoes
Documentar o contexto do aprendizado:

1. Qual erro, excecao, desafio ou **padrao de excelencia** foi identificado?
2. Qual era o comportamento esperado vs observado (para bugs) ou qual a vantagem competitiva (para padroes)?
3. Qual a stack/tecnologia envolvida?
4. Qual a frequencia/impacto ou potencial de reuso?

### Formato de Captura

```markdown
## Contexto

**Stack**: [Ex: Laravel 10 + PHP 8.2]
**Ambiente**: [Ex: Producao, Docker]
**Frequencia**: [Ex: Intermitente, sempre]
**Impacto**: [Ex: Critico, Alto, Medio, Baixo]

### Sintoma Observado
[Descricao do comportamento errado]

### Comportamento Esperado
[Descricao do comportamento correto]

### Evidencia
[Stack trace, log, screenshot]
```

### Criterios de Validacao
- [ ] Contexto claramente descrito
- [ ] Stack/tecnologia identificada
- [ ] Sintoma documentado com evidencia
- [ ] Impacto avaliado

---

## Step 2: Identificar Causa Raiz
**Checkpoint**: `root_cause_identified`

### Acoes
Aplicar tecnica dos 5 Porques:

1. Por que o erro ocorreu?
2. Por que essa condicao existia?
3. Por que nao foi detectado antes?
4. Por que o sistema permitiu?
5. Por que nao havia protecao?

### Formato

```markdown
## Causa Raiz

### Analise (5 Whys)
1. **Por que falhou?** [Resposta]
2. **Por que?** [Resposta]
3. **Por que?** [Resposta]
4. **Por que?** [Resposta]
5. **Por que?** [Causa raiz]

### Causa Raiz Identificada
[Explicacao tecnica clara e concisa]

### Tipo de Problema / Padrao
- [ ] Bug de codigo
- [ ] Configuracao incorreta
- [ ] Padrao de Codificacao (Sucesso)
- [ ] Decisao Arquitetural
- [ ] Otimizacao de Performance
- [ ] Falta de validacao
- [ ] Outro: ___
```

### Criterios de Validacao
- [ ] 5 Whys aplicados
- [ ] Causa raiz tecnica identificada
- [ ] Tipo de problema classificado
- [ ] Nao confunde sintoma com causa

---

## Step 3: Documentar Solucao
**Checkpoint**: `solution_documented`

### Acoes
Documentar a correcao de forma reproduzivel:

1. O que foi mudado (diff/codigo)
2. Por que essa solucao funciona
3. Alternativas consideradas
4. Testes que validam a correcao

### Formato

```markdown
## Solucao

### Correcao Aplicada
```[linguagem]
// Codigo que resolveu o problema
```

### Por Que Funciona
[Explicacao tecnica de por que essa solucao resolve a causa raiz]

### Alternativas Consideradas
| Alternativa | Por que nao escolhida |
|-------------|----------------------|
| ... | ... |

### Validacao
- Teste adicionado: `path/to/test.spec.ts`
- Comando de verificacao: `npm test -- --grep "..."`
```

### Criterios de Validacao
- [ ] Codigo/mudanca documentado
- [ ] Explicacao do porque funciona
- [ ] Teste de regressao adicionado
- [ ] Comando de verificacao funciona

---

## Step 4: Armazenar Licao
**Checkpoint**: `lesson_stored`

### Acoes

#### Memoria Local (Projeto)
Salvar em `.aidev/memory/kb/`:

```bash
# Nome do arquivo
YYYY-MM-DD-<slug-do-problema>.md
```

#### Memoria Global (Cross-Project)
Se aplicavel a outros projetos:

```bash
# Usar basic-memory MCP
mcp__basic-memory__write_note
```

### Template Final da Licao

```markdown
# Licao: [Titulo Curto e Descritivo]

**Data**: YYYY-MM-DD
**Stack**: [Tecnologias envolvidas]
**Tags**: [bug, success-pattern, arch-decision, performance, security, etc]

## Contexto
[Resumo do contexto - do Step 1]

## Problema
[Sintoma e evidencia - do Step 1]

## Causa Raiz
[Explicacao tecnica - do Step 2]

## Solucao
[Codigo/mudanca - do Step 3]

## Prevencao
Como evitar no futuro:
- [ ] Checklist item 1
- [ ] Checklist item 2

## Referencias
- [Link para PR/commit]
- [Link para documentacao]
```

### Criterios de Validacao
- [ ] Arquivo salvo no path correto
- [ ] Todas as secoes preenchidas
- [ ] Tags apropriadas adicionadas
- [ ] Memoria global atualizada (se aplicavel)

---

## Categorias de Licoes

| Categoria | Descricao | Exemplo |
|-----------|-----------|---------|
| `bug` | Correcao de erro | NPE em campo nullable |
| `success-pattern` | Padrao de Excelencia | Implementacao limpa de Repository |
| `arch-decision` | Decisao Arquitetural | Uso de Event Sourcing |
| `config` | Configuracao | Timeout de conexao |
| `performance` | Otimizacao | N+1 query |
| `security` | Vulnerabilidade | SQL injection |
| `integration` | Integracao | API externa |
| `deployment` | Deploy/infra | Docker/K8s |

---

## Transicoes

### Ao Completar com Sucesso
```bash
skill_add_artifact "learned-lesson" ".aidev/memory/kb/YYYY-MM-DD-<topic>.md" "lesson"
skill_complete "learned-lesson"
# Licao disponivel para consulta futura
```

### Em Caso de Falha
```bash
skill_fail "learned-lesson" "Motivo da falha"
# Salvar estado parcial para retomar depois
```

---

## Exemplo de Uso com Orquestrador

```bash
skill_init "learned-lesson"
skill_set_steps "learned-lesson" 4

# Step 1: Capturar contexto
skill_advance "learned-lesson" "Capturar contexto"
# ... documentar contexto, sintoma, evidencia ...
skill_validate_checkpoint "learned-lesson"

# Step 2: Identificar causa raiz
skill_advance "learned-lesson" "Identificar causa raiz"
# ... aplicar 5 whys, classificar problema ...
skill_validate_checkpoint "learned-lesson"

# Step 3: Documentar solucao
skill_advance "learned-lesson" "Documentar solucao"
# ... registrar correcao, validar com teste ...
skill_validate_checkpoint "learned-lesson"

# Step 4: Armazenar licao
skill_advance "learned-lesson" "Armazenar licao"
# ... salvar arquivo, atualizar memoria ...
skill_add_artifact "learned-lesson" ".aidev/memory/kb/2024-01-15-timeout-api.md" "lesson"
skill_validate_checkpoint "learned-lesson"

skill_complete "learned-lesson"
```

---

## Recuperacao de Licoes

Para consultar licoes em sessoes futuras:

```bash
# Listar e buscar (CLI Otimizado)
aidev lessons
aidev lessons --search "termo"
aidev lessons --read "nome-arquivo"

# Busca global (basic-memory)
mcp__basic-memory__search_notes query="palavra-chave"
```

---

## Principios

1. **Capture Imediatamente** - Conhecimento tacito se perde rapido
2. **Seja Especifico** - Detalhes tecnicos importam
3. **Inclua Codigo** - Exemplos concretos sao mais uteis
4. **Adicione Teste** - Previna regressoes
5. **Categorize Bem** - Facilita busca futura