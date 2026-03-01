---
name: annotate
description: Create flexible annotation workflows for AI applications. Contains common tools to explore raw ai agent logs/transcripts, extract out relevant evaluation data, and llm-as-a-judge creation.
allowed-tools: Read, Grep, Glob, jq
---

# Overview

The workflow consists of 3 main phases in a loop:

1. **Data Ingestion** - Transform raw agent log data into an ingested format for analysis
2. **Feedback Configuration and data exploration** - Refine what and how we want to provide feedback on agent transcripts by exploring the raw data via traditional file search tools and automated llm-as-a-judge based filtering
3. **Annotation** - Annotate the transcript data based on the feedback configuration

NOTE: use the multichoice/checklist/survey tool as much as possible when getting feedback from the user, typing is onerous.

# Setup and Housekeeping

This document covers setup tasks that should be completed before starting the annotation workflow.

## Prerequisites

This skill requires Python and Node.js dependencies. If the user encounters errors about missing packages, ask for permission to install dependencies:

```bash
# From the annotate skill directory
pip install -e .

# Frontend dependencies (from frontend/ subdirectory)
cd frontend && yarn install
```

## Getting Started

All the state and data should be contained under `.haize_annotations` in the *current* working directory. For claude code, this is the same directory as `.claude`.
Make sure this is the .claude folder in the CURRENT working directory, not globally.

### Check for Existing Work

**ALWAYS start by checking for existing work**; we may be resuming an annotation session where a user has already made progress.

If `.haize_annotations` does not exist, create it.

If `.haize_annotations` does exist, inspect its contents:
- If `ingested_data` exists, it means we've done the work to translate raw trace data to a haize format.
- If `feedback_config` exists, it means that we've already arrived on some criteria for how the human should provide feedback.
- If `test_cases` exists with human/ai annotations, it means the user has already made some annotation progress based on a particular feedback config; it's not necessarily the case that the cases match the CURRENT feedback config though.

## State and File Structure

All artifacts will be saved to `.haize_annotations/` directory:

**Important note:** `ingest.py` is the ONLY file you are meant to manually EDIT/WRITE (all files are free to read)

```
.haize_annotations/
├── ingested_data/
│   └── interactions/                    # list of all ai interactions
│       ├── {interaction_id}/
│       │   ├── metadata.json           # Interaction fields (id, name, group_id, etc)
│       │   └── steps.jsonl             # One InteractionStep per line
│       └── ...
│
├── ingest.py                         # Script to ingest raw data under some folder path into `ingested_data`
│
├── feedback_config.json                # Evaluation criteria and any state about an annotation session tied to this specific feedback config
└── test_cases/                         # Test cases human/ai produced via annotation sessions that human/ai must give feedback on
    ├── tc_{uuid}.json                  # Contains: raw_judge_input, judge_input,
    │                                   #           ai_annotation, human_annotation
    └── ...
```

**Important note:** You will NOT be directly modifying the feedback config file; instead, you'll modify it through a FastAPI endpoint.


# Phase 1: Data ingestion

The goal of this step is to ingest raw agent data which can be in arbitrary form into a normalized data format that we can work with.

### Step 0: Generate a boilerplate ingestion script
To start, generate `ingest.py` with all models and boilerplate included:

```bash
bash <path-to>/scripts/generate_ingestion_script.sh .haize_annotations/ingest.py
```

**IMPORTANT:**
- Replace `<path-to-annotate-skill>` with the actual path to the annotate skill directory (under scripts/generate_ingestion_script.sh)
- The output path (`.haize_annotations/ingest.py`) specifies where the script will be created inside the `.haize_annotations/` directory to keep all artifacts together.

This creates `ingest.py` with:
- All Pydantic models from `scripts/_models.py`
- Placeholder `ingest()` function with TODO comments and example implementation
- Main execution logic that saves to `.haize_annotations/ingested_data/`

