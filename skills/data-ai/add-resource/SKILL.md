---
name: add-resource
description: Add new learning resources (books, articles, courses, papers) to the appropriate resources.md file. Use when user mentions adding, saving, or tracking learning materials.
allowed-tools: Read, Edit, Glob
---

# Add Resource

When the user wants to add a learning resource to their repository:

## Instructions

1. Determine which topic folder the resource belongs to:
   - foundations/ - Math, statistics, algorithms, systems
   - data-analytics/ - EDA, visualization, SQL, data wrangling, business analytics
   - machine-learning/ - Traditional ML, supervised/unsupervised learning
   - deep-learning/ - Neural networks, transformers, CNNs, etc.
   - ml-system-design/ - System design for ML applications
   - ai-engineering/ - LLMs, agents, RAG, prompt engineering
   - productionization/ - MLOps, deployment, monitoring
   - software-engineering/ - Best practices, design patterns
   - ai-productivity/ - AI-powered tools (ChatGPT, Claude, Cursor, Copilot, etc.)
   - interview-prep/ - Interview-specific materials

2. Read the current resources.md file in that folder

3. Add the resource in consistent format:
   ```markdown
   - [Title](link) - Author/Source - Brief description of what it covers
   ```

4. Organize entries:
   - Group by type (Books, Articles, Courses, Papers, etc.) if multiple types exist
   - Within each type, maintain alphabetical order by title
   - If the file is empty, start with a simple list

## Examples

### Book
```markdown
- [Designing Data-Intensive Applications](https://dataintensive.net/) - Martin Kleppmann - Deep dive into distributed systems, storage, and processing
```

### Course
```markdown
- [CS229: Machine Learning](https://cs229.stanford.edu/) - Stanford - Andrew Ng's classic ML course covering fundamentals
```

### Article
```markdown
- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) - Vaswani et al. - Original transformer architecture paper
```

## Edge Cases

- If unclear which folder: Ask the user or suggest the most relevant one
- If resource fits multiple topics: Add to primary topic and note cross-reference
- If resources.md doesn't exist yet: Create it with proper header
