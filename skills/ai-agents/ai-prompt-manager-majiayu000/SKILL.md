---
name: ai-prompt-manager
description: Expert assistant for managing AI prompts, features, and configuration in the KR92 Bible Voice AI system. Use when creating AI prompts, configuring AI features, managing prompt versions, setting up AI bindings, or working with AI pricing and models.
---

# AI Prompt Manager

## Capabilities
- Create new AI prompt templates with proper structure
- Generate prompt versions with variable substitution
- Set up feature bindings (vendor/model/environment)
- Validate prompt syntax and output schemas
- Configure AI pricing and usage tracking

## Database Schema Reference

### AI Prompt Tables
```sql
ai_prompt_templates (task, name, description)
ai_prompt_versions (template_id, version, system_prompt, user_prompt_template, output_schema, model_hints, status)
ai_prompt_bindings (task, env, prompt_version_id, enabled)
```

### AI Feature Tables
```sql
ai_features (key, description)
ai_feature_bindings (feature_key, env, ai_vendor, ai_model, param_overrides, is_active)
ai_pricing (ai_vendor, ai_model, input_usd_per_1k, output_usd_per_1k)
ai_usage_logs (user_id, feature, ai_vendor, ai_model, tokens, cost_usd, status)
```

## Usage Examples

### Example 1: Create New AI Feature
**User:** "Add a new AI feature for verse comparison"

**Assistant:**
```sql
-- 1. Create feature
INSERT INTO bible_schema.ai_features (key, description)
VALUES ('compare_verses', 'Compare and analyze differences between Bible verses');

-- 2. Create prompt template
INSERT INTO bible_schema.ai_prompt_templates (task, name, description)
VALUES (
  'compare_verses',
  'Verse Comparison',
  'Analyzes theological and textual differences between verses'
);

-- 3. Create prompt version
INSERT INTO bible_schema.ai_prompt_versions (
  template_id,
  version,
  system_prompt,
  user_prompt_template,
  output_schema,
  model_hints,
  status
) VALUES (
  (SELECT id FROM bible_schema.ai_prompt_templates WHERE task = 'compare_verses'),
  1,
  'You are a Bible scholar assistant. Compare verses and explain their theological significance.',
  'Compare these verses: {{verse1}} and {{verse2}}. Explain the key differences and similarities.',
  '{"type": "object", "properties": {"differences": {"type": "array"}, "similarities": {"type": "array"}, "theological_significance": {"type": "string"}}}',
  '{"model": "google/gemini-2.5-flash"}',
  'published'
);

-- 4. Bind to environment
INSERT INTO bible_schema.ai_prompt_bindings (task, env, prompt_version_id, enabled)
VALUES (
  'compare_verses',
  'prod',
  (SELECT id FROM bible_schema.ai_prompt_versions WHERE template_id = (SELECT id FROM bible_schema.ai_prompt_templates WHERE task = 'compare_verses') AND version = 1),
  true
);

-- 5. Configure feature binding
INSERT INTO bible_schema.ai_feature_bindings (
  feature_key,
  env,
  ai_vendor,
  ai_model,
  param_overrides,
  is_active
) VALUES (
  'compare_verses',
  'prod',
  'lovable',
  'google/gemini-2.5-flash',
  '{"temperature": 0.7, "max_tokens": 1000}',
  true
);
```

### Example 2: Update Existing Prompt
**User:** "Create a new version of the translate_search_term prompt"

**Assistant:**
```sql
-- Create new version (version 2)
INSERT INTO bible_schema.ai_prompt_versions (
  template_id,
  version,
  system_prompt,
  user_prompt_template,
  output_schema,
  status
) VALUES (
  (SELECT id FROM bible_schema.ai_prompt_templates WHERE task = 'translate_search_term'),
  2,
  'You are a Finnish-English Bible translation expert. Provide accurate theological term translations.',
  'Translate the term "{{term}}" from {{source_language}} to {{target_language}}. Focus on Biblical context. Return JSON: {"main": "primary translation", "synonyms": ["alt1", "alt2"]}',
  '{"type": "object", "properties": {"main": {"type": "string"}, "synonyms": {"type": "array", "items": {"type": "string"}}}}',
  'draft'
);

-- Test the new version in dev environment
UPDATE bible_schema.ai_prompt_bindings
SET prompt_version_id = (
  SELECT id FROM bible_schema.ai_prompt_versions
  WHERE template_id = (SELECT id FROM bible_schema.ai_prompt_templates WHERE task = 'translate_search_term')
  AND version = 2
)
WHERE task = 'translate_search_term' AND env = 'dev';

-- After testing, publish and promote to prod
UPDATE bible_schema.ai_prompt_versions
SET status = 'published'
WHERE template_id = (SELECT id FROM bible_schema.ai_prompt_templates WHERE task = 'translate_search_term')
AND version = 2;

UPDATE bible_schema.ai_prompt_bindings
SET prompt_version_id = (SELECT id FROM bible_schema.ai_prompt_versions WHERE template_id = (SELECT id FROM bible_schema.ai_prompt_templates WHERE task = 'translate_search_term') AND version = 2)
WHERE task = 'translate_search_term' AND env = 'prod';
```

