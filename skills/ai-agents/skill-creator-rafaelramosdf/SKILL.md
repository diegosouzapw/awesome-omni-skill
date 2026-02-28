---
name: skill-creator
description: Guia para criar habilidades eficazes. Esta habilidade deve ser usada quando os usuários desejam criar uma nova habilidade (ou atualizar uma existente) que estenda as capacidades do Claude com conhecimento especializado, fluxos de trabalho ou integrações de ferramentas.
license: Termos completos em LICENSE.txt
---

# Criador de Habilidades (Skill Creator)

Esta habilidade fornece orientações para a criação de habilidades eficazes.

## Sobre Habilidades

Habilidades (Skills) são pacotes modulares e independentes que estendem as capacidades do Claude fornecendo conhecimento especializado, fluxos de trabalho e ferramentas. Pense nelas como "guias de integração" para domínios ou tarefas específicas — elas transformam o Claude de um agente de propósito geral em um agente especializado equipado com conhecimento processual que nenhum modelo pode possuir totalmente.

### O Que as Habilidades Fornecem

1.  **Fluxos de trabalho especializados** - Procedimentos de várias etapas para domínios específicos
2.  **Integrações de ferramentas** - Instruções para trabalhar com formatos de arquivo ou APIs específicas
3.  **Expertise de domínio** - Conhecimento específico da empresa, esquemas, lógica de negócios
4.  **Recursos agrupados** - Scripts, referências e ativos para tarefas complexas e repetitivas

## Princípios Fundamentais

### Concisão é Chave

A janela de contexto é um bem público. As habilidades compartilham a janela de contexto com tudo o mais que o Claude precisa: prompt do sistema, histórico da conversa, metadados de outras habilidades e a solicitação real do usuário.

**Suposição padrão: O Claude já é muito inteligente.** Adicione apenas o contexto que o Claude ainda não possui. Questione cada informação: "O Claude realmente precisa dessa explicação?" e "Este parágrafo justifica seu custo em tokens?"

Prefira exemplos concisos a explicações prolixas.

### Defina Graus de Liberdade Apropriados

Combine o nível de especificidade com a fragilidade e variabilidade da tarefa:

**Alta liberdade (instruções baseadas em texto)**: Use quando várias abordagens são válidas, as decisões dependem do contexto ou heurísticas guiam a abordagem.

**Média liberdade (pseudocódigo ou scripts com parâmetros)**: Use quando existe um padrão preferido, alguma variação é aceitável ou a configuração afeta o comportamento.

**Baixa liberdade (scripts específicos, poucos parâmetros)**: Use quando as operações são frágeis e propensas a erros, a consistência é crítica ou uma sequência específica deve ser seguida.

Pense no Claude explorando um caminho: uma ponte estreita com penhascos precisa de guard-rails específicos (baixa liberdade), enquanto um campo aberto permite muitas rotas (alta liberdade).

### Anatomia de uma Habilidade

Toda habilidade consiste em um arquivo `SKILL.md` obrigatório e recursos opcionais agrupados:

```
skill-name/
├── SKILL.md (obrigatório)
│   ├── Metadados YAML frontmatter (obrigatório)
│   │   ├── name: (obrigatório)
│   │   └── description: (obrigatório)
│   └── Instruções Markdown (obrigatório)
└── Bundled Resources (opcional)
    ├── scripts/          - Código executável (Python/Bash/etc.)
    ├── references/       - Documentação destinada a ser carregada no contexto conforme necessário
    └── assets/           - Arquivos usados na saída (modelos, ícones, fontes, etc.)
```

#### SKILL.md (obrigatório)

Todo `SKILL.md` consiste em:

- **Frontmatter** (YAML): Contém campos `name` e `description`. Estes são os únicos campos que o Claude lê para determinar quando a habilidade é usada, portanto, é muito importante ser claro e abrangente ao descrever o que é a habilidade e quando ela deve ser usada.
- **Corpo** (Markdown): Instruções e orientações para usar a habilidade. Carregado apenas DEPOIS que a habilidade é acionada (se for).

#### Recursos Agrupados (opcional)

