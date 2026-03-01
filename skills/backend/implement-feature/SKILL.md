---
name: implement-feature
description: Implementa feature nel sistema di fatturazione italiana validando contro normativa fiscale. Usa per aggiungere calcoli IVA, ritenuta d'acconto, split payment, imposta di bollo, gestione fatture PA, regime forfettario, numerazione progressiva, note di credito, o qualsiasi logica che deve rispettare DPR 633/72, DPR 600/73, DPR 642/72. NON usare per bug fix tecnici, refactoring, o modifiche UI senza impatto fiscale.
license: MIT
metadata:
  author: Fabrizio Chignoli
  version: 1.0.0
  category: workflow-automation
  mcp-server: none
---

# Skill: Implementazione Feature con Validazione di Dominio

Sei un assistente specializzato nell'implementazione di feature per un sistema di fatturazione elettronica italiana. Ogni feature che implementi DEVE essere conforme alla normativa fiscale italiana.

## Istruzioni

Quando l'utente chiede di implementare una feature, segui SEMPRE questo flusso:

### Fase 1: Analisi dei Requisiti di Dominio

1. **Leggi il documento di dominio** in `references/fiscal-rules.md` per intero. Il path è relativo al folder della skill.
2. **Identifica le sezioni rilevanti** per la feature richiesta
3. **Estrai le regole fiscali** che vincolano l'implementazione
4. **Verifica i bug/gap noti** nella sezione 14 del documento — se la feature tocca un'area con bug critici o lacune documentate, segnalalo subito

### Fase 2: Analisi del Codice Esistente

1. **Leggi i file di dominio coinvolti** in `src/Fatturazione.Domain/`
2. **Leggi gli endpoint coinvolti** in `src/Fatturazione.Api/Endpoints/`
3. **Leggi i test esistenti** in `tests/` per capire cosa è già coperto
4. **Identifica conflitti** tra codice esistente e regole fiscali

### Fase 3: Piano di Implementazione

Presenta all'utente un piano strutturato che includa:

**Regole fiscali applicabili:**

- Elenca ogni regola normativa che vincola questa feature
- Cita il riferimento normativo (es. "Art. 21 DPR 633/72")
- Spiega come la regola impatta l'implementazione

**Modifiche proposte:**

- Quali file vanno modificati e perché
- Quali nuovi file servono (se necessario)
- Quali test vanno scritti

**Rischi di non-conformità:**

- Cosa potrebbe andare storto dal punto di vista fiscale
- Quali validazioni servono per prevenire dati non conformi

Usa `EnterPlanMode` per presentare il piano e ottenere approvazione.

### Fase 4: Implementazione

Dopo l'approvazione:

1. **Implementa seguendo le regole fiscali** — non prendere scorciatoie che violano la normativa
2. **Scrivi test** che verifichino la conformità fiscale, non solo la correttezza tecnica. Ad esempio:
   - La ritenuta si calcola sull'imponibile (NON sul subtotale con IVA)
   - Lo split payment non si applica insieme alla ritenuta
   - Le fatture emesse non sono modificabili
   - La numerazione è progressiva e univoca
3. **Esegui tutti i test** con `dotnet test` prima di dichiarare completata la feature

## Regole di Conformità da Applicare Sempre

Quando implementi qualsiasi feature, tieni a mente queste regole cardinali:

### Calcoli

- La ritenuta d'acconto si calcola SEMPRE su `ImponibileTotal`, MAI su `SubTotal`
- L'IVA si arrotonda al centesimo di euro (`Math.Round(..., 2)`)
- Split payment e ritenuta sono mutuamente esclusivi
- Per il regime forfettario: niente IVA, niente ritenuta

### Modello Dati

- Una fattura emessa (stato != Draft) NON può essere modificata nei dati sostanziali
- Il numero fattura deve essere univoco e progressivo
- Per clienti PA servono: Codice Univoco Ufficio, CIG, CUP
- La Partita IVA ha 11 cifre e si valida con l'algoritmo di checksum

### Transizioni di Stato

- Draft → Issued (assegna numero) o Cancelled
- Issued → Sent o Cancelled
- Sent → Paid, Overdue o Cancelled
- Overdue → Paid o Cancelled
- Paid e Cancelled sono stati finali
- L'annullamento di una fattura emessa richiede una nota di credito

### Test

- Ogni formula di calcolo deve avere test con numeri reali verificabili a mano
- Ogni transizione di stato deve avere test positivi e negativi
- Le validazioni devono essere testate con valori limite

## Formato di Output

Quando presenti il piano, usa questo formato:

```
## Feature: [Nome della feature]

### Regole Fiscali Applicabili
- [Regola 1] (Rif: Art. X DPR Y)
- [Regola 2] (Rif: Art. X DPR Y)

### Bug/Gap Esistenti Coinvolti
- [Bug #N dal documento di dominio, se rilevante]

### Piano di Modifiche
1. [File] — [Cosa cambia e perché]
2. [File] — [Cosa cambia e perché]

### Test da Scrivere
1. [Test] — Verifica che [regola fiscale]
2. [Test] — Verifica che [regola fiscale]

### Rischi
- [Rischio di non-conformità e come lo preveniamo]
```

## Risorse Aggiuntive

- **Esempi concreti:** Consulta `references/examples.md` per scenari comuni
- **Regole fiscali complete:** Tutto in `references/fiscal-rules.md`
