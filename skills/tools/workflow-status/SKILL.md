---
name: workflow-status
description: "Display project workflow progress by reading handoff documents in .docs/ directory. This skill should be used when users want to check their workflow status, see what phase they're in, or when other workflow skills need to verify prerequisites. Provides reusable prerequisite-checking templates for integration with other workflow skills."
---

# workflow-status

<purpose>
Read all handoff documents in .docs/ subdirectory and present a clear picture of workflow progress, including completed phases, pending phases, termination point options, and actionable next steps. Also provide reusable prerequisite-checking logic for other workflow skills.
</purpose>

<output>
- Formatted status display showing workflow progress
- Project name detection from directory
- List of found handoff documents
- Termination point guidance
- Clear next step recommendation
- Reusable templates for other skills' prerequisite checks
</output>

---

<workflow>

<phase id="0" name="detect-context">
<action>Read project directory name and scan .docs/ for handoff documents.</action>

<detection-process>
1. Get current working directory name as project name
2. Check for .docs/ subdirectory existence
3. Scan .docs/ for all handoff documents
4. Note which documents are present vs missing
5. Check for project foundation (claude.md in project directory)
6. Check for CI/CD setup (.github/workflows/)
</detection-process>

<handoff-documents>
| Document | Phase | Location | Indicates |
|----------|-------|----------|-----------|
| PROJECT-MODE.md | 0 | .docs/ | Workflow mode established (LEARNING/BALANCED/DELIVERY) |
| brief-{project}.md | 0 | .docs/ | Project brief completed |
| tech-stack-decision.md | 1 | .docs/ | Technology stack selected |
| deployment-strategy.md | 2 | .docs/ | Deployment approach defined |
| project-foundation-complete.md | 3 | .docs/ | Project scaffolding generated |
| test-strategy.md | 4 | .docs/ | Test strategy defined |
| deployment-log.md | 5 | .docs/ | Deployment executed |
| ci.yml / deploy.yml | 6 | .github/workflows/ | CI/CD pipelines configured |
</handoff-documents>

<detection-rules>
- PROJECT-MODE.md: Contains mode selection (LEARNING/BALANCED/DELIVERY)
- Brief file: Matches pattern brief-*.md
- All planning documents expected in .docs/ subdirectory
- Missing .docs/ directory indicates workflow not started
- claude.md in project directory indicates Phase 3 complete
- .github/workflows/ indicates Phase 6 complete
</detection-rules>
</phase>

<phase id="1" name="analyze-progress">
<action>Determine completion status of each workflow phase based on found documents.</action>