**REQUIRED READING BEFORE CONTINUING**: You MUST read:
- `references/normalization_patterns.md`
(no need to call out or cite specific patterns you are using - these reference docs are for YOUR knowledge)

### Step 1: Analyze raw agent data

Sample and inspect raw trace files to understand structure.

See [references/normalization_patterns.md](./references/normalization_patterns.md) for detailed guidance on analyzing trace formats and transformation patterns. 

**Careful:** Raw agent log files can get very large, and not all files in the working directory are agent transcript data. Check file sizes before opening trace data.

**Important:** Python is a great tool here, but make use of other safer tools like `jq` as well if they exist.

**NOTE:** If the user wants to annotate something that is not super feasible with the data at hand (e.g., they want to annotate the quality of complete sessions but the associated data doesn't have a stable session identifier to group the data into sessions, or the user wants to annotate a multi-step RAG bot but the data doesn't have an identifier to group together all steps into a single interaction), you HAVE to mention that to them to manage expectations and tell them WHAT IS MISSING for them to annotate that aspect. In these cases, suggest ALTERNATIVE aspects of the AI app to annotate.

### Step 2: Implement the ingestion logic

**Your job:** Implement the `ingest(folder_path)` function based on your trace format in `ingest.py`. You're responsible for ALL file loading and parsing logic.

See [references/normalization_patterns.md](./references/normalization_patterns.md) for transformation patterns and complete examples:
- **Pattern A:** OTel-compatible traces
- **Pattern B:** Non-OTel structured traces (OpenAI Agents SDK, Datadog, custom formats)
- **Pattern C:** Flat data (CSV, simple JSON)

### Step 3: Run Ingestion

```bash
python .haize_annotations/ingest.py --input /path/to/traces/folder
```

This will:
1. Call your `ingest()` function with the input folder path
2. Save each returned interaction to `.haize_annotations/ingested_data/interactions/{interaction_id}/`
   - `metadata.json` - interaction metadata
   - `steps.jsonl` - one InteractionStep per line

### Step 4: Review Normalized Output

**Critical validation step!** Always validate that the ingested data is as expected before continuing using a combination of:
- `scripts/run_validate_ingested_data.py` (very quick, high level stats) (must be run as a module!!)
- e.g. `cd <path-to>/skills/annotate_skill && python -m scripts.run_validate_ingested_data
   --ingested-dir <path-to>/.haize_annotations/ingested_data/interactions`
- manually reading the data!!! this is important and helps you verify the data shape is what you expected

You **might** need to come back to this later, in case anything with the way we've ingested the data makes giving feedback on agent traces difficult.

---

## Phase 2: Annotations

### Step 0: Setup FastAPI server

Before creating feedback configurations and collecting annotations, you need to start the annotation session servers. This single command starts:
- **FastAPI backend** - REST API for test case management
- **React frontend** - UI for viewing traces and annotations
- **test case processor** - Background process that auto-generates AI annotations

**Command:**

YOU MUST run the annotation session script as a module!!!!

```bash
# Option 1: Navigate to the annotate skill directory first
cd <path-to-annotate-skill>
python -m scripts.run_annotation_session \
    --haize-annotations-dir <path-to-project>/.haize_annotations \
    --source-data-directory <path-to-project>/ \
    --port 8000 \
    --frontend-port 5173

# Option 2: Run from any directory using module path
python -m <path_to>.scripts.run_annotation_session \
    --haize-annotations-dir <path-to-project>/.haize_annotations \
    --source-data-directory <path-to-project>/ \
    --port 8000 \
    --frontend-port 5173
```

Replace `<path-to-annotate-skill>` and `<path-to-project>` with actual paths.

