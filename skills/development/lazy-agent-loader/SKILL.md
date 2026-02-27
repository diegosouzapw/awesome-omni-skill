---
name: lazy-agent-loader
description: "Load agent definitions on-demand to reduce context usage. Only loads full agent when needed."
autoInvoke: false
priority: high
triggers:
  - "when agent is selected"
  - "agent:load"
allowed-tools: Read, Glob
---

# Lazy Agent Loader

**Priority:** HIGH - Load agents on-demand
**Version:** 1.0.0

---

## Purpose

Reduce token usage by:
1. Loading only agent **summaries** initially (~50 tokens each)
2. Loading **full definition** only when agent is activated
3. Caching loaded agents in session state

---

## Agent Index (Summaries Only)

```toon
agent_index[24]{id,category,specialty,keywords}:
  mobile-react-native,dev,React Native/Expo mobile,react-native/expo/RN/mobile/ios/android
  mobile-flutter,dev,Flutter/Dart mobile,flutter/dart/bloc/mobile
  web-angular,dev,Angular frontend,angular/ngrx/rxjs/typescript
  web-vuejs,dev,Vue.js frontend,vue/vuejs/pinia/nuxt/composition
  web-reactjs,dev,React frontend,react/reactjs/jsx/hooks/redux
  web-nextjs,dev,Next.js fullstack,next/nextjs/ssr/ssg/app-router
  backend-nodejs,dev,Node.js backend,nodejs/express/nestjs/fastify/api
  backend-python,dev,Python backend,python/django/fastapi/flask/api
  backend-go,dev,Go backend,go/golang/gin/fiber/api
  backend-laravel,dev,Laravel/PHP backend,laravel/php/eloquent/artisan
  security-expert,quality,Security auditing,security/vulnerability/audit/owasp/penetration
  qa-automation,quality,Testing/QA,test/testing/coverage/qa/jest/cypress
  devops-cicd,ops,DevOps/CI-CD,deploy/docker/kubernetes/ci-cd/pipeline/terraform
  smart-agent-detector,infra,Agent detection,detect/agent/select/route
  pm-operations-orchestrator,infra,PM/orchestration,pm/project/orchestrate/manage
  project-manager,infra,Project management,project/detect/identify/config/context
```

---

## Loading Strategy

### Initial Load (~1200 tokens)
```
1. Load this index file (agent_index)
2. DO NOT load individual agent files
3. Use index for agent detection scoring
```

### On Agent Selection (~500-2000 tokens per agent)
```
1. Agent scores ≥80 (PRIMARY) → Load full definition
2. Agent scores 50-79 (SECONDARY) → Load summary only
3. Agent scores <50 (OPTIONAL) → Don't load
```

### Full Definition Location
```
agents/[agent-id].md
```

---

## Loading Commands

### Load Single Agent
```bash
# Load full agent definition
cat agents/mobile-react-native.md
```

### Load Agent Summary
```toon
agent_summary{id,role,focus}:
  mobile-react-native,Senior React Native Developer,Expo/RN mobile apps with TypeScript
```

---

## Integration with Agent Detector

```
Step 1: Score agents using agent_index keywords
Step 2: Identify PRIMARY agent(s) with score ≥80
Step 3: Load ONLY PRIMARY agent full definitions
Step 4: For SECONDARY agents, use summary from index
```

---

## Token Savings

```toon
comparison[4]{scenario,without_lazy,with_lazy,savings}:
  Initial load (24 agents),~48000,~1200,97.5%
  Single agent task,~48000,~2700,94.4%
  Dual agent task,~48000,~4200,91.3%
  Full stack (3 agents),~48000,~5700,88.1%
```

---

## Cache Strategy

### Session Cache
```
Loaded agents are cached in conversation context.
If agent already loaded, skip re-loading.
Track loaded agents: loaded_agents[]: mobile-react-native,qa-automation
```

### Force Reload
```
User: "reload agent mobile-react-native"
→ Clear cache for agent
→ Re-read full definition
```

---

## Example Flow

```
User: "Create a React Native screen for login"

1. Agent Detector scores all agents using keywords from agent_index
   - mobile-react-native: +60 (react-native) +20 (context) = 80 ✅ PRIMARY
   - ui-expert: +35 (screen/login implies UI) → OPTIONAL

2. Lazy Loader activates:
   - Load: agents/mobile-react-native.md (~1500 tokens)
   - Skip: ui-expert (score < 50)

3. Context loaded:
   - Agent index: ~1200 tokens
   - mobile-react-native full: ~1500 tokens
   - Total: ~2700 tokens (vs ~48000 without lazy loading)
```

---

## Agent Categories

```toon
categories[4]{name,count,when_to_load}:
  dev,11,When code/implementation requested
  quality,3,When review/test/design requested
  ops,5,When deploy/integrate/notify requested
  infra,5,Usually auto-loaded by system
```

---

**Note:** This skill is automatically used by `agent-detector` for optimized loading.
