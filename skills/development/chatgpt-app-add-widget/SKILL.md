---
name: chatgpt-app:add-widget
description: Add a new inline widget to your ChatGPT App with Tailwind CSS and Apps SDK integration.
---

# Add Inline Widget

You are helping the user add a new inline HTML widget with **Tailwind CSS** to their ChatGPT App.

## Widget Patterns Available

Choose from these polished widget patterns:

### 1. Card Grid Widget
- Modern cards with hover effects and badges
- Responsive grid layout
- Best for: task lists, product catalogs, search results

### 2. Stats Dashboard Widget
- Colorful stat cards with icons
- Trend indicators (up/down arrows)
- Best for: analytics, dashboards, KPIs

### 3. Table Widget
- Clean data table with hover rows
- Column alignment support
- Best for: data tables, reports, logs

### 4. Bar Chart Widget
- Animated bars with smooth transitions
- Auto-scaling height
- Best for: comparisons, distributions

### 5. Comparison Cards Widget
- Side-by-side cards with "recommended" badge
- Great for pricing or scenario comparison
- Best for: mortgages, plans, options

### 6. Timeline Widget
- Vertical timeline with dots
- Scrollable container
- Best for: schedules, amortization, history

## Workflow

1. **Gather Information**
   Ask the user:
   - What data will this widget display?
   - What actions should users be able to take?
   - Which pattern fits best?

2. **Define Data Shape**
   Create TypeScript interface for tool output.

3. **Add Widget Config**
   Add to the `widgets` array in `server/index.ts`:
   ```typescript
   {
     id: "my-widget",
     name: "My Widget",
     description: "Displays data visually",
     templateUri: "ui://widget/my-widget.html",
     invoking: "Loading...",
     invoked: "Ready",
     mockData: { /* sample data for preview */ },
   },
   ```

4. **Add Widget HTML with Tailwind**
   Add case to `generateWidgetHtml()` function using Tailwind CSS:
   ```typescript
   if (widgetId === "my-widget") {
     return `<!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>My Widget</title>
     <script src="https://cdn.tailwindcss.com"></script>
     ${previewScript}
   </head>
   <body class="bg-gradient-to-br from-gray-50 to-gray-100 text-gray-900 p-4 font-sans antialiased">
     <div id="root">
       <div class="flex items-center justify-center min-h-[200px] text-gray-400">
         <svg class="animate-spin h-6 w-6 mr-2" fill="none" viewBox="0 0 24 24">
           <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
           <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
         </svg>
         Loading...
       </div>
     </div>
     <script>
       (function() {
         let rendered = false;
         function render(data) {
           if (rendered || !data) return;
           rendered = true;
           document.getElementById('root').innerHTML = '...';
         }
         function tryRender() {
           if (window.PREVIEW_DATA) { render(window.PREVIEW_DATA); return; }
           if (window.openai?.toolOutput) { render(window.openai.toolOutput); }
         }
         window.addEventListener('openai:set_globals', tryRender);
         const poll = setInterval(() => {
           if (window.openai?.toolOutput || window.PREVIEW_DATA) { tryRender(); clearInterval(poll); }
         }, 100);
         setTimeout(() => clearInterval(poll), 10000);
         tryRender();
       })();
     </script>
   </body>
   </html>`;
   }
   ```

5. **Create/Update Tool**
   Add tool that returns widget data with `widgetId`.

6. **Test Widget**
   ```bash
   npm run dev
   open http://localhost:3000/preview/my-widget
   ```

## Tailwind Widget Patterns

### Card Grid
```javascript
function render(data) {
  if (rendered || !data?.items) return;
  rendered = true;

  const statusColors = {
    active: 'bg-emerald-100 text-emerald-700 ring-1 ring-emerald-600/20',
    pending: 'bg-amber-100 text-amber-700 ring-1 ring-amber-600/20',
    inactive: 'bg-gray-100 text-gray-600 ring-1 ring-gray-500/20',
  };

  document.getElementById('root').innerHTML = `
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
      ${data.items.map(item => `
        <div class="group bg-white rounded-xl shadow-sm hover:shadow-md transition-all duration-200 border border-gray-200 hover:border-gray-300 overflow-hidden">
          <div class="p-5">
            <div class="flex items-start justify-between mb-3">
              <h3 class="font-semibold text-gray-900 group-hover:text-blue-600 transition-colors">${item.title}</h3>
              <span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium ${statusColors[item.status] || statusColors.inactive}">
                ${item.status}
              </span>
            </div>
            <p class="text-sm text-gray-500">${item.description || ''}</p>
          </div>
        </div>
      `).join('')}
    </div>
  `;
}
```

### Stats Dashboard
```javascript
function render(data) {
  if (rendered || !data?.stats) return;
  rendered = true;

  const colors = ['bg-blue-500', 'bg-emerald-500', 'bg-violet-500', 'bg-amber-500'];
  const fmt = n => n >= 1000 ? (n/1000).toFixed(1) + 'K' : n.toLocaleString();

  document.getElementById('root').innerHTML = `
    <div class="grid grid-cols-2 sm:grid-cols-4 gap-4">
      ${data.stats.map((stat, i) => `
        <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-5 hover:shadow-md transition-shadow">
          <div class="flex items-center gap-3 mb-3">
            <div class="w-10 h-10 ${colors[i % colors.length]} rounded-lg flex items-center justify-center">
              <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7h8m0 0v8m0-8l-8 8-4-4-6 6"></path>
              </svg>
            </div>
            <span class="text-sm font-medium text-gray-500">${stat.label}</span>
          </div>
          <span class="text-3xl font-bold text-gray-900">${fmt(stat.value)}</span>
        </div>
      `).join('')}
    </div>
  `;
}
```

