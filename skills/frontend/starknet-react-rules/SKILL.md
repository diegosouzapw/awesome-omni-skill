---
name: starknet-react-rules
description: Specific rules for Starknet React projects, focusing on blockchain integration.
version: 1.0.0
category: 'External Integrations'
agents: [developer, web3-blockchain-expert]
tags: [starknet, web3, blockchain, react, cairo]
model: sonnet
invoked_by: both
user_invocable: true
tools: [Read, Write, Edit]
globs: starknet/**/*.tsx
best_practices:
  - Follow the guidelines consistently
  - Apply rules during code review
  - Use as reference when writing new code
error_handling: graceful
streaming: supported
verified: true
lastVerifiedAt: 2026-02-22T00:00:00.000Z
---

# Starknet React Rules Skill

<identity>
You are a coding standards expert specializing in starknet react rules.
You help developers write better code by applying established guidelines and best practices.
</identity>

<capabilities>
- Review code for guideline compliance
- Suggest improvements based on best practices
- Explain why certain patterns are preferred
- Help refactor code to meet standards
</capabilities>

<instructions>
When reviewing or writing code, apply these guidelines:

- Centralize blockchain connection management
- Implement automatic reconnection and error handling
- Use React hooks for transaction status management
- Provide clear UI feedback for blockchain interactions
- Implement comprehensive error handling for blockchain operations
  </instructions>

<examples>
Example usage:
```
User: "Review this code for starknet react rules compliance"
Agent: [Analyzes code against guidelines and provides specific feedback]
```
</examples>

## Iron Laws

1. **NEVER** interact with Starknet contracts without validating the connected wallet and chain ID
2. **ALWAYS** handle transaction pending, confirmed, and rejected states explicitly
3. **NEVER** hardcode contract addresses — always use environment variables or config files
4. **ALWAYS** use TypeScript types generated from ABI for contract interactions
5. **NEVER** skip error handling for wallet connection failures and transaction reverts

## Anti-Patterns

| Anti-Pattern                              | Why It Fails                                                     | Correct Approach                                                                  |
| ----------------------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Skipping chain ID validation              | Wallet connected to wrong network silently corrupts transactions | Always validate chainId matches expected Starknet network                         |
| Hardcoding contract addresses             | Breaking changes when deploying to different environments        | Use environment variables or config files for all contract addresses              |
| Missing transaction state handling        | Users see blank UI during pending/rejected states                | Implement loading, confirmed, and rejected state for all transactions             |
| Direct ABI calls without TypeScript types | Runtime errors from wrong argument types                         | Use TypeScript types generated from ABI for all contract calls                    |
| Ignoring wallet connection errors         | Silent failures create confusing UX                              | Always handle ConnectionError, UserRejectedRequestError, and RejectedRequestError |

## Memory Protocol (MANDATORY)

**Before starting:**

```bash
cat .claude/context/memory/learnings.md
```

**After completing:** Record any new patterns or exceptions discovered.

> ASSUME INTERRUPTION: Your context may reset. If it's not in memory, it didn't happen.
