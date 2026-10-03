# Unit 3 — Plan and build

Live-mode plan + build for Path Review issue
[#53](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/53)
(`PII scrubber fails to redact parenthesized US phone numbers`), graded
with the installed `plan-check` skill whose eval harness passed at
**20/20** (`beat-1-sandbox/unit-3/eval-run.txt`).

GitHub username: `JJC3321`

Skill upload path: `tools/plan-check/` (same content as
`~/.claude/skills/plan-check/`).

Working clone: `Documents/Code/pathreview-ai301-fa26-s3`
Branch: `fix/53-parenthesized-phone`
Local drafts: `plan.md`, `comment.md` (kept uncommitted)

## Run history

Ordered agreement scores from the harness runs that shaped this rubric:

1. Smoke run (`python run_eval.py … --limit 3`): first attempt failed on
   Windows `UnicodeEncodeError` (stdin cp1252); fixed `run_eval.py` to
   pass `encoding="utf-8"`. Second attempt failed with Claude OAuth
   expired; re-authed with `claude auth login`.
2. Full scored run (`python run_eval.py --rubric
   ~/.claude/skills/plan-check/rubric.md --evidence
   ~/.claude/skills/plan-check/references/evidence-guide.md --workers 5
   --save-run eval-run.txt`): **20/20** agreement (bar 18/20: **PASS**).
   Category floors: clear-accept 7/7, scope-creep 4/4,
   thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4. Transcript
   saved as `beat-1-sandbox/unit-3/eval-run.txt` (and
   `eval/eval-run.txt`; written 2026-10-03T07:17:03Z).
3. No revise loop after step 2: every scored package already agreed with
   gold, and every category had a match. Live plan-check on #53 then
   returned `accept` (re-run after Deviations also `accept`).

## Package analysis

Scored package: **`pkg-01`** (source: `httpie/cli#1838`, category
`wrong-cause`).

| Side | Verdict |
|---|---|
| Gold label | `reject` |
| Skill (this rubric) | `reject` |

Gold note: the plan blames the request-item tokenizer; the package's
own control run (same items, no `-v` flag) parses fine, ruling the
tokenizer out, and `--debug` shows argparse consuming positionals
before the item parser runs.

Skill reasoning: `diagnosis-grounded` fails because the stated cause
(tokenizer separator regex) contradicts the repro-evidence control —
step 2 shows the same `header1:xyz` / `x=1` items parse without `-v`,
and step 4 pins the failure on argparse before items reach the
tokenizer. Other required checks may still pass (the plan names files
and a test), but one required fail yields `reject`. This is the
wrong-cause pattern the rubric was written to catch, and it matches
gold.

## Check rationale

Quoted check from `tools/plan-check/rubric.md` (same text as the
installed skill copy):

> **`diagnosis-grounded`** — Evidence: The candidate plan's stated cause
> or diagnosis, read against the repro-evidence block (steps, artifacts,
> control runs) and the issue body's described behavior. Pass condition:
> The stated cause is consistent with what the repro evidence actually
> shows: it explains the failing artifact, and does not contradict a
> control run or measurement in that same evidence that rules the cause
> out. Fail when the plan blames a component the evidence's own controls
> already cleared, ignores a decisive control, or invents a cause with
> no footing in the shown steps/artifacts. Weight: required.

Why this wording: the fail clause names the gold wrong-cause family
explicitly (`pkg-01` tokenizer vs argparse control, `pkg-07` FES
tree-shake vs instance-method error still printing, `pkg-11` collect
operator vs top-level control, `pkg-16` post-read cast vs zeros already
gone in pyarrow). Requiring the cause to survive the package's own
controls stops confident plans that target the wrong layer. Structure
(section count, heading polish) is deliberately ignored.

Applied to my #53 package: the diagnosis (separator `[-.]?` cannot match
the space in `(555) 123-4567`) cites the scrub/detect miss and the
dashed control from my unit-2 repro, so the check passes.

## Trade-offs

- `diagnosis-grounded` as `required` rejects any plan whose cause the
  repro's controls already cleared. That gives up accepting a polished
  plan that happens to pick an adjacent layer; the gain is not shipping
  wrong-cause builds like `pkg-01`.
- `scope-bounded` rejects a correct core fix buried in a redesign
  (`pkg-15`, `pkg-06`). That gives up "while we're here" migrations that
  might be good engineering; the gain is holding plans to the
  reproduced defect.
- `thread-aware` + `ai-disclosure` are separate required checks so the
  two-package `thread-convention` category cannot be missed on volume:
  maintainer direction (`pkg-04`) and strict AI policy (`pkg-20`) each
  have a named fail mode.
- `honest-unknowns` is `preferred` only, so a terse ready plan without a
  risks section is not held. That gives up forcing every package to
  narrate uncertainty; the gain is not rejecting clear-accept terseness
  like `pkg-02` / calib-01.
