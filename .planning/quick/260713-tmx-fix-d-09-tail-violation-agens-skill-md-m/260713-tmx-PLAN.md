---
phase: quick-260713-tmx
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - .claude/skills/agens/SKILL.md
  - C:/Users/Simon/.claude/skills/agens/SKILL.md
autonomous: true
requirements: [D-09]

must_haves:
  truths:
    - "SKILL.md's present-as-is section states that agens emits nothing after the delegated skill's completion output — the turn ends at the target's own final line"
    - "The 'Wrapping the target's output' anti-pattern bullet names the tail case: no restated answer or summary after the completion output"
    - "The installed user-level copy is byte-identical to the committed repo copy"
  artifacts:
    - path: "C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md"
      provides: "Repo source of the agens skill with the D-09 tail rule"
      contains: "turn ends at the target's own final line"
    - path: "C:/Users/Simon/.claude/skills/agens/SKILL.md"
      provides: "Live installed copy carrying the same fix"
      contains: "turn ends at the target's own final line"
  key_links:
    - from: "C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md"
      to: "C:/Users/Simon/.claude/skills/agens/SKILL.md"
      via: "post-commit file copy (sync)"
      pattern: "diff exits 0"
---

<objective>
Close the D-09 tail violation found in Phase 2 UAT Test 4. After a successful
delegated Skill call, agens appended two paragraphs of its own — a restated
framework answer and an AI-SPEC.md summary — after the delegated skill's
completion banner. The SKILL.md present-as-is instruction bars paraphrase
mid-run but never states that the turn must end at the target's final line.

Purpose: D-09 requires agens to trigger the target and step back; the tail is
part of the hand-off. Without an explicit end-of-turn rule the model treats
the completion banner as an invitation to summarise.
Output: Amended SKILL.md in the agens repo (committed) and a synced installed
copy at the user-level skills root.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md
@C:/Users/Simon/claude-projects/agens/.planning/phases/02-delegation-wiring/02-HUMAN-UAT.md

Evidence (02-HUMAN-UAT.md, Test 4, line 29): mid-run hand-off was clean; the
final assistant message ended the GSD completion banner at its "Next step:"
line, then appended a restated framework answer and a section-by-section
summary of 02-AI-SPEC.md in agens' own voice. Gap entry at line 46 states the
required fix: after the delegated skill's completion output, agens emits
nothing — the turn ends at the target's own "Next step" line.

Cross-directory note: the target project is C:/Users/Simon/claude-projects/agens,
not the current working directory. Use absolute paths for every read and write.
Run git as `git -C "C:/Users/Simon/claude-projects/agens" ...`. The repo working
tree carries unrelated untracked files (README.md, two PATTERNS.md files) and a
modified .planning/config.json — stage ONLY .claude/skills/agens/SKILL.md.
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add the end-of-turn tail rule to SKILL.md and commit</name>
  <files>C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md</files>
  <action>
    Make two edits to the repo copy, in the voice of the surrounding text.

    Edit 1 — section "### 3. Present the target's output as-is" (currently
    lines 284-288). Keep the existing paragraph and append a second paragraph
    stating the tail rule. Use this text: "The hand-off ends the turn. Once the
    delegated skill emits its completion output, agens adds nothing after it —
    the assistant turn ends at the target's own final line (for example its
    'Next step:' line). No restated framework answer, no section-by-section
    summary of AI-SPEC.md, no closing commentary. The mid-run rule and the
    tail rule are the same rule: the target speaks last." Render the inline
    'Next step:' in double quotes to match the file's quoting style.

    Edit 2 — the "**Wrapping the target's output.**" bullet under
    "## Anti-patterns" (currently lines 327-329). Extend the bullet so it names
    the tail case. Append: "The tail counts: appending a restated answer or a
    summary after the target's completion output is the same violation — the
    turn ends at the target's final line (D-09)." Keep the existing "(D-09)"
    reference once; move it to the end of the extended bullet rather than
    citing it twice.

    Do not touch Step 0, the gate checks, the invocation section, or the fixed
    failure message. This closes the UAT gap only.

    Commit with: git -C "C:/Users/Simon/claude-projects/agens" add
    .claude/skills/agens/SKILL.md, then commit with message
    "fix(agens): end the turn at the delegated skill's final line (D-09 tail, UAT Test 4)".
    Stage nothing else — the working tree carries unrelated changes.
  </action>
  <verify>
    <automated>grep -c "turn ends at the target's own final line" "C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md" | grep -q 1 && git -C "C:/Users/Simon/claude-projects/agens" log -1 --pretty=%s | grep -q "D-09 tail"</automated>
  </verify>
  <done>Both edits present in the repo SKILL.md; commit exists touching only that file; unrelated untracked/modified files remain unstaged.</done>
</task>

<task type="auto">
  <name>Task 2: Sync the installed user-level copy</name>
  <files>C:/Users/Simon/.claude/skills/agens/SKILL.md</files>
  <action>
    Copy the committed repo file over the installed copy so the live skill
    carries the fix: cp
    "C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md"
    "C:/Users/Simon/.claude/skills/agens/SKILL.md". Then confirm the two files
    are byte-identical. If they differ after the copy, stop and report — do
    not hand-edit the installed copy.
  </action>
  <verify>
    <automated>diff "C:/Users/Simon/claude-projects/agens/.claude/skills/agens/SKILL.md" "C:/Users/Simon/.claude/skills/agens/SKILL.md" && grep -q "turn ends at the target's own final line" "C:/Users/Simon/.claude/skills/agens/SKILL.md"</automated>
  </verify>
  <done>Installed copy is byte-identical to the committed repo copy and contains the tail rule.</done>
</task>

</tasks>

<verification>
- Repo SKILL.md section "### 3. Present the target's output as-is" contains the
  end-of-turn paragraph; the anti-pattern bullet names the tail case.
- `git -C "C:/Users/Simon/claude-projects/agens" show --stat HEAD` lists only
  `.claude/skills/agens/SKILL.md`.
- `diff` between repo and installed copies exits 0.
</verification>

<success_criteria>
- The D-09 tail gap from 02-HUMAN-UAT.md line 46 is closed in the skill text:
  after the delegated skill's completion output, agens emits nothing.
- Both copies (repo source and user-level install) carry the identical fix.
- One focused commit in the agens repo; no unrelated files staged.
</success_criteria>

<output>
Create `C:/Users/Simon/claude-projects/agens/.planning/quick/260713-tmx-fix-d-09-tail-violation-agens-skill-md-m/260713-tmx-SUMMARY.md` when done.
</output>