##### Scripts (`scripts/`)

Código executável (Python/Bash/etc.) para tarefas que exigem confiabilidade determinística ou são reescritos repetidamente.

- **Quando incluir**: Quando o mesmo código está sendo reescrito repetidamente ou confiabilidade determinística é necessária
- **Exemplo**: `scripts/rotate_pdf.py` para tarefas de rotação de PDF
- **Benefícios**: Eficiente em tokens, determinístico, pode ser executado sem carregar no contexto
- **Nota**: Scripts ainda podem precisar ser lidos pelo Claude para patches ou ajustes específicos do ambiente

##### Referências (`references/`)

Documentação e material de referência destinados a serem carregados conforme necessário no contexto para informar o processo e o pensamento do Claude.

- **Quando incluir**: Para documentação que o Claude deve referenciar enquanto trabalha
- **Exemplos**: `references/finance.md` para esquemas financeiros, `references/mnda.md` para modelo de NDA da empresa, `references/policies.md` para políticas da empresa, `references/api_docs.md` para especificações de API
- **Casos de uso**: Esquemas de banco de dados, documentação de API, conhecimento de domínio, políticas da empresa, guias de fluxo de trabalho detalhados
- **Benefícios**: Mantém o `SKILL.md` enxuto, carregado apenas quando o Claude determina que é necessário
- **Melhor prática**: Se os arquivos forem grandes (>10k palavras), inclua padrões de busca grep no `SKILL.md`
- **Evite duplicação**: As informações devem viver no `SKILL.md` ou nos arquivos de referência, não em ambos. Prefira arquivos de referência para informações detalhadas, a menos que seja verdadeiramente central para a habilidade — isso mantém o `SKILL.md` enxuto enquanto torna as informações descobríveis sem ocupar a janela de contexto. Mantenha apenas instruções processuais essenciais e orientações de fluxo de trabalho no `SKILL.md`; mova material de referência detalhado, esquemas e exemplos para arquivos de referência.

##### Ativos (`assets/`)

Arquivos não destinados a serem carregados no contexto, mas sim usados dentro da saída que o Claude produz.

- **Quando incluir**: Quando a habilidade precisa de arquivos que serão usados na saída final
- **Exemplos**: `assets/logo.png` para ativos de marca, `assets/slides.pptx` para modelos do PowerPoint, `assets/frontend-template/` para boilerplate HTML/React, `assets/font.ttf` para tipografia
- **Casos de uso**: Modelos, imagens, ícones, código boilerplate, fontes, documentos de exemplo que são copiados ou modificados
- **Benefícios**: Separa recursos de saída da documentação, permite que o Claude use arquivos sem carregá-los na janela de contexto

#### O Que Não Incluir em uma Habilidade

Uma habilidade deve conter apenas arquivos essenciais que apoiem diretamente sua funcionalidade. NÃO crie documentação estranha ou arquivos auxiliares, incluindo:

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md
- etc.

A habilidade deve conter apenas as informações necessárias para um agente de IA fazer o trabalho em questão. Não deve conter contexto auxiliar sobre o processo de criação, procedimentos de configuração e teste, documentação voltada para o usuário, etc. Criar arquivos de documentação adicionais apenas adiciona desordem e confusão.

### Princípio de Design de Divulgação Progressiva

As habilidades usam um sistema de carregamento de três níveis para gerenciar o contexto de forma eficiente:

1.  **Metadados (name + description)** - Sempre no contexto (~100 palavras)
2.  **Corpo do SKILL.md** - Quando a habilidade é acionada (<5k palavras)
3.  **Recursos agrupados** - Conforme necessário pelo Claude (Ilimitado porque scripts podem ser executados sem ler na janela de contexto)

#### Padrões de Divulgação Progressiva

Mantenha o corpo do `SKILL.md` com o essencial e abaixo de 500 linhas para minimizar o inchaço do contexto. Divida o conteúdo em arquivos separados ao se aproximar desse limite. Ao separar o conteúdo em outros arquivos, é muito importante referenciá-los no `SKILL.md` e descrever claramente quando lê-los, para garantir que o leitor da habilidade saiba que eles existem e quando usá-los.

