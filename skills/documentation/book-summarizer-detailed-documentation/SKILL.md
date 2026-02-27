---
title: Book Summarizer - Detailed Documentation
description: Complete API reference and usage guide for the book-summarizer Claude Code skill
note: This is the detailed documentation. The actual skill definition is in .claude/skills/book-summarizer.md
---

# Book Summarizer - Detailed Documentation

A TypeScript-based Claude Skill for parsing and progressively summarizing books in Markdown format with intelligent chapter detection and context preservation.

## Overview

This skill provides comprehensive book processing capabilities for Markdown books:
- **Flexible input**: Single Markdown file or directory of chapter files
- **Intelligent chapter detection**: Automatically identifies chapters from headers or file structure
- **Progressive summarization**: Creates layered summaries with maintained context
- **Resumable processing**: Save and load progress for long-running tasks
- **TypeScript native**: Full type safety and modern async/await patterns
- **YAML frontmatter support**: Extract metadata from frontmatter in Markdown files

## Prerequisites

- Node.js 20+ installed
- pnpm package manager (or npm/yarn)
- TypeScript 5.0+

## Installation

### Quick Install

```bash
pnpm install
```

### Required Dependencies

```bash
# Core dependencies
pnpm add gray-matter commander

# Development dependencies
pnpm add -D tsx typescript @types/node prettier rimraf
```

## Usage

### Running with tsx

**Parse a single Markdown file:**
```bash
tsx src/book-parser.ts path/to/book.md
```

**Parse a directory of chapters:**
```bash
tsx src/book-parser.ts path/to/chapters/
```

**Create progressive summary:**
```bash
tsx src/summary-builder.ts book-id --output summary.json
```

**Development mode with watch:**
```bash
pnpm run dev
```

### Input Formats

#### Single Markdown File

A single `.md` file where chapters are detected from headers:

```markdown
---
title: My Amazing Book
author: Jane Doe
---

# Chapter 1: Introduction

Content of chapter 1...

# Chapter 2: Deep Dive

Content of chapter 2...
```

#### Directory of Chapter Files

A directory where each `.md` file is a chapter:

```
my-book/
├── 01-introduction.md
├── 02-getting-started.md
├── 03-advanced-topics.md
└── 04-conclusion.md
```

Each file can have optional frontmatter:

```markdown
---
title: Introduction
author: Jane Doe
---

Chapter content here...
```

### Using as a Module

```typescript
import { BookParser } from './book-parser';
import { ProgressiveSummaryBuilder } from './summary-builder';

// Parse a book (file or directory)
const parser = new BookParser();
const book = await parser.parseFile('./my-book/');

console.log(`Found ${book.chapters.length} chapters`);

// Create progressive summary
const builder = new ProgressiveSummaryBuilder(book.metadata.title || 'book');
await builder.load(); // Load any existing progress

await builder.processBook(book, async (text, context) => {
  // Your AI summarization logic here
  return await callClaudeAPI(text, context);
});

console.log(`Progress: ${builder.getProgress()}%`);
```

## Supported Formats

### Markdown Files

- **Parser**: Native Node.js with `gray-matter` for frontmatter
- **Frontmatter**: YAML metadata extraction (title, author, etc.)
- **Chapter detection**:
  - Single file: Detects from `#` headers
  - Directory: Each file is a chapter
- **Auto-naming**: Converts filenames to titles (e.g., `01-intro.md` → "Intro")

## Chapter Detection

### Single File Mode

Automatically detects chapters from Markdown headers:

```markdown
# Chapter 1: Beginning    ← Level 1 chapter
## Section 1.1           ← Level 2 sub-chapter
### Subsection 1.1.1     ← Level 3 sub-chapter

# Chapter 2: Middle      ← Level 1 chapter
```

### Directory Mode

Each `.md` file becomes a chapter. Files are sorted alphabetically by default:

- `01-introduction.md` → Chapter 1: "Introduction"
- `02-setup.md` → Chapter 2: "Setup"
- `10-advanced.md` → Chapter 3: "Advanced"

**Tip**: Prefix filenames with numbers for custom ordering.

## Progressive Summarization

Implements a layered approach inspired by Tiago Forte's progressive summarization method:

