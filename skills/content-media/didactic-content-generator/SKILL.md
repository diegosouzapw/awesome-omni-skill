---
name: didactic-content-generator
description: Gere conteúdo didático de alta qualidade em HTML/CSS com ilustrações SVG, usando um sistema de Temas pré definidos e reutilizáveis. Capaz de criar ou editar apostilas, tutoriais e materiais educacionais que seguem filosofias pedagógicas claras.
license: Termos completos em LICENSE.txt
---

Estas são instruções para criar **Conteúdo Didático Visual** - material educacional que é estruturado pedagogicamente e expresso visualmente com qualidade editorial. A saída final DEVE ser um único arquivo `.html`.

## ARQUIVOS E RECURSOS

- **`examples/`**: Contém exemplos de referência.
- **`themes.json`**: **CRÍTICO.** Este arquivo armazena as definições de estilo (cores, tipografia, CSS) dos temas pré definidos. O Agente DEVE ler este arquivo para operar.

---

## FUNCIONALIDADES PRINCIPAIS

Sempre verifique qual destas ações abaixo o usuário deseja realizar:

### 1. CRIAR/EDITAR CONTEÚDO DIDÁTICO

Cria ou modifica um arquivo HTML único contendo o conteúdo didático completo, rico em visual e pedagogia.

#### Estrutura Pedagógica (Conteúdo)

**Cabeçalho Editorial:** Título (H1) impactante, Subtítulo (H2) explicativo, Badges de metadados.
**Resumo Executivo:** Uma caixa destacada no início com os pontos chave (TL;DR).
**Desenvolvimento:** O conteúdo dividido em seções claras (H3/H4).
**Didática Visual:**

- Use **SVG Inline** para diagramas (obrigatório para conceitos complexos).
- Use _Callouts_ (boxes coloridos) para alertas e dicas.
- Use _Cards_ para mnemônicos ou fórmulas.
- _Exemplos:_ Blocos de código ou cenários de uso com fundo contrastante.

**Conclusão:** Resumo final recapitulativo.

#### Diretrizes Técnicas (HTML/CSS)

- **HTML Semântico:** Use `<article>`, `<section>`, `<aside>`, `<figure>`, `<figcaption>`.
- **CSS Embutido:** Todo o estilo deve estar em uma tag `<style>` no `<head>`.
- **Responsividade:** O layout deve funcionar em Mobile e Desktop. Use `max-width: 800px` para o contêiner de leitura principal para ergonomia visual.
- **Tipografia:** Use `@import` para fontes do Google Fonts. Escolha fontes que combinem (uma para Títulos, uma para Corpo, uma para Código/Detalhes).
  - _Exemplo:_ `Merriweather` (Corpo) + `Oswald` (Títulos) para um look clássico.
  - _Exemplo:_ `Inter` (Corpo) + `Space Grotesk` (Títulos) para tech.

#### Diretrizes de Ilustração (SVG/CSS)

O agente DEVE ser capaz de ilustrar conceitos. Não use imagens externas quebradas.

- **SVG Inline:** Desenhe diagramas, ícones e fluxogramas usando tags `<svg>` diretamente no HTML. Isso garante vetores perfeitos em qualquer zoom.
- **CSS Shapes:** Use divs com bordas e cores para criar caixas de destaque, separadores e elementos visuais simples.

#### Diretrizes de Qualidade (O "Canvas Digital")

Gere um arquivo HTML único. Este arquivo deve ser uma **OBRA DE ARTE EDITORIAL**.

- Não faça páginas web genéricas.
- Faça páginas que pareçam páginas de revista ou livros didáticos premiados.
- HTML & CSS (Engine do Tema): Ao aplicar um tema, mapeie o JSON para CSS Variables na raiz `:root`.
- Importe as fontes usando o campo `googleFontsUrl` do tema.

#### Processo de Criação

**1. Defina Tópico:** Se o usuário não informar o tópico, pergunte. Confirme o assunto (tópico e subtópicos), o público-alvo e o objetivo do conteúdo.
**2. Defina o Público-alvo:** Se o usuário não informar o público-alvo, pergunte. O público-alvo (Ex: Crianças de 5 anos, Doutorandos, Engenheiros Sênior).
**3. Defina o Tom:** Se o usuário não informar o tom, pergunte. Socrático, Direto, Lúdico, Acadêmico, "Explain Like I'm 5" (ELI5).
**4. Defina a Filosofia Didática:** Antes de escrever o conteúdo, defina a abordagem pedagógica e visual. Não pule esta etapa, ela garante coerência. (faça isso mentalmente).
**5. Defina a Estrutura Cognitiva:** Como o conhecimento será construído? (Ex: Conceito -> Exemplo -> Prática; ou Problema -> Solução -> Teoria). (faça isso mentalmente).
**6. Consulte Temas:** Leia `themes.json` e apresente a lista de temas disponíveis (Nome + Descrição) para o usuário escolher, ou selecione caso o usuário já tenha informado.
**7. Gere o HTML:**

- Use as variáveis de cor e tipografia do Tema escolhido.
- Insira o CSS no `<head>`.
- **IMPORTANTE**: Siga todas as diretrizes: Técnicas (HTML/CSS), de Ilustração (SVG/CSS) e de Qualidade (O "Canvas Digital") (**MANDATÓRIO**).

#### Processo de Edição

- Solicite o arquivo HTML ou o código fonte.
- Aplique as alterações solicitadas (conteúdo ou visual).
- Se for troca de tema, reescreva o bloco `<style>` com os dados do novo tema do `themes.json`.
- **IMPORTANTE**: Siga todas as diretrizes: Técnicas (HTML/CSS), de Ilustração (SVG/CSS) e de Qualidade (O "Canvas Digital") (**MANDATÓRIO**).

### 2. CRIAR/EDITAR TEMA

Cria ou atualiza definições no arquivo `themes.json`.

#### Estrutura do Objeto Tema (Referência)

```json
{
  "id": "string-kebab-case",
  "name": "Nome Legível",
  "description": "Breve descrição de uso.",
  "colors": {
    "primary": "#hex",
    "accent": "#hex",
    "secondary": "#hex",
    "text": "#hex",
    "background": "#hex",
    "surface": "#hex" // Para cartões/boxes
  },
  "typography": {
    "display": "Font Family para títulos",
    "body": "Font Family para texto",
    "code": "Font Family para código",
    "googleFontsUrl": "URL completa do Google Fonts"
  },
  "css_variables": {
    "--custom-var": "value" // Variáveis extras específicas
  },
  "extra_styles": "CSS cru para overrides específicos (ex: bordas de botões, sombras específicas)"
}
```

#### Processo de Criação

**Coletar Dados:** Pergunte ao usuário o "mood" (sério, lúdico, dark, tech), cores de preferência ou peça liberdade criativa.
**Gerar Identidade:**

- **Nome:** Crie um nome curto e intuitivo (Ex: `cyber-punk`, `infantil-pastel`, `direito-classico`).
- **ID:** Kebab-case do nome.
- **Cores/Tipografia:** Defina uma paleta coerente e fontes do Google Fonts.
  **Persistir:**
- Leia o `themes.json` atual.
- Adicione o novo objeto ao array.
- Sobrescreva o arquivo `themes.json` com o novo conteúdo JSON.

#### Processo de Edição

- Identifique o tema alvo.
- Altere os valores solicitados.
- Salve o `themes.json` atualizado.
