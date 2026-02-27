---
name: qa-check
description: Mandatory quality assurance for all dev work before publishing. Use BEFORE deploying any project to production. Validates build, tests browser functionality, checks mobile responsiveness, and ensures no broken links/images.
author: gizmolab
version: 1.0.0
tags: [qa, quality, testing, validation, deployment]
---

# QA Check

Mandatory pre-deployment quality assurance. Run this before ANY project goes live.

## When to Use

- Before `vercel --prod` or any production deploy
- Before announcing/sharing any project URL
- Before publishing skills to ClawHub
- After major code changes

## QA Checklist

### 1. Build Validation
```bash
# Ensure build succeeds without errors
cd <project-dir>
npm run build
```

**Fail criteria:** Build errors, warnings about missing dependencies

### 2. Browser Functional Test

Use browser tool to verify:
- [ ] Page loads without console errors
- [ ] All interactive elements work (buttons, links, forms)
- [ ] No broken images (check Network tab for 404s)
- [ ] No JavaScript errors in console

```
browser snapshot → check for errors
browser console → verify no red errors
```

### 3. Mobile Responsiveness

```
browser screenshot --mobile
```

Check:
- [ ] Content readable on mobile viewport
- [ ] No horizontal scroll
- [ ] Buttons/links tappable (not too small)
- [ ] Navigation works

### 4. Link Validation

```bash
# Check all external links resolve
grep -r "href=" src/ | grep -o 'https://[^"]*' | sort -u | while read url; do
  curl -s -o /dev/null -w "%{http_code} $url\n" "$url"
done
```

### 5. Performance Quick Check

- Page loads in < 3 seconds
- No massive bundle warnings (> 500KB)
- Images optimized (not raw screenshots)

### 6. SEO/Meta Basics

Verify in index.html:
- [ ] `<title>` set (not generic)
- [ ] `<meta name="description">` present
- [ ] `<meta property="og:*">` for social sharing
- [ ] Favicon present

## Pre-Deploy Command

```bash
# Run full QA suite
scripts/qa-check.sh <project-dir>
```

## Fail = No Deploy

If ANY check fails:
1. Document the issue
2. Fix it
3. Re-run QA
4. Only then deploy

## Quick Reference

| Check | Tool | Pass Criteria |
|-------|------|---------------|
| Build | `npm run build` | Exit 0, no errors |
| Browser | `browser snapshot` | No console errors |
| Mobile | `browser screenshot` | Readable, no scroll |
| Links | `curl` | All 200/301 |
| Speed | Browser | < 3s load |
| SEO | Read index.html | All meta present |

## Post-Deploy Verification

After deploy, ALWAYS:
1. Visit production URL in browser
2. Click through main features
3. Check mobile view
4. Verify analytics loading (if added)

Only THEN announce/share the project.
