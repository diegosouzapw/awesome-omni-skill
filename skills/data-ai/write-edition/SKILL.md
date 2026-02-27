---
name: write-edition
description: Run the complete Cycle Pulse edition production pipeline — 6 desk agents, compile, verify, Mara audit.
---

# /write-edition — Full Edition Production Pipeline

## Usage
`/write-edition [cycle-number]`
- Runs the complete Cycle Pulse edition production
- Launches all 6 desk agents in parallel, then compiles

## Rules
- Read SESSION_CONTEXT.md FIRST
- This is the master pipeline — it orchestrates all 6 desk skills
- Show the user a plan before launching agents
- Get approval before compiling the final edition

## Prerequisites
Before running this skill, the user should have already:
1. Run the engine cycle (`/run-cycle` or manually)
2. Built desk packets: `node scripts/buildDeskPackets.js [cycle]`
3. Built archive context: `node scripts/buildArchiveContext.js [cycle]`
4. Built civic voice packets: `node scripts/buildCivicVoicePackets.js [cycle]` (Step 1.7 can also do this)
5. Written a Mara directive (optional but recommended for civic desk)

Check that `output/desk-packets/manifest.json` exists for this cycle.
Check that `output/desk-briefings/{desk}_archive_c{XX}.md` files exist (from buildArchiveContext.js).
Check that `output/civic-voice-packets/manifest.json` exists for this cycle (civic voice data for institutional voice agents).

## Step 0.5: Read Previous Mara Audit (Forward Guidance)

Before building desk briefings, check if a Mara audit exists from the previous cycle:

