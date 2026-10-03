# Plan: parenthesized US phone numbers in PIIScrubber (#53)

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/53

## Diagnosis

The `phone_us` pattern in `safety/pii_scrubber.py` already allows optional
parentheses around the area code, but the separator after that group is only
`[-.]?` (an optional `-` or `.`). It does not allow a space. The issue's
failing form is `(555) 123-4567` — close-paren then space — so `scrub()` and
`detect()` miss it while dashed `555-123-4567` still matches.

Repro evidence this relies on (posted reproduction on #53, commit `2f4e82f`):

```
scrub : 'Call me at (555) 123-4567 or [REDACTED]'
detect paren: []
detect dashed: [{'type': 'phone_us', 'value': '555-123-4567', ...}]
```

Control: the same `PIIScrubber` redacts the dashed form in the same string,
so the scrubber path works; the miss is format-specific to the parenthesized
(spacing) shape. With `--runxfail`, the four named phone tests fail on
assertions that expect `[REDACTED]` / a phone detection for that form.

## Scope

**In scope**

- Widen the `phone_us` regex separators so parenthesized-with-space (and the
  other space-separated US forms already listed in
  `test_us_phone_formats`) match.
- Remove the `@pytest.mark.xfail` markers on the #53 phone tests once they
  pass for real.
- Keep `scrub()` / `detect()` behavior otherwise unchanged.

**Out of scope**

- International phone pattern redesign (`phone_intl`).
- New PII types, logging changes, or scrubber API changes.
- Broader "detect which pattern matched" cleanup noted in the scrubber
  comments.

## Files

- `safety/pii_scrubber.py` — `PII_PATTERNS["phone_us"]`
- `tests/unit/test_pii_scrubber.py` — drop xfail on the #53 phone cases

## Approach

1. Confirm the current pattern against the repro snippet (parenthesized
   miss + dashed control).
2. Change `phone_us` separators from `[-.]?` to a class that also allows
   whitespace (e.g. `[\s.-]?`), including after an optional `+1` prefix, so
   `(555) 123-4567` and `+1 555 123 4567` can match without loosening into
   arbitrary digit runs.
3. Re-run the phone-focused tests without xfail; remove the xfail markers
   when green.
4. Spot-check that existing dashed / dotted forms and
   `test_phone_at_end_of_text` still pass.

## Test plan

Re-run the unit-2 repro steps against the change; expect the opposite of
the before artifacts.

1. Issue snippet + dashed control:

```
.venv\Scripts\python -c "from safety.pii_scrubber import PIIScrubber; s = PIIScrubber(); print('scrub :', repr(s.scrub('Call me at (555) 123-4567 or 555-123-4567'))); print('detect paren:', s.detect('Call me at (555) 123-4567')); print('detect dashed:', s.detect('Call me at 555-123-4567'))"
```

After fix, expect both numbers redacted in `scrub`, and `detect paren` to
return a `phone_us` hit (dashed control still hits).

2. Phone tests without forcing xfail:

```
.venv\Scripts\python -m pytest tests/unit/test_pii_scrubber.py -k phone -v --tb=short
```

Expect the four previously-xfailed tests to **PASS** (not XFAIL), and
`test_international_phone_redaction` / `test_phone_at_end_of_text` still
pass.

3. Full scrubber unit file:

```
.venv\Scripts\python -m pytest tests/unit/test_pii_scrubber.py -q
```

Expect green; no new failures on email/SSN/address cases.

## Risks and unknowns

- Allowing `\s` in separators might over-match digit groups that are not
  phone numbers; the existing tests and a quick scan of other unit cases
  are the guardrail. If something over-redacts, tighten the pattern
  (e.g. require the closing `)` when a `(` was present) rather than
  expanding scope.
- `test_us_phone_formats` also includes `+1 555 123 4567`; the separator
  change should cover it, but that case was not in the original four
  `--runxfail` failures — verify explicitly after the change.
- Word-boundary behavior around a leading `(` is easy to get wrong; the
  start-of-text test (`test_phone_at_start_of_text`) is the check.

## Deviations

1. Leading boundary: `[\s.-]` alone was not enough. With a leading `\b`,
   the engine started the match at `555` and left a stray `(` (and a
   stray `+` on `+1 555 123 4567`). Switched the leading boundary to
   `(?<!\w)` so the opening `(` / `+1` stay inside the match. Same
   files; same separator widening; stronger boundary.
2. Fifth xfail: `test_mixed_pii_and_text` still fails after the phone
   fix because `street_address` case-insensitively matches `Pl` inside
   `applications`. That is out of scope for #53, so the marker stays
   with an updated reason. The four issue-named phone xfails were
   removed as planned.