### Example 3: Configure AI Provider
**User:** "Switch to Anthropic Claude for summaries in production"

**Assistant:**
```sql
-- Update feature binding
UPDATE bible_schema.ai_feature_bindings
SET
  ai_vendor = 'anthropic',
  ai_model = 'claude-3-5-sonnet-20241022',
  param_overrides = '{"max_tokens": 2048, "temperature": 0.7}'
WHERE feature_key = 'ai_summary' AND env = 'prod';

-- Ensure pricing is set
INSERT INTO bible_schema.ai_pricing (ai_vendor, ai_model, input_usd_per_1k, output_usd_per_1k)
VALUES ('anthropic', 'claude-3-5-sonnet-20241022', 0.003, 0.015)
ON CONFLICT (ai_vendor, ai_model) DO UPDATE
SET input_usd_per_1k = 0.003, output_usd_per_1k = 0.015;
```

## Prompt Variable Substitution

Use `{{variable}}` syntax in prompts:

```javascript
// System prompt
"You are a {{role}} assistant. Translate from {{source_language}} to {{target_language}}."

// User prompt template
"Translate: {{term}}"

// Edge Function usage
const prompt = await getPrompt('translate_search_term', {
  role: 'theological translation',
  term: 'armo',
  source_language: 'Finnish',
  target_language: 'English'
}, 'prod');
```

## Supported AI Providers

| Vendor | Endpoint | Models |
|--------|----------|--------|
| `lovable` | ai.gateway.lovable.dev | google/gemini-2.5-flash, openai/gpt-4o |
| `openai` | api.openai.com | gpt-4o, gpt-4o-mini, o1, o3 |
| `anthropic` | api.anthropic.com | claude-3-5-sonnet, claude-3-5-haiku |
| `openrouter` | openrouter.ai | Various models |

## Model Parameter Guidelines

### Newer Models (GPT-5, O3, O4)
- Use `max_completion_tokens` (not `max_tokens`)
- No `temperature` parameter
- Example: `{"max_completion_tokens": 1000}`

### Legacy Models (GPT-4o, Gemini)
- Use `max_tokens`
- Support `temperature`
- Example: `{"max_tokens": 1000, "temperature": 0.7}`

## Environment Strategy

1. **dev** - Experimental prompts and features
2. **stage** - Testing before production
3. **prod** - Live user-facing features

Always test in dev → stage → prod progression.

## Prompt Best Practices

1. **Clear instructions** - Be specific about the task
2. **JSON output** - Define explicit output schema
3. **Variable names** - Use descriptive `{{variable}}` names
4. **Examples** - Include examples in system prompt when possible
5. **Context** - Provide enough context for accurate responses
6. **Token limits** - Set appropriate `max_tokens` for the task
7. **Temperature** - Lower (0.3-0.5) for factual, higher (0.7-0.9) for creative

## Cost Optimization

1. Use cheaper models (gemini-2.5-flash) for simple tasks
2. Cache AI responses in `term_translations` table
3. Set reasonable `max_tokens` limits
4. Monitor usage in `ai_usage_logs`
5. Use fallback prompts for critical features

## Testing AI Features

Use the Admin AI page → Testaus tab:
1. Select feature
2. Input test variables
3. Review response, tokens, cost
4. Iterate on prompt if needed

## Related Documentation
- See `Docs/06-AI-ARCHITECTURE.md` for AI system details
- See `Docs/07-ADMIN-GUIDE.md` for Admin AI page usage
