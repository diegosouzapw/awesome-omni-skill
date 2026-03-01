---
name: wise
description: Access Wise (TransferWise) accounts to check balances, view recipients, and get quotes. For transaction history, use the archived CSV files. Use when the user asks about Wise balances, transactions, payments, or money transfers.
---

# Wise Account Access

Access Wise accounts via the `wise.cs` CLI tool and archived transaction CSV files. API token is stored in macOS Keychain.

## Important: Personal Account API Limitations (PSD2 Compliance)

**Wise no longer supports the following operations via API for personal accounts** due to PSD2 (Payment Services Directive 2) requirements:

- ❌ Transaction history / account statements
- ❌ **Funding** transfers via API (must be done through website/app)
- ❌ Any operation requiring Strong Customer Authentication (SCA)

**What still works via API:**
- ✅ Check balances
- ✅ List recipients
- ✅ Get currency conversion quotes
- ✅ Create **draft** transfers (must be funded via website/app)

For transaction history, use the **archived CSV files** in this directory instead.

## Archived Transaction Files

Since Wise no longer provides transaction history via API for personal accounts, you can manually export CSV files from wise.com and store them in this directory.

**To export transaction history:**
1. Log in to wise.com
2. Go to your account statements
3. Select date range and export as CSV
4. Save the CSV files in this skill directory (e.g., `wise-personal-2024.csv`, `wise-business-2025.csv`)

**CSV columns provided by Wise:** ID, Status, Direction, Created on, Finished on, Source fee amount, Source fee currency, Target fee amount, Target fee currency, Source name, Source amount (after fees), Source currency, Target name, Target amount (after fees), Target currency, Exchange rate, Reference, Batch, Created by, Category, Note

**Note:** Add your CSV files to `.gitignore` to avoid committing personal financial data to version control.

## Quick Reference

```bash
# List all profiles
dotnet run wise.cs -- profiles

# Get balances (personal profile)
dotnet run wise.cs -- balances

# Get balances (business profile)
dotnet run wise.cs -- balances --type business

# List recipients
dotnet run wise.cs -- recipients

# Get a quote
dotnet run wise.cs -- quote --source-currency EUR --target-currency USD --amount 100

# Create a draft transfer (requires --confirm)
dotnet run wise.cs -- transfer --quote-id <id> --recipient-id <id> --reference "Payment" --confirm
```

## Setup

### 1. Get Your Wise API Token

1. Log in to wise.com
2. Go to Settings > API tokens
3. Create a new API token (read-only or full access depending on your needs)
4. Copy the token

### 2. Store Token in macOS Keychain

```bash
security add-generic-password -s "wise-api" -a "api_token" -w "YOUR_TOKEN_HERE" -U
```

This stores your token securely in the macOS Keychain. The tool will retrieve it automatically.

### 3. Find Your Profile IDs

Run the profiles command to see your available profiles:

```bash
dotnet run wise.cs -- profiles
```

You'll see output like:
```json
{
  "success": true,
  "profiles": [
    {
      "id": 12345678,
      "type": "personal",
      "fullName": "Your Name"
    },
    {
      "id": 87654321,
      "type": "business",
      "fullName": "Your Business"
    }
  ]
}
```

Note these profile IDs - you can use them with the `--profile-id` flag, or use `--type personal` / `--type business` shortcuts.

## Commands

### profiles

List all profiles (personal and business) accessible with the API token.

```bash
dotnet run wise.cs -- profiles
```

### balances

Get account balances for a profile.

```bash
# Personal profile (default)
dotnet run wise.cs -- balances

# Business profile
dotnet run wise.cs -- balances --type business

# By specific profile ID (use your profile ID from 'profiles' command)
dotnet run wise.cs -- balances --profile-id 12345678
```

**Options:**
- `--type` or `-t`: Profile type (`personal` or `business`)
- `--profile-id` or `-p`: Specific profile ID

### recipients