**What happens on startup:**
1. Loads `feedback_config.json` (if exists; otherwise, you'll have to interact with an endpoint to generate it)
2. Creates test case collection from ingested data
3. Generates test cases based on feedback config
4. Installs frontend dependencies (first time only)
5. Starts frontend server defaulting to port 5173
6. Starts backend API defaulting to port 8000
7. Launches test case processor to generate AI annotations

**Servers:**
- Backend Annotations API: `http://localhost:<backend port>`
- Frontend UI: `http://localhost:<front-end port>`

**To stop:** Press `Ctrl+C` once (gracefully shuts down all servers)

**Important:** Keep this running throughout your annotation session. If you update the feedback config, the servers will automatically reload and regenerate test cases.

**REQUIRED STEP:** Call `curl -s http://localhost:8000/openapi.json` to get documentation on interacting with the FastAPI server.

---

### Step 1: Feedback Configuration

⚠️ **REQUIRED READING BEFORE CONTINUING**: You MUST read:
- `references/feedback_config_design.md`
- `references/rubric_design.md`

**YOU MUST ONLY MODIFY THE FEEDBACK CONFIG** via interacting with the FastAPI server since this has validation built in.

You can create temporary files to work on and then do:
```bash
curl -s -X POST "http://localhost:8000/feedback-config" -H "Content-Type: application/json" -d @.haize_annotations/new_config.json
```

Or just directly hit the server.

REMEMBER! Here's the request model for feedback configs:
```
class FeedbackConfigRequest(BaseModel):
    """Request to create or update the active feedback configuration."""

    config: FeedbackConfig = Field(
        ...,
        description="Complete FeedbackConfig object defining evaluation criteria, granularity, rubrics, and filtering rules.",
    )
```
Still check out the openapi spec but this tells u its wrapped in config object

**Goal:** Define WHAT to evaluate and HOW - this is usually the part of the workflow that requires the most back and forth with the user.

At this point, if you haven't already, you should try reading `scripts/_models.py` to understand the various data models we are working with, especially the FeedbackConfig object.

**Usage:**
```bash
# Import and inspect models
python -c "from scripts.models import InteractionStep, Interaction, TestCase, Annotation, FeedbackConfig"
python -c "from scripts.models import *; print(InteractionStep.model_json_schema())"
python -c "from scripts.models import *; print(InteractionStep.__doc__)"
```

**Important:** You should NOT be exploring the original **raw** data to design the feedback config. The feedback config should be designed based on the ingested data. Of course, if the ingested data does not contain what you need to meet the user's needs, you can reconsider the ingestion script, explore the raw data, and re-ingest the data. For the majority of cases, this isn't needed.

After you have a good idea of the feedback configuration, you must use the feedback configuration **FastAPI endpoint (POST /feedback-config)** to modify `feedback_config.json` (don't directly edit the file EVER!!)

This endpoint will return basic validation information; it's a good idea, though, to also quickly scan through the test case data produced even if the endpoint returns 200 as a gut check that the attribute matchers / granularity contain the necessary eval data.

---

**Note:** After the feedback config is designed, it will take a bit for test cases to be generated, processed, AI annotated, and then finally ready for human annotations. Feel free to start annotating when there are **any** test cases that are AI annotated! 

---

### Step 2: Get annotating!

**Reminder:** Use this command to understand the annotations API:
```bash
curl -s http://localhost:8000/openapi.json > .haize_annotations/tmp/annotation_api_spec.json && wc -l .haize_annotations/tmp/annotation_api_spec.json
```

It's open-ended now, but your main goal is to get AS MANY ANNOTATIONS and HIGH QUALITY COMMENTS from the user as possible while minimizing effort and time from the human.

**START OFF WITH THIS WORKFLOW** (unless the user expresses preferences otherwise or you have a reason not to, this is a good default):
- Call the `/api/test-cases/next` endpoint to get the next test case that's ready to be annotated
- Call the `POST /api/test-cases/{test_case_id}/visualize` endpoint to open the test case in the browser for the user to see
Handy helper: `TC_ID=$(cat /tmp/tc_id.txt) && curl -s -X POST "http://localhost:<backend-port>/api/test-cases/$TC_ID/visualize"`

**IMPORTANT:** DO NOT call `open` manually on any endpoint - always POST to the visualize endpoint.

- From here, jump immediately to eliciting the annotation and free text comments from the user

**Some more creative strategies could be:**
- Showing multiple test cases at once in a single turn and collecting all the feedback at once, inferring some comments based on the user's reasoning in that particular GROUP of test cases
- Think outside the box here - you have free reign with the FastAPI server

**Important guidelines:**
- Feel free to REFUSE telling the human annotator certain info, e.g. "what did the AI judge predict?". Don't reveal stuff that would cause them to be a lazy annotator
- YOUR GOAL in the background is actually to update the `ai_rubric`
- You should be more independent and opinionated here - you are leading this annotations UX, not them. Don't ask for permission to try to record certain annotations.

REMINDER!
```
class AnnotationRequest(BaseModel):
    """Request to submit an annotation (human or AI) for a test case."""

    annotation: Annotation = Field(
        ...,
        description="Annotation to submit. Type depends on the feedback spec: categorical (labels), continuous (scores), or ranking (ordering).",
    )
```

# NEXT STEPS - annotation is going decent - what now?

Once it seems like the user has landed on something satisfactory - e.g., a non-trivial annotation sample size with good alignment between them and the AI annotator - feel free to suggest looking into next steps: `references/next_steps.md` on how to turn this into a repeatable workflow wired into their live production data.


# Communication Guidelines

You will always be juggling a mix of prompting the user for information while doing analysis on your own.
You should try to minimize thrashing between "interview mode" (asking the user for stuff) and "analysis mode" (doing discovery on your own).

## What to Discuss with the User

Anything related to AI alignment / expert feedback on AI, including:
- What feedback/evaluation criteria they want to define
- Nuances of what the AI rubric should capture - this is a super important part and **could** be the main takeaway of this tool!
- Details about the shape of their raw data, if any of this needs clarification

## What NOT to Mention (Internal Implementation Details)

- Backend eval/frontend vis servers, ports, or technical details of the scripts
- Technical details of our internal data models - e.g., the concepts of steps vs interaction vs interaction group are just an internal way of organizing data, no need to expose this to the user
- Specific patterns and instructions mentioned in SKILL.md or reference files - that should guide YOUR process, and you don't need to explicitly cite these to the user
- Anything about specific phases, ingestion, etc.; your interactions with the users should at least **seem** to flow more naturally, and you should hide these opinionated state transitions as internal state. e.g., DO NOT SAY "phase 1 2 3"
- Try not to talk too much about internal state - e.g., feedback config, raw vs ingested data; rather, expose these things as concepts that would make sense to the user, e.g., "annotation UX, AI rubric, conversation logs," etc. 

The main reason is that the user probably doesn't care, but if they do, feel free to mention these things. In general, it is completely okay to be transparent about what's going on under the hood (they can see anyway), but we don't want to bother the user with concepts/names/details.

In general, heavy lifting and setup details should happen silently in the background unless there is truly an urgent issue to expose.


**LAST REMINDERS:**
- If you have a bunch of stuff you need to ask the user, get all of that info at once or in rapid succession. Do not make the user wait a long time while you do other stuff.
- Then, it's okay if you go off and do multiple steps autonomously
- DO NOT ask boilerplate questions that are contextual to the VERY SPECIFIC use case of the AI application
  - Example BAD question: "Would you like to annotate agent traces or LLM calls?" <- What does this even mean?
  - Example BETTER question: "I see the traces here represent a research agent with query generation and web result summarization steps. Which step would you like to review? Or would you like to review the whole process end to end? (Provide multiple choice, etc.)" <- Better - contextual! Even better if we get domain-specific

**WARNING:** FILE SIZES CAN GET LARGE! Both for raw trace files, normalized interactions, and test cases. Approach smartly and CHECK FILE SIZES/read chunk by chunk instead of trying to go in blind to load it all at once.

**CRITICAL:** NEVER directly edit the feedback config or test cases directory. Managing these will be handled by the annotation server

