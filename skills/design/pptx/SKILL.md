---
name: pptx
description: "Criação, edição e análise de apresentações. Quando o Claude precisar trabalhar com apresentações (.pptx files) para: (1) Criar novas apresentações, (2) Modificar ou editar conteúdo, (3) Trabalhar com layouts, (4) Adicionar comentários ou anotações do orador, ou quaisquer outras tarefas de apresentação"
license: Proprietário. LICENSE.txt tem termos completos
---

# PPTX: criação, edição e análise

## Visão Geral

Um usuário pode pedir para você criar, editar ou analisar o conteúdo de um arquivo .pptx. Um arquivo .pptx é essencialmente um arquivo ZIP contendo arquivos XML e outros recursos que você pode ler ou editar. Você tem diferentes ferramentas e fluxos de trabalho disponíveis para diferentes tarefas.

## Lendo e analisando conteúdo

### Extração de texto

Se você precisa apenas ler o conteúdo de texto de uma apresentação, você deve converter o documento para markdown:

```bash
# Converter documento para markdown
python -m markitdown path-to-file.pptx
```

### Acesso XML bruto

Você precisa de acesso XML bruto para: comentários, anotações do orador, layouts de slide, animações, elementos de design e formatação complexa. Para qualquer um desses recursos, você precisará descompactar uma apresentação e ler seu conteúdo XML bruto.

#### Descompactando um arquivo

`python ooxml/scripts/unpack.py <office_file> <output_dir>`

**Nota**: O script unpack.py está localizado em `skills/pptx/ooxml/scripts/unpack.py` relativo à raiz do projeto. Se o script não existir neste caminho, use `find . -name "unpack.py"` para localizá-lo.

#### Estruturas chave de arquivo

- `ppt/presentation.xml` - Metadados principais da apresentação e referências de slides
- `ppt/slides/slide{N}.xml` - Conteúdo individual do slide (slide1.xml, slide2.xml, etc.)
- `ppt/notesSlides/notesSlide{N}.xml` - Anotações do orador para cada slide
- `ppt/comments/modernComment_*.xml` - Comentários para slides específicos
- `ppt/slideLayouts/` - Modelos de layout para slides
- `ppt/slideMasters/` - Modelos mestre de slide
- `ppt/theme/` - Informações de tema e estilo
- `ppt/media/` - Imagens e outros arquivos de mídia

#### Tipografia e extração de cores

**Quando dado um exemplo de design para emular**: Sempre analise a tipografia e as cores da apresentação primeiro usando os métodos abaixo:

1. **Leia o arquivo de tema**: Verifique `ppt/theme/theme1.xml` para cores (`<a:clrScheme>`) e fontes (`<a:fontScheme>`)
2. **Amostra de conteúdo de slide**: Examine `ppt/slides/slide1.xml` para uso real de fonte (`<a:rPr>`) e cores
3. **Busca por padrões**: Use grep para encontrar cor (`<a:solidFill>`, `<a:srgbClr>`) e referências de fonte em todos os arquivos XML

## Criando uma nova apresentação PowerPoint **sem um modelo**

Ao criar uma nova apresentação PowerPoint do zero, use o fluxo de trabalho **html2pptx** para converter slides HTML para PowerPoint com posicionamento preciso.

### Princípios de Design

**CRÍTICO**: Antes de criar qualquer apresentação, analise o conteúdo e escolha elementos de design apropriados:

1. **Considere o assunto**: Sobre o que é esta apresentação? Que tom, indústria ou humor ela sugere?
2. **Verifique por branding**: Se o usuário mencionar uma empresa/organização, considere suas cores de marca e identidade
3. **Combine a paleta com o conteúdo**: Selecione cores que reflitam o assunto
4. **Declare sua abordagem**: Explique suas escolhas de design antes de escrever o código

**Requisitos**:

- ✅ Declare sua abordagem de design informada pelo conteúdo ANTES de escrever o código
- ✅ Use apenas fontes seguras para web: Arial, Helvetica, Times New Roman, Georgia, Courier New, Verdana, Tahoma, Trebuchet MS, Impact
- ✅ Crie hierarquia visual clara através de tamanho, peso e cor
- ✅ Garanta legibilidade: contraste forte, texto adequadamente dimensionado, alinhamento limpo
- ✅ Seja consistente: repita padrões, espaçamento e linguagem visual entre os slides

