---
name: asset-management
description: Complete asset management feature for Polkadot dApps using the Assets pallet. Use when user needs fungible token/asset functionality including creating custom tokens, minting tokens to accounts, transferring tokens between accounts, destroying tokens, viewing portfolios, or managing token metadata. Generates production-ready code (~2,200 lines across 15 files) with full lifecycle support (create→mint→transfer→destroy), real-time fee estimation, transaction tracking, and user-friendly error messages. Works with template infrastructure (WalletContext, ConnectionContext, TransactionContext, balance utilities, shared components). Load when user mentions assets, tokens, fungible tokens, token creation, minting, portfolio, or asset pallet.
---

# Asset Management Feature

Implement complete asset management functionality for Polkadot dApps.

## Implementation Overview

Generate asset management in this order:

1. **Pure functions** (lib/) - Asset operations, toast configs, error messages
2. **Custom hooks** - Mutation management, fee estimation, asset ID queries
3. **Components** - Forms for create/mint/transfer/destroy, asset lists, portfolio
4. **Integration** - Exports and routing

**Output:** 14 new files, 4 modified files, ~2,100 lines

**Template provides:** `useFee` hook and `FeeDisplay` component (used by all features)

## Critical Conventions

Follow template's CLAUDE.md strictly:

**State:** NEVER `useReducer` - use `useState` or context only
**TypeScript:** NEVER `any` or `as` - use `unknown` and narrow types
**Architecture:** Components presentational, logic in `lib/` and hooks
**Exports:** ALL exports through barrel files (`index.ts`)
**Balance:** ALWAYS use template's `toPlanck`/`fromPlanck` - NEVER create custom
**Components:** ALWAYS use template shared components - NEVER recreate
**Navigation:** Add links to EXISTING SIDEBAR in App.tsx - NEVER create separate tab navigation

## Common Mistakes

❌ **Creating tab navigation in page content** - Navigation belongs in App.tsx sidebar
❌ **Custom balance utilities** - Use template's `toPlanck`/`fromPlanck`
❌ **Recreating FeeDisplay or TransactionFormFooter** - Use template components
❌ **Using `@polkadot/api`** - Only use `polkadot-api`
❌ **Type assertions (`as`)** - Let types prove correctness

## Layer 1: Pure Functions

### 1. lib/assetOperations.ts

See `references/asset-operations.md` for complete patterns.

Exports:
- `createAssetBatch(api, params, signerAddress)` - Create + metadata + optional mint
- `mintTokens(api, params)` - Mint tokens to recipient
- `transferTokens(api, params)` - Transfer tokens
- `destroyAssetBatch(api, params)` - 5-step destruction

Key: Use `toPlanck` from template, `MultiAddress.Id()`, `Binary.fromText()`, `.decodedCall`, `Utility.batch_all()`

### 2. lib/assetToasts.ts

Toast configurations for all operations:

```typescript
import type { ToastConfig } from './toastConfigs'

export const createAssetToasts: ToastConfig<CreateAssetParams> = {
  signing: (params) => ({ description: `Creating ${params.symbol}...` }),
  broadcasted: (params) => ({ description: `${params.symbol} sent to network` }),
  inBlock: (params) => ({ description: `${params.symbol} in block` }),
  finalized: (params) => ({ title: 'Asset Created! 🎉', description: `${params.name} ready` }),
  error: (params, error) => ({ title: 'Creation Failed', description: parseError(error) }),
}
```

Create similar configs for mint, transfer, destroy.

### 3. lib/assetErrorMessages.ts

See `references/error-messages.md` for complete list.

Exports `ASSET_ERROR_MESSAGES` object and `getAssetErrorMessage(errorType)` function.

### 4. lib/assetQueryHelpers.ts (NEW file)

Asset-specific query invalidation helpers:

```typescript
import type { QueryClient } from '@tanstack/react-query'

export const invalidateAssetQueries = async (queryClient: QueryClient) => {
  await queryClient.invalidateQueries({ queryKey: ['assets'] })
  await queryClient.invalidateQueries({ queryKey: ['assetMetadata'] })
}

export const invalidateBalanceQueries = (
  queryClient: QueryClient,
  assetId: number,
  addresses: (string | undefined)[]
) => {
  addresses.forEach((address) => {
    if (address) {
      queryClient.invalidateQueries({ queryKey: ['assetBalance', assetId, address] })
    }
  })
}
```

**Note:** Template has base `queryHelpers.ts` - this adds asset-specific helpers.

## Layer 2: Custom Hooks

### 5. hooks/useAssetMutation.ts

Generic mutation hook:

```typescript
export const useAssetMutation = <TParams>({
  params,
  operationFn,
  toastConfig,
  onSuccess,
  transactionKey,
  isValid,
}: AssetMutationConfig<TParams>) => {
  const { selectedAccount } = useWalletContext()
  const { executeTransaction } = useTransaction<TParams>(toastConfig)

  const transaction = selectedAccount && (!isValid || isValid(params))
    ? operationFn(params)
    : null

  const mutation = useMutation({
    mutationFn: async () => {
      if (!selectedAccount || !transaction) throw new Error('No account or transaction')
      const observable = transaction.signSubmitAndWatch(selectedAccount.polkadotSigner)
      await executeTransaction(transactionKey, observable, params)
    },
    onSuccess,
  })

  return { mutation, transaction }
}
```

### 6. hooks/useNextAssetId.ts

Query next available asset ID:

