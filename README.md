# adversarial-review-skill

Agent skill for **same-session adversarial review** of a plan or design before coding.

- **Default:** the current model reviews adversarially in the same chat (verdict, findings, triage, fold).
- **Fallback:** invoke the Codex CLI read-only only if the model cannot do a strong hostile pass, or the user explicitly asks for Codex.
- **Legacy:** "prep for adv ai review" writes a cold review packet for a later session.

Works with Cursor, Claude Code, Codex, and similar agents that load `SKILL.md` from a skills directory.

## Install

Clone this repository into an agent skills directory (replace `<owner>` with the GitHub org or user that hosts the fork or upstream):

```powershell
git clone https://github.com/<owner>/adversarial-review-skill.git "$env:USERPROFILE\.cursor\skills\adversarial-review"
```

Optional mirrors for Claude Code / Codex:

```powershell
git clone https://github.com/<owner>/adversarial-review-skill.git "$env:USERPROFILE\.claude\skills\adversarial-review"
git clone https://github.com/<owner>/adversarial-review-skill.git "$env:USERPROFILE\.codex\skills\adversarial-review"
```

Or copy `SKILL.md` into `.cursor/skills/adversarial-review/` (or `.claude` / `.codex`) at the project or user level.

## Usage

Ask the agent to adversarially review a plan, or to stress-test a design before build. Prefer letting the current model run the review. Only ask for Codex when you specifically want a second-model pass.

## Structure

- `SKILL.md` — full workflow the agent follows

## License

MIT — see [LICENSE](LICENSE).
