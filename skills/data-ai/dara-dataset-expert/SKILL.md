---
name: dara-dataset-expert
description: Warehouse-Prozess-Analyse mit 207 Labels, 47 Prozessen, 8 Szenarien, 13 Triggern. Vollständige Expertise für DaRa Datensatz + REFA-Methodik + Validierungslogik + Szenarioerkennung + Lagerlayout + 74 Artikel-Stammdaten + BPMN-Validierung & IST/SOLL-Vergleich. 100% faktenbasiert ohne Halluzinationen. v5.0 mit Ground Truth Central v3.0 + Multi-Order (S7/S8) + Frame-Level Validation Rules.
---

# DaRa Dataset Expert Skill — Version 5.0

## Zweck

Dieser Skill ermöglicht Claude die **präzise, faktenbasierte Analyse des DaRa-Datensatzes** für intralogistische Warehouse-Prozesse. Er kombiniert die Datensatz-Dokumentation mit **arbeitswissenschaftlichen Methoden (REFA/MTM)**, formaler **Validierungslogik**, **automatischer Szenarioerkennung**, **vollständiger Lagerlayout-Dokumentation** und **BPMN-konformer Prozessvalidierung**.

Der Fokus liegt auf **epistemischer Integrität**: Alle Antworten basieren ausschließlich auf verifizierten Quellen ohne Halluzinationen, Spekulationen oder Annahmen.

---

## NEU in Version 5.0: Vollständige Neuerstellung & Konsolidierung

Version 5.0 ist ein **Major Release** mit systematischer Überarbeitung aller Skill-Dateien durch 7-Phasen-Analyse:

### 🎯 Kern-Verbesserungen

**1. Neue zentrale Validierungsregeln**
- **references/core/validation_rules_v5_0.md** (NEU): 798 Zeilen Frame-Level Validierung
  - Master-Slave-Abhängigkeiten (CC01 → CC02-CC05)
  - Label-Kombinationsregeln mit Python-Code
  - Spezielle Validierungen (Multi-Order, CL134 Global Interrupt)
  - BPMN-Prozess-Mappings im Anhang

**2. BPMN-Validierung massiv erweitert**
- **references/processes/bpmn_validation_v5_0.md** (NEUERSTELLT): 1.623 Zeilen
  - Detailed Process Flows (Figures A2-A7): Exakte Activity-Sequenzen
  - Scenario-Routing Matrix: S1-S8 Mapping mit Prozess-Pfaden
  - Error-Handling Details: CL135 Aktivierungsbedingungen
  - Cross-Process Consistency: Identische vs. unterschiedliche Aktivitäten

**3. Chunking-System vollständig dokumentiert**
- **references/auxiliary/chunking_v5_0.md** (REPAIRED): 1.212 Zeilen
  - Multi-Order Handling (Kapitel 4.6): S7/S8 Trigger-Integration
  - Ground Truth v3.0 Synchronisierung: T11-T13 Mapping
  - Erweiterte Trigger-Logik für Extensions-Kategorie

**4. Referenzintegrität garantiert**
- ✅ **0 fehlerhafte Referenzen** (11 Fehler behoben in Phase 6)
- ✅ **100% Konsistenz** aller Datei-Links
- ✅ **Alle Dateien mit `_v5_0`-Suffix** für eindeutige Versionierung

---

### 📊 Datensatz-Umfang (v5.0)

- **18 Probanden (S01-S18)** mit demografischen und Erfahrungsprofilen
- **Session-basierte Aufzeichnungen** mit 3 parallelen Subjekten pro Session
- **8 Szenarien (S1-S8)** für Retrieval- und Storage-Prozesse
- **12 Klassenkategorien (CC01-CC12)** mit insgesamt **207 Labels (CL001-CL207)**
- **74 Artikel** über 3 Orders (2904/2905/2906) mit Lagerorten
- **8 Regalkomplexe** in 5 Gassen (Aisle 1-5)
- **REFA/MTM-Zeitarten-Mapping** ($t_{R}$, $t_{MH}$, $t_{MN}$, $t_{v}$)
- **Validierungsregeln** (Master-Slave-Abhängigkeiten + Frame-Level + Szenario)
- **BPMN-Prozesslogik** für Warehouse-Kommissionierung und Einlagerung
- **Chunking-System** mit 13 Triggern (T1-T13) + Ground Truth v3.0 Integration

**Skill-Dateien:** 18 finale v5_0-Dateien (~324 KB, 9.655 Zeilen)  
**Datensatz-Stand:** 20.10.2025 | **Skill-Stand:** 05.02.2026 (v5.0)

