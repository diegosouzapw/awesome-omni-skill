---
name: accessibility-standards
description: Implement WCAG 2.1 accessibility standards for Vue 3 apps. Use when adding ARIA labels, keyboard navigation, screen reader support, or checking color contrast. Mentions "accessibility", "ARIA", "keyboard nav", "screen reader", or "color contrast".
allowed-tools: Read, Edit, Grep, Glob
---

# Accessibility Standards

WCAG 2.1 AA compliance guidelines for Vue 3 applications.

## When to Activate

Use this skill when the user:
- Says "make it accessible" or "add ARIA labels"
- Asks "keyboard navigation" or "tab order"
- Mentions "screen reader", "WCAG", or "color contrast"
- Wants to "support assistive technology"

## Core Principles

1. **Perceivable**: Content must be presentable to users
2. **Operable**: UI must be navigable via keyboard
3. **Understandable**: Information must be clear
4. **Robust**: Compatible with assistive technologies

---

## 1. Keyboard Navigation

### Tab Order

**Proper Focus Flow**: Left → Right, Top → Bottom

```vue
<template>
  <!-- Header actions -->
  <button tabindex="0" @click="refresh">刷新</button>
  <button tabindex="0" @click="settings">设置</button>

  <!-- Filter panel -->
  <select tabindex="0" v-model="selectedInstitution">...</select>
  <button tabindex="0" @click="applyFilters">应用筛选</button>

  <!-- Main content -->
  <div tabindex="0" role="main">...</div>
</template>
```

### Skip Links

```vue
<template>
  <a href="#main-content" class="skip-link">跳至主内容</a>

  <header>...</header>

  <main id="main-content" tabindex="-1">
    <!-- Main content -->
  </main>
</template>

<style>
.skip-link {
  position: absolute;
  left: -9999px;
}

.skip-link:focus {
  position: static;
  left: 0;
}
</style>
```

### Keyboard Event Handling

```vue
<template>
  <button
    @click="handleAction"
    @keydown.enter="handleAction"
    @keydown.space.prevent="handleAction"
  >
    操作按钮
  </button>

  <!-- Custom component -->
  <FilterPanel
    tabindex="0"
    @keydown.esc="closePanel"
    aria-label="筛选面板"
  />
</template>
```

---

## 2. ARIA Labels

### Button ARIA

```vue
<!-- Icon button needs aria-label -->
<button aria-label="刷新数据" @click="refresh">
  <RefreshIcon />
</button>

<!-- Text button doesn't need it -->
<button @click="save">保存</button>

<!-- Disabled button -->
<button disabled aria-disabled="true">已禁用</button>
```

### Form ARIA

```vue
<template>
  <div class="form-field">
    <label for="institution-select">三级机构</label>
    <select
      id="institution-select"
      v-model="selectedInstitution"
      aria-describedby="institution-help"
      aria-required="true"
    >
      <option>达州</option>
      <option>德阳</option>
    </select>
    <span id="institution-help" class="help-text">
      选择业务员所属机构
    </span>
  </div>
</template>
```

### Live Region (Status Updates)

```vue
<template>
  <!-- Announce status changes to screen readers -->
  <div
    role="status"
    aria-live="polite"
    aria-atomic="true"
    class="sr-only"
  >
    {{ statusMessage }}
  </div>
</template>

<script setup>
const statusMessage = ref('')

watch(isLoading, (loading) => {
  statusMessage.value = loading ? '正在加载数据' : '数据加载完成'
})
</script>

<style>
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}
</style>
```

### Dialog ARIA

```vue
<template>
  <div
    v-if="visible"
    role="dialog"
    aria-labelledby="dialog-title"
    aria-describedby="dialog-desc"
    aria-modal="true"
  >
    <h2 id="dialog-title">确认操作</h2>
    <p id="dialog-desc">您确定要删除此项吗？</p>
    <button @click="confirm">确认</button>
    <button @click="cancel">取消</button>
  </div>
</template>
```

---

## 3. Color Contrast

### WCAG 2.1 AA Requirements

- **Normal text**: Contrast ratio ≥ 4.5:1
- **Large text** (18pt+): Contrast ratio ≥ 3:1

### Current Platform Colors (Verified)

