---
name: copilotkit-pitch-deck
description: Production-ready CopilotKit pitch deck wizard in main application. Use when enhancing AI conversation features, optimizing Edge Function integration, debugging chat interface, or improving pitch deck generation flow. System is PRODUCTION READY (98/100).
---

# CopilotKit Pitch Deck Wizard - Production

## Purpose

Maintain and enhance the production-ready CopilotKit pitch deck wizard in the main Medellin Spark application. Uses CopilotKit Cloud API + Supabase Edge Functions (NOT LangGraph three-tier architecture).

---

## Architecture Overview

**Current Production Setup**:
```
Frontend (:8080) --> Supabase Edge Functions --> OpenAI API
  React 19 + Vite     Deno Runtime (cloud)      GPT-4o-mini
```

**Key Components**:
- **Frontend**: `src/pages/PitchDeckWizard.tsx` - CopilotKit v1.10.6 with chat interface
- **Edge Functions**: `supabase/functions/` - pitch-deck-assistant, generate-pitch-deck, chat
- **Database**: Supabase PostgreSQL with RLS enabled
- **Status**: 🟢 **PRODUCTION READY** (98/100)

---

## Production Status

**Tracker**: `/home/sk/template-copilot-kit-py/mvp-plan/progress/00-PRODUCTION-READINESS-TRACKER.md`
**Latest Validation**: `/home/sk/template-copilot-kit-py/mvp-plan/progress/01-FINAL-PRODUCTION-VALIDATION.md`

### Current Implementation ✅

**CopilotKit Integration** (`src/pages/PitchDeckWizard.tsx:189-192`):
```typescript
<CopilotKit
  publicApiKey={import.meta.env.VITE_COPILOT_CLOUD_PUBLIC_API_KEY}
  showDevConsole={import.meta.env.DEV}
>
  {/* Chat interface and wizard content */}
</CopilotKit>
```

**Edge Functions (All ACTIVE)**:
1. **chat** (v16) - OpenAI proxy for secure API calls
2. **pitch-deck-assistant** (v26) - Conversation handler with progress tracking
3. **generate-pitch-deck** (v28) - Full deck generation (10 slides)
4. **agent-example** (v8) - Agent SDK demonstration

**Features Working** ✅:
- AI-powered chat interface
- Progress tracking (0-100%)
- Data collection via conversation
- Generate deck button at 80%+ completion
- Presentation viewer/editor
- Real-time updates

### Performance Metrics

**Build Performance**:
```
Build Time: 4.71s ✅ (Target: <5s)
Bundle Size: 554KB ✅ (50% reduction from 1,109KB)
Gzip: 151KB ✅ (73% compression)
Warnings: 0 ✅
```

**Production Quality**:
```
TypeScript Errors: 0 ✅
Linter Warnings: 0 ✅
Routes Working: 24/24 ✅
Security Score: 100% ✅
Overall Score: 98/100 ✅
```

---

## Common Issues & Solutions

### Issue 1: Chat Interface Not Responding
**Cause**: Edge Function connection issue
**Check**:
```bash
# Verify Edge Functions deployed
supabase functions list

# Check function logs
supabase functions logs pitch-deck-assistant --tail
```
**Fix**: Ensure all Edge Functions deployed and ACTIVE

### Issue 2: API Key Errors
**Cause**: Missing or invalid OpenAI API key in Edge Function
**Check**: Edge Function secrets
```bash
# View secrets (won't show values)
supabase secrets list

# Set OpenAI key if missing
supabase secrets set OPENAI_API_KEY=sk-...
```
**Fix**: Ensure OPENAI_API_KEY set as Supabase secret

### Issue 3: Progress Bar Not Updating
**Cause**: Edge Function not returning completeness data
**Check**: Browser console for response data
**Debug**:
```typescript
// In PitchDeckWizard.tsx - check response structure
console.log('Response from Edge Function:', response);
// Should include: { completeness: number, ... }
```

### Issue 4: Build Errors
**Cause**: TypeScript errors or missing dependencies
**Check**:
```bash
# Type check
pnpm tsc --noEmit

# Check for missing dependencies
pnpm install
```
**Fix**: Resolve TypeScript errors before building

### Issue 5: Local Development Server Issues
**Cause**: Port conflicts or build cache
**Fix**:
```bash
# Kill any processes on port 8080
lsof -ti :8080 | xargs kill -9

# Clear Vite cache
rm -rf node_modules/.vite

# Restart dev server
pnpm dev
```

---

## Testing Workflow

### 1. Start Development Server
```bash
cd /home/sk/template-copilot-kit-py
pnpm dev
```

**Expected**:
```
VITE v7.x ready in 234 ms
➜  Local:   http://localhost:8080/
```

### 2. Navigate to Pitch Deck Wizard
**URL**: `http://localhost:8080/pitch-deck-wizard`

**Check**:
- ✅ Page loads without errors
- ✅ CopilotKit chat interface visible
- ✅ No console errors
- ✅ Dev console badge visible (in DEV mode)

### 3. Test Chat Flow
1. **Send**: "I want to create a pitch deck for TestCorp"
2. **Verify**: AI responds with questions
3. **Send**: "We're in AI software industry"
4. **Verify**: Conversation continues
5. **Continue**: Answer 4-5 more questions
6. **Verify**: Progress tracking works (if implemented)
7. **Verify**: "Generate Deck" appears at completion

