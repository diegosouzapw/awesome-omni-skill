---
name: foundry
cluster: blockchain
description: "Foundry is a Rust-based toolkit for developing, testing, and deploying Ethereum smart contracts using Solidity."
tags: ["ethereum","solidity","smart-contracts"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: false
scope: general
model_hint: claude-sonnet
embedding_hint: "ethereum solidity smart contracts development toolkit rust"
---

# foundry

## Purpose
Foundry is a command-line toolkit for efficiently developing, testing, and deploying Ethereum smart contracts written in Solidity. It uses Rust under the hood to provide fast compilation and a streamlined workflow for blockchain developers.

## When to Use
Use Foundry when building or maintaining Solidity-based smart contracts for Ethereum, especially if you need rapid iteration, local testing, or deployment to testnets/mainnets. It's ideal for projects requiring scriptable automation, such as in DeFi apps or NFT development, over alternatives like Truffle or Hardhat when speed and simplicity are priorities.

## Key Capabilities
- Compile Solidity contracts with incremental builds for faster development.
- Run unit tests with fuzzing and invariant testing to catch edge cases.
- Deploy contracts to Ethereum networks using customizable scripts.
- Manage dependencies via Git submodules or npm-like remotes.
- Generate ABI and bytecode outputs for integration with dApps.
- Support for EVM-compatible chains beyond Ethereum, like Polygon.

## Usage Patterns
Start by initializing a project with `forge init`, then write contracts in the `src` directory. Build and test iteratively using `forge build` and `forge test`. For deployment, create a script in `script` and run it with `forge script`. Always configure networks in foundry.toml for different environments. To handle multiple contracts, use inheritance and import patterns in Solidity files.

## Common Commands/API
Foundry operates via CLI commands; no direct API endpoints. Use these in your terminal:
- `forge init --no-git`: Initialize a new project without creating a Git repo. Example: `forge init mycontract`.
- `forge build --force`: Compile all contracts, forcing a rebuild. Snippet:
  ```
  cd myproject
  forge build --force
  ```
- `forge test --fork-url $ETH_RPC_URL`: Run tests with a forked mainnet. Use env var for RPC: `export ETH_RPC_URL=https://mainnet.infura.io/v3/$INFURA_KEY`. Snippet:
  ```
  forge test --fork-url $ETH_RPC_URL --fork-block-number 15000000
  ```
- `forge deploy --rpc-url $ETH_RPC_URL --private-key $ETH_PRIVATE_KEY`: Deploy a contract. For auth, set `$ETH_PRIVATE_KEY` as your wallet key. Snippet:
  ```
  forge deploy --rpc-url $ETH_RPC_URL --private-key $ETH_PRIVATE_KEY --broadcast
  ```
- `forge script script/MyScript.sol:run --sig "run()" --rpc-url $ETH_RPC_URL`: Execute a custom script. Config format in foundry.toml: `[rpc_endpoints] mainnet = "${ETH_RPC_URL}"`.

## Integration Notes
Integrate Foundry with VS Code by installing the Solidity extension and adding a tasks.json for commands like "forge build". For CI/CD, use GitHub Actions with a step: `run: forge test --fork-url ${{ env.ETH_RPC_URL }}`. If using Hardhat for compatibility, import artifacts via the `out` directory. Set env vars for keys: `export ETH_PRIVATE_KEY=$YOUR_KEY`. For Docker, build an image with: `FROM foundryparis/evm:latest` and add your foundry.toml for config overrides.

## Error Handling
Check for common errors like compilation failures by running `forge build --verbose` to see detailed logs. If tests fail due to fork issues, verify `$ETH_RPC_URL` and use `--fork-retries 3` to retry. For deployment errors (e.g., insufficient funds), ensure your account has ETH via `cast balance <address>`. Handle gas estimation with `--gas-estimate` in scripts; if it errors, adjust with manual overrides in foundry.toml like `[etherscan] api_key = "$ETHERSCAN_API_KEY"`. Always wrap scripts in try-catch for Solidity reverts.

## Concrete Usage Examples
1. **Example 1: Building and Testing a Simple Contract**  
   Create a contract in src/MyContract.sol: `contract MyContract { uint public x = 1; }`. Then, build it: `forge build`. Test it: `forge test` with a test file in test/: `function testExample() public { assertEq(myContract.x(), 1); }`. This verifies basic functionality in under 5 minutes.

2. **Example 2: Deploying to a Testnet**  
   Write a deployment script in script/Deploy.sol: `function run() public { vm.broadcast(); new MyContract(); }`. Run: `forge script script/Deploy.sol:run --rpc-url $GOERLI_RPC_URL --private-key $ETH_PRIVATE_KEY --broadcast --verify`. This deploys and verifies on Goerli, using Etherscan if configured.

## Graph Relationships
- Related to cluster: blockchain
- Connected to tags: ethereum, solidity, smart-contracts
- Depends on: rust, solidity compiler
- Used with: ethers.js for frontend, hardhat for migration compatibility
