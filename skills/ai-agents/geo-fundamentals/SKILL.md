---
name: geo-fundamentals
description: Otimização de Motores Generativos (GEO) para mecanismos de busca de IA (ChatGPT, Claude, Perplexity).
allowed-tools: Read, Glob, Grep
---

# Fundamentos de GEO

> Otimização para mecanismos de busca movidos por IA.

---

## 1. O que é GEO?

**GEO** = Generative Engine Optimization (Otimização de Motores Generativos)

| Objetivo | Plataforma |
| :--- | :--- |
| Ser citado nas respostas da IA | ChatGPT, Claude, Perplexity, Gemini |

### SEO vs GEO

| Aspecto | SEO | GEO |
| :--- | :--- | :--- |
| Objetivo | Ranking #1 | Citações da IA |
| Plataforma | Google | Motores de IA |
| Métricas | Rankings, CTR | Taxa de citação |
| Foco | Palavras-chave | Entidades, dados |

---

## 2. Panorama dos Motores de IA

| Motor | Estilo de Citação | Oportunidade |
| :--- | :--- | :--- |
| **Perplexity** | Numerado [1][2] | Maior taxa de citação |
| **ChatGPT** | Inline/Rodapés | GPTs customizados |
| **Claude** | Contextual | Conteúdo longo |
| **Gemini** | Seção de fontes | Cruzamento com SEO |

---

## 3. Fatores de Recuperação RAG

Como os motores de IA selecionam o conteúdo para citar:

| Fator | Peso |
| :--- | :--- |
| Relevância semântica | ~40% |
| Correspondência de palavra-chave | ~20% |
| Sinais de autoridade | ~15% |
| Atualidade (Freshness) | ~10% |
| Diversidade de fontes | ~15% |

---

## 4. Conteúdo que é Citado

| Elemento | Por que Funciona |
| :--- | :--- |
| **Estatísticas originais** | Dados únicos e citáveis |
| **Citações de especialistas** | Transferência de autoridade |
| **Definições claras** | Fácil de extrair |
| **Guias passo a passo** | Valor acionável |
| **Tabelas de comparação** | Informação estruturada |
| **Seções de FAQ** | Respostas diretas |

---

## 5. Checklist de Conteúdo GEO

### Elementos de Conteúdo

- [ ] Títulos baseados em perguntas.
- [ ] Resumo/TL;DR (Too Long; Didn't Read) no topo.
- [ ] Dados originais com fontes.
- [ ] Citações de especialistas (nome, cargo).
- [ ] Seção de FAQ (3-5 perguntas e respostas).
- [ ] Definições claras.
- [ ] Carimbo de data "Última atualização".
- [ ] Autor com credenciais.

### Elementos Técnicos

- [ ] Schema de Artigo com datas.
- [ ] Schema de Pessoa para o autor.
- [ ] Schema de FAQPage.
- [ ] Carregamento rápido (< 2.5s).
- [ ] Estrutura HTML limpa.

---

## 6. Construção de Entidade

| Ação | Propósito |
| :--- | :--- |
| Painel de Conhecimento do Google | Reconhecimento de entidade |
| Wikipedia (se notable/notável) | Fonte de autoridade |
| Informação consistente na web | Consolidação de entidade |
| Menções na indústria | Sinais de autoridade |

---

## 7. Acesso de Crawlers de IA

### Principais User-Agents de IA

| Crawler | Motor |
| :--- | :--- |
| GPTBot | ChatGPT/OpenAI |
| Claude-Web | Claude |
| PerplexityBot | Perplexity |
| Googlebot | Gemini (compartilhado) |

### Decisão de Acesso

| Estratégia | Quando |
| :--- | :--- |
| Permitir tudo | Deseja citações da IA |
| Bloquear GPTBot | Não quer treinamento da OpenAI |
| Seletivo | Permitir alguns, bloquear outros |

---

## 8. Mensuração

| Métrica | Como Rastrear |
| :--- | :--- |
| Citações da IA | Monitoramento manual |
| Menções "De acordo com [Marca]" | Busca na IA |
| Citações de concorrentes | Comparação de share |
| Tráfego via IA | Parâmetros UTM |

---

## 9. Anti-Padrões

| ❌ NÃO Faça | ✅ Faça |
| :--- | :--- |
| Publicar sem datas | Adicione carimbos de data |
| Atribuições vagas | Nomeie as fontes |
| Pular info do autor | Mostre as credenciais |
| Conteúdo raso | Cobertura abrangente |

---

> **Lembre-se:** A IA cita conteúdo que é claro, autoritativo e fácil de extrair. Seja a melhor resposta.

---

## Script

| Script | Propósito | Comando |
| :--- | :--- | :--- |
| `scripts/geo_checker.py` | Auditoria de GEO (Prontidão para citação por IA) | `python scripts/geo_checker.py <caminho_do_projeto>` |
