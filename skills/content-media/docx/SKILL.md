---
name: docx
description: "Criação, edição e análise abrangente de documentos com suporte para alterações controladas, comentários, preservação de formatação e extração de texto. Quando o Claude precisar trabalhar com documentos profissionais (.docx files) para: (1) Criar novos documentos, (2) Modificar ou editar conteúdo, (3) Trabalhar com alterações controladas, (4) Adicionar comentários ou quaisquer outras tarefas de documentos"
license: Proprietário. LICENSE.txt tem termos completos
---

# DOCX: criação, edição e análise

## Visão Geral

Um usuário pode pedir para você criar, editar ou analisar o conteúdo de um arquivo .docx. Um arquivo .docx é essencialmente um arquivo ZIP contendo arquivos XML e outros recursos que você pode ler ou editar. Você tem diferentes ferramentas e fluxos de trabalho disponíveis para diferentes tarefas.

## Árvore de Decisão de Fluxo de Trabalho

### Lendo/Analisando Conteúdo

Use as seções "Extração de texto" ou "Acesso XML bruto" abaixo

### Criando Novo Documento

Use o fluxo de trabalho "Criando um novo documento Word"

### Editando Documento Existente

- **Seu próprio documento + alterações simples**
  Use o fluxo de trabalho "Edição básica OOXML"

- **Documento de outra pessoa**
  Use o fluxo de trabalho **"Redlining workflow"** (padrão recomendado)

- **Documentos legais, acadêmicos, de negócios ou governamentais**
  Use o fluxo de trabalho **"Redlining workflow"** (obrigatório)

## Lendo e analisando conteúdo

### Extração de texto

Se você precisa apenas ler o conteúdo de texto de um documento, você deve converter o documento para markdown usando pandoc. O Pandoc fornece excelente suporte para preservar a estrutura do documento e pode mostrar alterações controladas:

```bash
# Converter documento para markdown com alterações controladas
pandoc --track-changes=all path-to-file.docx -o output.md
# Opções: --track-changes=accept/reject/all
```

### Acesso XML bruto

Você precisa de acesso XML bruto para: comentários, formatação complexa, estrutura do documento, mídia incorporada e metadados. Para qualquer um desses recursos, você precisará descompactar um documento e ler seu conteúdo XML bruto.

#### Descompactando um arquivo

`python ooxml/scripts/unpack.py <office_file> <output_directory>`

#### Estruturas chave de arquivo

- `word/document.xml` - Conteúdo principal do documento
- `word/comments.xml` - Comentários referenciados em document.xml
- `word/media/` - Imagens e arquivos de mídia incorporados
- Alterações controladas usam tags `<w:ins>` (inserções) e `<w:del>` (exclusões)

## Criando um novo documento Word

Ao criar um novo documento Word do zero, use **docx-js**, que permite criar documentos Word usando JavaScript/TypeScript.

### Fluxo de Trabalho

1. **OBRIGATÓRIO - LEIA O ARQUIVO INTEIRO**: Leia [`docx-js.md`](docx-js.md) (~500 linhas) completamente do início ao fim. **NUNCA defina limites de intervalo ao ler este arquivo.** Leia o conteúdo completo do arquivo para sintaxe detalhada, regras de formatação críticas e práticas recomendadas antes de prosseguir com a criação do documento.
2. Crie um arquivo JavaScript/TypeScript usando componentes Document, Paragraph, TextRun (Você pode assumir que todas as dependências estão instaladas, mas se não, consulte a seção de dependências abaixo)
3. Exporte como .docx usando Packer.toBuffer()

## Editando um documento Word existente

Ao editar um documento Word existente, use a **biblioteca Document** (uma biblioteca Python para manipulação de OOXML). A biblioteca lida automaticamente com a configuração da infraestrutura e fornece métodos para manipulação de documentos. Para cenários complexos, você pode acessar o DOM subjacente diretamente através da biblioteca.