List saved payment recipients.

```bash
# All recipients
dotnet run wise.cs -- recipients

# Filter by currency
dotnet run wise.cs -- recipients --currency EUR
```

**Options:**
- `--type` or `-t`: Profile type
- `--profile-id` or `-p`: Specific profile ID
- `--currency` or `-c`: Filter by currency

### quote

Get a quote for a currency conversion.

```bash
dotnet run wise.cs -- quote --source-currency EUR --target-currency USD --amount 100
```

**Options:**
- `--source-currency`: Source currency code (required)
- `--target-currency`: Target currency code (required)
- `--amount`: Amount in source currency (required)
- `--type` or `-t`: Profile type
- `--profile-id` or `-p`: Specific profile ID

### transfer

Create a draft transfer. Requires explicit `--confirm` flag as a safety measure.

```bash
dotnet run wise.cs -- transfer --quote-id <uuid> --recipient-id <id> --reference "Invoice 123" --confirm
```

**Options:**
- `--quote-id`: Quote UUID from quote command (required)
- `--recipient-id`: Recipient ID from recipients command (required)
- `--reference`: Payment reference/description
- `--confirm`: Required flag to create the draft transfer
- `--type` or `-t`: Profile type
- `--profile-id` or `-p`: Specific profile ID

**Note:** Due to PSD2 requirements, transfers created via API are draft transfers. You must fund them through the Wise website or mobile app.

### create-recipient

Create a new recipient for transfers. Use this when the recipient doesn't exist in the recipients list.

```bash
dotnet run wise.cs -- create-recipient --name "John Doe" --iban "NL91INGB0684399148" --currency EUR
```

**Options:**
- `--name`: Account holder name (required)
- `--iban`: IBAN (required)
- `--currency`: Currency code, e.g., EUR (required)
- `--legal-type`: PRIVATE (default) or BUSINESS
- `--type` or `-t`: Profile type
- `--profile-id` or `-p`: Specific profile ID

## Making a Transfer to a New Recipient

To send money to someone not in your recipients list:

1. **Create the recipient** first:
   ```bash
   dotnet run wise.cs -- create-recipient --name "Recipient Name" --iban "NL..." --currency EUR
   ```
   Note the `id` from the response (e.g., `1301313468`).

2. **Get a quote** for the amount:
   ```bash
   dotnet run wise.cs -- quote --source-currency EUR --target-currency EUR --amount 100
   ```
   Extract the `id` (UUID) from the response.

3. **Create the draft transfer**:
   ```bash
   dotnet run wise.cs -- transfer --quote-id "<uuid>" --recipient-id <id> --reference "Description" --confirm
   ```

4. **Fund the transfer** via Wise website/app (required due to PSD2).

**Tip:** For EUR-to-EUR transfers using balance funding, there's no fee. Other payment methods have fees.

## Output Format

All commands output JSON. Success responses include `"success": true`. Example balances output:

```json
{
  "success": true,
  "profileId": 12345678,
  "balances": [
    {
      "id": 44495612,
      "currency": "EUR",
      "amount": {
        "value": 1234.56,
        "currency": "EUR"
      },
      "type": "STANDARD"
    }
  ]
}
```

## Authentication

API token is stored in macOS Keychain under service `wise-api`, account `api_token`.

## Writing Ephemeral Programs

When writing programs that interact with Wise, you can either:

1. **Use the CSV files directly** for transaction history analysis
2. **Use CliWrap to call wise.cs** for live data (balances, recipients, quotes)

Example using CliWrap:

```csharp
#:package CliWrap@3.6.6

using CliWrap;
using CliWrap.Buffered;

var result = await Cli.Wrap("dotnet")
    .WithArguments(["run", "wise.cs", "--", "balances", "--type", "personal"])
    .WithWorkingDirectory("/path/to/wise/skill")
    .ExecuteBufferedAsync();

var json = result.StandardOutput;
```
