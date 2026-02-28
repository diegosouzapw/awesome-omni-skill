---
name: ai-model
description: Gère les modèles IA de Motivia. Utilise ce skill quand l'utilisateur demande d'ajouter un nouveau modèle IA, modifier un provider, ou configurer les options de génération. Supporte OpenAI, Anthropic, Google, Mistral et xAI.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Gestion des Modèles IA Motivia

## Fichiers concernés

```
utils/ai-model.tsx           # Registry des modèles et providers
prisma/schema.prisma         # Enum ApiProvider
components/icons/            # Icônes des providers
app/api/create/route.ts      # Endpoint de génération
```

## Structure d'un modèle

```typescript
export type AIModelOption = {
  provider: ApiProvider;     // Enum Prisma: OPENAI, ANTHROPIC, GOOGLE, MISTRAL, XAI
  model: string;             // ID du modèle (ex: "gpt-5-nano-2025-08-07")
  label: string;             // Nom affiché (ex: "GPT 5 nano")
  free: boolean;             // true = utilise les lettres gratuites
  cost: number;              // Coût relatif (pour affichage)
};
```

## Providers actuels

| Provider | Enum | SDK |
|----------|------|-----|
| OpenAI | `ApiProvider.OPENAI` | `@ai-sdk/openai` |
| Anthropic | `ApiProvider.ANTHROPIC` | `@ai-sdk/anthropic` |
| Google | `ApiProvider.GOOGLE` | `@ai-sdk/google` |
| Mistral | `ApiProvider.MISTRAL` | `@ai-sdk/mistral` |
| xAI | `ApiProvider.XAI` | `@ai-sdk/xai` |

## Ajouter un nouveau modèle (provider existant)

### 1. Ajouter à la liste des modèles

```typescript
// utils/ai-model.tsx
export const OPENAI_MODELS = [
  // ... existing models
  {
    provider: ApiProvider.OPENAI,
    model: "gpt-5-ultra-2025-xx-xx",  // ID exact du modèle
    label: "GPT 5 Ultra",              // Nom affiché
    free: false,                       // false = nécessite clé API
    cost: 50,                          // Coût relatif
  },
];
```

### 2. Vérifier l'export dans modelOptions

```typescript
export const modelOptions: AIModelOption[] = [
  ...OPENAI_MODELS,
  ...ANTHROPIC_MODELS,
  ...MISTRAL_MODELS,
  ...XAI_MODELS,
];
```

## Ajouter un nouveau provider

### 1. Installer le SDK

```bash
pnpm add @ai-sdk/newprovider
```

### 2. Ajouter l'enum Prisma

```prisma
// prisma/schema.prisma
enum ApiProvider {
  OPENAI
  ANTHROPIC
  GOOGLE
  MISTRAL
  XAI
  NEWPROVIDER  // Ajouter ici
}
```

### 3. Exécuter la migration

```bash
pnpm prisma migrate dev --name add_newprovider
```

### 4. Créer l'icône du provider

```typescript
// components/icons/newprovider-icon.tsx
export const NewProviderIcon = ({ className }: { className?: string }) => (
  <svg className={className} viewBox="0 0 24 24">
    {/* SVG path */}
  </svg>
);
```

### 5. Ajouter au registry

```typescript
// utils/ai-model.tsx
import { createNewProvider } from "@ai-sdk/newprovider";
import { NewProviderIcon } from "@/components/icons";

// Dans providersLabelAndIcon
{
  label: "New Provider",
  value: ApiProvider.NEWPROVIDER,
  image: <NewProviderIcon className="size-6" />,
},

// Dans les modèles
export const NEWPROVIDER_MODELS = [
  {
    provider: ApiProvider.NEWPROVIDER,
    model: "model-id",
    label: "Model Name",
    free: false,
    cost: 10,
  },
];

// Dans modelOptions
export const modelOptions: AIModelOption[] = [
  ...OPENAI_MODELS,
  ...ANTHROPIC_MODELS,
  ...MISTRAL_MODELS,
  ...XAI_MODELS,
  ...NEWPROVIDER_MODELS,  // Ajouter ici
];

// Dans registry
export const registry = (apiKey?: string, provider?: ApiProvider) =>
  createProviderRegistry({
    // ... existing
    NEWPROVIDER: createNewProvider({
      apiKey:
        provider === ApiProvider.NEWPROVIDER && apiKey
          ? apiKey
          : process.env.NEWPROVIDER_API_KEY,
    }),
  });
```

### 6. Ajouter la variable d'environnement

```env
NEWPROVIDER_API_KEY=sk-...
```

## Règles métier

### Modèles gratuits

- Un modèle `free: true` utilise les 5 lettres gratuites de l'utilisateur
- Si `freeLetters === 0`, l'utilisateur doit avoir sa propre clé API

### Coût

Le champ `cost` est indicatif et sert à l'affichage. Valeurs suggérées:

| Gamme | Cost | Exemple |
|-------|------|---------|
| Économique | 0-1 | nano, small |
| Standard | 2-5 | mini, medium |
| Premium | 10-15 | sonnet, medium |
| Ultra | 20+ | opus |

## Génération de lettre

Le flow dans `app/api/create/route.ts`:

1. Vérifier l'authentification
2. Valider le modèle demandé
3. Récupérer la clé API (user ou env)
4. Si free: décrémenter `freeLetters`
5. Streamer la génération via `streamObject`
6. Retourner `{ title, content }`

## Checklist nouveau modèle

1. [ ] ID du modèle correct (vérifier docs du provider)
2. [ ] Label clair et cohérent
3. [ ] `free` bien défini (true/false)
4. [ ] `cost` relatif défini
5. [ ] Ajouté à l'array `modelOptions`
6. [ ] Testé avec une clé API valide
