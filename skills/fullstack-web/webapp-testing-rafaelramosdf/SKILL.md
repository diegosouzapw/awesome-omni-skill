---
name: webapp-testing
description: Kit de ferramentas para interagir e testar aplicativos web locais usando Playwright. Suporta verificação de funcionalidade de frontend, depuração de comportamento de UI, captura de capturas de tela do navegador e visualização de logs do navegador.
license: Termos completos em LICENSE.txt
---

# Teste de Aplicação Web

Para testar aplicativos web locais, escreva scripts nativos Python Playwright.

**Scripts Auxiliares Disponíveis**:

- `scripts/with_server.py` - Gerencia o ciclo de vida do servidor (suporta múltiplos servidores)

**Sempre execute scripts com `--help` primeiro** para ver o uso. NÃO leia a fonte até tentar executar o script primeiro e descobrir que uma solução personalizada é absolutamente necessária. Esses scripts podem ser muito grandes e, portanto, poluir sua janela de contexto. Eles existem para serem chamados diretamente como scripts de caixa preta, em vez de ingeridos em sua janela de contexto.

## Árvore de Decisão: Escolhendo Sua Abordagem

```
Tarefa do usuário → É HTML estático?
    ├─ Sim → Leia o arquivo HTML diretamente para identificar seletores
    │         ├─ Sucesso → Escreva script Playwright usando seletores
    │         └─ Falha/Incompleto → Trate como dinâmico (abaixo)
    │
    └─ Não (webapp dinâmico) → O servidor já está rodando?
        ├─ Não → Execute: python scripts/with_server.py --help
        │        Então use o auxiliar + escreva script Playwright simplificado
        │
        └─ Sim → Reconhecimento-então-ação:
            1. Navegue e aguarde networkidle
            2. Tire captura de tela ou inspecione DOM
            3. Identifique seletores do estado renderizado
            4. Execute ações com seletores descobertos
```

## Exemplo: Usando with_server.py

Para iniciar um servidor, execute `--help` primeiro, depois use o auxiliar:

**Servidor único:**

```bash
python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
```

**Múltiplos servidores (por exemplo, backend + frontend):**

```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python your_automation.py
```

Para criar um script de automação, inclua apenas a lógica do Playwright (os servidores são gerenciados automaticamente):

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True) # Sempre inicie o chromium no modo headless
    page = browser.new_page()
    page.goto('http://localhost:5173') # Servidor já rodando e pronto
    page.wait_for_load_state('networkidle') # CRÍTICO: Aguarde a execução do JS
    # ... sua lógica de automação
    browser.close()
```

## Padrão Reconhecimento-Então-Ação

1. **Inspecione o DOM renderizado**:

   ```python
   page.screenshot(path='/tmp/inspect.png', full_page=True)
   content = page.content()
   page.locator('button').all()
   ```

2. **Identifique seletores** a partir dos resultados da inspeção

3. **Execute ações** usando seletores descobertos

## Armadilha Comum

❌ **Não** inspecione o DOM antes de aguardar `networkidle` em aplicativos dinâmicos
✅ **Faça** aguardar `page.wait_for_load_state('networkidle')` antes da inspeção

## Melhores Práticas

- **Use scripts empacotados como caixas pretas** - Para realizar uma tarefa, considere se um dos scripts disponíveis em `scripts/` pode ajudar. Esses scripts lidam com fluxos de trabalho comuns e complexos de forma confiável sem desordenar a janela de contexto. Use `--help` para ver o uso, depois invoque diretamente.
- Use `sync_playwright()` para scripts síncronos
- Sempre feche o navegador quando terminar
- Use seletores descritivos: `text=`, `role=`, seletores CSS ou IDs
- Adicione esperas apropriadas: `page.wait_for_selector()` ou `page.wait_for_timeout()`

## Arquivos de Referência

- **examples/** - Exemplos mostrando padrões comuns:
  - `element_discovery.py` - Descobrindo botões, links e entradas em uma página
  - `static_html_automation.py` - Usando URLs file:// para HTML local
  - `console_logging.py` - Capturando logs do console durante a automação
