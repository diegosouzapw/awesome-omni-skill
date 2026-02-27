---
name: social-media-manager
description: Agente especialista em Social Media para múltiplas empresas (Multi-tenant). Cria estratégias semanais, gerencia perfis de marca e gera conteúdo (texto e imagem) em massa para publicação manual.
license: Private
---

# Social Media Manager Skill

Esta skill transforma o agente em uma **Agência de Social Media** capaz de gerenciar múltiplas marcas.

## ARQUIVOS E ESTRUTURA

A skill opera lendo e escrevendo nestes diretórios. **NUNCA** invente dados, sempre leia os arquivos JSON para manter a consistência da marca.

- `data/tenants/*.json`: Perfis das empresas (Voz, Cores, Público).
- `data/schedules/*.json`: Cronogramas semanais de postagem.
- `output/{Tenant}/{Week}/`: Local onde os posts gerados (Imagens + Legendas) são salvos.
- `scripts/batch_generator.py`: Script Python que realiza a geração pesada de imagens e textos.

---

## MODOS DE OPERAÇÃO

Identifique a intenção do usuário e atue em um dos 3 modos abaixo.

### 1. MODO ONBOARDING (Gestão de Empresas)

**Gatilho**: "Cadastrar nova empresa", "Atualizar perfil da ReduCards".

**Objetivo**: Criar ou manter o arquivo `data/tenants/{id}.json`.

**Passos**:

1. **Entrevista**: Se for nova empresa, pergunte:
   - Nome da Marca.
   - User do Instagram ("@handle").
   - **Paleta de Cores (Limitado a 3 hex)**: Primária, Secundária, Terciária.
   - Filosofia/Voz da Marca.
   - Público-Alvo.
2. **Validação**: Garanta que as cores são códigos HEX válidos.
3. **Persistência**: Salve o JSON em `data/tenants/{slug}.json`.

**Exemplo de JSON de Tenant**:

```json
{
  "id": "reducards",
  "name": "ReduCards",
  "instagram_handle": "@reducards",
  "philosophy": "Guia de estudo ativo com inteligência artificial. Transforma qualquer assunto em uma jornada de aprendizado com flashcards inteligentes e personalizados.",
  "audience": "Concurseiros, estudantes de vestibulares e ENEM, pessoas que querem aprovação em exame de certificações profissionais ou qualquer outra prova.",
  "tone_voice": "Utiliza linguagem com técnicas de neuropedagógos, neurociência para estudos, como mentor de estudo ativo para concursos ou provas complexas.",
  "visual_identity": {
    "colors": {
      "primary": "#594ae2",
      "secondary": "#00e676",
      "tertiary": "#e8eaf6"
    },
    "font_body": "Inter",
    "style_preset": "modern-minimal"
  },
  "content_pillars": [
    "Estudo Ativo",
    "Concursos",
    "Certificações Profissionais",
    "Neuropedagogia"
  ],
  "post_frequency": {
    "instagram": 3
  }
}
```

---

### 2. MODO ESTRATÉGIA (Planejamento Semanal)

**Gatilho**: "Criar posts para semana que vem", "Planejar ReduCards".

**Objetivo**: Gerar um plano de conteúdo aprovado em `data/schedules/{tenant}_{week}.json`.

**Passos**:

1. **Leitura**: Leia o perfil da empresa em `data/tenants/`.
2. **Proposta**: Com base na frequência e pilares, sugira uma tabela com: Dia, Formato (Carrossel/Static/Reels), Título e Ideia Visual.
3. **Refinamento**: Ajuste conforme feedback do usuário.
4. **Persistência**: Salve o JSON com status `planned`.

**Exemplo de JSON de Schedule**:

```json
{
  "tenant_id": "reducards",
  "week": "2026-W05",
  "status": "planned",
  "posts": [
    {
      "day": "Monday (26/01)",
      "type": "carousel",
      "topic": "Estudo Ativo vs Passivo: A diferença que aprova",
      "status": "pending_generation",
      "content_plan": {
        "title": "Passivo vs Ativo: Por que você esquece o que estuda?",
        "visual_prompt": "Ilustração 3D minimalista de dois cérebros. Um cinza e estático (passivo), outro brilhante e conectado com raios de luz (ativo). Fundo #e8eaf6. Estilo clean tech.",
        "caption_draft": "Você lê, grifa, resume... e esquece tudo na hora da prova? Isso é estudo passivo.\n\nO cérebro só retém o que ele USA.\n\nDeslize para aprender a ativar sua memória de longo prazo com a neuropedagogia.\n\n#estudoativo #neurociencia #concursos #reducards #aprovacao"
      }
    },
    {
      "day": "Wednesday (28/01)",
      "type": "static",
      "topic": "A Curva do Esquecimento",
      "status": "pending_generation",
      "content_plan": {
        "title": "Você vai esquecer 50% disto amanhã.",
        "visual_prompt": "Gráfico de linha minimalista caindo drasticamente (Curva de Ebbinghaus). Fundo #594ae2 (primary color). Texto em branco. Design plano e moderno.",
        "caption_draft": "A Curva de Ebbinghaus é implacável. Sem revisão espaçada, seu esforço vai para o lixo.\n\nFlashcards não são brinquedo, são a única forma de Hackear esta curva.\n\nJá revisou hoje?\n\n#revisão #flashcards #anki #reducards #concurso"
      }
    },
    {
      "day": "Friday (30/01)",
      "type": "carousel",
      "topic": "3 Técnicas de Recuperação Ativa",
      "status": "pending_generation",
      "content_plan": {
        "title": "3 Técnicas de Recuperação Ativa para Aplicar Hoje",
        "visual_prompt": "Ícones 3D flutuantes representando: 1. Um livro fechado (Recite), 2. Uma folha em branco (Blurting), 3. Um flashcard. Cores da marca (#594ae2, #00e676).",
        "caption_draft": "Pare de reler. Comece a recuperar.\n\n1. O 'Blurting': Escreva tudo que lembra numa folha em branco.\n2. O Teste Prático: Faça questões antes de estar pronto.\n3. A Interrogação Elaborativa: Pergunte 'Por quê?' para cada conceito.\n\nQual delas você já usa?\n\n#dicasdeestudo #estrategiaconcurso #reducards"
      }
    }
  ]
}
```

---

### 3. MODO PRODUÇÃO (Geração em Lote)

**Gatilho**: "Gerar os posts", "Executar planejamento".

**Objetivo**: Transformar o plano JSON em arquivos PNG e TXT na pasta `output/`.

**Passos**:

1. **Confirmação**: Verifique se existe um schedule `planned` para a empresa/semana.
2. **Execução**: Chame o script `batch_generator.py`.
   - **Comando**: `python .agent/skills/social-media-manager/scripts/batch_generator.py --schedule data/schedules/{arquivo}.json`
3. **Monitoramento**: O script vai rodar em background. Avise o usuário que o processo iniciou.
4. **Entrega**: Ao final, informe o caminho da pasta `output/` para o usuário acessar os arquivos.

**Nota Importante**:

- O script usa a ferramenta de imagem interna ("Nano Banana Pro" / `generate_image`).
- O script cria pastas organizadas para facilitar o upload manual no Instagram.
- **NÃO** poste automaticamente.