```typescript
export function useNextAssetId() {
  const { api } = useConnectionContext()

  const { data, isLoading } = useQuery({
    queryKey: ['nextAssetId'],
    queryFn: async () => {
      const result = await api.query.Assets.NextAssetId.getValue()
      if (result === undefined) throw new Error('NextAssetId undefined')
      return result
    },
    staleTime: 0,
    gcTime: 0,
  })

  return { nextAssetId: data?.toString() ?? '', isLoading }
}
```

## Layer 3: Components

See `references/form-patterns.md` and `references/template-integration.md` for complete patterns.

### 8-11. Form Components

Create these forms using standard layout from `references/form-patterns.md`:

- **CreateAsset.tsx** - Create with `useNextAssetId()`, fields: name, symbol, decimals, minBalance, initialSupply
- **MintTokens.tsx** - Mint, fields: assetId, recipient, amount
- **TransferTokens.tsx** - Transfer, fields: assetId, recipient, amount
- **DestroyAsset.tsx** - Destroy with confirmation, field: assetId (type to confirm)

All forms use:
- `AccountDashboard` at top
- `TransactionReview` in right column
- `TransactionFormFooter` at bottom
- `FeatureErrorBoundary` wrapper

### 12-14. Display Components

**AssetList.tsx** - Query and display all assets:

```typescript
const { data: assets } = useQuery({
  queryKey: ['assets'],
  queryFn: async () => await api.query.Assets.Asset.getEntries(),
})
```

**AssetCard.tsx** - Individual asset display with action menu

**AssetBalance.tsx** - Display asset balance for account using `formatBalance` from template

### 15. AssetDashboard.tsx

Portfolio view combining `AccountDashboard` + `AssetList`.

**NO tab navigation in this component** - navigation is in App.tsx sidebar (see Layer 4).

## Layer 4: Integration

### 16-18. Exports

**components/index.ts** - Add:
```typescript
export { CreateAsset } from './CreateAsset'
export { MintTokens } from './MintTokens'
export { TransferTokens } from './TransferTokens'
export { DestroyAsset } from './DestroyAsset'
export { AssetList } from './AssetList'
export { AssetCard } from './AssetCard'
export { AssetBalance } from './AssetBalance'
export { AssetDashboard } from './AssetDashboard'
```

**hooks/index.ts** - Add:
```typescript
export { useAssetMutation } from './useAssetMutation'
export { useNextAssetId } from './useNextAssetId'
// Note: useFee is in template, not generated here
```

**lib/index.ts** - Add:
```typescript
export * from './assetOperations'
export { invalidateAssetQueries, invalidateBalanceQueries } from './assetQueryHelpers'
export { getAssetErrorMessage } from './assetErrorMessages'
```

### 19. App.tsx

**CRITICAL: Add navigation links to EXISTING SIDEBAR, not as separate tabs.**

Common mistake: Creating tab navigation in the main content area. Instead:

```typescript
// In App.tsx sidebar navigation
<nav className="sidebar">
  {/* Existing links */}
  <Link to="/dashboard">Dashboard</Link>
  
  {/* ADD asset management links HERE in sidebar */}
  <Link to="/assets/create">Create Asset</Link>
  <Link to="/assets/mint">Mint Tokens</Link>
  <Link to="/assets/transfer">Transfer Tokens</Link>
  <Link to="/assets/destroy">Destroy Asset</Link>
  <Link to="/assets/portfolio">Portfolio</Link>
</nav>

// In routes
<Routes>
  {/* Existing routes */}
  <Route path="/" element={<Dashboard />} />
  
  {/* ADD asset management routes */}
  <Route path="/assets/create" element={<CreateAsset />} />
  <Route path="/assets/mint" element={<MintTokens />} />
  <Route path="/assets/transfer" element={<TransferTokens />} />
  <Route path="/assets/destroy" element={<DestroyAsset />} />
  <Route path="/assets/portfolio" element={<AssetDashboard />} />
</Routes>
```

**DO NOT create separate tab navigation in the page content - use the existing sidebar.**

## Validation

After generation:

```bash
# REQUIRED
bash .claude/scripts/validate-typescript.sh

# Verify imports
grep -r "@polkadot/api" src/  # Should be ZERO
grep -r "parseUnits\|formatUnits" src/  # Should be ZERO (use template utilities)
```

## Expected Capabilities

After implementation:
- ✅ Create custom tokens with metadata
- ✅ Mint tokens to recipients
- ✅ Transfer tokens between accounts
- ✅ Destroy tokens (5-step process)
- ✅ View portfolio and balances
- ✅ Real-time fee estimation (via template's `useFee`)
- ✅ Transaction notifications (via template's TransactionContext)
- ✅ User-friendly error messages

## References

Load these as needed during implementation:

- **Asset operations:** `references/asset-operations.md`
- **Form patterns:** `references/form-patterns.md`
- **Error messages:** `references/error-messages.md`
- **Template integration:** `references/template-integration.md`

## Completion Checklist

- [ ] 14 new files generated (useFee is in template, not generated)
- [ ] 4 files modified (3 index.ts + App.tsx)
- [ ] **Navigation added to EXISTING SIDEBAR (not as separate tabs)**
- [ ] TypeScript validation passes
- [ ] Zero @polkadot/api imports
- [ ] Template utilities used: `toPlanck`, `fromPlanck`, `formatBalance`, `useFee`, `FeeDisplay`
- [ ] Shared components used: `TransactionFormFooter`, `TransactionReview`, `AccountDashboard`
- [ ] All exports through barrel files
