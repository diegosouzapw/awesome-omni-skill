---
name: systematic-debugging
description: Processo de 4 fases para encontrar causa raiz de bugs
triggers:
  - "bug"
  - "erro"
  - "debug"
  - "nao funciona"
  - "quebrou"
  - "falha"
globs:
  - "**/*.log"
  - ".aidev/state/lessons/**"
  - ".aidev/memory/kb/**"
steps: 4
checkpoints:
  - bug_reproduced
  - bug_isolated
  - root_cause_found
  - fix_verified
artifact: ".aidev/memory/kb/YYYY-MM-DD-<bug>-lesson.md"
next_skill: learned-lesson
---

# Systematic Debugging Skill

## Metadata
- **Total de Steps**: 4 (REPRODUCE, ISOLATE, ROOT CAUSE, FIX)
- **Tempo Estimado**: Variavel (depende da complexidade)
- **Artefato Final**: Licao aprendida em `.aidev/memory/kb/`
- **Proxima Skill**: `learned-lesson` (para registrar conhecimento)

## Quando Usar
Ao encontrar bugs, erros ou comportamentos inesperados.

## Principio Fundamental
> Nunca "chute" a causa de um bug.
> Siga o processo sistematicamente ate encontrar a causa raiz.

---

## Step 1: REPRODUCE (Reproduzir)
**Checkpoint**: `bug_reproduced`

### Objetivo
Fazer o bug acontecer de forma confiavel e previsivel.

### Acoes

#### 1.1 Reproducao Minima
- Encontrar os passos mais simples para provocar o bug
- Isolar de outros fatores (cache, estado anterior, etc)
- Documentar os passos exatos

```markdown
## Passos para Reproduzir
1. [Passo 1]
2. [Passo 2]
3. [Passo 3]

**Resultado Esperado**: [O que deveria acontecer]
**Resultado Atual**: [O que esta acontecendo]
```

#### 1.2 Capturar Evidencias
- Mensagens de erro completas
- Stack traces
- Logs relevantes
- Screenshots/videos (se UI)

#### 1.3 Criar Teste que Falha
```bash
# O teste DEVE falhar atualmente
# Este teste validara o fix depois
```

```javascript
it('should [comportamento esperado]', () => {
  // Arrange - setup que reproduz o bug
  // Act - acao que causa o bug
  // Assert - verificacao que falha
});
```

### Criterios de Validacao
- [ ] Bug reproduzido consistentemente
- [ ] Passos documentados
- [ ] Evidencias capturadas
- [ ] Teste que falha criado

### Confianca
- Se consegue reproduzir sempre: `confidence: 0.9`
- Se intermitente: `confidence: 0.5` (precisa mais investigacao)

---

## Step 2: ISOLATE (Isolar)
**Checkpoint**: `bug_isolated`

### Objetivo
Encontrar ONDE no codigo o bug se origina.

### Acoes

#### 2.1 Busca Binaria
```
1. Dividir o codigo ao meio
2. Verificar em qual metade o bug esta
3. Repetir ate encontrar o local exato
```

#### 2.2 Adicionar Logging Estrategico
```javascript
console.log('[DEBUG] Ponto A:', variavel);
// ... codigo ...
console.log('[DEBUG] Ponto B:', variavel);
```

#### 2.3 Verificar Suposicoes
- Os inputs estao corretos?
- O estado esta como esperado?
- As dependencias estao funcionando?

### Perguntas-Chave
- Em que arquivo o bug se manifesta?
- Em que funcao/metodo?
- Em que linha?
- Qual variavel/estado esta incorreto?

### Criterios de Validacao
- [ ] Local exato identificado (arquivo:linha)
- [ ] Variavel/estado incorreto identificado
- [ ] Suposicoes verificadas

---

## Step 3: ROOT CAUSE (Causa Raiz)
**Checkpoint**: `root_cause_found`

### Objetivo
Entender POR QUE o bug esta acontecendo.

### Acoes

#### 3.1 Rastrear para Tras
- Do sintoma ate a causa
- Seguir o fluxo de dados
- Verificar cada transformacao

#### 3.2 Tecnica dos 5 Porques
```
Sintoma: "O formulario nao submete"
  Por que? "A validacao falha"
  Por que? "O campo esta null"
  Por que? "O binding nao funcionou"
  Por que? "Falta wire:model no input"
  Por que? "Template foi copiado sem ajustar"

CAUSA RAIZ: Template copiado sem ajustar bindings
```

#### 3.3 Documentar Entendimento
```markdown
## Analise de Causa Raiz

**O que esta acontecendo:**
[Descricao do comportamento atual]

**O que deveria acontecer:**
[Descricao do comportamento esperado]

**Por que esta errado:**
[Explicacao tecnica da causa]

**Onde esta o problema:**
[Arquivo:linha ou componente]
```

