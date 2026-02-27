---
name: doc-coauthoring
description: Guia os usuários através de um fluxo de trabalho estruturado para coautoria de documentação. Use quando o usuário quiser escrever documentação, propostas, especificações técnicas, documentos de decisão ou conteúdo estruturado semelhante. Este fluxo de trabalho ajuda os usuários a transferir contexto de forma eficiente, refinar o conteúdo através de iteração e verificar se o documento funciona para os leitores. Acione quando o usuário mencionar escrever documentos, criar propostas, redigir especificações ou tarefas de documentação semelhantes.
---

# Fluxo de Trabalho de Coautoria de Documentos

Esta habilidade fornece um fluxo de trabalho estruturado para guiar os usuários através da criação colaborativa de documentos. Aja como um guia ativo, conduzindo os usuários por três estágios: Coleta de Contexto, Refinamento e Estrutura, e Teste com Leitor.

## Quando Oferecer Este Fluxo de Trabalho

**Condições de acionamento:**

- O usuário menciona escrever documentação: "escrever um doc", "redigir uma proposta", "criar uma especificação", "fazer um write up"
- O usuário menciona tipos específicos de documentos: "PRD", "documento de design", "documento de decisão", "RFC"
- O usuário parece estar começando uma tarefa substancial de escrita

**Oferta inicial:**
Ofereça ao usuário um fluxo de trabalho estruturado para coautoria do documento. Explique os três estágios:

1. **Coleta de Contexto**: O usuário fornece todo o contexto relevante enquanto o Claude faz perguntas de esclarecimento
2. **Refinamento e Estrutura**: Construa iterativamente cada seção através de brainstorming e edição
3. **Teste com Leitor**: Teste o documento com um Claude novo (sem contexto) para detectar pontos cegos antes que outros o leiam

Explique que essa abordagem ajuda a garantir que o documento funcione bem quando outros o lerem (incluindo quando o colarem no Claude). Pergunte se eles querem experimentar este fluxo de trabalho ou preferem trabalhar de forma livre.

Se o usuário recusar, trabalhe de forma livre. Se o usuário aceitar, prossiga para o Estágio 1.

## Estágio 1: Coleta de Contexto

**Objetivo:** Fechar a lacuna entre o que o usuário sabe e o que o Claude sabe, permitindo orientação inteligente mais tarde.

### Perguntas Iniciais

Comece pedindo ao usuário o meta-contexto sobre o documento:

1. Qual é o tipo de documento? (por exemplo, especificação técnica, documento de decisão, proposta)
2. Quem é o público principal?
3. Qual é o impacto desejado quando alguém ler isso?
4. Existe um modelo ou formato específico a seguir?
5. Alguma outra restrição ou contexto a saber?

Informe a eles que podem responder de forma abreviada ou despejar informações da maneira que for melhor para eles.

**Se o usuário fornecer um modelo ou mencionar um tipo de documento:**

- Pergunte se eles têm um documento modelo para compartilhar
- Se fornecerem um link para um documento compartilhado, use a integração apropriada para buscá-lo
- Se fornecerem um arquivo, leia-o

**Se o usuário mencionar editar um documento compartilhado existente:**

- Use a integração apropriada para ler o estado atual
- Verifique se há imagens sem texto alternativo
- Se existirem imagens sem texto alternativo, explique que quando outros usarem o Claude para entender o documento, o Claude não poderá vê-las. Pergunte se eles querem que o texto alternativo seja gerado. Se sim, solicite que colem cada imagem no chat para geração de texto alternativo descritivo.

### Despejo de Informações (Info Dumping)

Uma vez que as perguntas iniciais sejam respondidas, encoraje o usuário a despejar todo o contexto que eles têm. Solicite informações como:

- Antecedentes sobre o projeto/problema
- Discussões de equipe relacionadas ou documentos compartilhados
- Por que soluções alternativas não estão sendo usadas
- Contexto organizacional (dinâmica de equipe, incidentes passados, política)
- Pressões ou restrições de cronograma
- Arquitetura técnica ou dependências
- Preocupações das partes interessadas

Aconselhe-os a não se preocupar em organizá-lo - apenas coloque tudo para fora. Ofereça várias maneiras de fornecer contexto:

- Fluxo de consciência de despejo de informações
- Aponte para canais de equipe ou tópicos para ler
- Link para documentos compartilhados

**Se integrações estiverem disponíveis** (por exemplo, Slack, Teams, Google Drive, SharePoint ou outros servidores MCP), mencione que estes podem ser usados para extrair contexto diretamente.