### Fluxo de Trabalho

1. **OBRIGATÓRIO - LEIA O ARQUIVO INTEIRO**: Leia [`ooxml.md`](ooxml.md) (~600 linhas) completamente do início ao fim. **NUNCA defina limites de intervalo ao ler este arquivo.** Leia o conteúdo completo do arquivo para a API da biblioteca Document e padrões XML para editar diretamente arquivos de documentos.
2. Descompacte o documento: `python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. Crie e execute um script Python usando a biblioteca Document (veja a seção "Document Library" em ooxml.md)
4. Empacote o documento final: `python ooxml/scripts/pack.py <input_directory> <office_file>`

A biblioteca Document fornece métodos de alto nível para operações comuns e acesso direto ao DOM para cenários complexos.

## Fluxo de trabalho de Redlining para revisão de documentos

Este fluxo de trabalho permite que você planeje alterações controladas abrangentes usando markdown antes de implementá-las em OOXML. **CRÍTICO**: Para alterações controladas completas, você deve implementar TODAS as alterações sistematicamente.

**Estratégia de Loteamento**: Agrupe alterações relacionadas em lotes de 3-10 alterações. Isso torna a depuração gerenciável enquanto mantém a eficiência. Teste cada lote antes de passar para o próximo.

**Princípio: Edições Mínimas e Precisas**
Ao implementar alterações controladas, marque apenas o texto que realmente muda. Repetir texto inalterado torna as edições mais difíceis de revisar e parece pouco profissional. Quebre as substituições em: [texto inalterado] + [exclusão] + [inserção] + [texto inalterado]. Preserve o RSID da execução original para texto inalterado extraindo o elemento `<w:r>` do original e reutilizando-o.

Exemplo - Alterando "30 dias" para "60 dias" em uma frase:

```python
# RUIM - Substitui a frase inteira
'<w:del><w:r><w:delText>O prazo é de 30 dias.</w:delText></w:r></w:del><w:ins><w:r><w:t>O prazo é de 60 dias.</w:t></w:r></w:ins>'