### Criterios de Validacao
- [ ] Causa raiz identificada (nao apenas sintoma)
- [ ] Explicacao tecnica clara
- [ ] Entendimento documentado

---

## Step 4: FIX & PREVENT (Corrigir e Prevenir)
**Checkpoint**: `fix_verified`

### Objetivo
Corrigir o bug e prevenir que aconteca novamente.

### Acoes

#### 4.1 Verificar Teste Falha
```bash
# Confirmar que teste ainda falha antes do fix
npm test -- --testNamePattern="[nome do teste]"
```

#### 4.2 Implementar Fix Minimo
- Menor mudanca possivel para corrigir
- NAO refatorar enquanto corrige
- Focar APENAS no bug

```bash
# Antes de modificar
validation_check "safe_path" "$filepath"
```

#### 4.3 Verificar Teste Passa
```bash
# Teste deve passar agora
npm test -- --testNamePattern="[nome do teste]"
```

#### 4.4 Executar Suite Completa
```bash
# Nenhuma regressao
npm test
```

#### 4.5 Documentar Licao Aprendida
Ativar skill `learned-lesson` para registrar:

```markdown
# Licao: [Titulo do Bug]

**Data**: YYYY-MM-DD
**Contexto**: [Onde/quando ocorreu]

## Sintoma
[O que foi observado]

## Causa Raiz
[Por que aconteceu - explicacao tecnica]

## Correcao
[O que foi mudado - codigo ou configuracao]

## Prevencao
[Como evitar no futuro]
- [ ] Adicionar validacao X
- [ ] Criar teste para caso Y
- [ ] Documentar padrao Z
```

### Criterios de Validacao
- [ ] Teste que falhava agora passa
- [ ] Nenhuma regressao (suite completa passa)
- [ ] Licao documentada
- [ ] Commit atomico com mensagem descritiva

---

## Transicoes

### Ao Completar com Sucesso
```bash
skill_add_artifact "systematic-debugging" ".aidev/memory/kb/YYYY-MM-DD-<bug>.md" "lesson"
skill_complete "systematic-debugging"

# Registrar licao permanentemente
skill_init "learned-lesson"
```

### Se Causa Raiz Nao Encontrada
```bash
# Pedir ajuda ou mais contexto
confidence_log "Causa raiz nao identificada" 0.3 "low"
# Sugerir: revisar codigo com outro desenvolvedor, adicionar mais logs
```

---

## Anti-Patterns a Evitar

### 1. Chutar a Causa
```
ERRADO: "Acho que e cache, vou limpar"
CERTO:  Seguir o processo ate ter certeza
```

### 2. Corrigir Sintoma
```
ERRADO: Adicionar try-catch para esconder erro
CERTO:  Encontrar e corrigir a causa raiz
```

### 3. Refatorar Durante Debug
```
ERRADO: "Ja que estou aqui, vou melhorar esse codigo"
CERTO:  Focar APENAS no bug, refatorar depois
```

### 4. Nao Criar Teste
```
ERRADO: Corrigir sem teste de regressao
CERTO:  Sempre criar teste que falha antes do fix
```

---

## Exemplo de Uso com Orquestrador

```bash
skill_init "systematic-debugging"
skill_set_steps "systematic-debugging" 4

# Step 1: REPRODUCE
skill_advance "systematic-debugging" "REPRODUCE: Reproduzir o bug"
# ... documentar passos, criar teste que falha ...
skill_validate_checkpoint "systematic-debugging"

# Step 2: ISOLATE
skill_advance "systematic-debugging" "ISOLATE: Isolar origem"
# ... busca binaria, logging ...
skill_validate_checkpoint "systematic-debugging"

# Step 3: ROOT CAUSE
skill_advance "systematic-debugging" "ROOT CAUSE: 5 Porques"
# ... analise de causa raiz ...
skill_validate_checkpoint "systematic-debugging"

# Step 4: FIX & PREVENT
skill_advance "systematic-debugging" "FIX: Corrigir e verificar"
validation_check "tests_pass"
skill_add_artifact "systematic-debugging" ".aidev/memory/kb/2024-01-15-null-pointer.md" "lesson"
skill_validate_checkpoint "systematic-debugging"

skill_complete "systematic-debugging"
# Handoff para registrar licao permanentemente
agent_handoff "qa" "orchestrator" "Registrar licao aprendida" ".aidev/memory/kb/2024-01-15-null-pointer.md"
```

---

## Integracao com Knowledge Base

Apos resolver o bug, a licao deve ser salva em dois lugares:

### 1. Local (Projeto)
```
.aidev/memory/kb/YYYY-MM-DD-<titulo>.md
```

### 2. Global (Cross-Project via MCP)
```bash
# Se a solucao for generica (ex: bug de versao de biblioteca)
mcp__basic-memory__write_note(
  title: "Bug: [titulo]",
  content: "[conteudo da licao]",
  directory: "lessons"
)
```