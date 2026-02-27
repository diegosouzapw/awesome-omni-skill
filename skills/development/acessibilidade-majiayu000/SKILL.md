---
name: acessibilidade
description: Garante que páginas HTML sigam práticas de acessibilidade WCAG 2.1 AA. Use quando criar ou modificar páginas HTML, adicionar componentes interativos, ou o usuário mencionar acessibilidade, ARIA, leitores de tela, ou contraste de cores.
---

# Skill: Acessibilidade Web (WCAG 2.1 AA)

Esta skill garante que todas as páginas do projeto sigam as diretrizes de acessibilidade WCAG 2.1 nível AA, tornando o site acessível para todos os usuários, incluindo pessoas com deficiências.

## Quando Usar

- Ao criar novas páginas HTML
- Ao adicionar componentes interativos (accordion, navbar, modals)
- Quando o usuário mencionar "acessibilidade", "ARIA", "alt text", "leitores de tela"
- Ao revisar código HTML existente
- Quando adicionar imagens, ícones ou conteúdo visual
- Ao criar formulários (se aplicável)

## Princípios WCAG 2.1

### 1. Perceptível
Informação e componentes da interface devem ser apresentados de forma perceptível aos usuários.

### 2. Operável
Componentes da interface e navegação devem ser operáveis por todos os usuários.

### 3. Compreensível
Informação e operação da interface devem ser compreensíveis.

### 4. Robusto
Conteúdo deve ser robusto o suficiente para ser interpretado por diferentes tecnologias assistivas.

## Diretrizes Principais

### Estrutura Semântica

**Hierarquia de Headings:**
- Use h1, h2, h3... em ordem hierárquica
- Não pule níveis (h1 → h3 é incorreto)
- Apenas um h1 por página

**Tags Semânticas HTML5:**
```html
<nav>      <!-- Navegação -->
<main>     <!-- Conteúdo principal -->
<section>  <!-- Seções -->
<article>  <!-- Conteúdo independente -->
<footer>   <!-- Rodapé -->
```

### ARIA (Accessible Rich Internet Applications)

**Roles:**
```html
<div role="progressbar" aria-valuenow="85" aria-valuemin="0" aria-valuemax="100">
```

**Labels:**
```html
<button aria-label="Fechar menu">✕</button>
```

**Atributos ARIA Essenciais:**
- `aria-label`: Texto alternativo para leitores de tela
- `aria-expanded`: Estado expandido/colapsado
- `aria-controls`: Relacionamento entre elementos
- `aria-hidden`: Ocultar elementos decorativos
- `aria-live`: Regiões que atualizam dinamicamente

### Navegação por Teclado

**Requisitos:**
- Todos os elementos interativos devem ser acessíveis via Tab
- Ordem de tab lógica e sequencial
- Indicadores visuais de foco visíveis
- Não use `tabindex` com valores positivos

**Elementos Focáveis:**
- Links (`<a>`)
- Botões (`<button>`)
- Inputs (`<input>`, `<select>`, `<textarea>`)
- Elementos com `tabindex="0"`

### Contraste de Cores

**Requisitos WCAG AA:**
- Texto normal (< 18pt): mínimo 4.5:1
- Texto grande (≥ 18pt ou ≥ 14pt bold): mínimo 3:1

**Combinações Bootstrap Seguras:**
✅ `bg-dark` + `text-white`
✅ `bg-primary` + `text-white`
✅ `bg-light` + `text-dark`
✅ `bg-warning` + `text-dark`

⚠️ **Evite:**
❌ `bg-warning` sem `text-dark`
❌ Cores claras em fundos claros
❌ Texto cinza claro em fundo branco

### Imagens e Conteúdo Visual

**Atributo Alt:**
```html
<!-- Imagem informativa -->
<img src="carta.jpg" alt="Carta Pikachu Base Set graduada nota 9">

<!-- Imagem decorativa -->
<img src="decoracao.png" alt="">
<!-- ou -->
<div aria-hidden="true">🔥</div>
```

**Emojis:**
Como este projeto usa emojis decorativos:
```html
<div class="display-1" aria-hidden="true">🔥</div>
<h2>Charizard</h2>
```

### Links e Botões

**Texto Descritivo:**
```html
<!-- ✅ Bom -->
<a href="charizard.html">Ver detalhes do Charizard</a>

<!-- ❌ Ruim -->
<a href="charizard.html">Clique aqui</a>
```

**Área de Clique:**
- Mínimo 44x44px (Bootstrap já garante com `.btn`)
- Espaçamento adequado entre elementos clicáveis

### Componentes Bootstrap com ARIA

**Accordion:**
```html
<button class="accordion-button"
        aria-expanded="true"
        aria-controls="collapse1">
    Título
</button>
<div id="collapse1" class="accordion-collapse collapse show">
    Conteúdo
</div>
```

**Navbar Toggle:**
```html
<button class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarNav"
        aria-controls="navbarNav"
        aria-expanded="false"
        aria-label="Alternar navegação">
    <span class="navbar-toggler-icon"></span>
</button>
```

