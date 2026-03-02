---
name: asset-builder
description: Manage CSS/JS building with npm/wp-scripts. Use when working on login page styles or scripts.
---

# Asset Builder

## Instructions

When working with CSS/JS assets for the login page:

1. **Source files location**: Check `src/` or project root
2. **Build commands**: Use npm scripts from package.json
3. **Output location**: `dist/assets/`
4. **Enqueue hook**: Use `login_enqueue_scripts`

## Build Commands

| Command            | Purpose                              |
| ------------------ | ------------------------------------ |
| `npm run start`    | Watch mode - rebuild on file changes |
| `npm run build`    | Production build - minified assets   |
| `npm run lint:js`  | Lint JavaScript                      |
| `npm run lint:css` | Lint CSS                             |

## Enqueue on Login Page

```php
add_action('login_enqueue_scripts', 'retrologin_enqueue_assets');
function retrologin_enqueue_assets(): void {
    wp_enqueue_style(
        'retrologin-login',
        plugins_url('dist/assets/login.css', __FILE__)
    );
    wp_enqueue_script(
        'retrologin-login',
        plugins_url('dist/assets/login.js', __FILE__),
        ['wp-api-fetch'],
        '0.1.0',
        true
    );
}
```

## Login Page CSS Selectors

| Element       | Selector            |
| ------------- | ------------------- |
| Page wrapper  | `.login`            |
| Login form    | `#loginform`        |
| Logo          | `.login h1 a`       |
| Messages      | `.login .message`   |
| Submit button | `.wp-submit-button` |

## Guidelines

-   Keep login assets minimal for performance
-   Login page doesn't load theme styles
-   Use CSS variables for retro theming
-   Test assets in browser after building
