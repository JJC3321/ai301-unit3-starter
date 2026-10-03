# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

**Where it lives**

- Eval bundle: the candidate plan's diagnosis/cause paragraph; the
  Repro evidence block's steps, failing artifacts, and control runs;
  the Issue body's expected vs actual for the behavior being explained.
- Live mode: the draft `plan.md` diagnosis; the student's posted repro
  comment (or house repro pack quotes inside the drafts); the issue
  body. Do not credit unquoted local notes.

**What good looks like**

The stated cause cites behavior the repro evidence actually shows, and
survives the evidence's own controls. If a control clears component A,
blaming A fails. A diagnosis that never engages the failing artifact or
contradicts a shown measurement is not grounded.

## Scope

**Where it lives**

- Eval bundle: the candidate plan's in-scope / out-of-scope lines,
  proposed-changes list, and named files or areas; compare the breadth
  of that list to the defect the repro isolates.
- Live mode: the same sections of draft `plan.md`.

**What good looks like**

One bounded change (or an explicitly scoped-down slice with reasons)
tied to the reproduced defect. A correct core fix buried inside a
migration, printer rewrite, state-machine redesign, new option, or
cross-runtime abstraction is still unbounded. A clear not-in-scope
line that defers adjacent work is good.

## Executability

**Where it lives**

- Eval bundle: the candidate plan's files/areas list and approach or
  steps; absence of those sections is itself evidence.
- Live mode: draft `plan.md` approach and files sections.

**What good looks like**

A stranger can start: at least one concrete file or area and one chosen
approach. "Profile and see", "investigate the stack", or "fix upstream
or vendored, whichever is easier" with no files is not executable.
Terse plans still pass when the file and the change are named.

## Test plan

**Where it lives**

- Eval bundle: the candidate plan's test plan; map it onto the Repro
  evidence steps and expected/actual.
- Live mode: draft `plan.md` test plan; the posted repro steps for the
  before/after observables.

**What good looks like**

Names an observable outcome for the fix (exit 0 instead of panic,
`[REDACTED]` present, color flips without leaving the view, specific
assertion). Re-running the repro with a stated expected result is
strong. "Should feel fast", "nothing else broken", or "run the full
suite" with no fix-specific signal is not decisive.

## Honesty

**Where it lives**

- Eval bundle: the candidate plan's risks, unknowns, deviations, and
  any confidence language in the diagnosis or approach.
- Live mode: the same in `plan.md`, including a `## Deviations` note
  after a build that changed course.

**What good looks like**

Unresolved choices are labeled unknowns or deferred work. False
certainty about a cause the evidence did not settle fails honesty.
Recording a mid-build deviation under Deviations is honest; a silent
diff-only change is not. A short plan with no risks section can still
be honest if it does not overclaim.

## Comms

**Where it lives**

- Eval bundle: Thread highlights for OWNER/maintainer direction
  (culprit file, patched binary, preferred approach); the repo-facts
  contribution policy / AI disclosure line; the candidate plan comment.
- Live mode: the live issue thread (`gh` / issue page); `CONTRIBUTING.md`
  / `AI_POLICY.md` on the repo; draft `comment.md`. Path Review house
  rule: a classmate's plan does not block posting your own.

**What good looks like**

The comment is about this issue's plan in the author's own words, and
engages any explicit maintainer direction present in the thread (or
states why it diverges). If the repo requires AI disclosure, the
comment discloses tool and human verification. Boilerplate that ignores
an owner-posted culprit path, or silence under a strict AI policy, is
not thread/convention aware.
