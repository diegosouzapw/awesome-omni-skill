---
name: rhetorical-analysis
description: "Analyse rhétorique et épistémologique d'articles, discours et textes argumentatifs. Utiliser ce skill quand l'utilisateur demande d'analyser la qualité argumentative d'un texte, d'identifier des sophismes ou biais, d'évaluer la fiabilité des sources citées, de déconstruire la logique d'un raisonnement, ou de produire une réécriture critique structurée d'un document."
---

# Skill d'analyse rhétorique et épistémologique

Ce skill permet d'analyser systématiquement la qualité argumentative d'un texte en combinant plusieurs frameworks établis.

## Frameworks d'analyse intégrés

### 1. Modèle de Toulmin (structure argumentative)
Pour chaque argument identifié, extraire :
- **Claim** : La thèse défendue
- **Grounds** : Les preuves/données avancées
- **Warrant** : Le lien logique implicite entre preuves et thèse
- **Backing** : Les éléments qui soutiennent ce lien logique
- **Qualifier** : Les nuances ou restrictions de la thèse
- **Rebuttal** : Les contre-arguments reconnus ou ignorés

### 2. Test CRAAP (fiabilité des sources)
Pour chaque source citée, évaluer :
- **Currency** : Actualité de l'information
- **Relevance** : Pertinence pour le propos
- **Authority** : Crédibilité de l'auteur/source
- **Accuracy** : Vérifiabilité et exactitude
- **Purpose** : Intention (informer, persuader, vendre...)

### 3. Catalogue des sophismes
Voir `references/fallacies-catalog.md` pour la liste complète des sophismes à détecter.

### 4. Échelle de fiabilité (1-5)
- **5** : Fait établi, consensus scientifique, sources multiples vérifiables
- **4** : Sources sérieuses, raisonnement logique valide, nuances possibles
- **3** : Mélange faits/interprétations, sources partielles
- **2** : Raisonnement contestable, sophismes identifiés
- **1** : Affirmations non sourcées, erreurs logiques majeures

## Workflow d'analyse

1. **Segmentation** : Identifier les arguments distincts du texte
2. **Décomposition Toulmin** : Analyser chaque argument selon le modèle
3. **Évaluation des sources** : Appliquer le test CRAAP aux références
4. **Détection des sophismes** : Scanner pour les erreurs de raisonnement
5. **Synthèse** : Produire le tableau d'analyse et la synthèse critique

## Format de sortie

### Option 1 : Génération automatique via script

1. Produire l'analyse au format JSON (voir `assets/example_analysis.json` pour le schéma)
2. Exécuter le script de génération :

   ```bash
   python scripts/generate_analysis.py analysis.json output.[xlsx|json|md] --format [xlsx|json|md]
   ```

**Exemples :**
```bash
python scripts/generate_analysis.py analysis.json rapport_analyse.xlsx --format xlsx
python scripts/generate_analysis.py analysis.json analysis_output.json --format json
python scripts/generate_analysis.py analysis.json rapport.md --format md
```

Le script génère un fichier de sortie selon le format choisi. Pour le format XLSX, il contient 5 feuilles :
- **Analyse rhétorique** : Tableau principal
- **Détail Toulmin** : Structure complète de chaque argument
- **Évaluation sources (CRAAP)** : Scores des sources citées
- **Synthèse** : Points forts, faibles, patterns
- **Légende** : Échelles et définitions

### Option 2 : Génération manuelle

Utiliser le skill `xlsx` pour créer un fichier avec les colonnes :
- N° de l'argument
- Argument traité (résumé)
- Texte original (extrait clé)
- Affirmation / Thèse (Claim)
- Type de raisonnement (Warrant + évaluation)
- Fiabilité (1-5)
- Évaluation détaillée de la fiabilité
- Commentaire critique

## Précautions

- **Distinguer la conclusion de l'argument** : Un argument peut être faible mais mener à une conclusion juste
- **Éviter le biais de confirmation** : Analyser avec la même rigueur les textes qu'on approuve ou désapprouve
- **Reconnaître les limites** : Certaines affirmations sont impossibles à évaluer sans expertise domaine
- **Contexte militant assumé** : Si le texte est explicitement militant, le noter sans que cela invalide l'analyse
