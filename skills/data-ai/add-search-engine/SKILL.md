---
name: add-search-engine
description: Integrate a new LLM search provider into Mentha
---

# Add Search Engine Skill

When the user requests to add a new AI search engine (e.g., "Integrate Claude 3", "Add Anthropic"), follow these steps:

## Steps

1. **Create Provider File**
   - Create `src/infrastructure/search/{provider-name}.provider.ts`
   - Import the `ISearchProvider` interface from `src/domain/search/types.ts`
   - Implement all required methods

2. **Provider Template**
   ```typescript
   import { ISearchProvider, SearchOptions, SearchResult } from '../../domain/search/types.js';
   import { env } from '../../config/index.js';

   export class {ProviderName}Provider implements ISearchProvider {
     private readonly apiKey: string;

     constructor() {
       this.apiKey = env.{PROVIDER}_API_KEY ?? '';
       if (!this.apiKey) {
         throw new Error('{PROVIDER}_API_KEY is required');
       }
     }

     async search(query: string, options?: SearchOptions): Promise<SearchResult> {
       // Implementation
     }
   }
   ```

3. **Update Factory**
   - Edit `src/infrastructure/search/factory.ts`
   - Add the new provider to the `createProvider` function
   - Add the provider enum value

4. **Update Environment**
   - Add `{PROVIDER}_API_KEY` to `.env.example`
   - Add to the env schema in `src/config/env.ts`

5. **Update Schema**
   - Add provider to the engines enum in controller schemas

6. **Test**
   - Run `npm run typecheck` to verify compilation
   - Create a basic integration test

## Validation Checklist

- [ ] Provider implements `ISearchProvider` interface
- [ ] Environment variable documented in `.env.example`
- [ ] Factory updated with new provider
- [ ] Controller enum includes new provider
- [ ] No TypeScript errors
