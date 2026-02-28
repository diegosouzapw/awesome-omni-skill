---
name: slack-gif-creator
description: Conhecimento e utilitários para criar GIFs animados otimizados para Slack. Fornece restrições, ferramentas de validação e conceitos de animação. Use quando os usuários solicitarem GIFs animados para Slack como "faça-me um GIF de X fazendo Y para Slack."
license: Termos completos em LICENSE.txt
---

# Criador de GIF para Slack

Um kit de ferramentas fornecendo utilitários e conhecimentos para criar GIFs animados otimizados para Slack.

## Requisitos do Slack

**Dimensões:**

- GIFs de Emoji: 128x128 (recomendado)
- GIFs de Mensagem: 480x480

**Parâmetros:**

- FPS: 10-30 (menor é menor tamanho de arquivo)
- Cores: 48-128 (menos = menor tamanho de arquivo)
- Duração: Mantenha abaixo de 3 segundos para GIFs de emoji

## Fluxo de Trabalho Principal

```python
from core.gif_builder import GIFBuilder
from PIL import Image, ImageDraw

# 1. Crie o construtor
builder = GIFBuilder(width=128, height=128, fps=10)

# 2. Gere quadros
for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))
    draw = ImageDraw.Draw(frame)

    # Desenhe sua animação usando primitivos PIL
    # (círculos, polígonos, linhas, etc.)

    builder.add_frame(frame)

# 3. Salve com otimização
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

## Desenhando Gráficos

### Trabalhando com Imagens Enviadas pelo Usuário

Se um usuário enviar uma imagem, considere se ele quer:

- **Usá-la diretamente** (por exemplo, "anime isso", "divida isso em quadros")
- **Usá-la como inspiração** (por exemplo, "faça algo como isso")

Carregue e trabalhe com imagens usando PIL:

```python
from PIL import Image

uploaded = Image.open('file.png')
# Use diretamente, ou apenas como referência para cores/estilo
```

### Desenhando do Zero

Ao desenhar gráficos do zero, use primitivos PIL ImageDraw:

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(frame)

# Círculos/ovais
draw.ellipse([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)

# Estrelas, triângulos, qualquer polígono
points = [(x1, y1), (x2, y2), (x3, y3), ...]
draw.polygon(points, fill=(r, g, b), outline=(r, g, b), width=3)

# Linhas
draw.line([(x1, y1), (x2, y2)], fill=(r, g, b), width=5)

# Retângulos
draw.rectangle([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)
```

**Não use:** Fontes de emoji (não confiáveis em plataformas) ou presuma que existem gráficos pré-empacotados nesta habilidade.

### Fazendo Gráficos Parecerem Bons

Os gráficos devem parecer polidos e criativos, não básicos. Veja como:

**Use linhas mais grossas** - Sempre defina `width=2` ou superior para contornos e linhas. Linhas finas (width=1) parecem irregulares e amadoras.

**Adicione profundidade visual**:

- Use gradientes para fundos (`create_gradient_background`)
- Camada de várias formas para complexidade (por exemplo, uma estrela com uma estrela menor dentro)

**Torne as formas mais interessantes**:

- Não desenhe apenas um círculo simples - adicione destaques, anéis ou padrões
- Estrelas podem ter brilhos (desenhe versões maiores e semitransparentes atrás)
- Combine várias formas (estrelas + brilhos, círculos + anéis)

**Preste atenção às cores**:

- Use cores vibrantes e complementares
- Adicione contraste (contornos escuros em formas claras, contornos claros em formas escuras)
- Considere a composição geral

**Para formas complexas** (corações, flocos de neve, etc.):

- Use combinações de polígonos e elipses
- Calcule pontos cuidadosamente para simetria
- Adicione detalhes (um coração pode ter uma curva de destaque, flocos de neve têm ramos intrincados)

Seja criativo e detalhado! Um bom GIF Slack deve parecer polido, não como gráficos de espaço reservado.

## Utilitários Disponíveis

### GIFBuilder (`core.gif_builder`)

Monta quadros e otimiza para Slack:

```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)  # Adicionar PIL Image
builder.add_frames(frames)  # Adicionar lista de quadros
builder.save('out.gif', num_colors=48, optimize_for_emoji=True, remove_duplicates=True)
```

### Validadors (`core.validators`)

Verifique se o GIF atende aos requisitos do Slack:

```python
from core.validators import validate_gif, is_slack_ready

# Validação detalhada
passes, info = validate_gif('my.gif', is_emoji=True, verbose=True)

# Verificação rápida
if is_slack_ready('my.gif'):
    print("Pronto!")
```

