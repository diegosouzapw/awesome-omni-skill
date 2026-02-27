---
name: nextjs-production-debugger
description: Advanced debugging guide for Next.js App Router production issues including SSR/CSR bugs, hydration errors, runtime mismatches, performance, and caching.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Next.js Production Debugger (App Router)

Guía de supervivencia para debuggear aplicaciones Next.js App Router en producción.

## 1. Bugs SSR (Server) vs CSR (Client)

**Síntoma:** El bug ocurre solo en la carga inicial (F5) o solo al navegar (SPA transition).

| Escenario      | Comportamiento                                                           | Diagnóstico                                                                                   |
| :------------- | :----------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| **Bug en SSR** | La página carga rota con F5. Al navegar desde otra página funciona bien. | Revisa logs del TERMINAL del servidor. Deshabilita JS en el navegador para ver el HTML crudo. |
| **Bug en CSR** | La página carga bien con F5, pero falla al interactuar o navegar.        | Revisa la consola del NAVEGADOR. Usa React DevTools.                                          |

**Técnica de Aislamiento:**

1.  **Deshabilitar JavaScript:** Si el contenido está mal sin JS, el problema es el Server rendering.
2.  **Hard Refresh (Cmd+Shift+R):** Fuerza una nueva petición al servidor, ignorando el Router Cache del cliente.

## 2. Hydration Errors (The "Uncanny Valley")

**Error:** `Text content does not match server-rendered HTML` o `Hydration failed`.

**Causas Comunes:**

1.  **Timestamps/Random:** Usar `Date.now()` o `Math.random()` directamente en el cuerpo del componente.
    - _Solución:_ Mover a `useEffect` o usar una prop desde el servidor.
2.  **HTML Inválido:** Poner un `<div>` dentro de un `<p>`, o `<ul>` dentro de `<p>`.
    - _Check:_ Valida tu HTML. El navegador intenta corregirlo, creando discrepancias.
3.  **Extensiones del Browser:** Plugins que inyectan HTML/CSS pueden romper la hidratación.
    - _Prueba:_ Abre en modo Incógnito/Privado.
4.  **`typeof window !== 'undefined'`:** Chequeos condicionales de renderizado basados en `window`.
    - _Solución:_ Usa `useEffect` para montar componentes que dependen de `window`.

**Fix Rápido (Solo si es estético):**
`<div suppressHydrationWarning>{time}</div>`

## 3. Edge vs Node Runtime Mismatch

**Error:** `Module not found: Can't resolve 'fs'` o errores crípticos en Vercel/Middleware.

**Diagnóstico:**

- Revisa `export const runtime = 'edge'` en tus `page.tsx` o `route.ts`.
- **Problema:** El Edge Runtime es limitado (no tiene APIs de Node completas).
- **Librerías:** Algunas librerías npm asumen Node.js y fallan en Edge.

**Soluciones:**

1.  Cambiar a `runtime = 'nodejs'` si necesitas compatibilidad total.
2.  Si usas Middleware (siempre es Edge), asegúrate de usar APIs Web Standard (`fetch`, `Request`, `Response`) y no módulos de Node.
3.  Usa `server-only` package para asegurar que código de servidor no se filtre al cliente.

## 4. Performance en App Router

**Síntoma:** "La página tarda mucho en cargar".

**Culpables Habituales:**

1.  **Waterfalls (Fetch en Cascada):**
    - _Mal:_ Componente A espera fetch -> Renderiza B -> B espera fetch.
    - _Bien:_ `Promise.all([fetchA, fetchB])` en el padre, o preload patterns.
2.  **Client Components Gigantes:**
    - No hagas toda la página `use client`. Mueve la interactividad a las hojas (hojas del árbol de componentes).
3.  **Blocking SSR:**
    - Una petición de base de datos lenta en el `page.tsx` bloquea todo el HTML.
    - _Solución:_ Envuelve la parte lenta en `<Suspense fallback={<Skeleton />}>` para Streaming SSR.

**Herramientas:**

- React DevTools Profiler.
- Pestaña "Network" en Chrome (mira el TTFB vs Content Download).

## 5. Caching Mal Configurado (El enemigo silencioso)

**Síntoma:** "Actualicé la DB pero la web sigue mostrando datos viejos".

**Capas de Cache en Next.js:**

1.  **Request Memoization:** Dentro del mismo render pass. (Deduplica fetches iguales).
2.  **Data Cache:** Persistente entre deploys/requests. (El `fetch` nativo lo hace).
    - _Fix:_ `fetch(url, { cache: 'no-store' })` o `export const dynamic = 'force-dynamic'`.
3.  **Full Route Cache:** Páginas estáticas generadas al build time.
    - _Fix:_ `revalidatePath('/ruta')` en Server Actions.
4.  **Router Cache (Client):** Cache en el navegador al navegar (dura 30s o 5min).
    - _Fix:_ `router.refresh()` invalida este cache.

**Debug Flow:**

1.  ¿Es dato viejo en el cliente? -> Refresca la página.
2.  ¿Sigue viejo? -> Es Data Cache o Full Route Cache.
3.  ¿Sigue viejo tras nuevo deploy? -> Revisa `revalidate` time o `no-store`.

---

**Comando útil:**
Usa `next build` localmente para ver qué páginas se generan como Estáticas (O) o Dinámicas (ƒ).