---

## Wann diesen Skill nutzen

### ✅ Verwende diesen Skill für:

1. **Strukturelle Datensatz-Fragen**
   - "Wie viele Probanden gibt es?"
   - "Wie sind Sessions aufgebaut?"
   - "Welche Szenarien existieren?"
   - "Erkläre die Chunking-Trigger T1-T13"
   - "Was ist neu in v5.0?"

2. **Klassifikations-Queries**
   - "Welche Labels gehören zu CC04 (Left Hand)?"
   - "Was ist der Unterschied zwischen CC08, CC09 und CC10?"
   - "Zeige mir alle Tool-Labels"
   - "Welche Kategorie hat CL135?"

3. **REFA & Arbeitswissenschaft**
   - "Welche DaRa-Labels entsprechen der Haupttätigkeit ($t_{MH}$)?"
   - "Wie wird die Erholungszeit basierend auf CC03 berechnet?"
   - "Ist 'Travel Time' eine Nebentätigkeit?"
   - "Berechne die Auftragszeit für ein Szenario"
   - "Was ist MTM-Code B (Bend) und wie viele TMU hat er?"

4. **Validierung & Logik**
   - "Darf man 'Walking' annotieren, wenn die Beine 'Standing Still' sind?"
   - "Welche Low-Level-Prozesse sind im Retrieval-Prozess erlaubt?"
   - "Prüfe, ob 'Scanning' ohne Scanner-Tool möglich ist."
   - "Welche Abhängigkeiten bestehen zwischen CC01 und CC09?"
   - "Was sind die Master-Slave-Regeln für CC01?"

5. **BPMN-Prozess-Analysen**
   - "Erkläre den Retrieval-Pfad im BPMN"
   - "Was passiert nach 'Picking Pick Time'?"
   - "Welche Entscheidungspunkte gibt es im Storage-Prozess?"
   - "Validiere diesen Prozess gegen BPMN"
   - "IST/SOLL-Vergleich für Proband S14"
   - "Wann wird CL135 (Error-Handling) aktiviert?"

6. **Szenario-Erkennung**
   - "Wie unterscheiden sich S1 und S7?"
   - "Erkläre die 5-Schritt Decision-Logik"
   - "Was sind Multi-Order Szenarien?"
   - "Wann wird Trigger T11 aktiviert?"
   - "Wie erkenne ich 'Other_Waiting'?"

7. **Datenstruktur-Fragen**
   - "Wie sind Frames synchronisiert?"
   - "Was ist die CSV-Struktur?"
   - "Wie funktioniert die Session-Organisation?"

8. **Lagerlayout & Physik**
   - "Wie groß ist Aisle 3?"
   - "Wo liegt Artikel 2904-042?"
   - "Welche Location-Transitions sind erlaubt?"
   - "Was ist Teleportation-Detection?"

---

### ❌ Nutze diesen Skill NICHT für:

- Rohdaten-Analyse (keine CSV-Dateien im Skill) → Lade selbst hoch
- Statistische Auswertungen → Nutze Pandas/Python
- Modelltraining oder ML-Code → Außerhalb des Skill-Scopes
- Bild-/Videoanalyse → Keine Videodaten im Skill
- Vorhersagen → Skill ist deskriptiv, nicht prädiktiv

---

## Navigationslogik (Orchestrierung) — v5.0

**Schritt 1: Identifiziere die Fragedomäne**