### Table
```javascript
function render(data) {
  if (rendered || !data?.rows || !data?.columns) return;
  rendered = true;

  document.getElementById('root').innerHTML = `
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
      <table class="w-full">
        <thead>
          <tr class="bg-gray-50 border-b border-gray-200">
            ${data.columns.map(col => `
              <th class="px-4 py-3 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">${col.label}</th>
            `).join('')}
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          ${data.rows.map(row => `
            <tr class="hover:bg-gray-50 transition-colors">
              ${data.columns.map(col => `
                <td class="px-4 py-3 text-sm text-gray-700">${row[col.key] ?? '—'}</td>
              `).join('')}
            </tr>
          `).join('')}
        </tbody>
      </table>
    </div>
  `;
}
```

### Bar Chart
```javascript
function render(data) {
  if (rendered || !data?.bars) return;
  rendered = true;

  const max = Math.max(...data.bars.map(b => b.value));
  const colors = ['bg-blue-500', 'bg-emerald-500', 'bg-violet-500', 'bg-amber-500'];

  document.getElementById('root').innerHTML = `
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6">
      ${data.title ? `<h3 class="text-lg font-semibold text-gray-900 mb-6">${data.title}</h3>` : ''}
      <div class="flex items-end justify-between gap-2 h-[200px] mb-4">
        ${data.bars.map((bar, i) => {
          const height = max > 0 ? (bar.value / max) * 100 : 0;
          return `
            <div class="flex-1 flex flex-col items-center justify-end h-full">
              <span class="text-xs font-semibold text-gray-700 mb-1">${bar.value}</span>
              <div class="w-full rounded-t-lg ${colors[i % colors.length]}" style="height: ${height}%"></div>
            </div>
          `;
        }).join('')}
      </div>
      <div class="flex justify-between border-t border-gray-100 pt-3">
        ${data.bars.map(bar => `
          <div class="flex-1 text-center">
            <span class="text-xs text-gray-500">${bar.label}</span>
          </div>
        `).join('')}
      </div>
    </div>
  `;
}
```

### Comparison Cards
```javascript
function render(data) {
  if (rendered || !data?.scenarios) return;
  rendered = true;

  const fmt = n => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 }).format(n);

  document.getElementById('root').innerHTML = `
    <div class="grid grid-cols-1 md:grid-cols-${Math.min(data.scenarios.length, 3)} gap-4">
      ${data.scenarios.map(s => `
        <div class="relative bg-white rounded-xl shadow-sm border-2 ${s.recommended ? 'border-blue-500 ring-2 ring-blue-100' : 'border-gray-200'}">
          ${s.recommended ? `<div class="bg-blue-500 text-white text-xs font-semibold px-3 py-1 text-center">RECOMMENDED</div>` : ''}
          <div class="p-6">
            <h3 class="text-lg font-bold text-gray-900 mb-2">${s.label}</h3>
            <div class="text-3xl font-bold text-gray-900 mb-4">${fmt(s.monthlyPayment)}<span class="text-sm font-normal text-gray-500">/mo</span></div>
            <ul class="space-y-2 text-sm">
              ${Object.entries(s).filter(([k]) => !['label','recommended','monthlyPayment'].includes(k)).map(([k,v]) => `
                <li class="flex justify-between"><span class="text-gray-500">${k}</span><span class="font-medium">${typeof v === 'number' ? fmt(v) : v}</span></li>
              `).join('')}
            </ul>
          </div>
        </div>
      `).join('')}
    </div>
  `;
}
```

## Helper Functions

```javascript
function formatCurrency(n) {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 }).format(n);
}

function formatCompact(n) {
  if (n >= 1000000) return (n/1000000).toFixed(1) + 'M';
  if (n >= 1000) return (n/1000).toFixed(1) + 'K';
  return n.toLocaleString();
}

function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}
```

## Tailwind Best Practices

1. **Always include CDN** - `<script src="https://cdn.tailwindcss.com"></script>`
2. **Use gradient backgrounds** - `bg-gradient-to-br from-gray-50 to-gray-100`
3. **Add shadows and borders** - `shadow-sm border border-gray-200 rounded-xl`
4. **Include hover states** - `hover:shadow-md hover:border-gray-300 transition-all`
5. **Use consistent spacing** - `p-4 p-5 p-6` and `gap-4`
6. **Include loading spinner** - Animated SVG during load
7. **Handle hydration** - Check `rendered` flag

## Checklist

- [ ] Widget config added to `widgets` array
- [ ] Widget HTML with Tailwind added to `generateWidgetHtml()`
- [ ] Tool created/updated with `widgetId`
- [ ] Mock data provided for preview
- [ ] Preview tested at `/preview/{widget-id}`
- [ ] XSS prevention (escape user content)
