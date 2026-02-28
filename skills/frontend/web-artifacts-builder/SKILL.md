---
name: web-artifacts-builder
description: Conjunto de ferramentas para criar artefatos HTML claude.ai elaborados e multi-componentes usando tecnologias web frontend modernas (React, Tailwind CSS, shadcn/ui). Use para artefatos complexos que requerem gerenciamento de estado, roteamento ou componentes shadcn/ui - não para artefatos HTML/JSX simples de arquivo único.
license: Termos completos em LICENSE.txt
---

# Construtor de Artefatos Web

Para construir artefatos frontend claude.ai poderosos, siga estas etapas:

1. Inicialize o repositório frontend usando `scripts/init-artifact.sh`
2. Desenvolva seu artefato editando o código gerado
3. Empacote todo o código em um único arquivo HTML usando `scripts/bundle-artifact.sh`
4. Exiba o artefato para o usuário
5. (Opcional) Teste o artefato

**Stack**: React 18 + TypeScript + Vite + Parcel (bundling) + Tailwind CSS + shadcn/ui

## Diretrizes de Design e Estilo

MUITO IMPORTANTE: Para evitar o que é frequentemente referido como "gororoba de IA", evite usar layouts centralizados excessivos, gradientes roxos, cantos arredondados uniformes e fonte Inter.

## Início Rápido

### Passo 1: Inicializar Projeto

Execute o script de inicialização para criar um novo projeto React:

```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

Isso cria um projeto totalmente configurado com:

- ✅ React + TypeScript (via Vite)
- ✅ Tailwind CSS 3.4.1 com sistema de temas shadcn/ui
- ✅ Aliases de caminho (`@/`) configurados
- ✅ 40+ componentes shadcn/ui pré-instalados
- ✅ Todas as dependências Radix UI incluídas
- ✅ Parcel configurado para empacotamento (via .parcelrc)
- ✅ Compatibilidade com Node 18+ (detecta e fixa automaticamente a versão do Vite)

### Passo 2: Desenvolva Seu Artefato

Para construir o artefato, edite os arquivos gerados. Veja **Tarefas Comuns de Desenvolvimento** abaixo para orientação.

### Passo 3: Pacote para Arquivo HTML Único

Para empacotar o aplicativo React em um único artefato HTML:

```bash
bash scripts/bundle-artifact.sh
```

Isso cria `bundle.html` - um artefato autocontido com todo o JavaScript, CSS e dependências embutidos. Este arquivo pode ser compartilhado diretamente em conversas do Claude como um artefato.

**Requisitos**: Seu projeto deve ter um `index.html` no diretório raiz.

**O que o script faz**:

- Instala dependências de empacotamento (parcel, @parcel/config-default, parcel-resolver-tspaths, html-inline)
- Cria configuração `.parcelrc` com suporte a alias de caminho
- Compila com Parcel (sem mapas de origem)
- Incorpora todos os ativos em HTML único usando html-inline

### Passo 4: Compartilhe o Artefato com o Usuário

Finalmente, compartilhe o arquivo HTML empacotado na conversa com o usuário para que ele possa visualizá-lo como um artefato.

### Passo 5: Testando/Visualizando o Artefato (Opcional)

Nota: Esta é uma etapa completamente opcional. Execute apenas se necessário ou solicitado.

Para testar/visualizar o artefato, use as ferramentas disponíveis (incluindo outras Habilidades ou ferramentas integradas como Playwright ou Puppeteer). Em geral, evite testar o artefato antecipadamente, pois isso adiciona latência entre a solicitação e quando o artefato final pode ser visto. Teste mais tarde, após apresentar o artefato, se solicitado ou se surgirem problemas.

## Referência

- **Componentes shadcn/ui**: https://ui.shadcn.com/docs/components
