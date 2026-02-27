---
name: llm-evaluate
description: Evaluate LLM models for cost/performance ratio. Fetches current pricing and recommends optimal model for your use case. Use during project init or when optimizing costs.
disable-model-invocation: true
allowed-tools: Read, WebFetch, WebSearch, AskUserQuestion
argument-hint: [use-case]
---

# LLM Model Evaluation

Evaluiert LLM-Modelle basierend auf aktuellem Preis/Leistungs-Verhältnis.

---

## Wann nutzen?

- Während `/init-project` bei der Komplexitätsbewertung
- Bei Kosten-Optimierung bestehender Projekte
- Wenn neue Modelle erscheinen (regelmäßig checken)
- Vor größeren Production-Deployments

---

## Step 1: Use Case verstehen

Falls kein Argument übergeben, frage:

```
Was ist dein Use Case?

Beispiele:
• "Chat-Bot für Kundenservice" (High-Volume, schnelle Antworten)
• "Dokumenten-Analyse" (Langer Context, Reasoning)
• "Code-Generierung" (Präzision wichtig)
• "GDPR-konforme EU-App" (Compliance)
• "Budget-Projekt" (Kosten minimieren)
```

---

## Step 2: Aktuelle Preise holen

**WICHTIG:** Preise ändern sich häufig. Hole aktuelle Daten.

### 2.1 Web Search für aktuelle Preise

Suche nach aktuellen Preisen mit WebSearch:

```
Query: "[Provider] API pricing 2026"
```

Für jeden Provider:
- Anthropic Claude pricing
- OpenAI GPT pricing
- Google Gemini pricing
- DeepSeek pricing
- xAI Grok pricing
- Mistral pricing

### 2.2 Pricing Endpoints (falls verfügbar)

Einige Provider haben öffentliche Pricing-Pages:

| Provider | Pricing URL |
|----------|-------------|
| Anthropic | https://www.anthropic.com/pricing |
| OpenAI | https://openai.com/api/pricing |
| Google | https://ai.google.dev/pricing |
| DeepSeek | https://platform.deepseek.com/api-docs/pricing |
| Mistral | https://mistral.ai/technology/#pricing |
| xAI | https://x.ai/api |

### 2.3 Fallback: Cached Reference

Falls Web-Fetch fehlschlägt, nutze `.claude/reference/llm-configuration.md` als Fallback (aber weise auf möglicherweise veraltete Daten hin).

---

## Step 3: Modelle bewerten

### 3.1 Bewertungskriterien

| Kriterium | Gewichtung | Beschreibung |
|-----------|------------|--------------|
| **Kosten** | 30% | Input + Output Tokens |
| **Qualität** | 30% | Benchmark-Scores, Erfahrungswerte |
| **Latenz** | 20% | Time to first token, Throughput |
| **Context** | 10% | Max Context Window |
| **Features** | 10% | Vision, Tools, Streaming |

### 3.2 Use Case Mapping

| Use Case | Wichtig | Unwichtig |
|----------|---------|-----------|
| **Chat-Bot** | Latenz, Kosten | Context |
| **Dokument-Analyse** | Context, Qualität | Latenz |
| **Code-Gen** | Qualität | Kosten |
| **High-Volume** | Kosten, Latenz | Qualität |
| **GDPR** | Compliance | Kosten |

---

## Step 4: Empfehlung ausgeben