**Se nenhuma integração for detectada e estiver no Claude.ai ou aplicativo Claude:** Sugira que eles podem habilitar conectores em suas configurações do Claude para permitir a extração de contexto de aplicativos de mensagens e armazenamento de documentos diretamente.

Informe-os de que perguntas de esclarecimento serão feitas assim que fizerem seu despejo inicial.

**Durante a coleta de contexto:**

- Se o usuário mencionar canais de equipe ou documentos compartilhados:
  - Se integrações disponíveis: Informe-os de que o conteúdo será lido agora, então use a integração apropriada
  - Se integrações não disponíveis: Explique a falta de acesso. Sugira que habilitem conectores nas configurações do Claude ou colem o conteúdo relevante diretamente.

- Se o usuário mencionar entidades/projetos que são desconhecidos:
  - Pergunte se ferramentas conectadas devem ser pesquisadas para saber mais
  - Aguarde a confirmação do usuário antes de pesquisar

- À medida que o usuário fornece contexto, rastreie o que está sendo aprendido e o que ainda não está claro

**Fazendo perguntas de esclarecimento:**

Quando o usuário sinalizar que fez seu despejo inicial (ou após contexto substancial fornecido), faça perguntas de esclarecimento para garantir o entendimento:

Gere 5-10 perguntas numeradas com base em lacunas no contexto.

Informe a eles que podem usar taquigrafia para responder (por exemplo, "1: sim, 2: veja canal X, 3: não porque compatibilidade reversa"), vincular a mais documentos, apontar para canais para ler ou apenas continuar despejando informações. O que for mais eficiente para eles.

**Condição de saída:**
Contexto suficiente foi reunido quando as perguntas mostram entendimento - quando casos extremos e compensações podem ser perguntados sem precisar que o básico seja explicado.

**Transição:**
Pergunte se há mais algum contexto que eles queiram fornecer nesta fase ou se é hora de passar para a redação do documento.

Se o usuário quiser adicionar mais, deixe-o. Quando estiver pronto, prossiga para o Estágio 2.

## Estágio 2: Refinamento e Estrutura

**Objetivo:** Construir a seção do documento por seção através de brainstorming, curadoria e refinamento iterativo.

**Instruções para o usuário:**
Explique que o documento será construído seção por seção. Para cada seção:

1. Perguntas de esclarecimento serão feitas sobre o que incluir
2. 5-20 opções serão debatidas (brainstorming)
3. O usuário indicará o que manter/remover/combinar
4. A seção será redigida
5. Será refinada através de edições cirúrgicas

Comece com a seção que tiver mais incógnitas (geralmente a decisão/proposta principal), depois trabalhe no resto.

**Ordenação das seções:**

Se a estrutura do documento for clara:
Pergunte com qual seção eles gostariam de começar.

Sugira começar com qualquer seção que tenha mais incógnitas. Para documentos de decisão, geralmente é a proposta central. Para especificações, geralmente é a abordagem técnica. As seções de resumo são melhor deixadas para o final.

Se o usuário não souber quais seções precisa:
Com base no tipo de documento e modelo, sugira 3-5 seções apropriadas para o tipo de documento.

Pergunte se essa estrutura funciona ou se eles querem ajustá-la.

**Uma vez que a estrutura seja acordada:**

Crie a estrutura inicial do documento com texto de espaço reservado para todas as seções.

**Se o acesso a artefatos estiver disponível:**
Use `create_file` para criar um artefato. Isso dá tanto ao Claude quanto ao usuário um andaime para trabalhar.

Informe-os de que a estrutura inicial com espaços reservados para todas as seções será criada.

Crie um artefato com todos os cabeçalhos de seção e breve texto de espaço reservado como "[A ser escrito]" ou "[Conteúdo aqui]".

Forneça o link do andaime e indique que é hora de preencher cada seção.

**Se não houver acesso a artefatos:**
Crie um arquivo markdown no diretório de trabalho. Nomeie-o apropriadamente (por exemplo, `documento-decisao.md`, `especificacao-tecnica.md`).

Informe-os de que a estrutura inicial com espaços reservados para todas as seções será criada.

Crie um arquivo com todos os cabeçalhos de seção e texto de espaço reservado.

Confirme que o nome do arquivo foi criado e indique que é hora de preencher cada seção.

**Para cada seção:**

### Passo 1: Perguntas de Esclarecimento

Anuncie que o trabalho começará na seção [NOME DA SEÇÃO]. Faça 5-10 perguntas de esclarecimento sobre o que deve ser incluído:

Gere 5-10 perguntas específicas com base no contexto e no propósito da seção.

Informe a eles que podem responder de forma abreviada ou apenas indicar o que é importante cobrir.

### Passo 2: Brainstorming