```python
# 1. Grundlegende Datensatz-Informationen
if "Proband" or "Subjekt" or "S01" to "S18" or "Session" in query:
    view("references/auxiliary/dataset_core_v5_0.md")

# 2. Datenstruktur / Frame-Synchronisation
elif "Frame" or "Synchronisation" or "CSV" or "Zeile" or "Datenformat" in query:
    view("references/auxiliary/data_structure_v5_0.md")

# 3. Lagerlayout / Physische Umgebung
elif "Lager" or "Regal" or "Gasse" or "Aisle" or "Zone" or "Compartment" or "Location" in query:
    view("references/auxiliary/warehouse_physical_v5_0.md")

# 4. Chunking-Logik (mit Validierung & Multi-Order)
elif "Chunk" or "Trigger" or "T1" to "T13" or "Segment" or "Multi-Order S7/S8" in query:
    view("references/auxiliary/chunking_v5_0.md")

# 5. Semantik / Abhängigkeiten
elif "Semantik" or "Abhängigkeit" or "Bedeutung" or "Zusammenhang" in query:
    view("references/auxiliary/semantics_v5_0.md")

# 6. Label-Definitionen
elif "Label" or "CL" or "CC" or "Kategorie" or "Klassifikation" in query:
    view("references/core/labels_207_v5_0.md")

# 7. Artikel-Stammdaten
elif "Artikel" or "Order 2904" or "Order 2905" or "Order 2906" or "Gewicht" in query:
    view("references/core/articles_inventory_v5_0.md")

# 8. Szenarioerkennung (Ground Truth v3.0)
elif "Szenario" or "S1" to "S8" or "Erkennung" or "Ground Truth" or "5-Schritt" in query:
    view("references/core/ground_truth_central_v5_0.md")

# 9. Category Activation Matrices
elif "Category Activation" or "Szenario-Matrix" or "IT-System-Mapping" in query:
    view("references/core/category_activation_matrix_v5_0.md")

# 10. Frame-Level Validierungsregeln (NEU v5.0)
elif "Master-Slave" or "Label-Kombination" or "Frame-Validierung" or "Validierungsregel" in query:
    view("references/core/validation_rules_v5_0.md")

# 11. REFA / Zeitarten
elif "REFA" or "Zeitart" or "t_MH" or "Erholung" or "Verteilzeit" or "Auftragszeit" in query:
    view("references/processes/refa_analytics_v5_0.md")

# 12. MTM-Codes
elif "MTM" or "TMU" or "Reach" or "Grasp" or "Move" or "Grundbewegung" in query:
    view("references/processes/mtm_codes_v5_0.md")

# 13. Prozess-Hierarchie
elif "Prozess-Hierarchie" or "High-Level" or "Mid-Level" or "Low-Level" or "CC08/CC09/CC10" in query:
    view("references/processes/process_hierarchy_v5_0.md")

# 14. BPMN-Validierung & Prozessanalyse (Hauptlogik)
elif "BPMN" or "Validierung" or "Abweichung" or "IST SOLL" or "Conformity" or "Sequenzfehler" in query:
    view("references/processes/bpmn_validation_v5_0.md")

# 15. BPMN-Validierung Quick Start (Tutorial)
elif "BPMN Quick Start" or "BPMN Tutorial" or "BPMN Anwendung" in query:
    view("references/processes/bpmn_validation_quickstart_v5_0.md")

# 16. Query-Patterns (Skill-Anwendung)
elif "Query Pattern" or "Wie frage ich" or "Beispiel-Fragen" in query:
    view("assets/query_patterns_v5_0.md")

# 17. Report-Templates
elif "Report Template" or "Bericht-Vorlage" or "Ausgabeformat" in query:
    if "BPMN" in query:
        view("assets/bpmn_validation_report_template_v5_0.md")
    else:
        view("assets/scenario_report_template_v5_0.md")

# 18. v5.0 Dokumentation
elif "CHANGELOG" or "Was ist neu" or "Version 5.0" in query:
    view("docs/CHANGELOG_v5_0.md")

elif "Migration" or "Upgrade" or "v4" in query:
    view("docs/MIGRATION_v5_0.md")

elif "Struktur" or "Dateiübersicht" or "Dependencies" in query:
    view("docs/STRUCTURE_v5_0.md")

elif "README" or "Quick Start" or "Einstieg" in query:
    view("docs/README_v5_0.md")

# 19. Fallback
else:
    view("references/auxiliary/dataset_core_v5_0.md")
```

**Schritt 2: Präzise antworten**

- Nur dokumentierte Fakten verwenden
- Label-IDs korrekt zitieren (z.B. "CL115")
- Fachbegriffe korrekt verwenden (z.B. "Master-Slave", "$t_{MN}$")
- Quelle angeben (z.B. "Gemäß Regel V-B1 in references/core/validation_rules_v5_0.md...")
- Bei Unsicherheit: **"Diese Information ist nicht in den Skill-Dateien dokumentiert"**

---

## Antwort-Prinzipien

### 1. Unterscheidung Datensatz vs. Methode

Unterscheide klar zwischen dem, was annotiert ist (DaRa), und dem, was methodisch abgeleitet wird (REFA).

**❌ Falsch:** "CC09 ist die Haupttätigkeit."  
**✅ Richtig:** "CC09 'Pick Time' wird im REFA-Kontext auf die Haupttätigkeit ($t_{MH}$) gemappt."

### 2. Terminologie-Standard

