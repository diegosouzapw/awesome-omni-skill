---
name: vite-configuration
description: Vite 6.x configuration for React SPAs. Use when setting up or configuring Vite projects.
---

# Vite Configuration Skill

This skill covers Vite 6.x configuration for React single-page applications.

## When to Use

Use this skill when:
- Setting up new Vite React projects
- Configuring build optimization
- Adding plugins and aliases
- Configuring environment variables

## Core Principle

**FAST BY DEFAULT** - Vite is optimized out of the box. Only add configuration when needed.

## Basic Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

## Full Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],

  // Path aliases
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@lib': path.resolve(__dirname, './src/lib'),
    },
  },

  // Development server
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },

  // Build configuration
  build: {
    target: 'ES2022',
    sourcemap: true,
    minify: 'esbuild',
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-router': ['react-router-dom'],
        },
      },
    },
  },

  // Preview server (for built app)
  preview: {
    port: 4173,
  },
});
```

## TypeScript Path Aliases

Configure both Vite and TypeScript:

```typescript
// vite.config.ts
resolve: {
  alias: {
    '@': path.resolve(__dirname, './src'),
  },
},
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## Environment Variables

### Defining Variables

```bash
# .env
VITE_API_URL=http://localhost:8080
VITE_APP_TITLE=My App

# .env.development
VITE_API_URL=http://localhost:8080

# .env.production
VITE_API_URL=https://api.production.com
```

### Using Variables

```typescript
// In application code
const apiUrl = import.meta.env.VITE_API_URL;

// TypeScript declarations
// src/vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

## Plugin Configuration

### React Plugin Options

```typescript
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      // Babel plugins (for emotion, styled-components, etc.)
      babel: {
        plugins: ['@emotion/babel-plugin'],
      },
      // React Refresh options
      fastRefresh: true,
    }),
  ],
});
```

### Common Plugins

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { visualizer } from 'rollup-plugin-visualizer';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),

    // Bundle analyzer
    visualizer({
      filename: 'stats.html',
      open: true,
      gzipSize: true,
    }),

    // PWA support
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'My App',
        short_name: 'App',
        theme_color: '#ffffff',
      },
    }),
  ],
});
```

## Build Optimization

### Manual Chunks

```typescript
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        // Group React ecosystem
        'vendor-react': ['react', 'react-dom', 'react-router-dom'],
        // Group UI libraries
        'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        // Group state management
        'vendor-state': ['zustand', '@tanstack/react-query'],
      },
    },
  },
},
```

### Dynamic Chunks

```typescript
build: {
  rollupOptions: {
    output: {
      manualChunks(id) {
        if (id.includes('node_modules')) {
          if (id.includes('react')) {
            return 'vendor-react';
          }
          if (id.includes('@radix-ui')) {
            return 'vendor-ui';
          }
          return 'vendor';
        }
      },
    },
  },
},
```

### Chunk Size Warnings

```typescript
build: {
  chunkSizeWarningLimit: 500, // KB
},
```

## CSS Configuration

### CSS Modules

```typescript
css: {
  modules: {
    localsConvention: 'camelCase',
  },
},
```

### PostCSS

```typescript
css: {
  postcss: {
    plugins: [
      require('autoprefixer'),
      require('tailwindcss'),
    ],
  },
},
```

### Preprocessors

```typescript
css: {
  preprocessorOptions: {
    scss: {
      additionalData: `@import "@/styles/variables.scss";`,
    },
  },
},
```

## Proxy Configuration

```typescript
server: {
  proxy: {
    // String shorthand
    '/foo': 'http://localhost:4567',

    // With options
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true,
      rewrite: (path) => path.replace(/^\/api/, ''),
    },

    // Regex
    '^/api/.*': {
      target: 'http://localhost:8080',
      changeOrigin: true,
    },
  },
},
```

## Library Mode

For building component libraries:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import dts from 'vite-plugin-dts';
import path from 'path';

export default defineConfig({
  plugins: [
    react(),
    dts({ include: ['src'] }),
  ],
  build: {
    lib: {
      entry: path.resolve(__dirname, 'src/index.ts'),
      name: 'MyLibrary',
      formats: ['es', 'cjs'],
      fileName: (format) => `index.${format}.js`,
    },
    rollupOptions: {
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
      },
    },
  },
});
```

## Testing Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/__tests__/**/*.test.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

## Commands

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "analyze": "vite build && vite-bundle-visualizer"
  }
}
```

## Best Practices

1. **Use path aliases** - Clean imports with @/
2. **Environment variables** - Prefix with VITE_
3. **Manual chunks** - Split vendor code
4. **Proxy API calls** - Avoid CORS in development
5. **Type declarations** - Declare ImportMetaEnv
6. **Source maps** - Enable for debugging

## Notes

- Vite uses ESBuild for development (fast)
- Vite uses Rollup for production (optimized)
- Hot Module Replacement (HMR) works out of the box
- CSS is automatically extracted and minified