Para a seção [NOME DA SEÇÃO], faça um brainstorming de [5-20] coisas que podem ser incluídas, dependendo da complexidade da seção. Procure por:

- Contexto compartilhado que pode ter sido esquecido
- Ângulos ou considerações ainda não mencionados

Gere 5-20 opções numeradas com base na complexidade da seção. No final, ofereça para fazer brainstorming de mais se eles quiserem opções adicionais.

### Passo 3: Curadoria

Pergunte quais pontos devem ser mantidos, removidos ou combinados. Solicite breves justificativas para ajudar a aprender prioridades para as próximas seções.

Forneça exemplos:

- "Manter 1,4,7,9"
- "Remover 3 (duplica 1)"
- "Remover 6 (público já sabe isso)"
- "Combinar 11 e 12"

**Se o usuário der feedback livre** (por exemplo, "parece bom" ou "gosto da maior parte, mas...") em vez de seleções numeradas, extraia suas preferências e prossiga. Analise o que eles querem mantido/removido/alterado e aplique.

### Passo 4: Verificação de Lacunas

Com base no que eles selecionaram, pergunte se há algo importante faltando para a seção [NOME DA SEÇÃO].

### Passo 5: Redação

Use `str_replace` para substituir o texto de espaço reservado para esta seção pelo conteúdo real redigido.

Anuncie que a seção [NOME DA SEÇÃO] será redigida agora com base no que eles selecionaram.

**Se estiver usando artefatos:**
Após a redação, forneça um link para o artefato.

Peça para eles lerem e indicarem o que mudar. Observe que ser específico ajuda a aprender para as próximas seções.

**Se estiver usando um arquivo (sem artefatos):**
Após a redação, confirme a conclusão.

Informe a eles que a seção [NOME DA SEÇÃO] foi redigida em [nome do arquivo]. Peça para eles lerem e indicarem o que mudar. Observe que ser específico ajuda a aprender para as próximas seções.

**Instrução chave para o usuário (incluir ao redigir a primeira seção):**
Forneça uma nota: Em vez de editar o documento diretamente, peça para eles indicarem o que mudar. Isso ajuda a aprender seu estilo para seções futuras. Por exemplo: "Remova a bala X - já coberta por Y" ou "Torne o terceiro parágrafo mais conciso".

### Passo 6: Refinamento Iterativo

À medida que o usuário fornece feedback:

- Use `str_replace` para fazer edições (nunca reimprima o documento inteiro)
- **Se estiver usando artefatos:** Forneça link para o artefato após cada edição
- **Se estiver usando arquivos:** Apenas confirme que as edições estão completas
- Se o usuário editar o documento diretamente e pedir para lê-lo: mentalmente note as alterações que eles fizeram e mantenha-as em mente para seções futuras (isso mostra suas preferências)

**Continue iterando** até que o usuário esteja satisfeito com a seção.

### Verificação de Qualidade

Após 3 iterações consecutivas sem alterações substanciais, pergunte se algo pode ser removido sem perder informações importantes.

Quando a seção estiver pronta, confirme que [NOME DA SEÇÃO] está completa. Pergunte se está pronto para passar para a próxima seção.

**Repita para todas as seções.**

### Perto da Conclusão

Ao se aproximar da conclusão (80%+ das seções feitas), anuncie a intenção de reler todo o documento e verificar:

- Fluxo e consistência entre as seções
- Redundância ou contradições
- Qualquer coisa que pareça "encheção de linguiça" ou preenchimento genérico
- Se cada frase carrega peso

Leia todo o documento e forneça feedback.

**Quando todas as seções estiverem redigidas e refinadas:**
Anuncie que todas as seções estão redigidas. Indique intenção de revisar o documento completo mais uma vez.

Revise quanto à coerência geral, fluxo, completude.

Forneça quaisquer sugestões finais.

Pergunte se está pronto para passar para o Teste com Leitor ou se eles querem refinar mais alguma coisa.

## Estágio 3: Teste com Leitor

**Objetivo:** Testar o documento com um Claude novo (sem vazamento de contexto) para verificar se funciona para os leitores.

**Instruções para o usuário:**
Explique que o teste ocorrerá agora para ver se o documento realmente funciona para os leitores. Isso detecta pontos cegos - coisas que fazem sentido para os autores, mas podem confundir os outros.

### Abordagem de Teste

**Se o acesso a subagentes estiver disponível (por exemplo, no Claude Code):**

Realize o teste diretamente sem envolvimento do usuário.

### Passo 1: Prever Perguntas dos Leitores

Anuncie a intenção de prever quais perguntas os leitores podem fazer ao tentar descobrir este documento.