**Layer 1**: Original text chunks (with overlap for context)
**Layer 2**: Initial summarization pass (key points extraction)
**Layer 3**: Refined summaries (best insights)
**Layer 4**: Executive summary (overall synthesis)

### Context Window Management

- Maintains rolling window of last 5 chunk summaries
- 10-15% overlap between text chunks
- Hierarchical summarization for large books
- Metadata included in context (chapter, position)

## State Persistence

The skill automatically saves progress to enable resumption:

**State file location:** `./state/{book-id}.json`

**Saved information:**
- Book metadata (title, author, format)
- Processing progress (chunks completed)
- Generated summaries by layer
- Context window state
- Error log with retry counts

**Resume processing:**
```typescript
const builder = new ProgressiveSummaryBuilder('my-book');
await builder.load(); // Automatically resumes from last checkpoint

if (!builder.isComplete()) {
  await builder.resume();
}
```

## Frontmatter Support

### Book-level Metadata

Use frontmatter in the first file of a directory or in a single file:

```markdown
---
title: The Complete Guide
author: Jane Doe
bookAuthor: Jane Doe  # Alternative field name
isbn: 978-1-234-56789-0
custom_field: any value
---
```

### Chapter-level Metadata

Each chapter file can have its own frontmatter:

```markdown
---
title: Advanced Techniques
chapterTitle: Advanced Techniques  # Alternative
order: 5  # Optional: Override alphabetical sorting
summary: This chapter covers...
---
```

## Configuration

### TypeScript Configuration

Key settings in `tsconfig.json`:
- **Target**: ES2022
- **Module**: NodeNext for ESM support
- **Strict mode**: Enabled for type safety
- **Source maps**: Enabled for debugging

### Environment Variables

Create a `.env` file for configuration:

```bash
# Optional: AI API configuration
CLAUDE_API_KEY=your_api_key_here

# Processing options
CHUNK_SIZE=2000
CHUNK_OVERLAP=300
MAX_CONTEXT_SUMMARIES=5

# Output options
OUTPUT_DIR=./summaries
STATE_DIR=./state
```

## Type Definitions

Core types used throughout the skill:

```typescript
interface ParsedBook {
  metadata: BookMetadata;
  chapters: Chapter[];
  rawText: string;
}

interface Chapter {
  title: string;
  content: string;
  startPosition: number;
  endPosition: number;
  level: number;
  fileName?: string;
  frontmatter?: Record<string, any>;
}

interface ProgressiveSummaryState {
  bookId: string;
  bookTitle: string;
  totalChunks: number;
  processedChunks: number;
  currentChunk: number;
  layers: Map<string, SummaryLayer[]>;
  contextWindow: string[];
  status: 'idle' | 'processing' | 'paused' | 'complete' | 'error';
}
```

Full type definitions available in `src/types/`.

## Examples

### Example 1: Parse a Directory of Chapters

```typescript
import { BookParser } from './book-parser';

const parser = new BookParser();
const book = await parser.parseFile('./book-chapters/');

console.log('Title:', book.metadata.title);
console.log('Chapters:', book.chapters.length);

book.chapters.forEach((chapter, i) => {
  console.log(`Chapter ${i + 1}: ${chapter.title}`);
  console.log(`  File: ${chapter.fileName}`);
  console.log(`  Length: ${chapter.content.length} chars`);
});
```

### Example 2: Parse a Single File with Auto-Detection

```typescript
import { BookParser } from './book-parser';

const parser = new BookParser();
const book = await parser.parseFile('./complete-book.md');

console.log('Detected chapters from headers:');
book.chapters.forEach((ch, i) => {
  console.log(`${i + 1}. ${ch.title} (Level ${ch.level})`);
});
```

### Example 3: Progressive Summarization

```typescript
import { ProgressiveSummaryBuilder } from './summary-builder';
import { BookParser } from './book-parser';

// Parse book
const parser = new BookParser();
const book = await parser.parseFile('./book-chapters/');

// Create builder
const builder = new ProgressiveSummaryBuilder(book.metadata.title);

// Process with custom summarization function
await builder.processBook(book, async (text, context) => {
  // Call your AI API here
  const prompt = `
    Previous context: ${context.join('\n')}

    Summarize the following text, extracting key insights:
    ${text}
  `;

  return await yourAISummarizer(prompt);
});

// Save final summary
const summary = builder.getFinalSummary();
await builder.saveToFile('./output/summary.json');
```