#### Seleção de Paleta de Cores

**Escolhendo cores criativamente**:

- **Pense além dos padrões**: Quais cores genuinamente combinam com este tópico específico? Evite escolhas automáticas.
- **Considere vários ângulos**: Tópico, indústria, humor, nível de energia, público-alvo, identidade da marca (se mencionada)
- **Seja aventureiro**: Tente combinações inesperadas - uma apresentação de saúde não precisa ser verde, finanças não precisa ser azul marinho
- **Construa sua paleta**: Escolha 3-5 cores que funcionem juntas (cores dominantes + tons de apoio + destaque)
- **Garanta o contraste**: O texto deve ser claramente legível nos fundos

**Exemplos de paletas de cores** (use para despertar a criatividade - escolha uma, adapte-a ou crie a sua própria):

1. **Azul Clássico**: Azul marinho profundo (#1C2833), cinza ardósia (#2E4053), prata (#AAB7B8), off-white (#F4F6F6)
2. **Verde-azulado e Coral**: Verde-azulado (#5EA8A7), verde-azulado profundo (#277884), coral (#FE4447), branco (#FFFFFF)
3. **Vermelho Ousado**: Vermelho (#C0392B), vermelho brilhante (#E74C3C), laranja (#F39C12), amarelo (#F1C40F), verde (#2ECC71)
4. **Blush Quente**: Malva (#A49393), blush (#EED6D3), rosa (#E8B4B8), creme (#FAF7F2)
5. **Luxo Borgonha**: Borgonha (#5D1D2E), carmesim (#951233), ferrugem (#C15937), ouro (#997929)
6. **Roxo Profundo e Esmeralda**: Roxo (#B165FB), azul escuro (#181B24), esmeralda (#40695B), branco (#FFFFFF)
7. **Creme e Verde Floresta**: Creme (#FFE1C7), verde floresta (#40695B), branco (#FCFCFC)
8. **Rosa e Roxo**: Rosa (#F8275B), coral (#FF574A), rosa (#FF737D), roxo (#3D2F68)
9. **Lima e Ameixa**: Lima (#C5DE82), ameixa (#7C3A5F), coral (#FD8C6E), azul-cinza (#98ACB5)
10. **Preto e Ouro**: Ouro (#BF9A4A), preto (#000000), creme (#F4F6F6)
11. **Sálvia e Terracota**: Sálvia (#87A96B), terracota (#E07A5F), creme (#F4F1DE), carvão (#2C2C2C)
12. **Carvão e Vermelho**: Carvão (#292929), vermelho (#E33737), cinza claro (#CCCBCB)
13. **Laranja Vibrante**: Laranja (#F96D00), cinza claro (#F2F2F2), carvão (#222831)
14. **Verde Floresta**: Preto (#191A19), verde (#4E9F3D), verde escuro (#1E5128), branco (#FFFFFF)
15. **Arco-íris Retrô**: Roxo (#722880), rosa (#D72D51), laranja (#EB5C18), âmbar (#F08800), ouro (#DEB600)
16. **Terroso Vintage**: Mostarda (#E3B448), sálvia (#CBD18F), verde floresta (#3A6B35), creme (#F4F1DE)
17. **Rosa Costeira**: Rosa velha (#AD7670), castor (#B49886), casca de ovo (#F3ECDC), cinza cinza (#BFD5BE)
18. **Laranja e Turquesa**: Laranja claro (#FC993E), turquesa acinzentado (#667C6F), branco (#FCFCFC)

#### Opções de Detalhes Visuais

**Padrões Geométricos**:

- Divisores de seção diagonais em vez de horizontais
- Larguras de coluna assimétricas (30/70, 40/60, 25/75)
- Cabeçalhos de texto rotacionados a 90° ou 270°
- Molduras circulares/hexagonais para imagens
- Formas de destaque triangulares nos cantos
- Formas sobrepostas para profundidade

**Tratamentos de Borda e Moldura**:

- Bordas grossas de uma cor (10-20pt) em apenas um lado
- Bordas de linha dupla com cores contrastantes
- Colchetes de canto em vez de molduras completas
- Bordas em forma de L (topo+esquerda ou fundo+direita)
- Acentos sublinhados abaixo dos cabeçalhos (3-5pt de espessura)

**Tratamentos de Tipografia**:

- Contraste de tamanho extremo (manchetes de 72pt vs corpo de 11pt)
- Cabeçalhos em maiúsculas com espaçamento entre letras amplo
- Seções numeradas em tipo de exibição superdimensionado
- Monoespaçado (Courier New) para dados/estatísticas/conteúdo técnico
- Fontes condensadas (Arial Narrow) para informações densas
- Texto contornado para ênfase

**Estilo de Gráfico e Dados**:

- Gráficos monocromáticos com uma única cor de destaque para dados principais
- Gráficos de barras horizontais em vez de verticais
- Gráficos de pontos em vez de gráficos de barras
- Linhas de grade mínimas ou nenhuma
- Rótulos de dados diretamente nos elementos (sem legendas)
- Números superdimensionados para métricas principais

**Inovações de Layout**:

- Imagens de sangramento total com sobreposições de texto
- Coluna da barra lateral (20-30% de largura) para navegação/contexto
- Sistemas de grade modular (blocos 3×3, 4×4)
- Fluxo de conteúdo padrão Z ou padrão F
- Caixas de texto flutuantes sobre formas coloridas
- Layouts de várias colunas estilo revista

**Tratamentos de Fundo**:

- Blocos de cor sólida ocupando 40-60% do slide
- Preenchimentos de gradiente (vertical ou diagonal apenas)
- Fundos divididos (duas cores, diagonal ou vertical)
- Faixas de cor de ponta a ponta
- Espaço negativo como elemento de design

### Dicas de Layout

**Ao criar slides com gráficos ou tabelas:**

- **Layout de duas colunas (PREFERIDO)**: Use um cabeçalho abrangendo toda a largura, depois duas colunas abaixo - texto/marcadores em uma coluna e o conteúdo em destaque na outra. Isso fornece melhor equilíbrio e torna os gráficos/tabelas mais legíveis. Use flexbox com larguras de coluna desiguais (por exemplo, divisão 40%/60%) para otimizar o espaço para cada tipo de conteúdo.
- **Layout de slide completo**: Deixe o conteúdo em destaque (gráfico/tabela) ocupar todo o slide para máximo impacto e legibilidade
- **NUNCA empilhe verticalmente**: Não coloque gráficos/tabelas abaixo do texto em uma única coluna - isso causa má legibilidade e problemas de layout

### Fluxo de Trabalho

1. **OBRIGATÓRIO - LEIA O ARQUIVO INTEIRO**: Leia [`html2pptx.md`](html2pptx.md) completamente do início ao fim. **NUNCA defina limites de intervalo ao ler este arquivo.** Leia o conteúdo completo do arquivo para sintaxe detalhada, regras de formatação críticas e práticas recomendadas antes de prosseguir com a criação da apresentação.
2. Crie um arquivo HTML para cada slide com dimensões adequadas (por exemplo, 720pt × 405pt para 16:9)
   - Use `<p>`, `<h1>`-`<h6>`, `<ul>`, `<ol>` para todo o conteúdo de texto
   - Use `class="placeholder"` para áreas onde gráficos/tabelas serão adicionados (renderize com fundo cinza para visibilidade)
   - **CRÍTICO**: Rasterizar gradientes e ícones como imagens PNG PRIMEIRO usando Sharp, depois referencie no HTML
   - **LAYOUT**: Para slides com gráficos/tabelas/imagens, use layout de slide completo ou layout de duas colunas para melhor legibilidade
3. Crie e execute um arquivo JavaScript usando a biblioteca [`html2pptx.js`](scripts/html2pptx.js) para converter slides HTML para PowerPoint e salvar a apresentação
   - Use a função `html2pptx()` para processar cada arquivo HTML
   - Adicione gráficos e tabelas às áreas de espaço reservado usando a API PptxGenJS
   - Salve a apresentação usando `pptx.writeFile()`
4. **Validação visual**: Gere miniaturas e inspecione problemas de layout
   - Crie grade de miniaturas: `python scripts/thumbnail.py output.pptx workspace/thumbnails --cols 4`
   - Leia e examine cuidadosamente a imagem da miniatura para:
     - **Corte de texto**: Texto sendo cortado por barras de cabeçalho, formas ou bordas do slide
     - **Sobreposição de texto**: Texto se sobrepondo a outro texto ou formas
     - **Problemas de posicionamento**: Conteúdo muito próximo das bordas do slide ou outros elementos
     - **Problemas de contraste**: Contraste insuficiente entre texto e fundos
   - Se problemas encontrados, ajuste margens/espaçamento/cores HTML e regenere a apresentação
   - Repita até que todos os slides estejam visualmente corretos

## Editando uma apresentação PowerPoint existente

Ao editar slides em uma apresentação PowerPoint existente, você precisa trabalhar com o formato Office Open XML (OOXML) bruto. Isso envolve descompactar o arquivo .pptx, editar o conteúdo XML e reempacotá-lo.

### Fluxo de Trabalho

1. **OBRIGATÓRIO - LEIA O ARQUIVO INTEIRO**: Leia [`ooxml.md`](ooxml.md) (~500 linhas) completamente do início ao fim. **NUNCA defina limites de intervalo ao ler este arquivo.** Leia o conteúdo completo do arquivo para orientação detalhada sobre a estrutura OOXML e fluxos de trabalho de edição antes de qualquer edição de apresentação.
2. Descompacte a apresentação: `python ooxml/scripts/unpack.py <office_file> <output_dir>`
3. Edite os arquivos XML (principalmente `ppt/slides/slide{N}.xml` e arquivos relacionados)
4. **CRÍTICO**: Valide imediatamente após cada edição e corrija quaisquer erros de validação antes de prosseguir: `python ooxml/scripts/validate.py <dir> --original <file>`
5. Empacote a apresentação final: `python ooxml/scripts/pack.py <input_directory> <office_file>`

## Criando uma nova apresentação PowerPoint **usando um modelo**

Quando você precisa criar uma apresentação que segue o design de um modelo existente, você precisará duplicar e reorganizar os slides do modelo antes de substituir o contexto do espaço reservado.

### Fluxo de Trabalho

1. **Extrair texto do modelo E criar grade de miniaturas visuais**:
   - Extrair texto: `python -m markitdown template.pptx > template-content.md`
   - Leia `template-content.md`: Leia o arquivo inteiro para entender o conteúdo da apresentação do modelo. **NUNCA defina limites de intervalo ao ler este arquivo.**
   - Crie grades de miniaturas: `python scripts/thumbnail.py template.pptx`
   - Veja a seção [Criando Grades de Miniaturas](#creating-thumbnail-grids) para mais detalhes

2. **Analise o modelo e salve o inventário em um arquivo**:
   - **Análise Visual**: Revise a(s) grade(s) de miniaturas para entender layouts de slides, padrões de design e estrutura visual
   - Crie e salve um arquivo de inventário de modelo em `template-inventory.md` contendo:

     ```markdown
     # Análise de Inventário de Modelo

     **Total de Slides: [contagem]**
     **IMPORTANTE: Slides são indexados em 0 (primeiro slide = 0, último slide = contagem-1)**

     ## [Nome da Categoria]

     - Slide 0: [Código de layout se disponível] - Descrição/propósito
     - Slide 1: [Código de layout] - Descrição/propósito
     - Slide 2: [Código de layout] - Descrição/propósito
       [... CADA slide deve ser listado individualmente com seu índice ...]
     ```

   - **Usando a grade de miniaturas**: Referencie as miniaturas visuais para identificar:
     - Padrões de layout (slides de título, layouts de conteúdo, divisores de seção)
     - Locais e contagens de espaços reservados para imagens
     - Consistência de design entre grupos de slides
     - Hierarquia visual e estrutura
   - Este arquivo de inventário é NECESSÁRIO para selecionar modelos apropriados na próxima etapa

3. **Crie o esboço da apresentação com base no inventário do modelo**:
   - Revise os modelos disponíveis da etapa 2.
   - Escolha um modelo de introdução ou título para o primeiro slide. Este deve ser um dos primeiros modelos.
   - Escolha layouts seguros e baseados em texto para os outros slides.
   - **CRÍTICO: Combine a estrutura do layout com o conteúdo real**:
     - Layouts de coluna única: Use para narrativa unificada ou tópico único
     - Layouts de duas colunas: Use APENAS quando tiver exatamente 2 itens/conceitos distintos
     - Layouts de três colunas: Use APENAS quando tiver exatamente 3 itens/conceitos distintos
     - Layouts de imagem + texto: Use APENAS quando tiver imagens reais para inserir
     - Layouts de citação: Use APENAS para citações reais de pessoas (com atribuição), nunca para ênfase
     - Nunca use layouts com mais espaços reservados do que você tem conteúdo
     - Se você tiver 2 itens, não os force em um layout de 3 colunas
     - Se você tiver 4+ itens, considere dividir em vários slides ou usar um formato de lista
   - Conte seus pedaços de conteúdo reais ANTES de selecionar o layout
   - Verifique se cada espaço reservado no layout escolhido será preenchido com conteúdo significativo
   - Selecione uma opção representando o **melhor** layout para cada seção de conteúdo.
   - Salve `outline.md` com conteúdo E mapeamento de modelo que aproveita os designs disponíveis
   - Exemplo de mapeamento de modelo:
     ```
     # Slides de modelo para usar (indexação baseada em 0)
     # AVISO: Verifique se os índices estão dentro do intervalo! Modelo com 73 slides tem índices 0-72
     # Mapeamento: números de slides do esboço -> índices de slides do modelo
     template_mapping = [
         0,   # Use slide 0 (Título/Capa)
         34,  # Use slide 34 (B1: Título e corpo)
         34,  # Use slide 34 novamente (duplicata para segundo B1)
         50,  # Use slide 50 (E1: Citação)
         54,  # Use slide 54 (F2: Fechamento + Texto)
     ]
     ```

4. **Duplique, reordene e exclua slides usando `rearrange.py`**:
   - Use o script `scripts/rearrange.py` para criar uma nova apresentação com slides na ordem desejada:
     ```bash
     python scripts/rearrange.py template.pptx working.pptx 0,34,34,50,52
     ```
   - O script lida com a duplicação de slides repetidos, exclusão de slides não utilizados e reordenação automaticamente
   - Os índices de slides são baseados em 0 (o primeiro slide é 0, o segundo é 1, etc.)
   - O mesmo índice de slide pode aparecer várias vezes para duplicar esse slide

5. **Extraia TODO o texto usando o script `inventory.py`**:
   - **Execute extração de inventário**:
     ```bash
     python scripts/inventory.py working.pptx text-inventory.json
     ```
   - **Leia text-inventory.json**: Leia todo o arquivo text-inventory.json para entender todas as formas e suas propriedades. **NUNCA defina limites de intervalo ao ler este arquivo.**

   - A estrutura JSON do inventário:

     ```json
     {
       "slide-0": {
         "shape-0": {
           "placeholder_type": "TITLE", // ou null para não-placeholders
           "left": 1.5, // posição em polegadas
           "top": 2.0,
           "width": 7.5,
           "height": 1.2,
           "paragraphs": [
             {
               "text": "Texto do parágrafo",
               // Propriedades opcionais (apenas incluídas quando não padrão):
               "bullet": true, // marcador explícito detectado
               "level": 0, // apenas incluído quando bullet é true
               "alignment": "CENTER", // CENTER, RIGHT (não LEFT)
               "space_before": 10.0, // espaço antes do parágrafo em pontos
               "space_after": 6.0, // espaço após parágrafo em pontos
               "line_spacing": 22.4, // espaçamento entre linhas em pontos
               "font_name": "Arial", // da primeira execução
               "font_size": 14.0, // em pontos
               "bold": true,
               "italic": false,
               "underline": false,
               "color": "FF0000" // cor RGB
             }
           ]
         }
       }
     }
     ```

   - Principais características:
     - **Slides**: Nomeados como "slide-0", "slide-1", etc.
     - **Formas**: Ordenadas por posição visual (cima-baixo, esquerda-direita) como "shape-0", "shape-1", etc.
     - **Tipos de espaço reservado**: TITLE, CENTER_TITLE, SUBTITLE, BODY, OBJECT, ou null
     - **Tamanho de fonte padrão**: `default_font_size` em pontos extraído de espaços reservados de layout (quando disponível)
     - **Números de slide são filtrados**: Formas com tipo de espaço reservado SLIDE_NUMBER são excluídas automaticamente do inventário
     - **Marcadores**: Quando `bullet: true`, `level` é sempre incluído (mesmo se 0)
     - **Espaçamento**: `space_before`, `space_after` e `line_spacing` em pontos (apenas incluído quando definido)
     - **Cores**: `color` para RGB (por exemplo, "FF0000"), `theme_color` para cores do tema (por exemplo, "DARK_1")
     - **Propriedades**: Apenas valores não padrão são incluídos na saída

6. **Gere texto de substituição e salve os dados em um arquivo JSON**
   Com base no inventário de texto da etapa anterior:
   - **CRÍTICO**: Primeiro verifique quais formas existem no inventário - apenas referencie formas que estão realmente presentes
   - **VALIDAÇÃO**: O script replace.py validará se todas as formas em seu JSON de substituição existem no inventário
     - Se você referenciar uma forma inexistente, receberá um erro mostrando as formas disponíveis
     - Se você referenciar um slide inexistente, receberá um erro indicando que o slide não existe
     - Todos os erros de validação são mostrados de uma vez antes do script sair
   - **IMPORTANTE**: O script replace.py usa inventory.py internamente para identificar TODAS as formas de texto
   - **LIMPEZA AUTOMÁTICA**: TODAS as formas de texto do inventário serão limpas, a menos que você forneça "paragraphs" para elas
   - Adicione um campo "paragraphs" às formas que precisam de conteúdo (não "replacement_paragraphs")
   - Formas sem "paragraphs" no JSON de substituição terão seu texto limpo automaticamente
   - Parágrafos com marcadores serão automaticamente alinhados à esquerda. Não defina a propriedade `alignment` quando `"bullet": true`
   - Gere conteúdo de substituição apropriado para texto de espaço reservado
   - Use o tamanho da forma para determinar o comprimento de conteúdo apropriado
   - **CRÍTICO**: Inclua propriedades de parágrafo do inventário original - não forneça apenas texto
   - **IMPORTANTE**: Quando bullet: true, NÃO inclua símbolos de marcador (•, -, \*) no texto - eles são adicionados automaticamente
   - **REGRAS DE FORMATAÇÃO ESSENCIAIS**:
     - Cabeçalhos/títulos geralmente devem ter `"bold": true`
     - Itens de lista devem ter `"bullet": true, "level": 0` (nível é obrigatório quando bullet é true)
     - Preserve quaisquer propriedades de alinhamento (por exemplo, `"alignment": "CENTER"` para texto centralizado)
     - Inclua propriedades de fonte quando diferentes do padrão (por exemplo, `"font_size": 14.0`, `"font_name": "Lora"`)
     - Cores: Use `"color": "FF0000"` para RGB ou `"theme_color": "DARK_1"` para cores do tema
     - O script de substituição espera **parágrafos formatados corretamente**, não apenas cadeias de texto
     - **Formas sobrepostas**: Prefira formas com default_font_size maior ou placeholder_type mais apropriado
   - Salve o inventário atualizado com substituições em `replacement-text.json`
   - **AVISO**: Layouts de modelo diferentes têm contagens de formas diferentes - sempre verifique o inventário real antes de criar substituições

   Exemplo de campo paragraphs mostrando formatação adequada:

   ```json
   "paragraphs": [
     {
       "text": "Novo texto de título da apresentação",
       "alignment": "CENTER",
       "bold": true
     },
     {
       "text": "Cabeçalho da Seção",
       "bold": true
     },
     {
       "text": "Primeiro ponto de marcador sem símbolo de marcador",
       "bullet": true,
       "level": 0
     },
     {
       "text": "Texto colorido em vermelho",
       "color": "FF0000"
     },
     {
       "text": "Texto colorido do tema",
       "theme_color": "DARK_1"
     },
     {
       "text": "Texto de parágrafo regular sem formatação especial"
     }
   ]
   ```

   **Formas não listadas no JSON de substituição são limpas automaticamente**:

   ```json
   {
     "slide-0": {
       "shape-0": {
         "paragraphs": [...] // Esta forma recebe novo texto
       }
       // shape-1 e shape-2 do inventário serão limpos automaticamente
     }
   }
   ```

   **Padrões de formatação comuns para apresentações**:
   - Slides de título: Texto em negrito, às vezes centralizado
   - Cabeçalhos de seção dentro dos slides: Texto em negrito
   - Listas com marcadores: Cada item precisa de `"bullet": true, "level": 0`
   - Texto do corpo: Geralmente nenhuma propriedade especial necessária
   - Citações: Podem ter alinhamento especial ou propriedades de fonte

7. **Aplique substituições usando o script `replace.py`**

   ```bash
   python scripts/replace.py working.pptx replacement-text.json output.pptx
   ```

   O script irá:
   - Primeiro extrair o inventário de TODAS as formas de texto usando funções de inventory.py
   - Validar se todas as formas no JSON de substituição existem no inventário
   - Limpar texto de TODAS as formas identificadas no inventário
   - Aplicar novo texto apenas a formas com "paragraphs" definidos no JSON de substituição
   - Preservar a formatação aplicando propriedades de parágrafo do JSON
   - Lidar com marcadores, alinhamento, propriedades de fonte e cores automaticamente
   - Salvar a apresentação atualizada

   Exemplo de erros de validação:

   ```
   ERROR: Formas inválidas no JSON de substituição:
     - Forma 'shape-99' não encontrada no 'slide-0'. Formas disponíveis: shape-0, shape-1, shape-4
     - Slide 'slide-999' não encontrado no inventário
   ```

   ```
   ERROR: Texto de substituição piorou o estouro nestas formas:
     - slide-0/shape-2: estouro piorou em 1.25" (era 0.00", agora 1.25")
   ```

## Criando Grades de Miniaturas

Para criar grades de miniaturas visuais de slides do PowerPoint para análise rápida e referência:

```bash
python scripts/thumbnail.py template.pptx [output_prefix]
```

**Recursos**:

- Cria: `thumbnails.jpg` (ou `thumbnails-1.jpg`, `thumbnails-2.jpg`, etc. para grandes decks)
- Padrão: 5 colunas, máx 30 slides por grade (5×6)
- Prefixo personalizado: `python scripts/thumbnail.py template.pptx my-grid`
  - Nota: O prefixo de saída deve incluir o caminho se você quiser saída em um diretório específico (por exemplo, `workspace/my-grid`)
- Ajustar colunas: `--cols 4` (faixa: 3-6, afeta slides por grade)
- Limites de grade: 3 cols = 12 slides/grade, 4 cols = 20, 5 cols = 30, 6 cols = 42
- Slides são indexados em zero (Slide 0, Slide 1, etc.)

**Casos de uso**:

- Análise de modelo: Entenda rapidamente layouts de slides e padrões de design
- Revisão de conteúdo: Visão geral visual de toda a apresentação
- Referência de navegação: Encontre slides específicos por sua aparência visual
- Verificação de qualidade: Verifique se todos os slides estão formatados corretamente

**Exemplos**:

```bash
# Uso básico
python scripts/thumbnail.py presentation.pptx

# Combinar opções: nome personalizado, colunas
python scripts/thumbnail.py template.pptx analysis --cols 4
```

## Convertendo Slides para Imagens

Para analisar visualmente slides do PowerPoint, converta-os em imagens usando um processo de duas etapas:

1. **Converter PPTX para PDF**:

   ```bash
   soffice --headless --convert-to pdf template.pptx
   ```

2. **Converter páginas PDF para imagens JPEG**:
   ```bash
   pdftoppm -jpeg -r 150 template.pdf slide
   ```
   Isso cria arquivos como `slide-1.jpg`, `slide-2.jpg`, etc.

Opções:

- `-r 150`: Define a resolução para 150 DPI (ajuste para equilíbrio qualidade/tamanho)
- `-jpeg`: Saída no formato JPEG (use `-png` para PNG se preferir)
- `-f N`: Primeira página para converter (por exemplo, `-f 2` começa da página 2)
- `-l N`: Última página para converter (por exemplo, `-l 5` para na página 5)
- `slide`: Prefixo para arquivos de saída

Exemplo para intervalo específico:

```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 template.pdf slide  # Converte apenas páginas 2-5
```

## Diretrizes de Estilo de Código

**IMPORTANTE**: Ao gerar código para operações PPTX:

- Escreva código conciso
- Evite nomes de variáveis verbosos e operações redundantes
- Evite instruções de impressão desnecessárias

## Dependências

Dependências necessárias (já devem estar instaladas):

- **markitdown**: `pip install "markitdown[pptx]"` (para extração de texto de apresentações)
- **pptxgenjs**: `npm install -g pptxgenjs` (para criar apresentações via html2pptx)
- **playwright**: `npm install -g playwright` (para renderização HTML em html2pptx)
- **react-icons**: `npm install -g react-icons react react-dom` (para ícones)
- **sharp**: `npm install -g sharp` (para rasterização SVG e processamento de imagem)
- **LibreOffice**: `sudo apt-get install libreoffice` (para conversão PDF)
- **Poppler**: `sudo apt-get install poppler-utils` (para pdftoppm para converter PDF em imagens)
- **defusedxml**: `pip install defusedxml` (para análise XML segura)
