---
name: ai-organizer-ui-consolidation
description: Build a unified, ADHD-friendly web UI that consolidates 70+ CLI tools into a beautiful liquid glass interface for the AI File Organizer. Use when creating the complete frontend application that replaces all terminal interactions with a macOS-inspired dashboard for file organization, search, VEO prompts, and system management.
---

# AI Organizer UI Consolidation

## Overview

Transform the AI File Organizer from 70+ scattered CLI tools (31,415 lines of Python) into a unified, beautiful web application that eliminates all terminal interactions. The user (Ryan) has ADHD and cannot use the current system due to overwhelming context-switching between commands. This consolidation is mission-critical.

**Success Metric:** Ryan can organize files, search content, manage VEO prompts, and control system settings without ever opening the terminal.

**Timeline:** 4 weeks (28 days)

**Tech Stack:** React 18 + Vite + Tailwind CSS + ShadCN UI + TanStack Query + Framer Motion

## When to Use This Skill

Use this skill when:
- Building the complete frontend UI for the AI File Organizer
- Creating a unified dashboard that replaces CLI workflows
- Implementing ADHD-friendly interaction patterns
- Designing liquid glass (frosted glass) aesthetics
- Consolidating multiple backend Python services into REST APIs
- Creating file organization interfaces with drag-and-drop
- Building rollback/undo systems for file operations
- Integrating computer vision (Gemini) and audio analysis results into UI

## Design Philosophy

### Liquid Glass Aesthetic (macOS Big Sur/Monterey inspired)

```css
/* Core Design Tokens */
--glass-bg: rgba(30, 30, 46, 0.7);
--glass-border: rgba(255, 255, 255, 0.1);
--glass-blur: blur(12px);
--accent-blue: #0A84FF;
--accent-green: #30D158;
--accent-yellow: #FFD60A;
--accent-red: #FF453A;
--accent-purple: #BF5AF2;
```

**Key Principles:**
- Frosted glass cards (`backdrop-blur-xl`)
- Subtle borders and shadows for depth
- Smooth 300ms animations
- Binary choices over complex menus
- Immediate visual feedback
- Undo always visible

### ADHD-Optimized UX

See `references/adhd-design-principles.md` for complete guidelines. Core tenets:
1. **Binary choices** ("A or B") not multiple choice
2. **Immediate feedback** - toasts, progress indicators, visual confirmation
3. **Forgiving interactions** - Rollback Center always accessible
4. **Visual hierarchy** - Important actions stand out
5. **No surprises** - Every action is predictable and reversible

## 4-Week Implementation Workflow

### Phase 1: Foundation + Dashboard + Basic Organize (Week 1)

**Goal:** Core infrastructure + can upload & organize a single file

**Days 1-2: Project Setup**
1. Initialize Vite project: Use `scripts/init-frontend.sh`
2. Install dependencies from `assets/package.json`
3. Copy `assets/tailwind.config.ts` for liquid glass design tokens
4. Create Layout component (Sidebar + Header + Outlet)
5. Set up React Router structure

**Days 3-4: Dashboard**
1. Build SystemStatusCard - shows Drive sync, disk space, services
2. Build QuickActionPanel - Upload, Search, Rollback buttons
3. Build RecentActivityFeed - live stream of file operations
4. Connect to `/api/system/status` and `/api/rollback/operations`

**Days 5-7: Basic Organize**
1. Build FileUploadZone with react-dropzone
2. Create `/api/organize/upload` endpoint (see `references/api-endpoints.md`)
3. Build classification preview UI
4. Implement toast notifications (Sonner)
5. **Test:** Upload file → see it organized

**Week 1 Deliverable:** ✅ Working dashboard + single file organization

### Phase 2: Advanced Organize + Rollback Center (Week 2)

**Goal:** Complete file organization workflow + rollback system

**Days 1-2: Confidence System + Interactive Questions**
1. Build ConfidenceModeSelector (NEVER/MINIMAL/SMART/ALWAYS)
2. Create `/api/settings/confidence` endpoints
3. Build InteractiveQuestionModal
   - "Is this about Finn or business?" [A] [B]
   - Shows file preview, confidence score, reasoning
4. **Test:** Low-confidence file triggers question

**Days 3-4: Batch Processing**
1. Build BatchProcessingPanel
2. Create `/api/organize/batch` endpoint
3. Implement progress bars ("Processing 7 of 23 files...")
4. Add pause/cancel functionality
5. **Test:** Process folder of 20 files

**Days 5-7: Rollback Center (CRITICAL)**
1. Build OperationTimeline component
2. Build expandable OperationCard
   - Shows before/after file paths
   - Timestamp, operation type
3. Create `/api/rollback/*` endpoints (see `references/api-endpoints.md`)
4. Implement search/filter functionality
5. Add Emergency Undo button (red, prominent)
6. **Test:** Organize files → undo operation → verify restoration

**Week 2 Deliverable:** ✅ Full organize workflow + bulletproof rollback

### Phase 3: Search + Analysis + VEO (Week 3)

**Goal:** Unified search + content analysis visibility + VEO library

**Days 1-2: Unified Search**
1. Build SearchBar with mode toggle [Fast/Semantic/Auto]
2. Build FilterPanel (date, category, location)
3. Build ResultsList with react-window virtualization
4. Create FileResultCard, EmailResultCard, ClipResultCard
5. **Test:** Search "Finn contracts" → see PDF + email results

**Days 3-4: Analysis Section**
1. Build AnalysisQueue component
2. Build VisionResultCard (Gemini Vision output)
3. Build AudioResultCard (BPM, mood, waveform)
4. Create `/api/analysis/*` endpoints
5. **Test:** Analyze image → see Gemini description

