---
name: brainstorming
description: Socratic questioning protocol + user communication. MANDATORY for complex requests, new features, or unclear requirements. Includes progress reporting and error handling.
allowed-tools: Read, Glob, Grep
---

# Brainstorming & Communication Protocol

> **MANDATORY:** Use for complex/vague requests, new features, updates.

---

## 🛑 SOCRATIC GATE (ENFORCEMENT)

### When to Trigger

| Pattern | Action |
|---------|--------|
| "Build/Create/Make [thing]" without details | 🛑 ASK 3 questions |
| Complex feature or architecture | 🛑 Clarify before implementing |
| Update/change request | 🛑 Confirm scope |
| Vague requirements | 🛑 Ask purpose, users, constraints |

### 🚫 MANDATORY: 3 Questions Before Implementation

1. **STOP** - Do NOT start coding
2. **ASK** - Minimum 3 questions:
   - 🎯 Purpose: What problem are you solving?
   - 👥 Users: Who will use this?
   - 📦 Scope: Must-have vs nice-to-have?
3. **WAIT** - Get response before proceeding

---

## 🧠 Dynamic Question Generation

**⛔ NEVER use static templates.** Read `dynamic-questioning.md` for principles.

### Core Principles

| Principle | Meaning |
|-----------|---------|
| **Questions Reveal Consequences** | Each question connects to an architectural decision |
| **Context Before Content** | Understand greenfield/feature/refactor/debug context first |
| **Minimum Viable Questions** | Each question must eliminate implementation paths |
| **Generate Data, Not Assumptions** | Don't guess—ask with trade-offs |

### Question Generation Process

```
1. Parse request → Extract domain, features, scale indicators
2. Identify decision points → Blocking vs. deferable
3. Generate questions → Priority: P0 (blocking) > P1 (high-leverage) > P2 (nice-to-have)
4. Format with trade-offs → What, Why, Options, Default
```

### Question Format (MANDATORY)

**🔴 FORMATO OBRIGATÓRIO: Múltipla Escolha (A, B, C, D)**

Todas as perguntas DEVEM seguir este formato para permitir respostas rápidas:

```markdown
### [PRIORIDADE] **[PONTO DE DECISÃO]**

**Pergunta:** [Pergunta clara]

**Por que isso importa:**
- [Consequência arquitetural]
- [Afeta: custo/complexidade/prazo/escala]

**Opções:**
A) [Opção 1] - [Descrição breve] - Melhor para: [Caso de uso]
B) [Opção 2] - [Descrição breve] - Melhor para: [Caso de uso]
C) [Opção 3] - [Descrição breve] - Melhor para: [Caso de uso]
D) Outra: [Descreva sua solução]

**Se não especificado:** [Padrão + justificativa] (Sugestão: Opção [X])
```

### 📋 Formato de Resposta Rápida (Para o Usuário)

O usuário pode responder **MÚLTIPLAS PERGUNTAS** em uma única linha:

**Exemplo:**
```
Perguntas 1-6: A, C, C, B, D: "Quero usar PostgreSQL com Prisma", A
```

**Interpretação:**
- Pergunta 1: Opção A
- Pergunta 2: Opção C
- Pergunta 3: Opção C
- Pergunta 4: Opção B
- Pergunta 5: Opção D com explicação "Quero usar PostgreSQL com Prisma"
- Pergunta 6: Opção A

### 🔄 Processamento de Respostas

Quando o usuário responder no formato rápido:

1. **Parse a string** separando por vírgulas
2. **Identificar opção D:** Se houver "D: 'texto'", extrair o texto explicativo
3. **Mapear respostas** para cada pergunta na ordem
4. **Confirmar entendimento** antes de prosseguir:
   ```markdown
   ✅ **Entendi suas respostas:**
   1. [Pergunta 1]: Opção A - [Descrição]
   2. [Pergunta 2]: Opção C - [Descrição]
   ...
   
   Posso prosseguir com essa configuração?
   ```

### 🔴 Protocolo Flexível (NOVA FEATURE)

**O usuário tem TOTAL FLEXIBILIDADE:**

**Opções de Resposta:**
- **Responder tudo:** `A, C, B, D: "Explicação", A`
- **Pular questionário completo:** `skip` ou `pular` (usa padrões)
- **Responder seletivamente:** `skip, C, skip, B` (só perguntas 2 e 4)
- **Apenas importantes:** `skip, skip, skip, D: "Texto"` (só pergunta 4)

**Processamento de `skip`:**
1. Se `skip`/`pular` sozinho → Usar TODOS os padrões sugeridos
2. Se `skip` em posição específica → Usar padrão daquela pergunta
3. **SEMPRE confirmar** ALL escolhas (respondidas + padrões)

**Exemplo:**
```
Usuário: skip, C, skip, D: "Supabase"

✅ Entendi:
1. Framework: A - Next.js (padrão)
2. Estilização: C - Styled Components (sua escolha)
3. Auth: C - Não agora (padrão)  
4. Banco: D - Supabase (sua escolha)

Prosseguir?
```


**For detailed domain-specific question banks and algorithms**, see: `dynamic-questioning.md`