**✅ Korrekt:**
- "CC04 — Sub-Activity: Left Hand"
- "Label CL115: Picking — Travel Time"
- "Kategorie CC09 (Mid-Level Process)"
- "Storage Compartment ID R1.2.7.A"
- "Gewichtsklasse Large [L]"
- "Trigger T6: Order Addition/Removal"

**❌ Falsch:**
- "Linke Hand" (ohne CC04)
- "CL-115" (falsches Format)
- "Mid-level" (inkonsistente Schreibweise)
- "Regal 1.2.7.A" (ohne R-Präfix)
- "Trigger 6" (ohne T-Präfix)

### 3. Formale Korrektheit

**Zitiere immer:**
- Regel-IDs (z.B. "V-B1", "V-S1")
- Abschnitte (z.B. "Kapitel 4.6 in references/auxiliary/chunking_v5_0.md")
- BPMN-Figuren (z.B. "Figure A3: Picking Process")

**Bei Unsicherheit:**
- ❌ "Ich glaube, dass..."
- ❌ "Vermutlich ist..."
- ✅ "Diese Information ist nicht dokumentiert. Ich kann nur bestätigen, dass..."

### 4. Keine Halluzinationen

**Wenn etwas nicht dokumentiert ist:**
- ❌ Erfinde keine Regeln
- ❌ Extrapoliere nicht ohne Grundlage
- ✅ Sage klar: "Diese Information ist nicht in den Skill-Dateien enthalten"
- ✅ Biete an: "Ich kann aber verwandte Informationen aus [Datei X] teilen"

---

## Dateiübersicht v5.0

### CORE-DATEIEN (5)
1. **references/core/labels_207_v5_0.md** — Vollständiges Label-Inventar (CL001-CL207)
2. **references/core/articles_inventory_v5_0.md** — 74 Artikel-Stammdaten
3. **references/core/category_activation_matrix_v5_0.md** — Szenario-Label-Mappings
4. **references/core/ground_truth_central_v5_0.md** — Ground Truth v3.0
5. **references/core/validation_rules_v5_0.md** ⭐ NEU — Frame-Level Validierungsregeln

### AUXILIARY-DATEIEN (5)
6. **references/auxiliary/chunking_v5_0.md** — Chunking-System & Trigger T1-T13
7. **references/auxiliary/data_structure_v5_0.md** — Frame-Struktur (30 fps)
8. **references/auxiliary/dataset_core_v5_0.md** — Dataset-Kerndokumentation
9. **references/auxiliary/semantics_v5_0.md** — Semantische Grunddefinitionen
10. **references/auxiliary/warehouse_physical_v5_0.md** — OMNI Warehouse Layout

### PROCESSES-DATEIEN (5)
11. **references/processes/bpmn_validation_v5_0.md** ⭐ NEU — Vollständige BPMN-Validierungslogik
12. **references/processes/bpmn_validation_quickstart_v5_0.md** — BPMN-Validierung Tutorial
13. **references/processes/process_hierarchy_v5_0.md** — Prozess-Hierarchie (CC08/09/10)
14. **references/processes/refa_analytics_v5_0.md** — REFA-Zeitarten-Mapping
15. **references/processes/mtm_codes_v5_0.md** — MTM-1 Grundbewegungen

### ASSETS-DATEIEN (3)
16. **assets/query_patterns_v5_0.md** — Query-Routing-Logik
17. **assets/bpmn_validation_report_template_v5_0.md** — BPMN Report Template
18. **assets/scenario_report_template_v5_0.md** — Szenario Report Template

### RELEASE-DOKUMENTATION (4)
- **docs/CHANGELOG_v5_0.md** — Vollständige Änderungshistorie
- **docs/MIGRATION_v5_0.md** — Upgrade-Anleitung
- **docs/STRUCTURE_v5_0.md** — Detaillierte Dateistruktur
- **docs/README_v5_0.md** — Quick Start Guide

---

## Qualitätsgarantie v5.0

✅ **Referenzintegrität:** 0 fehlerhafte Referenzen  
✅ **Cross-File-Konsistenz:** 100% verifiziert  
✅ **Terminologie:** 207 CL-Codes, S1-S8, CC01-CC12 konsistent  
✅ **Versionierung:** Alle Dateien mit `_v5_0`-Suffix  
✅ **Dokumentation:** Vollständig  
✅ **Status:** Production-Ready

---

**Version:** 5.0.0  
**Release-Datum:** 05.02.2026  
**Status:** Finalisiert ✅  
**Prozess:** 7-Phasen-Analyse (Phase 1-7 abgeschlossen)