**Princípio chave:** Quando uma habilidade suporta várias variações, frameworks ou opções, mantenha apenas o fluxo de trabalho principal e a orientação de seleção no `SKILL.md`. Mova detalhes específicos da variante (padrões, exemplos, configuração) para arquivos de referência separados.

**Padrão 1: Guia de alto nível com referências**

```markdown
# Processamento de PDF

## Início rápido

Extraia texto com pdfplumber:
[exemplo de código]

## Recursos avançados

- **Preenchimento de formulários**: Veja [FORMS.md](FORMS.md) para o guia completo
- **Referência da API**: Veja [REFERENCE.md](REFERENCE.md) para todos os métodos
- **Exemplos**: Veja [EXAMPLES.md](EXAMPLES.md) para padrões comuns
```

O Claude carrega `FORMS.md`, `REFERENCE.md` ou `EXAMPLES.md` apenas quando necessário.

**Padrão 2: Organização específica do domínio**

Para Habilidades com vários domínios, organize o conteúdo por domínio para evitar carregar contexto irrelevante:

```
bigquery-skill/
├── SKILL.md (visão geral e navegação)
└── reference/
    ├── finance.md (receita, métricas de faturamento)
    ├── sales.md (oportunidades, pipeline)
    ├── product.md (uso da API, recursos)
    └── marketing.md (campanhas, atribuição)
```

Quando um usuário pergunta sobre métricas de vendas, o Claude lê apenas `sales.md`.

Da mesma forma, para habilidades que suportam vários frameworks ou variantes, organize por variante:

```
cloud-deploy/
├── SKILL.md (fluxo de trabalho + seleção de provedor)
└── references/
    ├── aws.md (padrões de implantação AWS)
    ├── gcp.md (padrões de implantação GCP)
    └── azure.md (padrões de implantação Azure)
```

Quando o usuário escolhe AWS, o Claude lê apenas `aws.md`.

**Padrão 3: Detalhes condicionais**

Mostre conteúdo básico, link para conteúdo avançado:

```markdown
# Processamento de DOCX

## Criando documentos

Use docx-js para novos documentos. Veja [DOCX-JS.md](DOCX-JS.md).

## Editando documentos

Para edições simples, modifique o XML diretamente.

**Para alterações controladas**: Veja [REDLINING.md](REDLINING.md)
**Para detalhes OOXML**: Veja [OOXML.md](OOXML.md)
```

O Claude lê `REDLINING.md` ou `OOXML.md` apenas quando o usuário precisa desses recursos.

**Diretrizes importantes:**

- **Evite referências profundamente aninhadas** - Mantenha as referências a um nível de profundidade do `SKILL.md`. Todos os arquivos de referência devem ter link direto do `SKILL.md`.
- **Estruture arquivos de referência mais longos** - Para arquivos com mais de 100 linhas, inclua um índice no topo para que o Claude possa ver o escopo completo ao visualizar.

## Processo de Criação de Habilidade

A criação de habilidades envolve estas etapas:

1.  Entender a habilidade com exemplos concretos
2.  Planejar conteúdos de habilidade reutilizáveis (scripts, referências, ativos)
3.  Inicializar a habilidade (executar `init_skill.py`)
4.  Editar a habilidade (implementar recursos e escrever `SKILL.md`)
5.  Pacotar a habilidade (executar `package_skill.py`)
6.  Iterar com base no uso real

Siga estas etapas em ordem, pulando apenas se houver uma razão clara pela qual elas não se aplicam.

### Etapa 1: Entendendo a Habilidade com Exemplos Concretos

Pule esta etapa apenas quando os padrões de uso da habilidade já forem claramente compreendidos. Ela permanece valiosa mesmo ao trabalhar com uma habilidade existente.

Para criar uma habilidade eficaz, entenda claramente exemplos concretos de como a habilidade será usada. Essa compreensão pode vir de exemplos diretos do usuário ou exemplos gerados validados com o feedback do usuário.

Por exemplo, ao construir uma habilidade de editor de imagem, perguntas relevantes incluem:

