---
name: viem
description: Viem blockchain client patterns for Ethereum interactions, transactions, signing, encoding, and smart contract calls. Triggers on viem, publicClient, walletClient, chain, abi.
triggers: ["viem", "ox", "publicClient", "walletClient", "transport", "chain", "abi", "transaction", "signature", "encode", "decode"]
---

<objective>
Build blockchain interactions using viem for Ethereum clients, transactions, signing, and smart contract interactions. Viem is the modern replacement for ethers.js with better TypeScript support.
</objective>

<mcp_first>
**CRITICAL: Always fetch viem documentation before implementing.**

```
MCPSearch({ query: "select:mcp__plugin_devtools_context7__query-docs" })
```

```typescript
// Client patterns
mcp__context7__query_docs({
  context7CompatibleLibraryID: "/wevm/viem",
  topic: "createPublicClient createWalletClient"
})

// Contract interactions
mcp__context7__query_docs({
  context7CompatibleLibraryID: "/wevm/viem",
  topic: "readContract writeContract simulateContract"
})

// Signing
mcp__context7__query_docs({
  context7CompatibleLibraryID: "/wevm/viem",
  topic: "signMessage signTypedData EIP-712"
})

// Encoding
mcp__context7__query_docs({
  context7CompatibleLibraryID: "/wevm/viem",
  topic: "encodeFunctionData decodeFunctionResult encodeAbiParameters"
})
```
</mcp_first>

<quick_start>
**Create clients:**

```typescript
import { createPublicClient, createWalletClient, http } from "viem";
import { mainnet } from "viem/chains";
import { privateKeyToAccount } from "viem/accounts";

const publicClient = createPublicClient({
  chain: mainnet,
  transport: http(),
});

const walletClient = createWalletClient({
  chain: mainnet,
  transport: http(),
  account: privateKeyToAccount("0x..."),
});
```

**Read contract:**

```typescript
const balance = await publicClient.readContract({
  address: "0x...",
  abi: erc20Abi,
  functionName: "balanceOf",
  args: [userAddress],
});
```

**Write contract:**

```typescript
const hash = await walletClient.writeContract({
  address: "0x...",
  abi: erc20Abi,
  functionName: "transfer",
  args: [recipient, amount],
});

const receipt = await publicClient.waitForTransactionReceipt({ hash });
```
</quick_start>

<client_types>
| Client | Purpose | Example Use |
|--------|---------|-------------|
| `PublicClient` | Reading blockchain data | `getBlockNumber()`, `readContract()` |
| `WalletClient` | Transaction signing/sending | `sendTransaction()`, `signMessage()` |
| `TestClient` | Testing and simulation | `mine()`, `setBalance()` |
| `BundlerClient` | ERC-4337 User Operations | `sendUserOperation()` |
</client_types>

<utilities>
**Unit conversions:**

```typescript
import { parseEther, formatEther, parseUnits, formatUnits } from "viem";

parseEther("1.5");           // 1500000000000000000n
formatEther(1500000000000000000n);  // "1.5"
parseUnits("100", 6);        // 100000000n (USDC)
formatUnits(100000000n, 6);  // "100"
```

**Address utilities:**

```typescript
import { getAddress, isAddress } from "viem";

getAddress("0xabc...");  // Checksummed address
isAddress("0x...");      // Validates format
```
</utilities>

<constraints>
**Banned:** SSRF protection for config-sourced URLs (they're developer-controlled)

**Required:**
- Lazy initialization for chain resolution
- Client caching by network name
- WebSocket cleanup on destroy/refresh
- Type-safe ABI interactions
</constraints>

<success_criteria>
- [ ] Context7 docs fetched for current API
- [ ] Uses lazy chain resolution
- [ ] Caches clients by network name
- [ ] Type-safe ABI interactions
- [ ] Proper error handling for blockchain ops
</success_criteria>