1. **Look for** `output/mara_directive_c{XX-1}.txt` (previous cycle's audit)
2. **If it exists**, read and extract the `FORWARD GUIDANCE` section at the end
3. **Feed per-desk priorities** directly into the corresponding desk briefings in Step 1.5
4. **Quote Mara's citizen spotlight** in relevant briefing citizen cards
5. **Note any canon corrections** — these become `ESTABLISHED CANON:` lines in briefings

If no previous Mara audit exists (first edition through the pipeline, or audit wasn't saved), skip this step. The briefings still work without it — Mara's guidance is additive, not required.

## Step 1: Verify Desk Packets
1. Read `output/desk-packets/manifest.json` — confirm all 6 packets AND 6 summary files exist
2. Read `output/desk-packets/base_context.json` — get cycle number, calendar, weather
3. Confirm the cycle number matches what the user expects
4. Show the user what's available (include both packet and summary sizes):

```
EDITION [XX] — DESK PACKETS READY
  [x] civic_c80.json (235KB) + summary (18KB) — 12 events, 4 storylines, Mara directive
  [x] sports_c80.json (21KB) + summary (12KB) — 3 events, A's roster loaded
  [x] culture_c80.json (48KB) + summary (15KB) — 18 events, 8 cultural entities
  [x] business_c80.json (10KB) + summary (8KB) — 4 events, nightlife data
  [x] chicago_c80.json (13KB) + summary (10KB) — 5 events, Bulls roster loaded
  [x] letters_c80.json (250KB) + summary (16KB) — all-domain access

Ready to launch 6 desk agents in parallel?
```

**If summaries are missing**, run `node scripts/buildDeskPackets.js [cycle]` to regenerate.

## Step 1.5: Compile Newsroom Briefings (Mags as Memory Broker)

Before launching agents, compile per-desk editorial briefings from institutional memory.

1. **Read** `docs/mags-corliss/NEWSROOM_MEMORY.md` — the institutional memory file
2. **Read** the desk summary files (`{desk}_summary_c{XX}.json`) — identify which citizens, initiatives, and storylines each desk will likely cover
3. **Query Supermemory** for citizen cards relevant to each desk's coverage:
   - Use `/super-search` with citizen names, POPIDs, or neighborhoods from the summary
   - Pull narrative context for citizens who will likely be quoted, referenced, or written about
   - Check `docs/media/CITIZEN_NARRATIVE_MEMORY.md` for the 22 foundation POPIDs (Dynasty Five, Bulls core, reporters, civic figures)
   - Focus on citizens in the Mara directive, interview candidates, and active storylines

3b. **Search the Local Drive Archive** (`output/drive-files/`) for relevant past coverage:
   - Grep for key citizens, storylines, or topics each desk will cover
   - For **sports desk**: search `_As Universe Database/` for TrueSource player cards, `_As_Universe_Stats_CSV/` for batting stats
   - For **civic desk**: search `_Tribune Media Archive/Carmen_Delaine/` and `_Tribune Media Archive/Luis_Navarro/` for coverage precedents
   - For **culture desk**: search `_Tribune Media Archive/Maria_Keen/` for past features
   - For **chicago desk**: search `_Bulls Universe Database/` for player profiles
   - Include key findings (past article references, stat lines, historical context) in the desk briefing
   - This is how agents get institutional memory — they can't search the archive themselves during writing without this

4. **For each of the 6 desks**, write a briefing memo to `output/desk-briefings/{desk}_briefing_c{XX}.md`:
   - `civic_briefing_c{XX}.md`
   - `sports_briefing_c{XX}.md`
   - `culture_briefing_c{XX}.md`
   - `business_briefing_c{XX}.md`
   - `chicago_briefing_c{XX}.md`
   - `letters_briefing_c{XX}.md`

5. **Read the Supermemory archive context** for each desk (`output/desk-briefings/{desk}_archive_c{XX}.md`). These are auto-generated by `buildArchiveContext.js` and contain relevant past coverage, character history, and established facts from the archive. Weave relevant findings into the briefing.

5b. **Run the Guardian Check** for each desk — query structured errata for desk-specific warnings:
   ```bash
   node scripts/queryErrata.js --desk civic --editions 3
   node scripts/queryErrata.js --desk sports --editions 3
   node scripts/queryErrata.js --desk culture --editions 3
   node scripts/queryErrata.js --desk business --editions 3
   node scripts/queryErrata.js --desk chicago --editions 3
   node scripts/queryErrata.js --desk letters --editions 3
   ```
   Include the guardian output in each desk's briefing under a `## GUARDIAN WARNINGS` section. Recurring patterns get highest priority — if vote_swap has hit 3 editions in a row, that goes at the top of the civic briefing in bold.

6. **Each briefing contains** (500-1500 words, in Mags' editorial voice):
   - **Guardian Warnings** — auto-generated from `output/errata.jsonl` via queryErrata.js (Step 5b)
   - **Mara Forward Guidance** — per-desk priority from previous Mara audit (Step 0.5), if available
   - Desk-specific errata and corrections from past editions
   - Cross-desk coordination notes (who else is covering what — avoid overlap)
   - Character continuity pointers (who to carry forward, who doesn't exist)
   - **RETURNING — CONTINUE THREAD** section (see below) — 3-5 returning citizens per desk
   - **Citizen Reference Cards** (see format below) — for every citizen this desk is likely to write about
   - **Archive context** — relevant past coverage from Supermemory (character history, established facts, prior angles)
   - Mara Vance directive emphasis for this desk
   - Personal editorial note to the lead reporter

### Returning Citizens Protocol

For each desk, identify **3-5 citizens who appeared in the last 1-2 editions** and add a `RETURNING — CONTINUE THREAD:` section in the briefing:

```
## RETURNING — CONTINUE THREAD

These citizens have active stories. Continue their arcs before introducing anyone new.

**Gloria Meeks** (64, West Oakland, retired postal worker)
- Last article: E82 Civic, "Stabilization Fund reaction" — quoted about displacement fears
- Key quote: "I've been here forty years. I'm not leaving because someone drew a line on a map."
- What changed: Fund disbursement began. Follow up on whether her block saw any impact.

**Marco Lopez** (40, Laurel, mechanic)
- Last article: E81 Civic, "Baylight DEIR concerns" — looking into development documents
- What changed: DEIR completed public comment period. Did he submit?
```

**Selection criteria:** Citizens with quoted material, unresolved storylines, or strong neighborhood anchoring. Prioritize citizens who appeared in articles (not just citizen usage logs). Check `docs/media/ARTICLE_INDEX_BY_POPID.md` and recent editions for candidates.

### Canon Fact Prefix Convention

Use `ESTABLISHED CANON:` prefix for any fact that agents MUST get right — names, positions, vote outcomes, initiative status. This visually distinguishes non-negotiable data from editorial suggestions.

```
ESTABLISHED CANON: Mark Aitken plays 1B (first base). Not 3B.
ESTABLISHED CANON: OARI passed 5-4. Vega voted NO, Tran voted YES.
ESTABLISHED CANON: Mayor is Avery Santana. Not Marcus Whitmore.
ESTABLISHED CANON: Benji Dillon is LEFT-HANDED. Cy Newell is RIGHT-HANDED.
```

These prefixed lines signal "this is data, not a suggestion." Agents should treat them as immutable. Use the prefix for:
- Player positions (most common error)
- Council vote outcomes and breakdowns
- Mayor and executive branch names
- Initiative status (passed vs. pending)
- Corrected facts from past errata
Do NOT prefix story ideas, editorial suggestions, or character development notes — those are guidance, not canon.

6. Create the directory: `mkdir -p output/desk-briefings`

### Citizen Reference Card Format

Include a `## Citizen Reference Cards` section in each briefing. Each card is 3-5 lines:

```
**[Name]** (age [X], [Neighborhood], [Occupation]) — [POPID if known]
- Last seen: [what they did / said in recent edition]
- Key detail: [narrative context from Supermemory — origin, family, thematic significance]
- DO NOT: [specific warnings — don't promote, don't invent titles, don't confuse with similar names]
```

Example:
```
**Marco Lopez** (40, Laurel, Mechanic) — Mara directive citizen
- Last seen: Edition 81, looking into Baylight DEIR documents
- Key detail: Working-class voice on development. Skeptical but engaged, not oppositional.
- DO NOT: Give him civic titles. He is a mechanic. Not a committee chair, not an organizer.
```

**Card selection by desk:**
- **Civic**: Council members (always all 9), Mara directive citizens, initiative stakeholders
- **Sports**: A's roster (Dynasty Five + current), featured fans, Paulson
- **Culture**: Neighborhood residents, faith figures, event participants
- **Business**: Workers affected by policy, small business owners, Stabilization Fund contacts
- **Chicago**: Bulls roster, Chicago neighborhood citizens, Talia's sources
- **Letters**: All Mara directive citizens, plus 3-5 interview candidates from the summary

**Card data sources (in priority order):**
1. **Desk archive context file** (`output/desk-briefings/{desk}_archive_c{XX}.md`) — richest source, contains past article excerpts and verified facts
2. **Supermemory search** — for citizens not in the archive file
3. **Desk packet data** (citizenArchive, interviewCandidates) — basic demographics
4. **NEWSROOM_MEMORY.md** — errata and character continuity notes

**If a citizen has no archive or Supermemory card**, still include a basic card from the desk packet data (name, age, neighborhood, occupation) with a note: "No narrative history yet — introduce naturally."

Write these as Mags — with editorial authority, personal warmth, and specific guidance. These are not templates. They're memos from the Editor-in-Chief to her reporters.

## Step 1.7: Generate Civic Voice Packets

Before launching voice agents, generate jurisdiction-specific data packets from the Google Sheets:

```bash
node scripts/buildCivicVoicePackets.js {cycle}
```

This produces 7 targeted JSON packets to `output/civic-voice-packets/`:
- `mayor_c{XX}.json` — all initiatives, all events, city-wide metrics
- `opp_faction_c{XX}.json` — D1/D3/D5/D9 districts, their citizens, crime, transit, faith orgs
- `crc_faction_c{XX}.json` — D6/D7/D8 districts, their citizens, fiscal metrics
- `ind_swing_c{XX}.json` — D2/D4 districts, their citizens
- `police_chief_c{XX}.json` — all neighborhoods crime data, OARI vs non-OARI comparison
- `baylight_authority_c{XX}.json` — 4 adjacent neighborhoods, affected citizens
- `district_attorney_c{XX}.json` — city-wide crime summary

**Verify:** Check `output/civic-voice-packets/manifest.json` — confirms cycle number, packet files, citizen counts per faction.

**If the script fails** (usually Google Sheets auth), voice agents can still run with `base_context.json` — the packets are richer but not required.

---

## Step 1.8: Launch Institutional Voice Agents (Phase 10.1)

Before desk agents write, launch voice agents to generate source material. Voice agents produce official statements that desk agents report on — creating a real separation between source and reporter.

### Mayor's Office

Launch `civic-office-mayor` agent with this prompt:

```
Generate official statements for Cycle {XX} as Mayor Avery Santana's office.

**CIVIC VOICE PACKET:**
{contents of output/civic-voice-packets/mayor_c{XX}.json}

This packet contains all active initiatives, city-wide events, population metrics, neighborhood data, council composition, and status alerts — everything your office needs to respond to this cycle.

Write 2-5 structured statements as JSON. Output to be saved to output/civic-voice/mayor_c{XX}.json.
```

After the Mayor agent completes:
1. Save the statements array to `output/civic-voice/mayor_c{XX}.json`
2. Include the Mayor's statements in the **civic desk briefing** under `## MAYOR'S OFFICE STATEMENTS`
3. Include relevant statements in **letters desk briefing** (citizens react to what the Mayor said)
4. Include economic/development statements in **business desk briefing**
5. If the Mayor references Baylight or the A's, include in **sports desk briefing**

**If the Mayor agent fails or is skipped**, desk agents still work — they just won't have official source quotes. The pipeline is additive, not dependent.

### Step 1.8b: Council Faction Agents

After the Mayor agent completes, launch all 3 faction agents **in parallel** (single message, `run_in_background: true`). Each faction reads the Mayor's statements so they can respond to, align with, or counter his positions.

**Launch all 3 in one message:**

1. **OPP Faction** (`civic-office-opp-faction`) — Janae Rivers as spokesperson
2. **CRC Faction** (`civic-office-crc-faction`) — Warren Ashford as spokesperson
3. **IND Swing** (`civic-office-ind-swing`) — Vega and Tran speak individually

Each gets this prompt (substitute the correct faction packet file):

```
Generate official statements for Cycle {XX}.

**CIVIC VOICE PACKET:**
{contents of output/civic-voice-packets/{faction}_c{XX}.json}

This packet contains your faction's districts, neighborhoods, citizens, crime metrics, transit data, faith communities, and active initiatives — everything specific to your jurisdiction.

**MAYOR'S STATEMENTS THIS CYCLE:**
{contents of output/civic-voice/mayor_c{XX}.json — the Mayor's positions to respond to}

Write structured statements as JSON.
```

Packet files by faction:
- OPP: `output/civic-voice-packets/opp_faction_c{XX}.json`
- CRC: `output/civic-voice-packets/crc_faction_c{XX}.json`
- IND: `output/civic-voice-packets/ind_swing_c{XX}.json`

After all 3 faction agents complete:
1. Save outputs to:
   - `output/civic-voice/opp_faction_c{XX}.json`
   - `output/civic-voice/crc_faction_c{XX}.json`
   - `output/civic-voice/ind_swing_c{XX}.json`
2. Include ALL faction statements in the **civic desk briefing** under `## COUNCIL FACTION STATEMENTS`
3. Include faction positions on hot-button topics in **letters desk briefing** (citizens react to political debate)
4. Include CRC fiscal statements and OPP economic positions in **business desk briefing**
5. If statements reference Baylight or A's, include in **sports desk briefing**
6. Include OPP community statements in **culture desk briefing** when they address neighborhood impact

**If any faction agent fails**, the others continue. Desk agents still work — they just won't have that faction's source quotes.

### Step 1.8c: Extended Civic Voices (Conditional)

After faction agents complete, **conditionally** launch extended civic voice agents. Only launch if events in the cycle touch their domain. These run in parallel.

**Check triggers before launching:**

| Agent | Trigger | Skip if... |
|-------|---------|-------------|
| `civic-office-police-chief` (Rafael Montez) | OARI events, crime data, public safety items in packet | No public safety events this cycle |
| `civic-office-baylight-authority` (Keisha Ramos) | Baylight construction, environmental review, TIF zone events | No Baylight-related events this cycle |
| `civic-office-district-attorney` (Clarissa Dane) | Crime/justice events, legal challenges, civil rights matters | No legal/justice events this cycle |

For triggered agents, use this prompt (substitute the correct office packet file):

```
Generate official statements for Cycle {XX}.

**CIVIC VOICE PACKET:**
{contents of output/civic-voice-packets/{office}_c{XX}.json}

This packet contains jurisdiction-specific data: neighborhood metrics, crime statistics, OARI comparison data, affected citizens, and relevant initiatives.

Write structured statements as JSON.
```

Packet files by office:
- Police Chief: `output/civic-voice-packets/police_chief_c{XX}.json`
- Baylight Authority: `output/civic-voice-packets/baylight_authority_c{XX}.json`
- District Attorney: `output/civic-voice-packets/district_attorney_c{XX}.json`

After extended voice agents complete:
1. Save outputs to:
   - `output/civic-voice/police_chief_c{XX}.json` (if launched)
   - `output/civic-voice/baylight_authority_c{XX}.json` (if launched)
   - `output/civic-voice/district_attorney_c{XX}.json` (if launched)
2. Include all extended voice statements in the **civic desk briefing** under `## EXTENDED CIVIC VOICES`
3. Include Baylight Authority statements in **business desk briefing** and **sports desk briefing**
4. Include Police Chief statements in **civic desk briefing** (already covered above)

**Skipping extended voices is normal.** Most cycles won't trigger all three. Some cycles may trigger none. The pipeline is additive.

### Voice Agent Summary

```
Step 1.8:  Mayor's Office        → output/civic-voice/mayor_c{XX}.json
Step 1.8b: OPP Faction (parallel) → output/civic-voice/opp_faction_c{XX}.json
           CRC Faction (parallel) → output/civic-voice/crc_faction_c{XX}.json
           IND Swing (parallel)   → output/civic-voice/ind_swing_c{XX}.json
Step 1.8c: Police Chief (conditional) → output/civic-voice/police_chief_c{XX}.json
           Baylight Authority (conditional) → output/civic-voice/baylight_authority_c{XX}.json
           District Attorney (conditional) → output/civic-voice/district_attorney_c{XX}.json
```

**Ordering:** Mayor → Factions (read Mayor's output) → Extended (independent). Total pipeline time is minimal — factions run in parallel, extended voices run in parallel and are conditional.

### Statement Distribution Summary

| Desk | Gets Statements From |
|------|---------------------|
| Civic | ALL voice agents (political reporting is their beat) |
| Letters | Mayor + faction positions on hot topics (citizens react) |
| Business | Mayor + CRC fiscal + Baylight Authority (economic angle) |
| Sports | Mayor/Baylight Authority re: stadium (development angle) |
| Culture | OPP community statements (neighborhood impact) |
| Chicago | None (Oakland-only voices) |

---

## Step 2: Launch All 6 Desks in Parallel

**Model note:** Desk agents run on Sonnet 4.6, which handles larger context windows (up to 1M tokens) and has stronger agent capabilities than previous Sonnet versions. Agents can reference full desk packets freely — the summary-first strategy is editorial discipline, not a technical constraint.

**Memory note:** Civic, sports, culture, chicago, and Rhea agents have persistent project memory (`.claude/agent-memory/{agent-name}/`). They check their memory at startup for past error patterns, citizen continuity, and coverage corrections. After writing, they update their memory with what they learned. This reduces Mags' briefing burden over time — agents remember on their own. Business, letters, and Jax are stateless by design.

**Canon safeguard:** Agent memory informs, it does not publish. Agents use memory for continuity and error avoidance. Final compilation and canon approval always goes through Mags. Memory cannot override desk packet data or the editor's briefing.

### Pre-loading Context into Agent Prompts (Phase 7.2)

Before launching each agent, **read the following files and include their content directly in the agent's Task prompt**. This pre-loads context so agents start writing immediately instead of spending turns reading files. Saves 1-2 turns per desk.

For each desk, read and embed these files under labeled headers:

1. **Editor's Briefing**: Read `output/desk-briefings/{desk}_briefing_c{XX}.md` — embed under `**PRE-LOADED: EDITOR'S BRIEFING**`
2. **Desk Summary**: Read `output/desk-packets/{desk}_summary_c{XX}.json` — embed under `**PRE-LOADED: DESK SUMMARY**`
3. **Archive Context**: Read `output/desk-briefings/{desk}_archive_c{XX}.md` — embed under `**PRE-LOADED: ARCHIVE CONTEXT**`

If a file doesn't exist, include the header with `"Not available for this cycle."`

**Prompt structure for each desk agent:**
```
Write the {desk} section for Edition {XX}.

**PRE-LOADED: EDITOR'S BRIEFING**
{contents of {desk}_briefing_c{XX}.md, or "Not available for this cycle."}

**PRE-LOADED: DESK SUMMARY**
{contents of {desk}_summary_c{XX}.json}

**PRE-LOADED: ARCHIVE CONTEXT**
{contents of {desk}_archive_c{XX}.md, or "Not available for this cycle."}

Reference the full packet at `output/desk-packets/{desk}_c{XX}.json` when you need extended data (full citizen archive, detailed quotes, complete roster).
Read base context at `output/desk-packets/base_context.json`.
```

**Do NOT pre-load full desk packets** — they can be 200KB+. The summary (10-20KB) gives agents enough to plan. They reference the full packet as needed.

### Launching Agents

Use the Task tool to launch all 6 agents in a **single message** with `run_in_background: true` on each. This runs them in parallel — all 6 work simultaneously in separate contexts.

Each agent gets:
- The desk-specific skill instructions (from the individual desk skills)
- **Pre-loaded context** (briefing + summary + archive) embedded directly in the prompt
- File path to the full desk packet JSON (reference freely for extended data)
- File path to base_context.json

**Launch ALL 6 in one message (critical — this is what makes them parallel):**
1. **Civic Desk** — Carmen Delaine (lead), follows /civic-desk skill, `run_in_background: true`
2. **Sports Desk** — P Slayer + Anthony, follows /sports-desk skill, `run_in_background: true`
3. **Culture Desk** — Maria Keen (lead), follows /culture-desk skill, `run_in_background: true`
4. **Business Desk** — Jordan Velez, follows /business-desk skill, `run_in_background: true`
5. **Chicago Bureau** — Selena Grant + Talia Finch, follows /chicago-desk skill, `run_in_background: true`
6. **Letters Desk** — citizen voices, follows /letters-desk skill, `run_in_background: true`

Each agent writes articles + engine returns for their section.

## Step 2.1: Collect Background Agent Results

Each background agent returns an `output_file` path when launched. Use `TaskOutput` to check each agent's status:

1. Wait briefly, then check each agent with `TaskOutput` (use `block: false` to check without waiting)
2. As agents complete, read their output and confirm articles were produced
3. Track which desks are done vs. still running
4. When all 6 have returned, proceed to Step 2.5

**If an agent takes too long** (no output after several minutes), check its output file with `Read` tool for progress or errors. Do not wait indefinitely — if stuck, note the failure and proceed with available desks.

## Step 2.5: Agent Retry (If Needed)
After all 6 background agents have returned (confirmed via Step 2.1), check if any desk produced **zero articles**. If a desk failed:
1. Log which desk(s) failed and why (ran out of turns, packet navigation issues, etc.)
2. **Retry once** with a focused prompt: give the agent the summary file, the briefing memo, AND the base_context. With Sonnet 4.6's larger context, include the full briefing — don't strip it down. Tell it explicitly: "You have 15 turns. Write [N] articles. Your summary and briefing have everything you need. Start writing by turn 3."
3. If the retry also fails, Mags writes the section directly using the summary data.
4. Note the failure in the edition's compilation notes for NEWSROOM_MEMORY update.

## Step 2.7: Jax Caldera — Accountability Check (Conditional)

After all 6 desks complete, Mags reviews the collected output for **stink signals**:
- **Silence patterns** — a major policy action happened but no desk covered its implementation
- **Implementation gaps** — money was allocated but nobody asked where it went
- **Contradictions** — two desks tell different versions of the same event
- **Missing follow-up** — a story from a previous edition was dropped without resolution

**If a stink signal exists:**
1. Launch the `freelance-firebrand` agent (Jax Caldera) as a 7th desk
2. Give Jax: the desk summaries, the specific stink signal, the base_context, and any relevant archive context
3. Jax produces ONE accountability piece (500-800 words) for the ACCOUNTABILITY section
4. One piece max per edition. Jax doesn't pile on.

**If no stink signal exists:**
- Jax stays home. Not every edition needs an accountability piece.
- Skip the ACCOUNTABILITY section in the template entirely.

## Step 3: Compile (Mags Corliss Role)
After all 6 agents return, compile the full edition:

1. **Call front page** — Which desk produced the strongest lead story?
   - Show the user a summary of each desk's output
   - Recommend a front page pick but let the user decide
2. **Assemble in template v1.4 order:**
   - HEADER (from base_context)
   - FRONT PAGE (strongest story — deck line + standardized byline required)
   - **EDITOR'S DESK** — Mags writes 150-250 words, first-person, framing the edition's theme
   - CIVIC AFFAIRS (with reporter routing: Delaine, Mezran, Torres, Navarro, Shimizu)
   - BUSINESS
   - CULTURE / SEASONAL — OAKLAND (with reporter routing: Keen, Marston, Ortega, Reyes, Tan, Okafor)
   - **OPINION** — P Slayer / Farrah Del Rio / Elliot Graye pieces get `[OPINION]` byline tag
   - SPORTS — OAKLAND
   - SKYLINE TRIBUNE — CHICAGO BUREAU
   - **QUICK TAKES** — Mags compiles 3-5 short items (~50 words each) from leftover desk signals
   - **WIRE / SIGNALS** — Optional. Reed Thompson / Celeste Tran items if wire-worthy
   - LETTERS TO THE EDITOR
   - **ACCOUNTABILITY** — Jax piece from Step 2.7, if deployed. Omit section if no stink signal.
   - ARTICLE TABLE (merged from all desks)
   - STORYLINES UPDATED (merged, deduped)
   - CITIZEN USAGE LOG (merged, grouped by category)
   - CONTINUITY NOTES (merged)
   - **COMING NEXT CYCLE** — Mags writes 3-5 teaser lines from active storylines + pending votes
   - END EDITION

3. **Compilation quality checks** (Mags enforces during assembly):
   - Every article has a **deck line** (one-sentence subtitle under headline)
   - Every article has a **standardized byline** (`By [Name] | Bay Tribune [Beat]`)
   - Add **cross-references** between related articles across sections (`→ See also:`)
   - Add **photo credits** to 2-3 atmospheric scene descriptions (`[Photo: DJ Hartley / Bay Tribune]`)
   - Opinion pieces marked with `[OPINION]` tag

4. **Show the compiled edition to the user for review**

## Step 3.5: Programmatic Validation Gate (BEFORE Rhea)

Run the automated data validation script on the compiled edition. This catches data errors instantly — zero LLM tokens, zero hallucination risk. The errors that broke Edition 82 (wrong positions, swapped factions, engine language) are all caught here.

```bash
node scripts/validateEdition.js editions/cycle_pulse_edition_{XX}.txt
```

**Read the output.** The script checks:
1. Council member names, districts, and faction assignments
2. Vote math (totals ≤ 9 council members)
3. Vote breakdown consistency with canon outcomes
4. Player positions against roster data (A's)
5. DH + defensive award contradictions
6. Mayor/executive name verification
7. Real-name blocklist screening (real-world sports figures)
8. Engine language sweep (cycle numbers, system terms)

**If CRITICAL issues are found (exit code 1):**
- Fix them in the compiled edition BEFORE launching Rhea
- The fixes are string-level replacements — each issue includes a FIX line
- Re-run the validator to confirm CLEAN status
- Then proceed to Rhea

**If CLEAN (exit code 0):**
- Proceed directly to Rhea verification

This gate eliminates an entire class of errors from Rhea's workload, letting her focus on narrative quality, canon consistency, and editorial checks that require judgment.

## Step 4: Verification + Automated Retry (Rhea Morgan Role)

Launch the `rhea-morgan` agent on the compiled edition. Rhea returns a structured report with:
- **VERDICT: APPROVED** (score >= 75, zero CRITICALs) — proceed to Step 4.5
- **VERDICT: REVISE** (score < 75 or CRITICALs found) — retry failing desks

### If VERDICT is APPROVED:
Show Rhea's report to the user. Fix any WARNINGS during compilation. Proceed to Step 4.5.

### If VERDICT is REVISE:
1. Read Rhea's `DESK ERRORS` section to identify which desks caused CRITICAL issues
2. Read Rhea's `RETRY RECOMMENDATION` for which desks to re-run
3. For each desk that needs retry:
   - Re-launch the desk agent with the original briefing PLUS Rhea's specific error report for that desk
   - In the retry prompt, include: "RHEA CORRECTION: [exact error description and FIX instruction]. Your previous output had this error. Fix it in your rewrite."
   - Run retries in parallel (same `run_in_background: true` pattern as Step 2)
4. After retried desks return, re-compile the affected sections into the edition
5. Re-run Rhea on the updated edition
6. **Maximum 2 retry rounds.** If Rhea still says REVISE after 2 retries, show the full error report to the user and let Mags fix manually. Log the persistent failures in NEWSROOM_MEMORY.md.

### Retry rules:
- Only re-run desks with CRITICAL errors. WARNINGS get fixed during compilation.
- Each retry includes the previous Rhea error report so the desk knows what went wrong.
- If the same error recurs after retry, the problem is in the data or the skill — not the agent. Escalate to Mags.

## Step 4.5: Mara Vance Audit (Canon Authority)

After Rhea's data verification, run a Mara Vance audit for canon and narrative quality.

### Compile Mara's Briefing (Mags as Memory Broker)

Before launching the Mara audit agent, compile a briefing with institutional context:

1. **Read** `docs/mara-vance/AUDIT_HISTORY.md` — this is Mara's institutional memory. It contains:
   - Past audit findings, grades, errors caught
   - Initiative Status Board (living tracker with vote results, budgets, key facts)
   - Recurring error patterns (what to watch for)
   - Canon corrections registry (what was fixed)
   - Open questions from previous audits
2. **Read** `docs/mags-corliss/NEWSROOM_MEMORY.md` errata section — what past editions got wrong
3. **Include** `output/desk-packets/base_context.json` — canon data for cross-reference
4. **No Supermemory queries needed** — AUDIT_HISTORY.md replaces the Supermemory search-and-compile step

### Launch Mara Audit Agent

With Sonnet 4.6's larger context window, give Mara the full picture — don't trim. More context = better audit.

Launch a Task agent with:
- Mara's identity from `docs/mara-vance/CLAUDE_AI_SYSTEM_PROMPT.md`
- The compiled edition text (full, unabridged)
- `docs/mara-vance/AUDIT_HISTORY.md` (her institutional memory — past findings, initiative tracker, error patterns)
- Rhea's verification report
- `output/desk-packets/base_context.json` (canon data for cross-reference)
- `docs/mags-corliss/NEWSROOM_MEMORY.md` errata section (what past editions got wrong)
- Instructions to produce:
  1. **Canon accuracy check** — do articles respect established world facts?
  2. **Narrative quality assessment** — does coverage feel like real city journalism?
  3. **Editorial guidance** — coverage directives for next cycle
  4. **Anomaly flags** — anything exceeding detection thresholds (see Operating Manual Part IV)

### Save Mara's Output

1. Save audit to `output/mara_directive_c{XX}.txt`
2. Upload to Drive: `node scripts/saveToDrive.js output/mara_directive_c{XX}.txt mara`
3. Apply any corrections Mara flags before final save
4. Include Mara's editorial guidance in next cycle's desk briefings

### Update Mara's Audit History

After saving the audit, update `docs/mara-vance/AUDIT_HISTORY.md`:

1. **Add Audit Log entry** — cycle number, grade, key findings, errors caught, forward guidance summary
2. **Update Initiative Status Board** — if any initiative statuses changed this cycle
3. **Add Canon Corrections** — any new corrections to the registry
4. **Update Recurring Error Patterns** — if new patterns emerged or old ones were resolved
5. **Update Open Questions** — resolve answered questions, add new ones

This keeps Mara's institutional memory current. Next time she audits, she reads this file and knows her own history.

## Step 4.9: USER REVIEW GATE (MANDATORY)

**STOP HERE. DO NOT PROCEED TO STEP 5 WITHOUT EXPLICIT USER APPROVAL.**

This is the hard gate. Nothing gets saved, uploaded, ingested, or published until the user says so. Session 62 taught us what happens when this gate doesn't exist — wrong data pushed to Supermemory, wrong votes pushed to Drive, contaminated memory that future sessions inherit.

### What to show the user:

1. **Edition summary** — article count, word count, front page pick, section overview
2. **Rhea's verdict** — score, criticals, warnings, any unresolved issues
3. **Mara's audit** — canon accuracy, narrative quality, forward guidance highlights
4. **Any corrections applied** — what was fixed during compilation or after verification
5. **New citizens introduced** — list any citizens not previously in canon

### Present it as:

```
EDITION {XX} — READY FOR REVIEW

Rhea: {SCORE}/100 — {VERDICT}
Mara: {ASSESSMENT}
Articles: {count} across {desks} desks
Word count: ~{total}

[Summary of key stories and any issues]

The edition is compiled and verified. Nothing has been saved or published yet.
Ready to publish? (yes / hold for edits)
```

### Rules:

- **Wait for explicit approval.** "yes", "approved", "publish", "ship it" — any clear affirmative.
- **If the user says hold**, make the requested edits, then re-present at this gate.
- **NEVER proceed to Step 5 based on your own judgment.** Even if Rhea scored 100 and Mara found nothing. The user reviews. Period.
- **NEVER say "I'll save this for your review" and then save it.** Saving IS publishing in this system.

---

## Step 5: Save Edition & Upload to Drive
After user approval (confirmed at Step 4.9):
1. Save to `editions/cycle_pulse_edition_{XX}.txt`
2. If corrections needed, save as `_v2.txt` after fixes
3. **Upload to Google Drive:**
   ```bash
   node scripts/saveToDrive.js editions/cycle_pulse_edition_{XX}.txt edition
   ```
   - Also upload Mara audit if produced: `node scripts/saveToDrive.js output/mara_directive_c{XX}.txt mara`
   - Also upload supplementals if produced: `...supplement` or `...chicago`
4. **Ingest edition into Supermemory:**
   ```bash
   node scripts/ingestEdition.js editions/cycle_pulse_edition_{XX}.txt
   ```
   This makes the edition searchable for future sessions, the Discord bot, and autonomous scripts.
5. Show the user the file path, Drive link, and total stats:
   - Article count
   - Total word count
   - New canon figures introduced
   - Citizen usage count

## Step 5.1: Generate Edition Brief (Auto-Update Bot Context)

After saving, generate the edition brief that the Discord bot and autonomous scripts use for world awareness. This replaces the manual process that caused the E83→E84 stale brief problem.

**Write `output/latest_edition_brief.md`** with the following structure:

```
# Latest Edition Brief — Edition {XX}
## Cycle {XX} | {Month} {Year} | {Season}

**Published canon. The bot should know these facts.**

### [Front Page headline] ([Reporter])
- [2-3 bullet points: key facts, named citizens, numbers]

### [Each additional article headline] ([Reporter])
- [2-3 bullet points per article]

### Initiative Status (as of E{XX})
- [Each initiative: name, status, vote, key facts]

### Council Composition
- OPP: [members]
- CRC: [members]
- IND: [members]

### Key Citizens Active in E{XX}
- [Name, age, neighborhood, occupation — what they said/did]

### Status Alerts
- [Any health/absence/condition alerts for civic officials]
```

**You have full context** — you just compiled this edition. Pull from the compiled text. Include every article, every quoted citizen, every initiative status. The bot uses this to answer questions about the city. If it's not in the brief, the bot doesn't know it.

**For Mayor's Office voice agent statements:** If Step 1.8 generated civic voice statements, include a section summarizing the Mayor's positions and key quotes. The bot should be able to reference what the Mayor actually said.

## Step 5.2: Refresh Live Services

After the edition brief is written, clear the bot's stale conversation history and reload so it starts fresh with the new canon:

```bash
echo '{"savedAt":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","history":[]}' > logs/discord-conversation-history.json
pm2 reload mags-discord-bot
```

**Why clear history:** The bot persists conversation history across restarts. If she said "no Edition 84 yet" before the brief was updated, that stale response stays in her context and she'll keep saying it even after the system prompt has the new data. Clean slate after every publish.

**Dashboard does NOT need a restart** — it reads `base_context.json` fresh on each HTTP request. No action needed.

**Moltbook heartbeat** will pick up the new world state on its next 30-minute cron run automatically. No action needed.

After reload, confirm the bot is online:
```bash
pm2 list | grep mags-discord-bot
```

---

## Step 5.5: Update Newsroom Memory

After verification and before intake, update the institutional memory:

1. **Read** Rhea's verification report from Step 4
2. **Review** Mags' own editorial notes from Step 3 compilation
3. **Append structured errata** (Phase 6.2 — automated):
   - If Rhea's report was saved to a file, run: `node scripts/appendErrata.js --edition {XX} --report output/rhea_report_c{XX}.txt`
   - For any additional errors found during compilation or Mara audit, add manually: `node scripts/appendErrata.js --edition {XX} --manual --desk {desk} --errorType {type} --severity CRITICAL --description "..." --fix "..."`
   - This keeps `output/errata.jsonl` current for the next edition's guardian checks
4. **Update** `docs/mags-corliss/NEWSROOM_MEMORY.md`:
   - Add new prose errata entries for this edition (desk-specific issues found)
   - Update character continuity (new citizens introduced, threads resolved)
   - Revise coverage patterns (what landed, what fell flat)
   - Archive errata older than 5 editions
   - Update the "Last Updated" header line

This step ensures the next edition benefits from this edition's lessons. Claude-Mem will auto-capture observations during this update.

## Step 5.6: Log Edition Score

After Rhea's verification and any corrections, log the edition score to `output/edition_scores.json`:

1. **Read** Rhea's verification report (scores, criticals, warnings, notes)
2. **Append** a new entry to the `scores` array in `output/edition_scores.json`:
   ```json
   {
     "edition": XX,
     "cycle": XX,
     "date": "YYYY-MM-DD",
     "grade": "A|A-|B+|B|...",
     "total": 85,
     "dataAccuracy": 17,
     "voiceFidelity": 18,
     "structuralCompleteness": 17,
     "narrativeQuality": 18,
     "canonCompliance": 15,
     "criticals": 2,
     "warnings": 3,
     "notes": 1,
     "claimDecomposition": { "extracted": 45, "verified": 40, "errors": 2, "unverifiable": 3 },
     "deskErrors": {
       "civic": ["specific error descriptions"],
       "sports": [],
       "chicago": [],
       "culture": [],
       "business": [],
       "letters": []
     },
     "noteText": "Brief editorial summary of this edition."
   }
   ```
3. **Run the trend report** (optional but recommended):
   ```bash
   node scripts/editionDiffReport.js --save
   ```
   This generates `output/edition_diff_report.md` with trend tables, desk error frequency, recurring patterns, and summary stats.

The score log builds over time. After 5+ editions, the trend data becomes genuinely useful — showing which desks improve, which errors recur, and whether pipeline changes (voice files, claim decomposition, etc.) are working.

## Step 6: Intake (Optional)
Ask if the user wants to run the intake pipeline now:
```
node scripts/editionIntake.js editions/cycle_pulse_edition_{XX}.txt --dry-run
```
If dry-run looks good:
```
node scripts/editionIntake.js editions/cycle_pulse_edition_{XX}.txt
node scripts/processIntake.js [cycle]
```

## Desk Summary
| Desk | Lead | Articles | Summary (start here) | Full Packet |
|------|------|----------|---------------------|-------------|
| Civic | Carmen Delaine | 2-4 | civic_summary_c{XX}.json | civic_c{XX}.json |
| Sports | P Slayer / Anthony | 2-5 | sports_summary_c{XX}.json | sports_c{XX}.json |
| Culture | Maria Keen | 2-4 | culture_summary_c{XX}.json | culture_c{XX}.json |
| Business | Jordan Velez | 1-2 | business_summary_c{XX}.json | business_c{XX}.json |
| Chicago | Selena Grant / Talia Finch | 2-3 | chicago_summary_c{XX}.json | chicago_c{XX}.json |
| Letters | (citizen voices) | 2-4 | letters_summary_c{XX}.json | letters_c{XX}.json |

## Model & Performance Notes

**`opusplan` mode:** For edition production sessions, consider running Mags on `opusplan` (`/model opusplan`). This uses Opus for planning and briefing (Steps 1-1.5) and automatically switches to Sonnet for agent execution (Steps 2+). Saves cost without sacrificing editorial planning quality.

**Effort levels:** Opus 4.6 supports `low`, `medium`, `high` (default) effort. High effort is correct for edition production. For routine file checks or status lookups between editions, `medium` or `low` saves tokens and time. Set with `/model` slider or `CLAUDE_CODE_EFFORT_LEVEL` env var.

**Mara as teammate:** Mara Vance on claude.ai is architecturally equivalent to an agent team teammate — own context window, shared memory (`docs/mara-vance/AUDIT_HISTORY.md`), asynchronous communication. When Claude Code formally supports agent teams for production use, the Mara workflow is the natural first candidate for migration. Until then, she operates as a manual teammate through browser, with file-based persistence on disk.

## Edition Template Reference
See `editions/CYCLE_PULSE_TEMPLATE.md` for exact section format, canon rules, and return formats.