# BOM - Marca apenas o que mudou, preserva <w:r> original para texto inalterado
'<w:r w:rsidR="00AB12CD"><w:t>O prazo é de </w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t> dias.</w:t></w:r>'
```

### Fluxo de trabalho de alterações controladas

1. **Obter representação markdown**: Converta o documento para markdown com alterações controladas preservadas:

   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **Identificar e agrupar alterações**: Revise o documento e identifique TODAS as alterações necessárias, organizando-as em lotes lógicos:

   **Métodos de localização** (para encontrar alterações no XML):
   - Números de seção/título (por exemplo, "Seção 3.2", "Artigo IV")
   - Identificadores de parágrafo se numerados
   - Padrões Grep com texto circundante único
   - Estrutura do documento (por exemplo, "primeiro parágrafo", "bloco de assinatura")
   - **NÃO use números de linha markdown** - eles não mapeiam para a estrutura XML

   **Organização de lote** (agrupe 3-10 alterações relacionadas por lote):
   - Por seção: "Lote 1: alterações da Seção 2", "Lote 2: atualizações da Seção 5"
   - Por tipo: "Lote 1: Correções de data", "Lote 2: Alterações de nome das partes"
   - Por complexidade: Comece com substituições de texto simples, depois aborde alterações estruturais complexas
   - Sequencial: "Lote 1: Páginas 1-3", "Lote 2: Páginas 4-6"

3. **Ler documentação e descompactar**:
   - **OBRIGATÓRIO - LEIA O ARQUIVO INTEIRO**: Leia [`ooxml.md`](ooxml.md) (~600 linhas) completamente do início ao fim. **NUNCA defina limites de intervalo ao ler este arquivo.** Preste atenção especial às seções "Document Library" e "Tracked Change Patterns".
   - **Descompacte o documento**: `python ooxml/scripts/unpack.py <file.docx> <dir>`
   - **Anote o RSID sugerido**: O script de descompactação sugerirá um RSID para usar em suas alterações controladas. Copie este RSID para uso na etapa 4b.

4. **Implementar alterações em lotes**: Agrupe alterações logicamente (por seção, por tipo ou por proximidade) e implemente-as juntas em um único script. Esta abordagem:
   - Torna a depuração mais fácil (lote menor = mais fácil isolar erros)
   - Permite progresso incremental
   - Mantém a eficiência (tamanho de lote de 3-10 alterações funciona bem)

   **Agrupamentos de lote sugeridos:**
   - Por seção do documento (por exemplo, "alterações da Seção 3", "Definições", "Cláusula de rescisão")
   - Por tipo de alteração (por exemplo, "Alterações de data", "Atualizações de nome das partes", "Substituições de termos legais")
   - Por proximidade (por exemplo, "Alterações nas páginas 1-3", "Alterações na primeira metade do documento")

   Para cada lote de alterações relacionadas:

   **a. Mapear texto para XML**: Grep por texto em `word/document.xml` para verificar como o texto é dividido em elementos `<w:r>`.

   **b. Criar e executar script**: Use `get_node` para encontrar nós, implementar alterações, depois `doc.save()`. Veja a seção **"Document Library"** em ooxml.md para padrões.

   **Nota**: Sempre faça grep em `word/document.xml` imediatamente antes de escrever um script para obter os números de linha atuais e verificar o conteúdo do texto. Os números de linha mudam após cada execução de script.

5. **Empacotar o documento**: Após todos os lotes estarem completos, converta o diretório descompactado de volta para .docx:

   ```bash
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **Verificação final**: Faça uma verificação abrangente do documento completo:
   - Converta o documento final para markdown:
     ```bash
     pandoc --track-changes=all reviewed-document.docx -o verification.md
     ```
   - Verifique se TODAS as alterações foram aplicadas corretamente:
     ```bash
     grep "frase original" verification.md  # NÃO deve encontrar
     grep "frase de substituição" verification.md  # Deve encontrar
     ```
   - Verifique se nenhuma alteração não intencional foi introduzida

## Convertendo Documentos para Imagens

Para analisar visualmente documentos Word, converta-os em imagens usando um processo de duas etapas:

1. **Converter DOCX para PDF**:

   ```bash
   soffice --headless --convert-to pdf document.docx
   ```

2. **Converter páginas PDF para imagens JPEG**:
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   ```
   Isso cria arquivos como `page-1.jpg`, `page-2.jpg`, etc.

Opções:

- `-r 150`: Define a resolução para 150 DPI (ajuste para equilíbrio qualidade/tamanho)
- `-jpeg`: Saída no formato JPEG (use `-png` para PNG se preferir)
- `-f N`: Primeira página para converter (por exemplo, `-f 2` começa da página 2)
- `-l N`: Última página para converter (por exemplo, `-l 5` para na página 5)
- `page`: Prefixo para arquivos de saída

Exemplo para intervalo específico:

```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 document.pdf page  # Converte apenas páginas 2-5
```

## Diretrizes de Estilo de Código

**IMPORTANTE**: Ao gerar código para operações DOCX:

- Escreva código conciso
- Evite nomes de variáveis verbosos e operações redundantes
- Evite instruções de impressão (print) desnecessárias

## Dependências

Dependências necessárias (instale se não estiverem disponíveis):

- **pandoc**: `sudo apt-get install pandoc` (para extração de texto)
- **docx**: `npm install -g docx` (para criar novos documentos)
- **LibreOffice**: `sudo apt-get install libreoffice` (para conversão PDF)
- **Poppler**: `sudo apt-get install poppler-utils` (para pdftoppm converter PDF para imagens)
- **defusedxml**: `pip install defusedxml` (para análise XML segura)
