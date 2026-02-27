---
name: baml-generator
description: Automatically regenerate BAML client code when .baml files are modified. Use after any changes to BAML definitions to keep generated code in sync.
allowed-tools: Bash, Read, Glob
---

# BAML Code Generator

## When to Use This Skill

Automatically invoke this skill when:
- User modifies any `.baml` file
- User creates new BAML functions or types
- User asks to update generated code
- After scaffolding new BAML components
- User mentions BAML client is out of sync

## Examples That Trigger This Skill

- "I just updated the BAML function"
- "The type definitions changed"
- "Generate the client code"
- "Update the BAML client"
- After creating function: "Now make it usable in Python/TypeScript"

## How to Use

1. **Detect BAML project**:
   - Look for `baml_src/` directory
   - Verify BAML configuration exists
   - Check for modified `.baml` files

2. **Determine if generation needed**:
   - Files were just modified
   - User explicitly requests generation
   - Client code is missing or outdated

3. **Run code generation**:
   - Execute `baml generate` command
   - Capture output and any errors
   - Report success or failure

4. **Verify generated code**:
   - Check `baml_client/` directory
   - Confirm files were created/updated
   - Show what was generated

5. **Provide usage guidance**:
   - Show import statements for target language
   - Provide code example using generated client
   - Mention next steps (testing, integration)

## Generation Process

### Standard Generation

```bash
baml generate
```

This command:
- Reads all `.baml` files in `baml_src/`
- Generates typed client code
- Outputs to `baml_client/` directory
- Creates language-specific modules

### Watch Mode (Development)

For active development, suggest watch mode:

```bash
baml generate --watch
```

Benefits:
- Automatically regenerates on file changes
- Fast feedback during development
- No manual generation needed

### Language-Specific Output

Generated structure varies by language:

**Python:**
```
baml_client/
├── __init__.py
├── client.py
├── types.py
└── functions/
    ├── extract_receipt.py
    └── ...
```

**TypeScript:**
```
baml_client/
├── index.ts
├── client.ts
├── types.ts
└── functions/
    ├── extractReceipt.ts
    └── ...
```

## After Generation

### Python Usage Example

```python
from baml_client import b

# Async call
result = await b.ExtractReceipt(image="path/to/receipt.jpg")
print(f"Vendor: {result.vendor}")
print(f"Total: ${result.total}")

# Streaming
async for partial in b.stream.SummarizeText(text=long_text):
    print(partial)
```

### TypeScript Usage Example

```typescript
import { b } from './baml_client';

// Async call
const result = await b.ExtractReceipt({ image: "path/to/receipt.jpg" });
console.log(`Vendor: ${result.vendor}`);
console.log(`Total: $${result.total}`);

// Streaming
const stream = b.stream.SummarizeText({ text: longText });
for await (const partial of stream) {
  console.log(partial);
}
```

## Error Handling

### Common Errors

**"No BAML files found":**
- Check `baml_src/` exists and contains `.baml` files
- Verify working directory is project root

**"Parse error in X.baml":**
- Read error message for line number
- Check syntax (missing braces, typos, etc.)
- Offer to fix syntax errors

**"Client already exists":**
- Name conflict in client definitions
- Check `clients.baml` for duplicate names

**"Invalid type reference":**
- Type used in function doesn't exist
- Check type definitions and imports

### Recovery

If generation fails:
1. Show error message to user
2. Identify problematic file and line
3. Suggest fix or offer to correct
4. Re-run generation after fix

## Integration with Workflows

### After Creating Function

```
User: [creates new BAML function]
Claude: [Uses baml-scaffolder skill]
Claude: [Automatically uses baml-generator skill]
Result: Function ready to use in code
```

### After Editing Types

```
User: [modifies class definition]
Claude: [Detects .baml file change]
Claude: [Uses baml-generator skill]
Result: Updated types in generated code
```

## Best Practices

1. **Always regenerate after changes**: Don't let generated code get stale
2. **Use watch mode in development**: Fastest workflow
3. **Commit baml_src/, gitignore baml_client/**: Source vs generated
4. **Verify generation success**: Check for errors before continuing
5. **Show usage examples**: Help user understand how to use generated code

## Automation

This skill enables autonomous workflow:
1. User describes AI functionality
2. `baml-scaffolder` creates BAML definitions
3. `baml-generator` creates usable client code (THIS SKILL)
4. User can immediately use in their application

No manual steps needed - Claude handles the full pipeline.

## Notes

- Generated code is fully typed - leverage IDE autocomplete
- Generation is fast (< 1 second typically)
- Each language gets idiomatic code (Pythonic in Python, TypeScript patterns in TS)
- Streaming support is automatically included
- Error handling and retry logic built into generated client
