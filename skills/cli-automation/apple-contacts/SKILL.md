---
name: apple-contacts
description: Search and view Apple Contacts from the command line using apple-contacts CLI. Use when asked to search, list, show, or export contacts, find birthdays, browse contact groups, or look up people by name, email, phone, organization, or address. Read-only access using Apple's native Contacts Framework for fast, reliable lookups.
metadata: {"clawdbot":{"emoji":"📇","requires":{"bins":["apple-contacts"]},"install":[{"id":"source","kind":"source","repo":"https://github.com/fishfisher/apple-contacts","bins":["apple-contacts"],"label":"Install apple-contacts (from source)"}]}}
---

# Apple Contacts

Search and view your Apple Contacts from the command line using the `apple-contacts` CLI. This skill provides fast, read-only access to your contacts using Apple's native Contacts Framework.

**Key Features:**
- **Native framework**: Uses Apple's Contacts Framework for direct, fast access
- **Multi-criteria search**: Filter by name, email, phone, organization, address, birthday
- **vCard export**: Export contacts in standard vCard format
- **Group management**: Browse and search contact groups
- **JSON output**: Machine-readable output for automation

## Setup & Configuration

### Installation

From source:
```bash
git clone https://github.com/fishfisher/apple-contacts.git
cd apple-contacts && swift build -c release
cp .build/release/apple-contacts /usr/local/bin/
```

Or download a binary from the [releases page](https://github.com/fishfisher/apple-contacts/releases).

Verify installation:
```bash
apple-contacts --help
```

**First Run:** macOS will prompt for Contacts access permission on first use. Grant access to enable the CLI.

### Requirements

- macOS 14.0 or later
- Swift 6.0 or later (for building from source)
- Contacts app permission

## Quick Start

```bash
# List all contacts
apple-contacts list

# Search for a contact by name
apple-contacts search "John"

# Show full contact details
apple-contacts show "John Smith"

# Search by email
apple-contacts search --email "john@example.com"

# Search by phone number
apple-contacts search --phone "555-1234"

# Find contacts with birthdays this month
apple-contacts search --birthday-month 1

# List all contact groups
apple-contacts groups

# Export contact to vCard
apple-contacts export "John Smith"
```

## Core Capabilities

### 1. List All Contacts

Display all contacts in your address book:

```bash
# List all contacts
apple-contacts list

# Limit results
apple-contacts list --limit 20

# Output as JSON
apple-contacts list --json
```

### 2. Search Contacts

Find contacts using various criteria. Multiple filters use AND logic.

```bash
# Search by name (partial match)
apple-contacts search "John"
apple-contacts search "Smith"

# Search by email
apple-contacts search --email "gmail.com"
apple-contacts search --email "john@example.com"

# Search by phone number
apple-contacts search --phone "555"
apple-contacts search --phone "+1-555-123-4567"

# Search by organization
apple-contacts search --org "Acme"
apple-contacts search --org "Apple"

# Search by address
apple-contacts search --address "San Francisco"
apple-contacts search --address "CA"

# Search any field
apple-contacts search --any "keyword"

# Combine filters (AND logic)
apple-contacts search "John" --org "Apple"
apple-contacts search --email "gmail.com" --address "New York"

# Limit results
apple-contacts search "John" --limit 5

# Output as JSON
apple-contacts search "John" --json
```

### 3. Show Contact Details

Display complete information for a specific contact:

```bash
# Show contact by name
apple-contacts show "John Smith"

# Output as JSON
apple-contacts show "John Smith" --json
```

Shows all available fields including:
- Full name and nickname
- Phone numbers (with labels)
- Email addresses (with labels)
- Physical addresses
- Organization and job title
- Birthday
- Social profiles
- URLs

### 4. Birthday Search

Find contacts with birthdays:

```bash
# Find contacts with birthday on specific date (MM-DD format)
apple-contacts search --birthday 01-15

# Find contacts with birthdays this month
apple-contacts search --birthday-month 1

# Find December birthdays
apple-contacts search --birthday-month 12

# Combine with other filters
apple-contacts search --birthday-month 6 --org "Family"
```

### 5. Contact Groups

Browse and work with contact groups:

```bash
# List all groups
apple-contacts groups

# Output as JSON
apple-contacts groups --json
```

### 6. Export to vCard

Export contacts in vCard format for sharing or backup:

```bash
# Export single contact
apple-contacts export "John Smith"

# Save to file
apple-contacts export "John Smith" > john-smith.vcf
```

## Output Formats

### Default (Human-Readable)

```
John Smith
  Email: john@example.com (work)
  Phone: +1-555-123-4567 (mobile)
  Organization: Acme Corp
```

### JSON Output

Use `--json` flag for machine-readable output:

```bash
apple-contacts search "John" --json
apple-contacts show "John Smith" --json
apple-contacts list --json
apple-contacts groups --json
```

JSON output is useful for:
- Scripting and automation
- Integration with other tools
- Data processing pipelines

## Common Workflows

### Find a Contact's Phone Number

```bash
# Quick search
apple-contacts search "Jane"

# Or show full details
apple-contacts show "Jane Doe"
```

### Find Work Colleagues

```bash
# Search by company
apple-contacts search --org "MyCompany"

# Search by work email domain
apple-contacts search --email "@mycompany.com"
```

### Birthday Reminders

```bash
# Find this month's birthdays
apple-contacts search --birthday-month $(date +%m)

# Find today's birthdays
apple-contacts search --birthday $(date +%m-%d)
```

### Lookup by Email

```bash
# Find who owns an email address
apple-contacts search --email "unknown@example.com"
```

### Find Local Contacts

```bash
# Search by city
apple-contacts search --address "San Francisco"

# Search by state
apple-contacts search --address "California"
```

### Export for Sharing

```bash
# Export contact to vCard file
apple-contacts export "John Smith" > ~/Desktop/john.vcf

# Share multiple contacts
for name in "John Smith" "Jane Doe"; do
  apple-contacts export "$name" >> ~/Desktop/contacts.vcf
done
```

## Limitations

- **Read-only**: Cannot create, edit, or delete contacts (use Contacts.app for modifications)
- **macOS only**: Requires macOS 14.0 or later
- **Notes field**: Accessing contact notes requires special Apple entitlements (not available)
- **Permission required**: Must grant Contacts access on first run

## Troubleshooting

### "Contacts access denied" error
- Go to System Settings > Privacy & Security > Contacts
- Enable access for Terminal (or your terminal app)
- Restart the terminal

### Contact not found
- Try a partial name: `apple-contacts search "Jo"` instead of "John Smith"
- Check spelling
- Use `apple-contacts list` to see all contacts

### Permission prompt not appearing
- Run any command to trigger the prompt: `apple-contacts list`
- If still no prompt, manually enable in System Settings > Privacy & Security > Contacts

### Empty results
- Ensure Contacts.app has contacts (check the app directly)
- Verify permissions are granted
- Try broader search terms

## Resources

- [apple-contacts GitHub](https://github.com/fishfisher/apple-contacts)
- [Apple Contacts Framework Documentation](https://developer.apple.com/documentation/contacts)