### 4.1 Empfehlungs-Template

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  LLM EVALUATION - [Use Case]                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📅 Preise Stand: [Datum der Abfrage]                                       │
│                                                                             │
│  TOP 3 EMPFEHLUNGEN:                                                        │
│                                                                             │
│  🥇 #1: [Modell]                                                            │
│      Provider: [Provider]                                                   │
│      Input:    $[X]/1M tokens                                               │
│      Output:   $[X]/1M tokens                                               │
│      Context:  [X]K                                                         │
│      Score:    [X]/100 (basierend auf Use Case)                             │
│      Warum:    [Begründung]                                                 │
│                                                                             │
│  🥈 #2: [Modell]                                                            │
│      ...                                                                    │
│                                                                             │
│  🥉 #3: [Modell]                                                            │
│      ...                                                                    │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  KOSTEN-SCHÄTZUNG (bei 1M Requests/Monat, 1000 Tokens avg):                │
│                                                                             │
│  Modell #1: ~$[X]/Monat                                                     │
│  Modell #2: ~$[X]/Monat                                                     │
│  Modell #3: ~$[X]/Monat                                                     │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  FALLBACK-STRATEGIE:                                                        │
│                                                                             │
│  Primary:  [Modell #1]                                                      │
│  Fallback: [Modell #2]                                                      │
│  Budget:   [Modell #3]                                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Portkey Config generieren

Falls gewünscht, generiere die Portkey-Konfiguration:

```typescript
// Empfohlene Portkey Konfiguration für [Use Case]
const config = {
  strategy: {
    mode: 'fallback',
  },
  targets: [
    { provider: '[primary]', model: '[model]' },
    { provider: '[fallback]', model: '[model]' },
  ],
  cache: {
    mode: 'semantic',
    ttl: 3600,
  },
};
```

---

## Step 5: Dokumentation aktualisieren

Falls signifikante Preisänderungen gefunden wurden:

1. Weise den User darauf hin
2. Frage ob `.claude/reference/llm-configuration.md` aktualisiert werden soll
3. Bei "Ja": Update die Preistabellen

---

## Automatische Intervall-Checks

### Weekly Reminder

Dieser Skill sollte regelmäßig genutzt werden:

```
Empfehlung: Führe /llm-evaluate monatlich aus um:
- Neue Modelle zu entdecken
- Preisänderungen zu berücksichtigen
- Kosten-Optimierung zu prüfen
```

### Bei Projekt-Init

Während `/init-project` wird dieser Skill automatisch bei der Komplexitätsbewertung (Step 0.2) aufgerufen um das optimale Modell für den Use Case zu empfehlen.

---

## Modell-Datenbank (Referenz)

### Anthropic

| Modell | Input/1M | Output/1M | Context | Stärken |
|--------|----------|-----------|---------|---------|
| Claude Opus 4.5 | $15 | $75 | 200K | Best reasoning |
| Claude Sonnet 4 | $3 | $15 | 200K | Best coding |
| Claude Haiku 3.5 | $0.25 | $1.25 | 200K | Fast, cheap |

### OpenAI

| Modell | Input/1M | Output/1M | Context | Stärken |
|--------|----------|-----------|---------|---------|
| GPT-4o | $5 | $15 | 128K | Multimodal |
| GPT-4o-mini | $0.15 | $0.60 | 128K | Budget GPT-4 |
| o1 | $15 | $60 | 200K | Deep reasoning |

### Google

| Modell | Input/1M | Output/1M | Context | Stärken |
|--------|----------|-----------|---------|---------|
| Gemini 2.0 Pro | $7 | $21 | 1M | Long context |
| Gemini 2.0 Flash | $0.10 | $0.40 | 1M | Fast, cheap |
| Gemini Flash 8B | $0.04 | $0.15 | 1M | Ultra cheap |

### DeepSeek

| Modell | Input/1M | Output/1M | Context | Stärken |
|--------|----------|-----------|---------|---------|
| DeepSeek V3 | $0.27 | $1.10 | 64K | Open-weight |
| DeepSeek R1 | $0.55 | $2.19 | 64K | Reasoning |

### xAI

| Modell | Input/1M | Output/1M | Context | Stärken |
|--------|----------|-----------|---------|---------|
| Grok 2 | $2 | $10 | 128K | Realtime data |
| Grok 2 Vision | $2 | $10 | 32K | Image understanding |

### Mistral

| Modell | Input/1M | Output/1M | Context | Stärken |
|--------|----------|-----------|---------|---------|
| Mistral Large | $2 | $6 | 128K | EU-hosted |
| Codestral | $0.30 | $0.90 | 256K | Code |
| Mistral Small | $0.20 | $0.60 | 128K | Budget |

---

## Beispiel-Ausführung

```
User: /llm-evaluate Chat-Bot für Kundenservice

Claude:
┌─────────────────────────────────────────────────────────────────────────────┐
│  LLM EVALUATION - Customer Service Chat Bot                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📅 Preise Stand: Januar 2026                                               │
│                                                                             │
│  TOP 3 EMPFEHLUNGEN:                                                        │
│                                                                             │
│  🥇 #1: Claude Haiku 3.5                                                    │
│      Provider: Anthropic                                                    │
│      Input:    $0.25/1M tokens                                              │
│      Output:   $1.25/1M tokens                                              │
│      Context:  200K                                                         │
│      Score:    92/100                                                       │
│      Warum:    Schnell, günstig, gute Qualität für Chat                     │
│                                                                             │
│  🥈 #2: Gemini 2.0 Flash                                                    │
│      Provider: Google                                                       │
│      Input:    $0.10/1M tokens                                              │
│      Output:   $0.40/1M tokens                                              │
│      Context:  1M                                                           │
│      Score:    88/100                                                       │
│      Warum:    Noch günstiger, aber weniger konsistent                      │
│                                                                             │
│  🥉 #3: DeepSeek V3                                                         │
│      Provider: DeepSeek                                                     │
│      Input:    $0.27/1M tokens                                              │
│      Output:   $1.10/1M tokens                                              │
│      Context:  64K                                                          │
│      Score:    82/100                                                       │
│      Warum:    Sehr günstig, aber China-based (Compliance?)                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Integration mit init-project

Dieser Skill wird automatisch während `/init-project` aufgerufen:

1. User beschreibt Projekt (Step 0.1)
2. Komplexitätsstufe wird ermittelt (Step 0.2)
3. **`/llm-evaluate` läuft im Hintergrund**
4. Stack-Empfehlung inkl. LLM-Empfehlung (Step 0.3)

---

**Version:** 1.0
**Last Updated:** January 2026
