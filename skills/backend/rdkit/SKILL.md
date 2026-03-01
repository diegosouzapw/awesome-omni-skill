---
name: rdkit
description: Modern RDKit workflows for cheminformatics, including molecular fingerprints, drawing, and property calculations. Use when working with molecules, SMILES, molecular fingerprints (Morgan, ECFP, RDKit, atom pairs, topological torsions), molecule visualization/drawing, substructure search, or chemical property calculations. This skill provides up-to-date syntax patterns as RDKit's API evolves.
---

# RDKit Skill

This skill provides up-to-date RDKit patterns. **LLMs often have outdated RDKit syntax** — always follow the patterns here.

## Installation

```bash
uv pip install rdkit
```

## Core Imports

```python
from rdkit import Chem
from rdkit.Chem import AllChem, Descriptors, Draw, rdFingerprintGenerator
from rdkit.Chem import rdDepictor
from rdkit import DataStructs
```

## Molecule I/O

```python
# From SMILES
mol = Chem.MolFromSmiles('CCO')

# From file
mol = Chem.MolFromMolFile('molecule.mol')
mols = [m for m in Chem.SDMolSupplier('molecules.sdf') if m is not None]

# To SMILES
smiles = Chem.MolToSmiles(mol)
```

## Fingerprints — USE THE NEW API

**CRITICAL**: Use `rdFingerprintGenerator` module, NOT the old functions like `GetMorganFingerprint()`.

See `references/fingerprints.md` for complete documentation.

### Quick Reference

```python
from rdkit.Chem import rdFingerprintGenerator

# Morgan/ECFP fingerprints (ECFP4 = radius 2)
mfpgen = rdFingerprintGenerator.GetMorganGenerator(radius=2, fpSize=2048)

# RDKit fingerprint
rdkgen = rdFingerprintGenerator.GetRDKitFPGenerator(fpSize=2048)

# Atom pairs
apgen = rdFingerprintGenerator.GetAtomPairGenerator(fpSize=2048)

# Topological torsions
ttgen = rdFingerprintGenerator.GetTopologicalTorsionGenerator(fpSize=2048)

# Generate fingerprints (same API for all types)
fp = mfpgen.GetFingerprint(mol)           # bit vector
cfp = mfpgen.GetCountFingerprint(mol)     # count vector
np_fp = mfpgen.GetFingerprintAsNumPy(mol) # numpy array
```

## Drawing Molecules

**Use `MolDraw2DCairo` or `MolDraw2DSVG`** with `drawOptions()` for customization.

See `references/drawing.md` for complete documentation.

### Quick Reference

```python
from rdkit.Chem import Draw, rdDepictor

# Generate 2D coordinates
rdDepictor.Compute2DCoords(mol)
rdDepictor.StraightenDepiction(mol)

# Simple drawing
img = Draw.MolToImage(mol, size=(300, 300))
img.save('molecule.png')

# Customized drawing
d2d = Draw.MolDraw2DCairo(350, 300)
dopts = d2d.drawOptions()
dopts.addAtomIndices = True  # show atom indices
d2d.DrawMolecule(mol)
d2d.FinishDrawing()
with open('molecule.png', 'wb') as f:
    f.write(d2d.GetDrawingText())

# Grid of molecules
img = Draw.MolsToGridImage(mols, molsPerRow=4, subImgSize=(200, 200))
```

## Molecular Properties

```python
from rdkit.Chem import Descriptors

mw = Descriptors.MolWt(mol)
logp = Descriptors.MolLogP(mol)
hbd = Descriptors.NumHDonors(mol)
hba = Descriptors.NumHAcceptors(mol)
tpsa = Descriptors.TPSA(mol)
rotatable = Descriptors.NumRotatableBonds(mol)
```

## Substructure Search

```python
# SMARTS pattern matching
pattern = Chem.MolFromSmarts('[OH]')
matches = mol.GetSubstructMatches(pattern)  # returns tuple of tuples

# Check if has substructure
has_oh = mol.HasSubstructMatch(pattern)
```

## Similarity

**Always specify which fingerprint** when reporting similarity — different fingerprints give very different values for the same molecule pair (e.g., RDKit FP: 0.79 vs Morgan2: 0.43).

```python
from rdkit import DataStructs

# Tanimoto similarity between two fingerprints
sim = DataStructs.TanimotoSimilarity(fp1, fp2)

# Bulk similarity (one vs many)
sims = DataStructs.BulkTanimotoSimilarity(fp1, [fp2, fp3, fp4])
```

## Molecule Properties

```python
# Get/set/check properties on molecules, atoms, bonds
mol.SetProp('name', 'aspirin')
mol.GetProp('name')
mol.HasProp('name')
mol.ClearProp('name')

# Typed getters/setters
mol.SetDoubleProp('score', 3.14)
mol.GetDoubleProp('score')
mol.SetIntProp('count', 42)

# Get all properties
mol.GetPropsAsDict()  # {key: value, ...}

# Private props (start with _) hidden by default
mol.GetPropsAsDict(includePrivate=True)
```

### Pickling Gotcha

Properties are **lost by default** when pickling/serializing:

```python
from rdkit import Chem

# Enable property preservation
Chem.SetDefaultPickleProperties(Chem.PropertyPickleOptions.AllProps)

# Or per-molecule
binary = mol.ToBinary(Chem.PropertyPickleOptions.AllProps)
```

## Parsing & Sanitization

Control chemistry perception during parsing. See `references/parsing.md` for details.

```python
# Custom parsing without sanitization
params = Chem.SmilesParserParams()
params.sanitize = False
params.removeHs = False
mol = Chem.MolFromSmiles(smiles, params=params)

# Manual sanitization after preprocessing
Chem.SanitizeMol(mol)
Chem.AssignStereochemistry(mol, cleanIt=True, force=True)
```