**Days 5-7: VEO Studio**
1. Build ClipLibrary grid view
2. Build ClipDetailView with Monaco JSON editor
3. Enhance existing `/api/veo/*` endpoints
4. Improve ContinuityGraph (hover tooltips, click navigation)
5. Build BatchUploadModal for videos
6. **Test:** Upload video → edit JSON → view continuity

**Week 3 Deliverable:** ✅ Search everything + analysis visible + VEO functional

### Phase 4: Settings + Polish + Launch (Week 4)

**Goal:** System management + final refinements + production launch

**Days 1-2: Settings Tabs**
1. Build Google Drive panel (`/api/settings/gdrive`)
2. Build Space Protection panel (`/api/settings/space`)
3. Build Background Services monitor
4. Build Adaptive Learning panel
5. Build Deduplication panel
6. **Test:** Change confidence mode, reconnect Drive

**Days 3-4: Polish & Animations**
1. Implement Framer Motion animations
2. Add keyboard shortcuts (Cmd+K search, Cmd+U upload)
3. Mobile responsive adjustments
4. Dark/light mode toggle (optional)
5. Loading skeletons for async content

**Days 5-7: Testing & Optimization**
1. E2E test critical flows
2. Performance optimization (code splitting, lazy loading)
3. Accessibility (ARIA labels, keyboard navigation)
4. Final UI polish
5. **Test with Ryan:** 1 week of daily use without terminal

**Week 4 Deliverable:** ✅ Production-ready UI replacing all CLI tools

## Component Architecture

### Layout Structure

```jsx
<BrowserRouter>
  <Layout>
    <Sidebar />              {/* Persistent navigation */}
    <MainContent>
      <Header />             {/* Breadcrumbs, notifications */}
      <Outlet />             {/* Route content */}
    </MainContent>
  </Layout>
</BrowserRouter>
```

### Main Sections (Routes)

1. **Dashboard** (`/`) - System status, quick actions, recent activity
2. **Organize** (`/organize`) - File upload, classification, batch processing
3. **Search** (`/search`) - Unified search across files, emails, VEO clips
4. **VEO Studio** (`/veo`) - Video prompt library, continuity graph, JSON editor
5. **Analysis** (`/analysis`) - Vision & audio analysis results gallery
6. **Rollback Center** (`/rollback`) - Operation history, undo functionality
7. **Settings** (`/settings`) - Drive, space, services, learning, dedup

See `references/component-specifications.md` for detailed component props, state management, and API integration patterns.

## Backend Service Layer Strategy

**Do NOT rewrite Python modules.** Instead, wrap them:

```python
# backend/services/organizer_service.py
from interactive_organizer import InteractiveOrganizer

class OrganizerService:
    """Wraps existing CLI tool for API use."""

    async def classify_file(self, file_path: str):
        result = await self.classifier.classify(file_path)
        return ClassificationResult(
            category=result['category'],
            confidence=result['confidence'],
            reasoning=result['reasoning']
        )
```

**Benefits:**
- Reuse 31,415 lines of battle-tested Python logic
- Faster development (no rewrites)
- Easier debugging (Python unchanged)

See `references/api-endpoints.md` for complete API specification covering all 70+ CLI commands.

## Resources

### scripts/

**`init-frontend.sh`** - Initialize Vite project with all required dependencies. Run this first:
```bash
./scripts/init-frontend.sh
```

### references/

**Essential documentation to load as needed:**

- **`cli-tool-audit.md`** - Complete categorization of all 70+ CLI tools by function (organization, search, analysis, safety, etc.)
- **`api-endpoints.md`** - Full REST API specification for all required endpoints
- **`component-specifications.md`** - Detailed specs for all React components (props, state, API calls)
- **`adhd-design-principles.md`** - ADHD-specific UX guidelines (cognitive load, feedback, forgiveness, hierarchy)
- **`liquid-glass-design-system.md`** - Complete design system (colors, typography, animations, component patterns)

**Usage:** Load these files when implementing specific features. For example, when building the Organize section, read `component-specifications.md` for FileUploadZone and ConfidenceModeSelector details.

### assets/

**Code templates and configuration files:**

- **`package.json`** - All required npm dependencies (React, Vite, Tailwind, TanStack Query, etc.)
- **`tailwind.config.ts`** - Tailwind configuration with liquid glass design tokens
- **`vite.config.ts`** - Vite configuration for optimal build
- **`component-template.tsx`** - Example React component with proper TypeScript, hooks, and styling
- **`tsconfig.json`** - TypeScript configuration

**Usage:** Copy these files directly into the frontend project during initialization.

## Critical Success Factors

1. **Start with Dashboard + Basic Organize** - Get core loop working first
2. **Rollback Center is Non-Negotiable** - This is the trust mechanism
3. **Test with Real Files** - Use Ryan's actual PDFs, contracts, emails
4. **ADHD-First UX Review** - After each component: "Can I do this without thinking?"
5. **Performance Matters** - Target <100ms UI response for all interactions

## Definition of Done

This project is complete when:

1. ✅ Ryan uses the system for 1 week without opening the terminal
2. ✅ All 70+ CLI commands have UI equivalents
3. ✅ Rollback system works flawlessly (tested with real mistakes)
4. ✅ Search finds files across all sources (local, Drive, emails, VEO)
5. ✅ UI feels fast, beautiful, and calming (not overwhelming)
6. ✅ System status is always visible (no "is this working?" anxiety)

**Final Test:** Ask Ryan: "Would you recommend this to another person with ADHD?"

If yes, **mission accomplished.** 🎉

---

*Version: 1.0*
*Status: Ready for implementation*