- "Que funcionalidade a habilidade de editor de imagem deve suportar? Edição, rotação, algo mais?"
- "Você pode dar alguns exemplos de como essa habilidade seria usada?"
- "Posso imaginar usuários pedindo coisas como 'Remover os olhos vermelhos desta imagem' ou 'Girar esta imagem'. Existem outras maneiras que você imagina essa habilidade sendo usada?"
- "O que um usuário diria que deveria acionar essa habilidade?"

Para evitar sobrecarregar os usuários, evite fazer muitas perguntas em uma única mensagem. Comece com as perguntas mais importantes e faça o acompanhamento conforme necessário para melhor eficácia.

Conclua esta etapa quando houver uma noção clara da funcionalidade que a habilidade deve suportar.

### Etapa 2: Planejando os Conteúdos Reutilizáveis da Habilidade

Para transformar exemplos concretos em uma habilidade eficaz, analise cada exemplo:

1.  Considerando como executar o exemplo do zero
2.  Identificando quais scripts, referências e ativos seriam úteis ao executar esses fluxos de trabalho repetidamente

Exemplo: Ao construir uma habilidade `pdf-editor` para lidar com consultas como "Ajude-me a girar este PDF", a análise mostra:

1.  Girar um PDF requer reescrever o mesmo código a cada vez
2.  Um script `scripts/rotate_pdf.py` seria útil para armazenar na habilidade

Exemplo: Ao projetar uma habilidade `frontend-webapp-builder` para consultas como "Construa um aplicativo de tarefas" ou "Construa um painel para rastrear meus passos", a análise mostra:

1.  Escrever um webapp frontend requer o mesmo HTML/React boilerplate a cada vez
2.  Um modelo `assets/hello-world/` contendo os arquivos de projeto HTML/React boilerplate seria útil para armazenar na habilidade

Exemplo: Ao construir uma habilidade `big-query` para lidar com consultas como "Quantos usuários fizeram login hoje?" a análise mostra:

1.  Consultar o BigQuery requer redescobrir os esquemas de tabela e relacionamentos a cada vez
2.  Um arquivo `references/schema.md` documentando os esquemas de tabela seria útil para armazenar na habilidade

Para estabelecer o conteúdo da habilidade, analise cada exemplo concreto para criar uma lista dos recursos reutilizáveis a incluir: scripts, referências e ativos.

### Etapa 3: Inicializando a Habilidade

Neste ponto, é hora de realmente criar a habilidade.

Pule esta etapa apenas se a habilidade sendo desenvolvida já existir e a iteração ou empacotamento for necessário. Neste caso, continue para a próxima etapa.

Ao criar uma nova habilidade do zero, sempre execute o script `init_skill.py`. O script gera convenientemente um diretório de habilidade de modelo que inclui automaticamente tudo o que uma habilidade requer, tornando o processo de criação de habilidade muito mais eficiente e confiável.