### Funções de Easing (`core.easing`)

Movimento suave em vez de linear:

```python
from core.easing import interpolate

# Progresso de 0.0 a 1.0
t = i / (num_frames - 1)

# Aplicar easing
y = interpolate(start=0, end=400, t=t, easing='ease_out')

# Disponível: linear, ease_in, ease_out, ease_in_out,
#           bounce_out, elastic_out, back_out
```

### Auxiliares de Quadro (`core.frame_composer`)

Funções de conveniência para necessidades comuns:

```python
from core.frame_composer import (
    create_blank_frame,         # Fundo de cor sólida
    create_gradient_background,  # Gradiente vertical
    draw_circle,                # Auxiliar para círculos
    draw_text,                  # Renderização de texto simples
    draw_star                   # Estrela de 5 pontas
)
```

## Conceitos de Animação

### Tremer/Vibrar

Deslocar a posição do objeto com oscilação:

- Use `math.sin()` ou `math.cos()` com índice de quadro
- Adicione pequenas variações aleatórias para sensação natural
- Aplique à posição x e/ou y

### Pulso/Batimento Cardíaco

Escalar o tamanho do objeto ritmicamente:

- Use `math.sin(t * frequency * 2 * math.pi)` para pulso suave
- Para batimento cardíaco: dois pulsos rápidos depois pausa (ajuste onda senoidal)
- Escala entre 0.8 e 1.2 do tamanho base

### Quicar

Objeto cai e quica:

- Use `interpolate()` com `easing='bounce_out'` para aterrissagem
- Use `easing='ease_in'` para cair (acelerando)
- Aplique gravidade aumentando a velocidade y a cada quadro

### Girar/Rotacionar

Gire o objeto ao redor do centro:

- PIL: `image.rotate(angle, resample=Image.BICUBIC)`
- Para oscilação: use onda senoidal para ângulo em vez de linear

### Fade In/Out

Apareça ou desapareça gradualmente:

- Crie imagem RGBA, ajuste canal alfa
- Ou use `Image.blend(image1, image2, alpha)`
- Fade in: alfa de 0 a 1
- Fade out: alfa de 1 a 0

### Deslizar

Mova o objeto de fora da tela para a posição:

- Posição inicial: fora dos limites do quadro
- Posição final: local de destino
- Use `interpolate()` com `easing='ease_out'` para parada suave
- Para overshoot: use `easing='back_out'`

### Zoom

Escala e posição para efeito de zoom:

- Zoom in: escala de 0.1 a 2.0, corte o centro
- Zoom out: escala de 2.0 a 1.0
- Pode adicionar desfoque de movimento para drama (filtro PIL)

### Explodir/Explosão de Partículas

Crie partículas irradiando para fora:

- Gere partículas com ângulos e velocidades aleatórios
- Atualize cada partícula: `x += vx`, `y += vy`
- Adicione gravidade: `vy += gravity_constant`
- Desapareça partículas ao longo do tempo (reduza alfa)

## Estratégias de Otimização

Apenas quando solicitado para diminuir o tamanho do arquivo, implemente alguns dos seguintes métodos:

1. **Menos quadros** - FPS mais baixo (10 em vez de 20) ou duração mais curta
2. **Menos cores** - `num_colors=48` em vez de 128
3. **Dimensões menores** - 128x128 em vez de 480x480
4. **Remover duplicatas** - `remove_duplicates=True` em save()
5. **Modo Emoji** - `optimize_for_emoji=True` auto-otimiza

```python
# Otimização máxima para emoji
builder.save(
    'emoji.gif',
    num_colors=48,
    optimize_for_emoji=True,
    remove_duplicates=True
)
```

## Filosofia

Esta habilidade fornece:

- **Conhecimento**: Requisitos do Slack e conceitos de animação
- **Utilitários**: GIFBuilder, validadores, funções de easing
- **Flexibilidade**: Crie a lógica de animação usando primitivos PIL

NÃO fornece:

- Modelos de animação rígidos ou funções pré-fabricadas
- Renderização de fonte de emoji (não confiável entre plataformas)
- Uma biblioteca de gráficos pré-empacotados incorporados na habilidade

**Nota sobre uploads de usuários**: Esta habilidade não inclui gráficos pré-configurados, mas se um usuário enviar uma imagem, use PIL para carregar e trabalhar com ela - interprete com base em sua solicitação se ele quer que seja usado diretamente ou apenas como inspiração.

Seja criativo! Combine conceitos (quicar + girar, pulsar + deslizar, etc.) e use os recursos completos do PIL.

## Dependências

```bash
pip install pillow imageio numpy
```