**Progress Bars:**
```html
<div class="progress" style="height: 30px;">
    <div class="progress-bar bg-success"
         role="progressbar"
         style="width: 85%;"
         aria-valuenow="85"
         aria-valuemin="0"
         aria-valuemax="100">
        <span class="fw-bold">85%</span>
    </div>
</div>
```

**Modals:**
```html
<div class="modal" id="modalFrente" tabindex="-1" aria-labelledby="modalFrenteLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="modalFrenteLabel">Título</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Fechar"></button>
            </div>
        </div>
    </div>
</div>
```

### Responsividade e Zoom

**Viewport:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**Requisitos:**
- Conteúdo deve ser legível até 200% de zoom
- Não bloqueie zoom (não use `maximum-scale=1.0`)
- Layout deve adaptar sem scroll horizontal

### Idioma

**Tag HTML:**
```html
<html lang="pt-BR">
```

**Mudanças de Idioma:**
```html
<p>Este é um <span lang="en">Base Set</span> Charizard</p>
```

## Checklist de Acessibilidade

Ao criar ou modificar uma página, verifique:

**Estrutura:**
- ✅ `<html lang="pt-BR">` definido
- ✅ Hierarquia de headings correta (h1, h2, h3...)
- ✅ Tags semânticas HTML5 usadas apropriadamente

**Conteúdo:**
- ✅ Alt text em todas as imagens informativas
- ✅ `aria-hidden="true"` em imagens/emojis decorativos
- ✅ Links com texto descritivo (não "clique aqui")
- ✅ Contraste de cores adequado (4.5:1 mínimo)

**Interatividade:**
- ✅ Navegação por teclado funcional
- ✅ Indicadores de foco visíveis
- ✅ ARIA labels em botões sem texto
- ✅ Navbar com atributos ARIA corretos
- ✅ Accordion com `aria-expanded` e `aria-controls`
- ✅ Progress bars com `role` e atributos `aria-value*`
- ✅ Modals com `aria-labelledby` e `aria-hidden`

**Responsividade:**
- ✅ Viewport meta tag configurada
- ✅ Funciona em 200% de zoom
- ✅ Sem scroll horizontal em mobile

## Testes de Acessibilidade

### Testes Manuais

1. **Navegação por Teclado:**
   - Use apenas Tab, Enter, Espaço, Setas
   - Verifique se todos os elementos interativos são alcançáveis
   - Confirme que a ordem de foco é lógica

2. **Leitor de Tela:**
   - Windows: NVDA (gratuito)
   - Mac: VoiceOver (nativo)
   - Verifique se todo o conteúdo é lido corretamente

3. **Zoom:**
   - Teste com 200% de zoom no navegador
   - Verifique se não há quebra de layout
   - Confirme que todo o conteúdo permanece legível

4. **Desabilitar CSS:**
   - Veja se o conteúdo ainda faz sentido
   - Verifique se a hierarquia está correta

### Ferramentas Automatizadas

- **axe DevTools:** Extensão para Chrome/Firefox
- **WAVE:** Avaliador de acessibilidade web
- **Lighthouse:** Painel no Chrome DevTools (aba Accessibility)
- **WebAIM Contrast Checker:** https://webaim.org/resources/contrastchecker/

## Correções Comuns

### Problema: Imagem sem alt
```html
<!-- ❌ Antes -->
<img src="carta.jpg">

<!-- ✅ Depois -->
<img src="carta.jpg" alt="Carta Charizard Base Set">
```

### Problema: Link não descritivo
```html
<!-- ❌ Antes -->
<a href="detalhes.html">Clique aqui</a>

<!-- ✅ Depois -->
<a href="detalhes.html">Ver detalhes da carta Charizard</a>
```

### Problema: Botão sem label
```html
<!-- ❌ Antes -->
<button class="btn-close" data-bs-dismiss="modal"></button>

<!-- ✅ Depois -->
<button class="btn-close" data-bs-dismiss="modal" aria-label="Fechar"></button>
```

### Problema: Emoji sem aria-hidden
```html
<!-- ❌ Antes -->
<div class="display-1">🔥</div>

<!-- ✅ Depois -->
<div class="display-1" aria-hidden="true">🔥</div>
```

### Problema: Contraste insuficiente
```html
<!-- ❌ Antes -->
<span class="badge bg-warning">Aviso</span>

<!-- ✅ Depois -->
<span class="badge bg-warning text-dark">Aviso</span>
```

## Recursos

- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [Bootstrap 5 Accessibility](https://getbootstrap.com/docs/5.3/getting-started/accessibility/)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [axe DevTools](https://www.deque.com/axe/devtools/)
- [WAVE Extension](https://wave.webaim.org/extension/)

## Integração com Outras Skills

Esta skill trabalha em conjunto com:
- **bootstrap-guidelines:** Para usar classes Bootstrap acessíveis
- **codigo-html:** Para estrutura HTML semântica
- **estrutura-paginas:** Para layout acessível de páginas

Ao criar ou modificar páginas, sempre considere acessibilidade como prioridade, não como complemento.