---

## 📚 Exemplo Completo do Novo Formato

### Exemplo de Perguntas do Agente:

```markdown
Preciso de algumas informações antes de começar:

### [P0] **1. FRAMEWORK FRONTEND**
**Pergunta:** Qual framework você quer usar para o frontend?
**Por que isso importa:** Define a estrutura do projeto, biblioteca de componentes e tooling.
**Opções:**
A) React + Next.js - Framework completo com SSR - Melhor para: SEO e performance
B) Vue + Nuxt - Framework progressivo - Melhor para: Projetos médios, curva de aprendizado suave
C) React (Vite) - Build rápido, sem SSR - Melhor para: SPAs simples
D) Outra: [Descreva qual framework]
**Se não especificado:** React + Next.js (Sugestão: Opção A)

### [P0] **2. ESTILIZAÇÃO**
**Pergunta:** Como prefere estilizar os componentes?
**Opções:**
A) Tailwind CSS - Utility-first - Melhor para: Desenvolvimento rápido
B) CSS Modules - Scoped CSS - Melhor para: Manutenção de longo prazo
C) Styled Components - CSS-in-JS - Melhor para: Temas dinâmicos
D) Outra: [Descreva sua preferência]
**Se não especificado:** Tailwind CSS (Sugestão: Opção A)

### [P1] **3. AUTENTICAÇÃO**
**Pergunta:** Precisa de autenticação de usuários?
**Opções:**
A) Sim, com email/senha
B) Sim, com OAuth (Google, GitHub, etc.)
C) Não precisa agora
D) Outra: [Descreva o método]
**Se não especificado:** Não precisa agora (Sugestão: Opção C)

**📋 Responda todas de uma vez:** `A, B, C` ou `A, C, D: "Explicação"` etc.
```

### Exemplo de Resposta do Usuário:

```
A, A, B
```

### Confirmação do Agente:

```markdown
✅ **Entendi suas respostas:**

1. **Framework Frontend:** Opção A - React + Next.js (Framework completo com SSR)
2. **Estilização:** Opção A - Tailwind CSS (Utility-first)
3. **Autenticação:** Opção B - Sim, com OAuth (Google, GitHub, etc.)

Posso prosseguir com essa configuração?
```

### Exemplo com Opção D (Outra):

**Usuário responde:**
```
A, D: "Quero usar Emotion para CSS-in-JS", C
```

**Agente confirma:**
```markdown
✅ **Entendi suas respostas:**

1. **Framework Frontend:** Opção A - React + Next.js
2. **Estilização:** Opção D - Quero usar Emotion para CSS-in-JS
3. **Autenticação:** Opção C - Não precisa agora

Posso prosseguir com essa configuração?
```

---

## Progress Reporting (PRINCIPLE-BASED)

**PRINCIPLE:** Transparency builds trust. Status must be visible and actionable.

### Status Board Format

| Agent | Status | Current Task | Progress |
|-------|--------|--------------|----------|
| [Agent Name] | ✅🔄⏳❌⚠️ | [Task description] | [% or count] |

### Status Icons

| Icon | Meaning | Usage |
|------|---------|-------|
| ✅ | Completed | Task finished successfully |
| 🔄 | Running | Currently executing |
| ⏳ | Waiting | Blocked, waiting for dependency |
| ❌ | Error | Failed, needs attention |
| ⚠️ | Warning | Potential issue, not blocking |

---

## Error Handling (PRINCIPLE-BASED)

**PRINCIPLE:** Errors are opportunities for clear communication.

### Error Response Pattern

```
1. Acknowledge the error
2. Explain what happened (user-friendly)
3. Offer specific solutions with trade-offs
4. Ask user to choose or provide alternative
```

### Error Categories

| Category | Response Strategy |
|----------|-------------------|
| **Port Conflict** | Offer alternative port or close existing |
| **Dependency Missing** | Auto-install or ask permission |
| **Build Failure** | Show specific error + suggested fix |
| **Unclear Error** | Ask for specifics: screenshot, console output |

---

## Completion Message (PRINCIPLE-BASED)

**PRINCIPLE:** Celebrate success, guide next steps.

### Completion Structure

```
1. Success confirmation (celebrate briefly)
2. Summary of what was done (concrete)
3. How to verify/test (actionable)
4. Next steps suggestion (proactive)
```

---

## Communication Principles

| Principle | Implementation |
|-----------|----------------|
| **Concise** | No unnecessary details, get to point |
| **Visual** | Use emojis (✅🔄⏳❌) for quick scanning |
| **Specific** | "~2 minutes" not "wait a bit" |
| **Alternatives** | Offer multiple paths when stuck |
| **Proactive** | Suggest next step after completion |

---

## Anti-Patterns (AVOID)

| Anti-Pattern | Why |
|--------------|-----|
| Jumping to solutions before understanding | Wastes time on wrong problem |
| Assuming requirements without asking | Creates wrong output |
| Over-engineering first version | Delays value delivery |
| Ignoring constraints | Creates unusable solutions |
| "I think" phrases | Uncertainty → Ask instead |

---