Gere 5-10 perguntas que os leitores realisticamente fariam.

### Passo 2: Teste com Subagente

Anuncie que essas perguntas serão testadas com uma nova instância do Claude (sem contexto desta conversa).

Para cada pergunta, invoque um subagente apenas com o conteúdo do documento e a pergunta.

Resuma o que o Leitor Claude acertou/errou para cada pergunta.

### Passo 3: Executar Verificações Adicionais

Anuncie que verificações adicionais serão realizadas.

Invoque o subagente para verificar ambiguidade, suposições falsas, contradições.

Resuma quaisquer problemas encontrados.

### Passo 4: Relatar e Corrigir

Se problemas forem encontrados:
Relate que o Leitor Claude teve dificuldades com problemas específicos.

Liste os problemas específicos.

Indique intenção de corrigir essas lacunas.

Volte para o refinamento para seções problemáticas.

---

**Se não houver acesso a subagentes (por exemplo, interface web claude.ai):**

O usuário precisará fazer o teste manualmente.

### Passo 1: Prever Perguntas dos Leitores

Pergunte quais perguntas as pessoas podem fazer ao tentar descobrir este documento. O que elas digitariam no Claude.ai?

Gere 5-10 perguntas que os leitores realisticamente fariam.

### Passo 2: Configurar Teste

Forneça instruções de teste:

1. Abra uma nova conversa do Claude: https://claude.ai
2. Cole ou compartilhe o conteúdo do documento (se estiver usando uma plataforma de documentos compartilhados com conectores habilitados, forneça o link)
3. Faça ao Leitor Claude as perguntas geradas

Para cada pergunta, instrua o Leitor Claude a fornecer:

- A resposta
- Se algo foi ambíguo ou pouco claro
- Que conhecimento/contexto o documento assume que já é conhecido

Verifique se o Leitor Claude dá respostas corretas ou interpreta mal alguma coisa.

### Passo 3: Verificações Adicionais

Também pergunte ao Leitor Claude:

- "O que neste documento pode ser ambíguo ou pouco claro para os leitores?"
- "Que conhecimento ou contexto este documento assume que os leitores já têm?"
- "Existem contradições internas ou inconsistências?"

### Passo 4: Iterar com Base nos Resultados

Pergunte o que o Leitor Claude errou ou teve dificuldades. Indique a intenção de corrigir essas lacunas.

Volte para o refinamento para quaisquer seções problemáticas.

---

### Condição de Saída (Ambas as Abordagens)

Quando o Leitor Claude responde consistentemente às perguntas corretamente e não apresenta novas lacunas ou ambiguidades, o documento está pronto.

## Revisão Final

Quando o Teste com Leitor passar:
Anuncie que o documento passou no teste do Leitor Claude. Antes da conclusão:

1. Recomende que eles façam uma leitura final eles mesmos - eles são donos deste documento e são responsáveis por sua qualidade
2. Sugira verificar novamente quaisquer fatos, links ou detalhes técnicos
3. Peça que verifiquem se ele atinge o impacto que desejavam

Pergunte se eles querem mais uma revisão ou se o trabalho está concluído.

**Se o usuário quiser revisão final, forneça-a. Caso contrário:**
Anuncie a conclusão do documento. Forneça algumas dicas finais:

- Considere vincular esta conversa em um apêndice para que os leitores possam ver como o documento foi desenvolvido
- Use apêndices para fornecer profundidade sem inchar o documento principal
- Atualize o documento conforme o feedback for recebido de leitores reais

## Dicas para Orientação Eficaz

**Tom:**

- Seja direto e procedimental
- Explique a lógica brevemente quando afetar o comportamento do usuário
- Não tente "vender" a abordagem - apenas execute-a

**Lidando com Desvios:**

- Se o usuário quiser pular um estágio: Pergunte se eles querem pular isso e escrever de forma livre
- Se o usuário parecer frustrado: Reconheça que isso está demorando mais do que o esperado. Sugira maneiras de mover mais rápido
- Sempre dê ao usuário agência para ajustar o processo

**Gerenciamento de Contexto:**

- Durante todo o processo, se o contexto estiver faltando sobre algo mencionado, pergunte proativamente
- Não deixe lacunas se acumularem - aborde-as conforme surgirem

**Gerenciamento de Artefatos:**

- Use `create_file` para redigir seções completas
- Use `str_replace` para todas as edições
- Forneça link do artefato após cada alteração
- Nunca use artefatos para listas de brainstorming - isso é apenas conversa

**Qualidade sobre Velocidade:**

- Não corra através dos estágios
- Cada iteração deve fazer melhorias significativas
- O objetivo é um documento que realmente funcione para os leitores