- Why nothing changed after the 20/20 run: every category floor was
  already full. Editing further would only risk introducing a new miss
  without fixing a disagreement.

## Plan comment

GitHub username: `JJC3321`

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/53

Posted URL: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/53#issuecomment-5967019324

Pasted plan comment text:

```text
Reproduced on `main` at `2f4e82f` (report above): `(555) 123-4567` is left
intact by `scrub()` / `detect()` while `555-123-4567` in the same run is
redacted.

Plan: the `phone_us` regex in `safety/pii_scrubber.py` already allows
optional parentheses, but the separator after the area code is only
`[-.]?`, so the space in `(555) 123-4567` never matches. I'll widen those
separators to allow whitespace (`[\s.-]?`), switch the leading boundary
from `\b` to `(?<!\w)` so a leading `(` / `+1` stays inside the match,
leave `phone_intl` and other PII patterns alone, drop the four #53 phone
`xfail` markers in `tests/unit/test_pii_scrubber.py` once green, and
re-run the issue snippet plus the phone tests as the check (both formats
`[REDACTED]`, `detect()` hits the parenthesized form). Leaving the
unrelated `test_mixed_pii_and_text` xfail alone — that failure is a
`street_address` false positive on `applications`, not this phone case.

Per course AI-use norms: I used an AI assistant to organize this plan; I
ran the reproduction myself and will verify every edit before committing.
```

## Branch

`fix/53-parenthesized-phone` on the Path Review clone (commit
`9fd4d7e` — only `safety/pii_scrubber.py` and
`tests/unit/test_pii_scrubber.py`; `plan.md` / `comment.md` untracked).

## plan.md (Deviations)

Full draft lives at the clone root `plan.md`. Deviations filled after
the build:

1. Leading boundary: `[\s.-]` alone left a stray `(` / `+` under `\b`;
   switched to `(?<!\w)`.
2. Kept `test_mixed_pii_and_text` xfail (street_address `Pl` false
   positive); removed only the four issue-named phone xfails.

## Evidence

Before (unit 2 repro on `2f4e82f` / main):

```
$ .venv\Scripts\python -c "from safety.pii_scrubber import PIIScrubber; s = PIIScrubber(); print('scrub :', repr(s.scrub('Call me at (555) 123-4567 or 555-123-4567'))); print('detect paren:', s.detect('Call me at (555) 123-4567')); print('detect dashed:', s.detect('Call me at 555-123-4567'))"
scrub : 'Call me at (555) 123-4567 or [REDACTED]'
detect paren: []
detect dashed: [{'type': 'phone_us', 'value': '555-123-4567', 'start': 11, 'end': 23}]
```

Phone tests with `--runxfail` on the four named cases: 4 failed.

After (`fix/53-parenthesized-phone`):

```
scrub : 'Call me at [REDACTED] or [REDACTED]'
detect paren: [{'type': 'phone_us', 'value': '(555) 123-4567', 'start': 11, 'end': 25}]
detect dashed: [{'type': 'phone_us', 'value': '555-123-4567', 'start': 11, 'end': 23}]
plus1: 'Contact: [REDACTED]'
start: '[REDACTED] is my phone number.'
```

```
$ .venv\Scripts\python -m pytest tests/unit/test_pii_scrubber.py -k phone -v --tb=short
test_us_phone_number_redaction PASSED
test_us_phone_formats PASSED
test_international_phone_redaction PASSED
test_detect_phone_pii PASSED
test_phone_at_start_of_text PASSED
test_phone_at_end_of_text PASSED
6 passed, 19 deselected

$ .venv\Scripts\python -m pytest tests/unit/test_pii_scrubber.py -q
24 passed, 1 xfailed
```

The remaining xfail is `test_mixed_pii_and_text` (street_address), not
the phone formats.

## Issue link

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/53

## Self-check against the rubric (before posting)

| Check | Grade | Evidence |
|---|---|---|
| `diagnosis-grounded` | pass | Cause cites space-missing separator; matches scrub/detect miss + dashed control |
| `scope-bounded` | pass | Only `phone_us` + phone xfails; street_address deferred in Deviations |
| `executable` | pass | Names `safety/pii_scrubber.py` and concrete separator/boundary change |
| `test-decisive` | pass | Repro snippet + phone tests with observable `[REDACTED]` / PASS |
| `thread-aware` | pass | No OWNER direction on #53; classmates do not block |
| `ai-disclosure` | pass | Path Review silent; comment discloses AI anyway |
| `honest-unknowns` | pass | Risks + Deviations label boundary and fifth-xfail unknowns |

Verdict under the rubric rule (all required pass): **accept** — ready to
post (live skill agreed).
