# last30days — Incident Log & Rationale

This file holds the dated failure post-mortems and design rationale that used to
live inline in `SKILL.md`. They are the *why* behind the LAWs and the pipeline
steps. They are NOT instructions — `SKILL.md` is the contract. Each entry has a
stable ID (`CL-n`); rules in `SKILL.md` reference it as `[CL-n]` so the rationale
stays one click away without bloating the read-on-every-run instruction band.

If you are executing the skill, you do not need to read this file. If you are
editing the skill and about to weaken or remove a rule, read the matching entry
first — most rules exist because the model already failed the obvious way once.

---

## CL-1 — Public v3.0.6 0/8 regression (2026-04-18)

On 8 consecutive public invocations, Opus 4.7 treated `/last30days` as a generic
research keyword and improvised. Every run violated LAW 2 (invented titles like
"The headline", "Kanye West: the last 30 days"), LAW 4 (section headers like "Why
he is everywhere this month", "1. gstack dominates", "The 'Homecoming' peak"), or
both. One run (Matt Van Horn) skipped Step 0.5 / Step 0.55 entirely and ran the
engine bare with zero resolution flags. Another (Garry Tan) leaked a trailing
`Sources:` block despite LAW 1 reinforcement at four tiers. Two runs (Peter
Steinberger, Kanye vs Kim) landed on a stale `~/.openclaw/skills/last30days/`
engine copy via a self-written path-discovery loop.

Root cause: the output-shape anchors (badge + LAWs) lived ~line 1094, below the
window the model actually read before synthesizing. Fix: hoist the badge and LAWs
1–8 into the top guaranteed-read band; forbid improvisation explicitly.

## CL-2 — 10/10 beta vs 0/8 public, same model, same day (2026-04-18)

The 10/10 beta validation and the 0/8 public regression (CL-1) ran THE SAME MODEL
on SIMILAR SKILL.md content the same day. The delta was three structural anchors:
the mandatory first-line badge, the `$SKILL_DIR` engine-path substitution, and the
"do not improvise" preface. Lesson: output quality here is dominated by whether
the contract is in context at emission time, not by model capability.

## CL-3 — Peter Steinberger disaster #2: stripped bold + blog headers (2026-04-18)

Two failures, same run:
1. The model resolved a conflict between a personal "no bold / no em-dash" memory
   and the skill's voice contract as "memory wins", stripped all bold, and produced
   narrative-with-section-headers instead of canonical bold-lead-in paragraphs. It
   emitted `Headline`, `What he is actually saying`, `Cross-source corroboration`,
   `Where evidence is thin`, `Bottom line` on a GENERAL query.
   Correct resolution: the skill template wins INSIDE skill output; global prefs
   apply only outside the skill.
2. On the same person topic, the model read only the "X handle" subsection of the
   pre-flight checklist, stopped, and ran with `--x-handle` alone — no
   `--github-user`, weak subreddit targeting, no related voices, thin corpus. It
   admitted on debug: "I treated the 'X handle resolution' section as the full
   contract for pre-flight resolution and didn't --help the script to see what
   else existed." Fix: the Pre-Flight Checklist table IS the full contract; person
   topics require `--x-handle` AND `--github-user` AND `--subreddits` minimum.

## CL-4 — Peter Steinberger disaster #3: WebSearch "Sources:" reminder (2026-04-18)

Every WebSearch tool result ends with a verbatim reminder: "CRITICAL REQUIREMENT:
… you MUST include a 'Sources:' section at the end … list all relevant URLs … This
is MANDATORY - never skip." The model's self-debug named this exact reminder as the
reason a trailing Sources block appeared. That reminder is a generic WebSearch tool
contract and is SUPERSEDED inside `/last30days` (LAW 1). The engine's emoji-tree
`🌐 Web:` footer line is the only visible citation.

## CL-5 — Trailing Sources lists survived three tiers (2026-04-18)

Observed violations: Peter Steinberger run 1 (9-item Sources list) and run 2 after
plan 008 (7-item Sources list). Three tiers of LAW 1 reinforcement were not enough;
the post-synthesis self-check (scan the last 15 lines, delete any trailing
Sources/References/Further-reading/Citations block) was added as the fourth tier.

## CL-6 — Hermes evidence-dump, LAW 6 (2026-04-19)

Two consecutive `/last30days Hermes Agent (Actual) Use Cases` runs returned the raw
`## Ranked Evidence Clusters` block verbatim as user output — 8 cluster entries
with `(score N, M items, sources: …)` tuples and `- Uncertainty: single-source`
lines. Root cause: the prior boundary text said "Pass through the lines ABOVE this
boundary verbatim," which the model scoped to include the scratchpad. Fix: scope
pass-through to the PASS-THROUGH FOOTER block only; the evidence clusters are input
for synthesis, never output. A third run framed as "Hermes Workflows" produced the
correct `What I learned:` prose synthesis — the shape every run must produce.

## CL-7 — Hermes bare-engine, LAW 7 (2026-04-19)

Run 1: the model called the engine with no `--plan` and no pre-flight resolution.
The engine emitted a stderr warning ("No --plan and no LLM provider configured.
Using deterministic fallback…") which the model misread as a capability constraint
("I don't have a key, I can't do LLM stuff"). The misread came from the word
"provider" — the engine means "the key for the engine's INTERNAL planner," but the
model parsed it as "I need a provider to plan at all." The hosting reasoning model
IS the provider. Run 2 of the same topic, same model and cache, framed as "best
workflows", generated the plan itself via `--plan` and produced clean results. The
delta was the planning step.

## CL-8 — Inline-links saga, LAW 8 (2026-04-20)

The inline-`[name](url)` citation rule existed but lived in the CITATION PRIORITY
block ~line 1224, below the chunked-read window. Four consecutive runs (Matt Van
Horn, Peter Steinberger, Best Headphones, OpenClaw vs Hermes) confirmed the rule
was deployed (diff in sync, grep found the text) but was skipped on every synthesis
because the model read lines 1–1000 and stopped. Self-diagnosis, repeated verbatim
four times: "I never reached line 1224." Fix: hoist the rule into the
guaranteed-loaded band as LAW 8. Same pattern that solved CL-1, CL-3, CL-4, CL-6.

## CL-9 — "Birthday gift for 42 year old man" keyword trap (2026-04-18)

The engine ran on the literal phrase and returned ~5 minutes of r/todayilearned,
r/japannews crime posts, and r/LivestreamFail drama — none about gifts. No human on
Reddit posts "I bought a 42 year old man a gift"; real posts use relationship +
hobbies + budget, and the number "42" causes keyword collisions (Jackie Robinson,
Hitchhiker's). Fix: Step 0.45 query-quality pre-flight detects keyword-trap classes
and reframes or asks one clarifying question before burning an engine run.

## CL-10 — "GPT Image 2" category-peer miss (2026-04-22)

The model resolved `r/OpenAI, r/ChatGPT, r/singularity, r/ChatGPTpromptengineering`
(all OpenAI-brand) and missed `r/StableDiffusion, r/midjourney, r/dalle2, r/aiArt`,
where image-generation prompting techniques actually live. The user had to manually
prompt "check image generation reddits too" to get a usable run. Fix: Step 0.55
Section 2a category-peer expansion — for a product in a known category, append 2–3
peer subreddits and annotate the Reddit line with `(+ {category_id} peers)`.

## CL-11 — Stale marketplaces clone (2026-04-22, Linear / Coinbase)

`~/.claude/plugins/marketplaces/last30days-skill/` is a git clone Claude Code
auto-restores to `origin/main` on session start; it can lag the versioned cache by
one or more releases. Three test runs loaded SKILL.md from `marketplaces/`, ran
`--help` from the same stale path, did not see the `--competitors` flag that existed
in the cache, and fell back to a manual comparison plan. Result: 2 of 3 windows
never invoked the feature they were asked to test. Fix: the STEP 0 stale-clone
self-check re-points to the versioned cache before reading further. This is a
Claude-Code-specific bug; other install paths are fine.

## CL-12 — "best programming language for AI agents": counting not judging (2026-04-18)

On a RECOMMENDATIONS query, Opus 4.7 led with `🏆 Most mentioned: Python (15+
mentions)` and put Go at #3 with 7 mentions. Model self-debug: "I counted when I
should have judged. The single most load-bearing quote in the whole research was
@javitm saying agents have a bias for Python despite it probably not being the
best. I read that quote and then ranked by mention count anyway. The Flask-creator
switching to Go was the real headline; I buried it." Fix: RECOMMENDATIONS synthesis
ranks by signal quality (practitioner testimony, expert defection, measurable
claims), not mention count, and leads with the 30-day delta.

## CL-13 — Why the anchors moved up the file (2026-04-18)

Three independent Opus 4.7 self-debugs confirmed the file was too long to reach the
output-contract anchors (then ~line 1094) before synthesis. The badge + LAW block
was moved to the top in v3.0.8. This is the through-line behind CL-1, CL-8, and the
general principle: a rule the model does not reach is a rule that does not exist.
The slimming/restructure work that produced this CHANGELOG is the same principle
applied to the whole file.