Uso:

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
```

O script:

- Cria o diretório da habilidade no caminho especificado
- Gera um modelo `SKILL.md` com frontmatter adequado e marcadores TODO
- Cria diretórios de recursos de exemplo: `scripts/`, `references/` e `assets/`
- Adiciona arquivos de exemplo em cada diretório que podem ser personalizados ou excluídos

Após a inicialização, personalize ou remova o `SKILL.md` gerado e os arquivos de exemplo conforme necessário.

### Etapa 4: Editar a Habilidade

Ao editar a habilidade (recém-gerada ou existente), lembre-se de que a habilidade está sendo criada para outra instância do Claude usar. Inclua informações que seriam benéficas e não óbvias para o Claude. Considere qual conhecimento processual, detalhes específicos do domínio ou ativos reutilizáveis ajudariam outra instância do Claude a executar essas tarefas de forma mais eficaz.

#### Aprenda Padrões de Design Comprovados

Consulte estes guias úteis com base nas necessidades da sua habilidade:

- **Processos de várias etapas**: Veja `references/workflows.md` para fluxos de trabalho sequenciais e lógica condicional
- **Formatos de saída específicos ou padrões de qualidade**: Veja `references/output-patterns.md` para modelos e padrões de exemplo

Esses arquivos contêm melhores práticas estabelecidas para design de habilidade eficaz.

#### Comece com Conteúdos Reutilizáveis da Habilidade

Para iniciar a implementação, comece com os recursos reutilizáveis identificados acima: arquivos `scripts/`, `references/` e `assets/`. Observe que esta etapa pode exigir entrada do usuário. Por exemplo, ao implementar uma habilidade `brand-guidelines`, o usuário pode precisar fornecer ativos de marca ou modelos para armazenar em `assets/`, ou documentação para armazenar em `references/`.

Os scripts adicionados devem ser testados executando-os realmente para garantir que não haja bugs e que a saída corresponda ao esperado. Se houver muitos scripts semelhantes, apenas uma amostra representativa precisa ser testada para garantir confiança de que todos funcionam, equilibrando o tempo até a conclusão.

Quaisquer arquivos e diretórios de exemplo não necessários para a habilidade devem ser excluídos. O script de inicialização cria arquivos de exemplo em `scripts/`, `references/` e `assets/` para demonstrar a estrutura, mas a maioria das habilidades não precisará de todos eles.

#### Atualize o SKILL.md

**Diretrizes de Escrita:** Sempre use a forma imperativa/infinitiva.

##### Frontmatter

Escreva o frontmatter YAML com `name` e `description`:

- `name`: O nome da habilidade
- `description`: Este é o mecanismo de acionamento principal para sua habilidade e ajuda o Claude a entender quando usar a habilidade.
  - Inclua tanto o que a Habilidade faz quanto gatilhos/contextos específicos para quando usá-la.
  - Inclua todas as informações de "quando usar" aqui - Não no corpo. O corpo é carregado apenas após o acionamento, portanto, seções "Quando usar esta habilidade" no corpo não são úteis para o Claude.
  - Exemplo de descrição para uma habilidade `docx`: "Criação, edição e análise abrangente de documentos com suporte para alterações controladas, comentários, preservação de formatação e extração de texto. Use quando o Claude precisar trabalhar com documentos profissionais (.docx files) para: (1) Criar novos documentos, (2) Modificar ou editar conteúdo, (3) Trabalhar com alterações controladas, (4) Adicionar comentários ou quaisquer outras tarefas de documentos"

Não inclua outros campos no frontmatter YAML.

##### Corpo

Escreva instruções para usar a habilidade e seus recursos agrupados.

### Etapa 5: Empacotando uma Habilidade

Uma vez que o desenvolvimento da habilidade esteja completo, ela deve ser empacotada em um arquivo .skill distribuível que é compartilhado com o usuário. O processo de empacotamento valida automaticamente a habilidade primeiro para garantir que ela atenda a todos os requisitos:

```bash
scripts/package_skill.py <path/to/skill-folder>
```

Especificação opcional do diretório de saída:

```bash
scripts/package_skill.py <path/to/skill-folder> ./dist
```

O script de empacotamento irá:

1.  **Validar** a habilidade automaticamente, verificando:
    - Formato do frontmatter YAML e campos obrigatórios
    - Convenções de nomenclatura de habilidade e estrutura de diretório
    - Completude e qualidade da descrição
    - Organização de arquivos e referências de recursos

2.  **Empacotar** a habilidade se a validação passar, criando um arquivo .skill nomeado após a habilidade (por exemplo, `my-skill.skill`) que inclui todos os arquivos e mantém a estrutura de diretório adequada para distribuição. O arquivo .skill é um arquivo zip com uma extensão .skill.

Se a validação falhar, o script relatará os erros e sairá sem criar um pacote. Corrija quaisquer erros de validação e execute o comando de empacotamento novamente.

### Etapa 6: Iterar

Após testar a habilidade, os usuários podem solicitar melhorias. Frequentemente, isso acontece logo após o uso da habilidade, com contexto novo de como a habilidade funcionou.

**Fluxo de trabalho de iteração:**

1.  Use a habilidade em tarefas reais
2.  Observe dificuldades ou ineficiências
3.  Identifique como o `SKILL.md` ou recursos agrupados devem ser atualizados
4.  Implemente alterações e teste novamente