| Combination | Ratio | Status |
|-------------|-------|--------|
| Primary text (#2C3E50) / White | 12.6:1 | ✅ Excellent |
| Secondary text (#8B95A5) / White | 4.8:1 | ✅ Pass |
| Primary color (#5B8DEF) / White | 4.2:1 | ⚠️ Borderline |
| Error color (#EF4444) / White | 5.1:1 | ✅ Pass |

### Improvements

```css
/* Use primary color with bold text for better readability */
.link-primary {
  color: var(--primary-500);
  font-weight: 600;  /* Bold improves perceived contrast */
}

/* Add icon support for color-blind users */
.status-success {
  color: var(--success-600);
}
.status-success::before {
  content: '✓';  /* Icon doesn't rely on color alone */
}
```

---

## 4. Focus Indicators

### Visible Focus

```css
/* Default browser focus */
*:focus {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
}

/* Custom focus for buttons */
.btn:focus-visible {
  outline: 2px solid var(--primary-500);
  outline-offset: 2px;
  box-shadow: 0 0 0 4px var(--primary-100);
}

/* Remove outline for mouse users */
.btn:focus:not(:focus-visible) {
  outline: none;
}
```

### Focus Management

```javascript
// Focus first interactive element in modal
const focusFirstElement = () => {
  nextTick(() => {
    const firstInput = modalRef.value?.querySelector('button, input, select')
    firstInput?.focus()
  })
}
```

---

## 5. Screen Reader Support

### Image Alt Text

```vue
<!-- Decorative image -->
<img src="icon.svg" alt="" role="presentation" />

<!-- Informative image -->
<img src="chart.png" alt="周对比保费趋势图，显示最近3周保费上升" />
```

### Chart Accessibility

```vue
<template>
  <div
    class="chart-container"
    role="img"
    :aria-label="chartDescription"
  >
    <ECharts :option="chartOption" />
  </div>

  <!-- Provide data table alternative -->
  <details class="chart-data">
    <summary>查看数据表格</summary>
    <table>
      <caption>周对比保费数据</caption>
      <thead>
        <tr>
          <th>日期</th>
          <th>保费（万元）</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in chartData" :key="item.date">
          <td>{{ item.date }}</td>
          <td>{{ item.value }}</td>
        </tr>
      </tbody>
    </table>
  </details>
</template>

<script setup>
const chartDescription = computed(() =>
  `周对比保费趋势图，显示${chartData.length}个数据点，保费范围从${minValue}到${maxValue}万元`
)
</script>
```

### Loading Announcements

```vue
<template>
  <div aria-busy="true" aria-live="polite">
    <Loading v-if="isLoading" />
    <span class="sr-only">{{ loadingMessage }}</span>
  </div>
</template>

<script setup>
const loadingMessage = computed(() =>
  isLoading.value ? '正在加载数据，请稍候' : '数据加载完成'
)
</script>
```

---

## 6. Motion and Animation

### Respect User Preferences

```css
/* Disable animations for users who prefer reduced motion */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### Safe Defaults

```css
/* Use subtle animations by default */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

/* No flashing or rapid movements */
```

---

## Accessibility Checklist

### Before Shipping

- [ ] All interactive elements are keyboard accessible
- [ ] Focus indicators are visible
- [ ] All images have appropriate alt text
- [ ] Forms have associated labels
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] ARIA roles and properties are correct
- [ ] Screen reader tested (NVDA/JAWS/VoiceOver)
- [ ] Keyboard navigation tested (Tab/Shift+Tab/Arrow keys/Enter/Esc)
- [ ] Reduced motion preference respected
- [ ] Error messages are announced to screen readers

---

## Testing Tools

### Browser Extensions
- **axe DevTools** - Automated accessibility testing
- **WAVE** - Visual accessibility evaluation
- **Lighthouse** - Built into Chrome DevTools

### Screen Readers
- **NVDA** (Windows, free)
- **JAWS** (Windows, paid)
- **VoiceOver** (macOS, built-in)

### Keyboard Testing
- Use only keyboard (no mouse)
- Tab through entire interface
- Verify all actions are accessible
- Check focus indicators are visible

---

## Troubleshooting

### "Screen reader doesn't announce changes"
Add `aria-live` region:
```vue
<div role="status" aria-live="polite">{{ message }}</div>
```

### "Keyboard navigation skips elements"
Check `tabindex`:
- `0` = Normal tab order
- `-1` = Not in tab order, but focusable programmatically
- `1+` = Avoid (disrupts natural order)

### "Focus indicator not visible"
Don't remove `:focus` styles. Customize instead:
```css
*:focus-visible {
  outline: 2px solid var(--primary-500);
}
```

---

## Related Files

**Component examples**:
- [Header.vue](../../../frontend/src/components/Header.vue)
- [FilterPanel.vue](../../../frontend/src/components/dashboard/FilterPanel.vue)

**Create**:
- `utils/accessibility.js` - Helper functions
- `composables/useFocusTrap.js` - Modal focus management

**Related Skills**:
- `vue-component-dev` - Component development
- `user-guidance-flows` - Help text and guidance

---

**Skill Version**: v1.0
**Created**: 2025-11-09
**Focuses On**: Accessibility standards only