### 4. Test Deck Generation
1. **Click**: "Generate Deck" button
2. **Verify**: Loading state shows
3. **Wait**: 5-10 seconds for generation
4. **Verify**: Redirect to `/presentations/{id}/outline`
5. **Verify**: All 10 slides render correctly

### 5. Production Build Test
```bash
# Build for production
pnpm build

# Should complete in <5s with 0 warnings
# Check output for bundle sizes
```

---

## Key Files Reference

### Frontend Files (Main Application)
- `src/pages/PitchDeckWizard.tsx` - CopilotKit integration and chat interface
- `src/lib/apiClient.ts` - Edge Function API calls
- `package.json` - Dependencies and build scripts
- `vite.config.ts` - Build configuration (optimized with code splitting)

### Edge Function Files
- `supabase/functions/chat/index.ts` - OpenAI proxy (v16)
- `supabase/functions/pitch-deck-assistant/index.ts` - Conversation handler (v26)
- `supabase/functions/generate-pitch-deck/index.ts` - Deck generator (v28)
- `supabase/functions/agent-example/index.ts` - Agent SDK demo (v8)

### Database Files
- `supabase/migrations/` - Database schema migrations
- SQL tables: presentations, pitch_conversations, profiles

### Documentation
- `CLAUDE.md` - Project standards and best practices
- `mvp-plan/progress/00-PRODUCTION-READINESS-TRACKER.md` - Production status
- `mvp-plan/progress/01-FINAL-PRODUCTION-VALIDATION.md` - Latest validation
- `SITEMAP-2.md` - Complete route map

---

## Production Verification Checklist

### Code Quality ✅
- [x] TypeScript: 0 errors (`pnpm tsc --noEmit`)
- [x] Linter: Clean (`pnpm lint`)
- [x] No console.log in production code
- [x] Error boundaries implemented
- [x] Loading states added

### CopilotKit Integration ✅
- [x] CopilotKit v1.10.6 installed
- [x] Public API key configured in .env
- [x] Chat interface renders correctly
- [x] Dev console enabled in DEV mode
- [x] No CORS errors

### Edge Functions ✅
- [x] All 4 functions deployed and ACTIVE
- [x] OPENAI_API_KEY set as secret
- [x] Functions return proper responses
- [x] Error handling comprehensive
- [x] Logs accessible via `supabase functions logs`

### Features ✅
- [x] Chat conversation works end-to-end
- [x] AI responds appropriately
- [x] Data collection functional
- [x] Generate deck button appears
- [x] Deck generation creates 10 slides
- [x] Presentation viewer/editor working

### Build & Performance ✅
- [x] Production build succeeds (<5s)
- [x] Bundle optimized (554KB, 50% reduction)
- [x] No build warnings
- [x] Gzip compression enabled (151KB)
- [x] Code splitting implemented

---

## Quick Commands

**Start Development**:
```bash
cd /home/sk/template-copilot-kit-py
pnpm dev  # Runs on http://localhost:8080
```

**Type Check**:
```bash
pnpm tsc --noEmit  # Should return 0 errors
```

**Production Build**:
```bash
pnpm build  # Should complete in <5s with 0 warnings
```

**Deploy Edge Functions**:
```bash
# Deploy single function
supabase functions deploy chat

# List all functions
supabase functions list

# View logs
supabase functions logs pitch-deck-assistant --tail
```

**Database Commands**:
```bash
# Push migrations
npx supabase db push

# Check status
supabase status
```

---

## Dependencies

**Frontend** (`package.json`):
- `@copilotkit/react-core` ^1.10.6 - CopilotKit SDK
- `@copilotkit/react-ui` ^1.10.6 - Chat interface
- `react` ^19.x
- `vite` ^7.x
- `@supabase/supabase-js` - Supabase client

**Edge Functions** (Deno runtime):
- OpenAI SDK for API calls
- Supabase client for database access
- Deno standard library

---

## Environment Variables

**Required in `.env`**:
```bash
# CopilotKit (frontend)
VITE_COPILOT_CLOUD_PUBLIC_API_KEY=your_public_key

# Supabase (frontend)
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your_anon_key

# Edge Functions (Supabase secrets)
OPENAI_API_KEY=sk-...  # Set via: supabase secrets set
```

---

## Performance Best Practices

✅ **Implemented**:
- Bundle code splitting (5 optimized chunks)
- Gzip compression (73% reduction)
- React.memo for expensive components
- Lazy loading for routes
- Error boundaries for graceful failures

🟡 **Optional Enhancements**:
- Add request debouncing for API calls
- Implement response caching
- Monitor Core Web Vitals
- Add service worker for offline support

---

## Related Resources

- **CopilotKit Docs**: https://docs.copilotkit.ai
- **Supabase Edge Functions**: https://supabase.com/docs/guides/functions
- **Project Documentation**: `/home/sk/template-copilot-kit-py/CLAUDE.md`
- **Production Tracker**: `/home/sk/template-copilot-kit-py/mvp-plan/progress/`

---

## Next Steps (Post-Production)

1. 🟡 Monitor production errors and Edge Function logs
2. 🟡 Collect user feedback on AI conversation quality
3. 🟡 Add analytics tracking for usage metrics
4. 🟡 Implement E2E test suite with Playwright
5. 🟡 Optimize prompts based on user interactions

---

*This skill provides complete guidance for the production-ready CopilotKit pitch deck wizard in the main Medellin Spark application.*
