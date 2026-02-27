---
name: laravel-type-bridge-development
description: Generate TypeScript/JavaScript type artifacts from Laravel PHP definitions — enums, i18n translations, and enum translator composables.
---

# Laravel Type Bridge Development

## When to use this skill

Use this skill when:
- Generating TypeScript or JavaScript enums from PHP backed enums
- Generating frontend i18n translation files from Laravel lang files
- Generating enum translator composables for frontend use
- Interacting with or integrating the generated enums/translations/translator composables in frontend code
- Configuring or extending the type bridge generation pipeline

## Installation & Setup

```bash
composer require gaiatools/laravel-type-bridge
php artisan type-bridge:publish
# Alternative:
# php artisan vendor:publish --tag=type-bridge-config
```

This publishes `config/type-bridge.php` where all generation is configured.

## Configuration (`config/type-bridge.php`)

```php
return [
    'output_format' => 'ts',          // 'ts' | 'js'
    'max_line_length' => 120,         // ESLint disable-line threshold (0 to disable)
    'trailing_commas' => true,
    'i18n_library' => 'i18next',      // 'i18next' | 'vue-i18n'

    'enums' => [
        'generate_backed_enums' => true,
        'discovery' => [
            'include_paths' => [app_path('Enums')],
            'exclude_paths' => [],
        ],
        'output_path' => 'js/enums/generated',    // relative to resources/
        'import_base' => '@/enums/generated',
    ],

    'translations' => [
        'discovery' => [
            'include_paths' => [base_path('lang')],
            'exclude_paths' => [],
        ],
        'output_path' => 'js/lang/generated',
        'custom_adapter' => null,    // optional: \App\TypeBridge\CustomI18nAdapter::class
    ],

    'enum_translators' => [
        'discovery' => [
            'include_paths' => [app_path('Enums')],
            'exclude_paths' => [],
        ],
        'translator_output_path' => 'js/composables/generated',
        'utils_composables_output_path' => 'js/composables',
        'utils_composables_import_path' => '@/composables',
        'utils_lib_output_path' => 'js/lib',
        'utils_lib_import_path' => '@/lib',
    ],
];
```

## Artisan Commands

### Generate everything (enums + translations + translators)

```bash
php artisan type-bridge:generate
php artisan type-bridge:generate en
php artisan type-bridge:generate --enums=Status --enums=App\\Enums\\Role
php artisan type-bridge:generate --translations-format=json
```

### Generate TypeScript/JavaScript enums from PHP backed enums

```bash
php artisan type-bridge:enums              # uses config output_format
php artisan type-bridge:enums --format=ts  # force TypeScript
php artisan type-bridge:enums --format=js  # force JavaScript
php artisan type-bridge:enums --check      # CI mode: fails if output is out of sync
php artisan type-bridge:enums --dirty      # only write enums that are out of sync
```

**Input:** PHP backed enums in `app/Enums/` (e.g. `Status::class`)
**Output:** `resources/js/enums/generated/Status.ts`

```typescript
// !!!!
// This is a generated file.
// Do not manually change this file
// !!!!
export const Status = {
    Active: 'active',
    Inactive: 'inactive',
} as const;

export type Status = typeof Status[keyof typeof Status];
```

### Generate frontend i18n translation files

```bash
php artisan type-bridge:translations en
php artisan type-bridge:translations en --format=json
php artisan type-bridge:translations en --format=js
php artisan type-bridge:translations en --format=ts
php artisan type-bridge:translations en --flat    # flatten nested keys
```

**Input:** `lang/en/` PHP translation arrays
**Output:** `resources/js/lang/generated/en.ts` (i18next or vue-i18n syntax based on `i18n_library` config)

### Generate enum translator composables

```bash
php artisan type-bridge:enum-translators
php artisan type-bridge:enum-translators --format=ts
php artisan type-bridge:enum-translators --dry    # preview candidates without writing
php artisan type-bridge:publish-translator-utils  # publish shared helper utilities
php artisan type-bridge:publish-translator-utils --force
```

**Output:** `resources/js/composables/generated/useStatusTranslator.ts`

Only enums that are both in the frontend enum generation set and have translations are eligible for translator generation.

### Publish translator utilities (required once per project)

```bash
php artisan type-bridge:publish-translator-utils [--force]
```

Defaults (configurable via `enum_translators.*`):
- composables: `resources/js/composables`
- lib: `resources/js/lib`

The file extensions follow `type-bridge.output_format`.

## Using the Generated JavaScript

These examples assume your frontend resolves `@` to `resources/js` (or equivalent) and you are generating JavaScript (`output_format` = `js`).

### Enums

```javascript
import { Status } from '@/enums/generated/Status';

if (status === Status.Active) {
    // ...
}
```

### Translations

The translation files are plain objects. The output shape depends on `i18n_library`.

i18next-style output:
```javascript
import en from '@/lang/generated/en';

i18next.addResources('en', 'translation', en);
```

vue-i18n-style output:
```javascript
import en from '@/lang/generated/en';

const i18n = createI18n({
    locale: 'en',
    messages: { en },
});
```

### Enum Translator Composables

Generated composables return a translator function (with helper methods). The exact API depends on the published utils in `js/composables` and `js/lib`.

```javascript
import { useStatusTranslator } from '@/composables/generated/useStatusTranslator';

const statusTranslator = useStatusTranslator();

statusTranslator('active'); // "Active"
statusTranslator.options(); // [{ value: 'active', label: 'Active' }, ...]
```
Destructuring is optional if you prefer direct access:
```javascript
const translateStatus = useStatusTranslator();
translateStatus('active');
translateStatus.options();
```

## Opt-in Enum Generation

Set `type-bridge.enums.generate_backed_enums=false` to restrict enum output to enums marked with `#[GenerateEnum]`.

```php
use GaiaTools\TypeBridge\Attributes\GenerateEnum;

#[GenerateEnum]
enum ThemeVisibility: string
{
    case Private = 'private';
    case Unlisted = 'unlisted';
    case Public = 'public';
}
```

## Global Translation Engine Setup

Generated translators call a global translation engine. Configure it once at app bootstrap:

```ts
import i18n from '@/i18n';
import { configureTranslationEngine } from '@/composables/useTranslator';

configureTranslationEngine({
  t: (key: string) => i18n.t(key),
});
```

## CI Drift Detection

Use `--check` flag in CI pipelines to fail if generated files are out of sync with source:

```bash
# In CI (GitHub Actions, etc.)
php artisan type-bridge:enums --check
```

Returns non-zero exit code when generated output does not match what would be generated.

## Extending the Pipeline

The generation pipeline follows: **Discoverer → Transformer → OutputFormatter → FileWriter**

To add a custom i18n adapter, implement `TranslationSyntaxAdapter` and set `translations.custom_adapter` in config:

```php
'custom_adapter' => \App\TypeBridge\MyAdapter::class,
```

Discovery paths are fully configurable per generator. Add multiple paths or exclude specific directories using `include_paths` / `exclude_paths` arrays in each generator's `discovery` config block.

## File Output Conventions

All generated files include a header warning:

```
// !!!!
// This is a generated file.
// Do not manually change this file
// !!!!
```

Files exceeding `max_line_length` get ESLint disable directives automatically. Set `max_line_length` to `0` to disable this behavior.

Output paths are always relative to `resources/`.
