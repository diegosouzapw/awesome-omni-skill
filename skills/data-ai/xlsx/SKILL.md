---
name: xlsx
description: "Criação, edição e análise abrangente de planilhas com suporte para fórmulas, formatação, análise de dados e visualização. Quando o Claude precisar trabalhar com planilhas (.xlsx, .xlsm, .csv, .tsv, etc) para: (1) Criar novas planilhas com fórmulas e formatação, (2) Ler ou analisar dados, (3) Modificar planilhas existentes preservando fórmulas, (4) Análise de dados e visualização em planilhas, ou (5) Recalcular fórmulas"
license: Proprietário. LICENSE.txt tem termos completos
---

# Requisitos para Saídas

## Todos os arquivos Excel

### Zero Erros de Fórmula

- Todo modelo Excel DEVE ser entregue com ZERO erros de fórmula (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?)

### Preservar Modelos Existentes (ao atualizar modelos)

- Estude e combine EXATAMENTE o formato, estilo e convenções existentes ao modificar arquivos
- Nunca imponha formatação padronizada em arquivos com padrões estabelecidos
- As convenções de modelo existentes SEMPRE substituem estas diretrizes

## Modelos financeiros

### Padrões de Codificação por Cores

A menos que declarado de outra forma pelo usuário ou modelo existente

#### Convenções de Cores Padrão da Indústria

- **Texto azul (RGB: 0,0,255)**: Entradas codificadas e números que os usuários mudarão para cenários
- **Texto preto (RGB: 0,0,0)**: TODAS as fórmulas e cálculos
- **Texto verde (RGB: 0,128,0)**: Links puxando de outras planilhas dentro da mesma pasta de trabalho
- **Texto vermelho (RGB: 255,0,0)**: Links externos para outros arquivos
- **Fundo amarelo (RGB: 255,255,0)**: Premissas principais precisando de atenção ou células que precisam ser atualizadas

### Padrões de Formatação de Números

#### Regras de Formato Necessárias

- **Anos**: Formate como cadeias de texto (por exemplo, "2024" não "2.024")
- **Moeda**: Use o formato $#.##0; SEMPRE especifique unidades nos cabeçalhos ("Receita ($mm)")
- **Zeros**: Use formatação de número para tornar todos os zeros "-", incluindo porcentagens (por exemplo, "$#.##0;($#.##0);-")
- **Porcentagens**: Padrão para formato 0,0% (uma casa decimal)
- **Múltiplos**: Formate como 0,0x para múltiplos de avaliação (EV/EBITDA, P/L)
- **Números negativos**: Use parênteses (123) não menos -123

### Regras de Construção de Fórmulas

#### Colocação de Premissas

- Coloque TODAS as premissas (taxas de crescimento, margens, múltiplos, etc.) em células de premissa separadas
- Use referências de célula em vez de valores codificados em fórmulas
- Exemplo: Use =B5*(1+$B$6) em vez de =B5*1.05

#### Prevenção de Erro de Fórmula

- Verifique se todas as referências de célula estão corretas
- Verifique se há erros off-by-one em intervalos
- Garanta fórmulas consistentes em todos os períodos de projeção
- Teste com casos extremos (valores zero, números negativos)
- Verifique se não há referências circulares não intencionais

#### Requisitos de Documentação para Hardcodes

- Comente ou coloque em células ao lado (se fim da tabela). Formato: "Fonte: [Sistema/Documento], [Data], [Referência Específica], [URL se aplicável]"
- Exemplos:
  - "Fonte: Empresa 10-K, AF2024, Página 45, Nota de Receita, [URL SEC EDGAR]"
  - "Fonte: Empresa 10-Q, T2 2025, Exposição 99.1, [URL SEC EDGAR]"
  - "Fonte: Terminal Bloomberg, 15/08/2025, AAPL US Equity"
  - "Fonte: FactSet, 20/08/2025, Tela de Estimativas de Consenso"

# XLSX: criação, edição e análise

## Visão Geral

Um usuário pode pedir para você criar, editar ou analisar o conteúdo de um arquivo .xlsx. Você tem diferentes ferramentas e fluxos de trabalho disponíveis para diferentes tarefas.

## Requisitos Importantes

