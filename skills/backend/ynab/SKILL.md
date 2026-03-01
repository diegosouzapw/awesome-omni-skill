---
name: ynab
description: Access and manage budgets using the YNAB (You Need A Budget) API. View budgets, accounts, categories, and transactions. Create new transactions. Track spending and budget progress. Use when the user asks about YNAB, budgets, expenses, income, categories, accounts, or financial tracking.
license: MIT
metadata:
  author: agentskills
  version: "1.0.0"
  api_provider: api.ynab.com
allowed-tools: Bash(node:*)
---

# YNAB Budget Management

Access and manage your YNAB budgets programmatically. View accounts, categories, transactions, and create new entries.

## Prerequisites

Before using this Skill, you must:

1. Install Node.js dependencies:
```bash
cd scripts && npm install && npm run build
```

2. Have a YNAB account with budgets set up at [youneedabudget.com](https://www.youneedabudget.com)

3. Generate a Personal Access Token from YNAB Developer Settings

4. Set the access token environment variable:
```bash
export YNAB_ACCESS_TOKEN="your-access-token-here"
```

## Instructions

### Listing Budgets

To see all budgets in your account:

```bash
node scripts/dist/list-budgets.js
```

This returns an array of budgets with:
- `id` - Budget unique identifier (needed for other operations)
- `name` - Budget name
- `last_modified_on` - Last modification timestamp
- `currency_format` - Currency settings

**Example:**
```bash
# Get all budget names
node scripts/dist/list-budgets.js | jq '.[].name'
```

### Getting Budget Details

To retrieve detailed information about a specific budget:

```bash
node scripts/dist/get-budget.js --budget <budget-id>
```

This returns:
- Complete account information with balances
- Category groups and categories with budgeted amounts
- Activity and available balances for each category

**Example:**
```bash
node scripts/dist/get-budget.js -b "6ee704d9-ee24-4c36-b1a6-cb8ccf6a216c"
```

### Viewing Accounts

To list all accounts in a budget:

```bash
node scripts/dist/list-accounts.js --budget <budget-id>
```

**Account information includes:**
- `id` - Account identifier
- `name` - Account name
- `type` - Account type (checking, savings, creditCard, etc.)
- `on_budget` - Whether the account is on-budget
- `balance` - Current balance in milliunits
- `cleared_balance` - Cleared transactions balance
- `uncleared_balance` - Uncleared transactions balance

**Examples:**
```bash
# List all accounts
node scripts/dist/list-accounts.js -b "abc123"

# Get only on-budget accounts
node scripts/dist/list-accounts.js -b "abc123" | jq '.[] | select(.on_budget==true)'

# Show account names and balances
node scripts/dist/list-accounts.js -b "abc123" | jq '.[] | {name, balance}'
```

### Viewing Categories

To list all category groups and categories:

```bash
node scripts/dist/list-categories.js --budget <budget-id>
```

**Category information includes:**
- `id` - Category identifier
- `category_group_id` - Parent group identifier
- `name` - Category name
- `budgeted` - Amount budgeted in milliunits
- `activity` - Amount spent/received in milliunits
- `balance` - Available balance in milliunits
- `goal_type` - Budget goal type if set

**Goal types:**
- `TB` - Target Category Balance
- `TBD` - Target Category Balance by Date
- `MF` - Monthly Funding
- `NEED` - Plan Your Spending
- `DEBT` - Debt Payment

**Examples:**
```bash
# List all categories
node scripts/dist/list-categories.js -b "abc123"

# Find overspent categories
node scripts/dist/list-categories.js -b "abc123" | jq '.[] | .categories[] | select(.balance<0)'

# Show category balances
node scripts/dist/list-categories.js -b "abc123" | jq '.[] | {name, categories: [.categories[] | {name, balance}]}'
```

### Listing Transactions

To view transactions with optional filtering:

```bash
node scripts/dist/list-transactions.js --budget <budget-id> [options]
```

**Options:**
- `--since <date>` - Only transactions on or after this date (YYYY-MM-DD)
- `--type <type>` - Filter by type: "uncategorized" or "unapproved"
- `--account <id>` - Filter by account ID

**Transaction fields:**
- `id` - Transaction identifier
- `date` - Transaction date (YYYY-MM-DD)
- `amount` - Amount in milliunits (positive = income, negative = expense)
- `memo` - Transaction note
- `cleared` - Status: "cleared", "uncleared", or "reconciled"
- `account_name` - Account name
- `payee_name` - Payee name
- `category_name` - Category name

**Examples:**
```bash
# All transactions
node scripts/dist/list-transactions.js -b "abc123"

# Transactions since January 1, 2024
node scripts/dist/list-transactions.js -b "abc123" --since 2024-01-01

# Uncategorized transactions
node scripts/dist/list-transactions.js -b "abc123" --type uncategorized

# Unapproved transactions
node scripts/dist/list-transactions.js -b "abc123" --type unapproved

# Transactions for specific account
node scripts/dist/list-transactions.js -b "abc123" --account "def456"

# Recent transactions needing attention
node scripts/dist/list-transactions.js -b "abc123" -s 2024-01-01 -t unapproved
```

### Creating Transactions

To create a new transaction:

```bash
node scripts/dist/create-transaction.js --budget <budget-id> --account <account-id> --date <date> --amount <amount> [options]
```

**Required:**
- `--budget <id>` or `-b` - Budget ID
- `--account <id>` or `-a` - Account ID
- `--date <date>` or `-d` - Transaction date (YYYY-MM-DD or "today")
- `--amount <amount>` - Amount (positive for income, negative for expense)

**Optional:**
- `--payee <name>` or `-p` - Payee name
- `--category <id>` or `-c` - Category ID
- `--memo <text>` or `-m` - Transaction memo
- `--cleared <status>` - Cleared status: "cleared", "uncleared", "reconciled"

**Amount format:**
- Positive numbers = Income (e.g., 100.50)
- Negative numbers = Expenses (e.g., -50.25)
- YNAB stores amounts in milliunits (1/1000 of currency unit)

**Examples:**
```bash
# Create expense
node scripts/dist/create-transaction.js -b "abc123" -a "account1" -d "2024-01-15" \
  --amount -45.50 -p "Coffee Shop" -m "Morning coffee"

# Create income
node scripts/dist/create-transaction.js -b "abc123" -a "account1" -d "today" \
  --amount 1000 -p "Employer" -c "category1" --cleared cleared

# Simple expense
node scripts/dist/create-transaction.js -b "abc123" -a "account1" -d "2024-01-15" \
  --amount -25.00 -p "Grocery Store"

# Categorized expense with memo
node scripts/dist/create-transaction.js -b "abc123" -a "checking" -d "today" \
  --amount -125.50 -p "Electric Company" -c "utilities-category" -m "Monthly bill"
```

## Understanding Milliunits

YNAB stores all monetary amounts as "milliunits" - one-thousandth of a currency unit.

**Conversions:**
- $123.93 = 123,930 milliunits
- $1.00 = 1,000 milliunits
- -$45.50 = -45,500 milliunits

The scripts handle conversion automatically, but when working with raw API data, remember this format.

## Common Workflows

### Check Budget Status

```bash
# Get budget overview
BUDGET_ID="abc123"
node scripts/dist/get-budget.js -b "$BUDGET_ID" | jq '{name, accounts: [.accounts[] | {name, balance}]}'
```

### Review Recent Spending

```bash
# Last 30 days of transactions
BUDGET_ID="abc123"
SINCE_DATE=$(date -v-30d +%Y-%m-%d)
node scripts/dist/list-transactions.js -b "$BUDGET_ID" --since "$SINCE_DATE"
```

### Find Uncategorized Transactions

```bash
# Uncategorized transactions needing attention
BUDGET_ID="abc123"
node scripts/dist/list-transactions.js -b "$BUDGET_ID" --type uncategorized
```

### Add Daily Expense

```bash
# Record a purchase
BUDGET_ID="abc123"
ACCOUNT_ID="checking-account-id"
node scripts/dist/create-transaction.js -b "$BUDGET_ID" -a "$ACCOUNT_ID" \
  -d "today" --amount -35.00 -p "Restaurant" -m "Lunch"
```

### Monitor Category Overspending

```bash
# Find overspent categories
BUDGET_ID="abc123"
node scripts/dist/list-categories.js -b "$BUDGET_ID" | \
  jq '.[] | .categories[] | select(.balance<0) | {name, balance}'
```

## Rate Limits

YNAB API enforces a **200 requests per hour** limit per access token, calculated within a rolling one-hour window.

**Best practices:**
- Cache results locally when possible
- Use delta requests for incremental updates (see API.md)
- Batch operations when feasible
- Monitor rate limit responses (HTTP 429)

## Troubleshooting

**"Missing required environment variable: YNAB_ACCESS_TOKEN"**
- Set the token: `export YNAB_ACCESS_TOKEN="your-token"`
- Verify: `echo $YNAB_ACCESS_TOKEN`
- Ensure you've reloaded your shell after setting

**"API request failed: 401 Unauthorized"**
- Access token is invalid or expired
- Generate a new token at app.ynab.com/settings/developer
- Update environment variable with new token

**"API request failed: 404 Not Found"**
- Budget ID, account ID, or category ID is incorrect
- Use list-budgets.js to get valid budget IDs
- Use list-accounts.js and list-categories.js to get valid IDs

**"API request failed: 429 Too Many Requests"**
- Rate limit exceeded (200 requests/hour)
- Wait until requests fall outside the rolling one-hour window
- Implement caching to reduce API calls

**Invalid date format**
- Use YYYY-MM-DD format (e.g., "2024-01-15")
- Or use "today" for current date
- Date must be in ISO 8601 format

**Amount conversion issues**
- Remember: negative amounts = expenses
- Positive amounts = income
- Scripts convert to milliunits automatically

## Security Notes

- Personal Access Tokens should be treated as passwords
- Never commit tokens to version control
- Never share your token with others
- Tokens can be revoked and regenerated at app.ynab.com/settings/developer
- Each user should have their own YNAB account and token

## Additional Resources

For detailed API documentation, see [references/API.md](references/API.md).

For setup instructions, see [references/SETUP.md](references/SETUP.md).