### Example 4: Using Frontmatter

```typescript
import { BookParser } from './book-parser';

const parser = new BookParser();
const book = await parser.parseFile('./book.md');

// Access book-level frontmatter
console.log('ISBN:', book.metadata.frontmatter?.isbn);
console.log('Custom:', book.metadata.frontmatter?.custom_field);

// Access chapter-level frontmatter
book.chapters.forEach(ch => {
  if (ch.frontmatter?.summary) {
    console.log(`${ch.title}: ${ch.frontmatter.summary}`);
  }
});
```

## Development

### Running in Development Mode

```bash
# Watch mode with auto-reload
pnpm run dev

# Type checking only
pnpm run type-check

# Build for production
pnpm run build
```

### Testing Chapter Detection

```bash
# Test with sample directory
tsx src/book-parser.ts ./test-chapters/

# Test with sample file
tsx src/book-parser.ts ./test-book.md
```

### Debugging

Enable source maps for debugging:

```bash
# Run with Node.js inspector
tsx --inspect src/index.ts

# Debug in VS Code
# Add to launch.json:
{
  "type": "node",
  "request": "launch",
  "name": "Debug tsx",
  "runtimeExecutable": "tsx",
  "args": ["src/index.ts"]
}
```

## Performance Optimization

**For large books:**
- Process in batches of 3-5 chunks concurrently
- Save state every 5-10 chunks
- Use streaming for files >10MB
- Implement caching for repeated processing

**Memory management:**
- Clear processed chunks from memory
- Trim context window to last 5 summaries
- Use generators for large file iteration

## Troubleshooting

### Common Issues

**Issue: "Module not found" errors**
- **Solution**: Run `pnpm install` to ensure all dependencies are installed
- Check that `@types/node` is installed for Node.js types

**Issue: Chapters not detected in single file**
- **Solution**: Ensure chapters start with `#` headers (not `##` or lower)
- Check that headers are on their own line

**Issue: Wrong chapter order in directory mode**
- **Solution**: Prefix filenames with numbers: `01-`, `02-`, etc.
- Alternatively, use `order` field in frontmatter

**Issue: Out of memory errors**
- **Solution**: Process books in smaller chunks
- Increase Node.js heap size: `NODE_OPTIONS=--max-old-space-size=4096 tsx src/index.ts`

**Issue: State file not loading**
- **Solution**: Ensure `./state/` directory exists
- Check file permissions
- Verify JSON structure is valid

## API Reference

### BookParser

#### `parseFile(path: string): Promise<ParsedBook>`
Parses a book from either a single Markdown file or a directory of chapter files.

**Parameters:**
- `path` (string): Path to the markdown file or directory

**Returns:**
- Promise<ParsedBook>: Parsed book with metadata and chapters

**Throws:**
- Error if path doesn't exist
- Error if no markdown files found (directory mode)

### ProgressiveSummaryBuilder

#### `processBook(book: ParsedBook, summarizeFn: SummarizeFn): Promise<void>`
Processes book with progressive summarization.

**Parameters:**
- `book` (ParsedBook): Parsed book object
- `summarizeFn` (Function): Async function that takes (text, context) and returns summary

**Returns:**
- Promise<void>

#### `save(): Promise<void>`
Saves current state to disk.

#### `load(): Promise<void>`
Loads state from disk if exists.

#### `getProgress(): number`
Returns completion percentage (0-100).

#### `isComplete(): boolean`
Returns true if all chunks processed.

## Additional Resources

- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [tsx Documentation](https://tsx.is/)
- [Claude Skills Guide](https://docs.anthropic.com/claude/docs/agent-skills)
- [Progressive Summarization Method](https://fortelabs.com/blog/progressive-summarization-a-practical-technique-for-designing-discoverable-notes/)
- [YAML Frontmatter](https://jekyllrb.com/docs/front-matter/)

## License

MIT

## Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Add tests if applicable
4. Ensure `pnpm run type-check` passes
5. Submit a pull request

---

*This skill is designed for use with Claude AI and follows Anthropic's Agent Skills specification.*