**LibreOffice Necessário para Recálculo de Fórmula**: Você pode assumir que o LibreOffice está instalado para recalcular valores de fórmula usando o script `recalc.py`. O script configura automaticamente o LibreOffice na primeira execução.

## Lendo e analisando dados

### Análise de dados com pandas

Para análise de dados, visualização e operações básicas, use **pandas**, que fornece recursos poderosos de manipulação de dados:

```python
import pandas as pd

# Ler Excel
df = pd.read_excel('file.xlsx')  # Padrão: primeira planilha
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)  # Todas as planilhas como dict

# Analisar
df.head()      # Visualizar dados
df.info()      # Info da coluna
df.describe()  # Estatísticas

# Escrever Excel
df.to_excel('output.xlsx', index=False)
```

## Fluxos de Trabalho de Arquivo Excel

## CRÍTICO: Use Fórmulas, Não Valores Codificados

**Sempre use fórmulas do Excel em vez de calcular valores em Python e codificá-los.** Isso garante que a planilha permaneça dinâmica e atualizável.

### ❌ ERRADO - Codificando Valores Calculados

```python
# Ruim: Calculando em Python e codificando resultado
total = df['Sales'].sum()
sheet['B10'] = total  # Codifica 5000

# Ruim: Computando taxa de crescimento em Python
growth = (df.iloc[-1]['Revenue'] - df.iloc[0]['Revenue']) / df.iloc[0]['Revenue']
sheet['C5'] = growth  # Codifica 0.15

# Ruim: Cálculo Python para média
avg = sum(values) / len(values)
sheet['D20'] = avg  # Codifica 42.5
```

### ✅ CORRETO - Usando Fórmulas Excel

```python
# Bom: Deixe o Excel calcular a soma
sheet['B10'] = '=SUM(B2:B9)'

# Bom: Taxa de crescimento como fórmula Excel
sheet['C5'] = '=(C4-C2)/C2'

# Bom: Média usando função Excel
sheet['D20'] = '=AVERAGE(D2:D19)'
```

Isso se aplica a TODOS os cálculos - totais, porcentagens, índices, diferenças, etc. A planilha deve ser capaz de recalcular quando os dados de origem mudam.

## Fluxo de Trabalho Comum

1. **Escolha a ferramenta**: pandas para dados, openpyxl para fórmulas/formatação
2. **Criar/Carregar**: Crie nova pasta de trabalho ou carregue arquivo existente
3. **Modificar**: Adicione/edite dados, fórmulas e formatação
4. **Salvar**: Escreva no arquivo
5. **Recalcular fórmulas (OBRIGATÓRIO SE USAR FÓRMULAS)**: Use o script recalc.py
   ```bash
   python recalc.py output.xlsx
   ```
6. **Verificar e corrigir quaisquer erros**:
   - O script retorna JSON com detalhes do erro
   - Se `status` for `errors_found`, verifique `error_summary` para tipos de erro e locais específicos
   - Corrija os erros identificados e recalcule novamente
   - Erros comuns para corrigir:
     - `#REF!`: Referências de célula inválidas
     - `#DIV/0!`: Divisão por zero
     - `#VALUE!`: Tipo de dados errado na fórmula
     - `#NAME?`: Nome de fórmula não reconhecido

### Criando novos arquivos Excel

```python
# Usando openpyxl para fórmulas e formatação
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active

# Adicionar dados
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['Row', 'of', 'data'])

# Adicionar fórmula
sheet['B2'] = '=SUM(A1:A10)'

# Formatação
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet['A1'].alignment = Alignment(horizontal='center')

# Largura da coluna
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### Editando arquivos Excel existentes

```python
# Usando openpyxl para preservar fórmulas e formatação
from openpyxl import load_workbook

# Carregar arquivo existente
wb = load_workbook('existing.xlsx')
sheet = wb.active  # ou wb['SheetName'] para planilha específica

# Trabalhando com múltiplas planilhas
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]
    print(f"Planilha: {sheet_name}")

# Modificar células
sheet['A1'] = 'Novo Valor'
sheet.insert_rows(2)  # Inserir linha na posição 2
sheet.delete_cols(3)  # Excluir coluna 3

# Adicionar nova planilha
new_sheet = wb.create_sheet('NewSheet')
new_sheet['A1'] = 'Dados'

