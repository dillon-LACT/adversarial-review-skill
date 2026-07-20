---
name: adversarial-review
description: >-
  Same-session adversarial review of a plan/design before coding. Default: the
  current agent reviews adversarially itself (verdict, findings, triage, fold).
  Fallback only: invoke Codex CLI read-only if the model cannot do the review
  well, or the user explicitly asks for Codex. Also covers legacy "prep for adv
  ai review" packets. Use for adv review, Codex review, plan stress-tests.
---

# Adversarial review

**Same session always.** Never hand the user a file and send them to another app.

## Default — you review (preferred)

Capable models should **just do the adversarial review** in this chat. Do not
call Codex CLI out of habit.

Triggers: "adv review", "adversarial review", "stress-test this plan", phase-0
review todo, or similar — **unless** the user explicitly names Codex.

Flow:

1. Read the plan + enough codebase context (auth, schema, multi-user seams)
2. Write artifacts under `reviews/` (optional but useful):
   - `reviews/adv-review-YYYY-MM-DD-<slug>-brief.md` — what you attacked
   - `reviews/adv-review-YYYY-MM-DD-<slug>-review.md` — full review
3. Return the structured review (shape below)
4. Triage if you were also the builder: **agree / push back**, fold agreed
   CRITICAL into the plan (or ask the user to confirm fold if findings are sharp)
5. Stop before feature coding unless the user said go

Keep secrets out of artifacts. Do not create tracker tickets just for the review
unless the user asks.

### Review shape (always)

1. **Verdict** — ship / revise / kill (one line)
2. **Critical findings** — must fix before coding
3. **Important findings** — should fix in plan before build
4. **Nits / optional**
5. **Concrete plan amendments** — exact wording
6. **What to keep**

Severity tags: `CRITICAL` / `IMPORTANT` / `NIT`. Short bullets. Be skeptical.
Prefer simpler systems. Attack auth, data model, multi-user leaks, yes-man
residue, over-scoping.

### Triage reply (when you also wrote the plan)

1. Link brief + review if written
2. Verdict
3. **Biggest hits you agree with** → fold into plan
4. **What you push back on** → one-line why
5. Ready when the user says go

---

## Fallback — call Codex CLI

Use **only if**:

- The user **explicitly** says "send to Codex" / "Codex adv review", **or**
- You honestly cannot produce a strong review (missing domain context, tool
  limits, you are the wrong model for a hostile pass) — say so briefly, then call Codex

Do **not** call Codex "just in case" when you can review competently yourself.

Discover the CLI (do not hardcode a username path):

```powershell
$codex = (Get-Command codex -ErrorAction SilentlyContinue)?.Source
if (-not $codex) {
  $candidate = Join-Path $env:LOCALAPPDATA "Programs\OpenAI\Codex\bin\codex.exe"
  if (Test-Path $candidate) { $codex = $candidate }
}
if (-not $codex) { throw "Codex CLI not found on PATH or under LOCALAPPDATA\\Programs\\OpenAI\\Codex" }

$repo  = "<repo absolute path>"
$planPath = "<plan absolute path>"
$briefPath = Join-Path $repo "reviews\codex-adversarial-YYYY-MM-DD-<slug>-brief.md"
$out = Join-Path $repo "reviews\codex-adversarial-YYYY-MM-DD-<slug>-review.md"
$promptPath = Join-Path $repo "reviews\_codex_prompt_tmp.txt"

$plan = Get-Content $planPath -Raw
$brief = Get-Content $briefPath -Raw
@"
Read the adversarial review brief and the full plan below, plus inspect the
repo as needed (read-only). Produce the structured adversarial review described
in the brief. Write nothing except the review markdown in your final message.

=== BRIEF ===
$brief

=== FULL PLAN ===
$plan
"@ | Set-Content -Encoding utf8 $promptPath

Get-Content $promptPath -Raw | & $codex exec `
  -C $repo `
  -s read-only `
  --skip-git-repo-check `
  -o $out `
  -

Remove-Item $promptPath -ErrorAction SilentlyContinue
```

On macOS/Linux, prefer `$(command -v codex)` and the same `codex exec` flags.

Rules when falling back:

- Always `-s read-only`
- Wait for completion (often 2–5+ minutes)
- You still own triage + fold in **this** chat after Codex returns
- If CLI missing, say so — do not invent a fake Codex review

Brief for Codex should state: adversarial reviewer not implementer; no app code;
do not edit the plan; locked decisions; focus areas; required output shape above.

---

## Legacy only — prep packet (no review run)

Only when the user explicitly says **"prep for adv ai review"** and wants a cold
packet for a *later* separate session:

1. Write `reviews/adv-ai-review-YYYY-MM-DD-<topic>.md`
2. Objective, status, files, diffs/snippets, rationale, risks, validation, review questions
3. Cold-reviewer complete; redact secrets
4. Path only — do not implement feedback until asked

If they want the review **run now**, use Default (you) or Fallback (Codex) —
not a dead packet.

---

## Do not

- Default to Codex when you can review yourself
- Hand the user only a path and tell them to open another app
- Start feature coding in the review turn unless they said go
- Let a fallback Codex edit app code or the plan
- Treat every finding as gospel — triage when you were also the builder
- Open a tracker ticket just for the review unless asked

---

## Install

Clone into an agent skills directory (pick the tool you use):

```powershell
git clone https://github.com/<owner>/adversarial-review-skill.git "$env:USERPROFILE\.cursor\skills\adversarial-review"
# optional mirrors:
# git clone ... "$env:USERPROFILE\.claude\skills\adversarial-review"
# git clone ... "$env:USERPROFILE\.codex\skills\adversarial-review"
```

Or copy `SKILL.md` into `.cursor/skills/adversarial-review/` (or `.claude` / `.codex`) at the project or user level.

If you maintain multiple copies, keep them identical.
