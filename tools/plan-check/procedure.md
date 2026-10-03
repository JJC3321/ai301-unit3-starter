# Procedure: how this skill grades a plan package

## Read order

1. Confirm mode. In live mode, read `scope.md` first and refuse any
   issue outside the scoped repo; note Path Review house rules. In
   eval mode, skip `scope.md` and treat the bundle as the whole world.
2. Read `rubric.md` and list every check name, its evidence column,
   its pass condition, its weight, and the verdict rule (including how
   `unclear` is treated).
3. Read `references/evidence-guide.md` so each evidence family has a
   known home before gathering starts.
4. Read the package in this order, and note one line from each part
   before grading anything:
   1. Issue context (title, body excerpt, expected behavior).
   2. Thread highlights (any OWNER/maintainer direction, culprit file,
      patched binary, preferred approach).
   3. Repo-facts block (contribution policy / AI disclosure rule).
   4. Repro-evidence block (environment, steps, artifacts, controls,
      expected vs actual). Note what behavior the evidence pins down
      and what any control run rules out.
   5. Candidate plan (diagnosis, scope, files, approach, test plan,
      risks/unknowns).
   6. Candidate plan comment.
5. Why this order: later checks compare the plan's cause and tests to
   the repro, and the comment to the thread/policy. Reading the plan
   first invites grading polish instead of grounding.

## Evidence gathering

For each family below, pull the fact from the place named and record a
short quote or concrete fact (not a paraphrase of "looks fine"):

1. **Diagnosis and grounding** — From the plan's diagnosis/cause
   statement; from the repro-evidence failing artifact and every
   control run. Record: stated cause; what the failing artifact shows;
   what each control shows.
2. **Scope** — From the plan's in/out scope lines, change list, and
   named files/areas. Record: what is in, what is out, and whether
   extra campaigns (migrations, rewrites, new options) appear.
3. **Executability** — From the plan's files/areas and approach steps.
   Record: named path(s) and the chosen approach (or note absence).
4. **Test plan** — From the plan's test section; map each claimed check
   onto a repro step or artifact. Record: the observable success
   signal (or note that none is named).
5. **Honesty** — From risks/unknowns/deviations and any certainty
   claims in the plan. Record: labeled unknowns vs asserted facts.
6. **Comms / thread / AI** — From thread highlights (maintainer
   direction) and repo-facts AI/contribution policy; from the plan
   comment's words. Record: whether a direction exists; whether the
   comment engages it; whether disclosure is required and present.

Live mode only: gather issue/thread/policy via `gh` or the issue page
per the evidence guide; take repro evidence from the student's posted
repro comment (or house repro pack quotes in the drafts). Grade only
what the drafts contain and quote.

Eval mode only: quote from the bundle sections; do not fetch.

## Check execution

1. Grade checks in this fixed order so two executors match:
   `diagnosis-grounded`, `scope-bounded`, `executable`,
   `test-decisive`, `thread-aware`, `ai-disclosure`,
   `honest-unknowns`.
2. For each check, use only the gathered evidence that check's Evidence
   column names. Apply the Pass condition as a yes/no on the outcome,
   not on writing quality or section count.
3. Assign exactly one grade:
   - `pass` — the pass condition is met by a named fact/quote.
   - `fail` — the pass condition is not met; quote the contradicting
     fact.
   - `unclear` — the evidence the check needs is genuinely absent from
     the package (not that you skipped looking).
4. If evidence for a check is absent, grade `unclear` immediately; do
   not invent missing diagnosis, files, or tests.
5. You may grade from the gathered notes without re-reading the whole
   package unless the notes lack a deciding quote; then re-open only
   the relevant section.
6. Write one evidence line per check for the output JSON: the fact or
   quote that decided the grade.

## Verdict assembly

1. Apply the rubric's verdict rule exactly: accept only if every
   `required` check is `pass`; any required `fail` or `unclear`
   yields `reject`; preferred checks never change the verdict.
2. Treat `unclear` on a required check as `fail` for the verdict (per
   the rubric rule).
3. In live mode, also hold the plan comment against `voice-guide.md`
   and note any broken rule in the readable summary; voice notes do
   not change the verdict unless a rubric check reads them.
4. Emit the short readable summary (one line per check, plus any voice
   notes), then the fenced JSON block with `item`, `checks`, and
   `verdict`, and nothing after that block. Quote the deciding fact in
   each check's `evidence` field.