wb.save('modified.xlsx')
```

## Recalculando fórmulas

Arquivos Excel criados ou modificados pelo openpyxl contêm fórmulas como cadeias de caracteres, mas não valores calculados. Use o script `recalc.py` fornecido para recalcular fórmulas:

```bash
python recalc.py <excel_file> [timeout_seconds]
```

Exemplo:

```bash
python recalc.py output.xlsx 30
```

O script:

- Configura automaticamente macro LibreOffice na primeira execução
- Recalcula todas as fórmulas em todas as planilhas
- Verifica TODAS as células quanto a erros do Excel (#REF!, #DIV/0!, etc.)
- Retorna JSON com locais e contagens de erro detalhados
- Funciona em Linux e macOS

## Lista de Verificação de Fórmula

Verificações rápidas para garantir que as fórmulas funcionem corretamente:

### Verificação Essencial

- [ ] **Teste 2-3 referências de amostra**: Verifique se elas puxam valores corretos antes de construir o modelo completo
- [ ] **Mapeamento de colunas**: Confirme se as colunas do Excel correspondem (por exemplo, coluna 64 = BL, não BK)
- [ ] **Deslocamento de linha**: Lembre-se de que as linhas do Excel são indexadas em 1 (linha DataFrame 5 = linha Excel 6)

### Armadilhas Comuns

- [ ] **Manipulação de NaN**: Verifique se há valores nulos com `pd.notna()`
- [ ] **Colunas da extrema direita**: Dados AF geralmente em colunas 50+
- [ ] **Múltiplas correspondências**: Pesquise todas as ocorrências, não apenas a primeira
- [ ] **Divisão por zero**: Verifique denominadores antes de usar `/` em fórmulas (#DIV/0!)
- [ ] **Referências erradas**: Verifique se todas as referências de célula apontam para as células pretendidas (#REF!)
- [ ] **Referências entre planilhas**: Use o formato correto (Planilha1!A1) para vincular planilhas

### Estratégia de Teste de Fórmula

- [ ] **Comece pequeno**: Teste fórmulas em 2-3 células antes de aplicar amplamente
- [ ] **Verifique dependências**: Verifique se todas as células referenciadas em fórmulas existem
- [ ] **Teste casos extremos**: Inclua zero, negativo e valores muito grandes

### Interpretando a saída do recalc.py

O script retorna JSON com detalhes do erro:

```json
{
  "status": "success", // ou "errors_found"
  "total_errors": 0, // Contagem total de erros
  "total_formulas": 42, // Número de fórmulas no arquivo
  "error_summary": {
    // Apenas presente se erros encontrados
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    }
  }
}
```

## Melhores Práticas

### Seleção de Biblioteca

- **pandas**: Melhor para análise de dados, operações em massa e exportação de dados simples
- **openpyxl**: Melhor para formatação complexa, fórmulas e recursos específicos do Excel

### Trabalhando com openpyxl

- Os índices de célula são baseados em 1 (linha=1, coluna=1 refere-se à célula A1)
- Use `data_only=True` para ler valores calculados: `load_workbook('file.xlsx', data_only=True)`
- **Aviso**: Se aberto com `data_only=True` e salvo, as fórmulas são substituídas por valores e permanentemente perdidas
- Para arquivos grandes: Use `read_only=True` para leitura ou `write_only=True` para escrita
- Fórmulas são preservadas, mas não avaliadas - use recalc.py para atualizar valores

### Trabalhando com pandas

- Especifique tipos de dados para evitar problemas de inferência: `pd.read_excel('file.xlsx', dtype={'id': str})`
- Para arquivos grandes, leia colunas específicas: `pd.read_excel('file.xlsx', usecols=['A', 'C', 'E'])`
- Lide com datas corretamente: `pd.read_excel('file.xlsx', parse_dates=['date_column'])`

## Diretrizes de Estilo de Código

**IMPORTANTE**: Ao gerar código Python para operações Excel:

- Escreva código Python mínimo e conciso, sem comentários desnecessários
- Evite nomes de variáveis verbosos e operações redundantes
- Evite instruções de impressão desnecessárias

**Para os próprios arquivos Excel**:

- Adicione comentários às células com fórmulas complexas ou premissas importantes
- Documente fontes de dados para valores codificados
- Inclua notas para cálculos principais e seções de modelo