<phase-mapping>
| Phase | Name | Required Documents | Status Logic |
|-------|------|-------------------|--------------|
| 0 | Project Brief | PROJECT-MODE.md + brief-*.md | Both present = COMPLETE |
| 1 | Tech Stack Selection | tech-stack-decision.md | Present = COMPLETE |
| 2 | Deployment Strategy | deployment-strategy.md | Present = COMPLETE |
| 3 | Project Foundation | claude.md in project dir | Present = COMPLETE |
| 4 | Test Strategy | test-strategy.md | Present = COMPLETE |
| 5 | Deployment | deployment-log.md | Present = COMPLETE |
| 6 | CI/CD | .github/workflows/*.yml | Present = COMPLETE |
</phase-mapping>

<status-determination>
For each phase:
- COMPLETE: All required documents present
- IN PROGRESS: Previous phase complete, this phase started but incomplete
- PENDING: Previous phase complete, this phase not started
- AVAILABLE: Can be started (flexible entry allows skipping)
- BLOCKED: Required for workflow continuation (only Phase 0 truly blocks)
</status-determination>

<mode-detection>
Read .docs/PROJECT-MODE.md to determine:
- LEARNING mode: Full guided workflow with checkpoints
- BALANCED mode: Moderate exploration
- DELIVERY mode: Streamlined generation
Display mode prominently in status output.
</mode-detection>

<termination-detection>
Identify if project is at a termination point:
- After Phase 3: Check if deployment-strategy.md indicates localhost
- After Phase 5: Check if deployment-log.md exists without CI/CD
- After Phase 6: Full workflow complete
</termination-detection>
</phase>

<phase id="2" name="determine-next-step">
<action>Identify what the user should do next based on current progress and termination points.</action>

<next-step-logic>
If no .docs/ directory:
  -> "Start workflow with project-brief-writer skill"

If missing PROJECT-MODE.md or brief:
  -> "Complete project brief with project-brief-writer skill"

If missing tech-stack-decision.md:
  -> "Select technology stack with tech-stack-advisor skill"

If missing deployment-strategy.md:
  -> "Define deployment approach with deployment-advisor skill"

If missing project foundation:
  -> "Generate project foundation with project-spinup skill"

If project foundation complete AND deployment indicates localhost:
  -> "TERMINATION POINT: Workflow complete for localhost project"
  -> "Optional: Continue to test-orchestrator when ready for testing infrastructure"

If project foundation complete AND deployment indicates public:
  -> "Ready to build features! When ready to deploy:"
  -> "  - Use test-orchestrator skill to set up testing (optional)"
  -> "  - Use deploy-guide skill to deploy your application"

If deployment-log.md exists AND no CI/CD:
  -> "TERMINATION POINT: Manual deployment complete"
  -> "Optional: Use ci-cd-implement skill to automate deployments"

If CI/CD exists:
  -> "WORKFLOW COMPLETE: Full automation configured"
</next-step-logic>

<termination-options>
At key points, present termination options:

After Phase 3 (project-spinup):
"Your project foundation is ready. You can:
1. Start development (this is a natural stopping point for localhost/learning projects)
2. Continue workflow when ready to deploy publicly"

After Phase 5 (deploy-guide):
"Your application is deployed. You can:
1. Stop here (manual deployment workflow established)
2. Use ci-cd-implement to automate future deployments"

After Phase 6 (ci-cd-implement):
"Workflow complete! Your project has full CI/CD automation."
</termination-options>

<next-step-details>
For each recommendation, include:
- The skill to invoke
- What it will produce
- Key decisions it will help make
- Whether it's required or optional at this point
</next-step-details>
</phase>

<phase id="3" name="display-status">
<action>Present formatted status output with all findings.</action>

<status-template>
## Project Workflow Status

**Project:** [{project-name}]
**Mode:** {LEARNING|BALANCED|DELIVERY}
**Current Phase:** {phase-number}: {phase-name}

---

### Phase Progress

{phase-status-list}

---

### Termination Points

{termination-status}

---

### Next Step

{next-step-recommendation}

---

### Handoff Documents

{document-list}

---

**Tip:** Any skill can be invoked standalone. Missing prerequisites will be gathered conversationally.
</status-template>

<phase-status-format>
For each phase, display:

PLANNING PHASES:
- [x] Phase 0: Project Brief
- [x] Phase 1: Tech Stack Selection
- [x] Phase 2: Deployment Strategy

SETUP PHASE:
- [x] Phase 3: Project Foundation <- TERMINATION POINT (localhost)

QUALITY PHASE:
- [ ] Phase 4: Test Strategy (optional)

RELEASE PHASES:
- [ ] Phase 5: Deployment <- TERMINATION POINT (manual deploy)
- [ ] Phase 6: CI/CD <- TERMINATION POINT (full automation)

Status indicators:
- [x] COMPLETE with key details
- [>] IN PROGRESS
- [ ] PENDING or AVAILABLE
</phase-status-format>

<termination-status-format>
Display based on current position:

If at Phase 3 with localhost:
"CURRENT TERMINATION: Localhost project - workflow complete for local development"

If at Phase 3 with public deployment planned:
"NEXT TERMINATION: After deploy-guide (Phase 5) for manual deployment"

If at Phase 5:
"CURRENT TERMINATION: Manual deployment - workflow complete if no CI/CD needed"

If at Phase 6:
"WORKFLOW COMPLETE: Full automation configured"
</termination-status-format>

<empty-state>
If .docs/ directory not found:

## No Workflow Detected

No workflow detected in current directory.

To start a new project workflow:
1. Create or navigate to your project directory
2. Say: "Use project-brief-writer skill"

This will establish your project brief and workflow mode.

**Note:** Skills can also be used standalone without the full workflow.
</empty-state>
</phase>

</workflow>

---

<reusable-templates>

<prerequisite-check-template>
Other workflow skills should include this pattern in their Phase 0:

<flexible-prerequisite-check>
<action>Check for handoff documents and gather missing information conversationally.</action>

<check-process>
1. Scan .docs/ for expected handoff documents
2. If found: Load context and summarize conversationally
3. If missing: Gather equivalent information through questions
4. Proceed with skill workflow regardless
</check-process>

<when-prerequisites-exist>
"I can see you've completed {previous-phase} ({key-detail}).
Ready to {current-phase-action}?"

Then proceed with the skill's main workflow.
</when-prerequisites-exist>

<when-prerequisites-missing>
"I don't see {expected-documents}. No problem - let me gather what I need.

{Targeted questions to get equivalent information}

Got it. Let's proceed with {current-phase-action}."

Then proceed with the skill's main workflow.
</when-prerequisites-missing>

<key-principle>
Skills NEVER block on missing prerequisites. They gather information conversationally and proceed.
</key-principle>
</flexible-prerequisite-check>
</prerequisite-check-template>

<skill-specific-prerequisites>
| Skill | Expected Documents | Questions if Missing |
|-------|-------------------|---------------------|
| tech-stack-advisor | PROJECT-MODE.md, brief-*.md | "What are you building? Learning or delivery focus?" |
| deployment-advisor | tech-stack-decision.md | "What's your tech stack? Frontend, backend, database?" |
| project-spinup | deployment-strategy.md | "Where will this deploy? What's your hosting preference?" |
| test-orchestrator | claude.md, project structure | "What framework are you using? What do you want to test?" |
| deploy-guide | deployment-strategy.md | "Where are you deploying? VPS, cloud, shared hosting?" |
| ci-cd-implement | project structure | "What's your deployment target? What should CI test?" |
</skill-specific-prerequisites>

</reusable-templates>

---

<guardrails>

<must-do>
- Always check .docs/ subdirectory for handoff documents
- Detect project name from current directory
- Provide clear, actionable next step recommendation
- Show which documents were found
- Display workflow mode (LEARNING/BALANCED/DELIVERY) when detected
- Use consistent status indicators throughout
- Highlight termination point options
- Explain flexible entry (skills can be used standalone)
</must-do>

<must-not-do>
- Assume documents exist without checking
- Skip displaying the next step recommendation
- Show technical errors to user (handle gracefully)
- Present workflow as strictly linear (termination points exist)
- Display raw file paths (use document names)
- Suggest skills must be run in strict order
</must-not-do>

</guardrails>

---

<integration-notes>

<workflow-position>
Standalone utility skill - not part of linear chain.
Can be invoked at any point during workflow.
Other skills reference reusable-templates for their prerequisite handling.
</workflow-position>

<reference-file>
HANDOFF-DOCUMENTATION.md in docs folder provides:
- Complete handoff system documentation
- Revision guidance for each phase
- Troubleshooting for common issues
</reference-file>

<termination-points>
The workflow has three natural stopping points:
1. After Phase 3 (project-spinup) - Localhost/learning projects
2. After Phase 5 (deploy-guide) - Manual deployment only
3. After Phase 6 (ci-cd-implement) - Full automation

workflow-status should clearly indicate which termination point applies.
</termination-points>

</integration-notes>
