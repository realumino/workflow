# AGENTS.md

## Workspace
- `.localworkspace/` is the ONLY place for intermediate output: plans, exploration
  reports, temporary files, and scripts. Anything not meant for the Git repo goes here.
- Directly under `.localworkspace/` is one directory per mission (or milestone).
  A mission may span multiple chat sessions.
- The operator names each mission directory. If a new mission needs a directory and
  none is named, create one following the naming convention used by existing
  directories, and state the name you chose in your reply (so the operator can
  correct it without being asked first).

## Files in a mission directory
| File | Written by | When | Rule |
|---|---|---|---|
| `INSTRUCTION.md` | operator | start of mission | read-only for the LLM |
| `EXPLORATIONS.md` | LLM | phase 1 | answer every open question; mark what remains unknown. Read on demand, not at session start |
| `PLAN.md` | LLM + operator approval | after operator decision | self-contained: current state + the exploration findings the plan depends on + executable steps, so a new session need not re-explore. The ONLY file requiring operator approval; keep a decision log in it |
| `HANDOFF.md` | LLM | end of every turn | OVERWRITE, never append. Write ONLY if the state changed materially (see below); otherwise leave the file untouched. Content: current state, files/assets changed and why, done / next / blocked-on-operator |

`HANDOFF.md` is a snapshot. History belongs in `PLAN.md`.
`PLAN.md` is a living document: a locked decision can change only with the operator's
approval, recorded as a new entry in its decision log. Never silently follow a plan
you know is wrong — stop and ask.

## When `HANDOFF.md` must be rewritten
A turn = one operator message plus the LLM's reply, including all tool calls in
that reply. At the end of that reply — after the last tool call finishes — check
whether the state changed. 

Material (write):
- any file or asset created, edited, deleted, or renamed
- a PLAN step completed, skipped, or blocked
- a decision made, changed, or found to be wrong
- state moved between "waiting on operator" and "executable"
- a new blocker or open question appeared

Not material (do not write):
- answering a question, explaining, or discussing without changing state
- read-only exploration that produced no new conclusion
- restating an unchanged state
- the act of writing `HANDOFF.md` itself

When in doubt, write.

## Session start
1. Read this file (auto).
2. If the current mission directory is not already clear from context, the operator
   will reference a file inside it; treat that file's mission directory as the current
   one. If there is no reference, classify the request using the Material / Not
   material lists in "When `HANDOFF.md` must be rewritten" — one criterion, two uses:
   - **Not material** (a small question, a cheap read-only check whose answer fits
     in one message): answer directly. Skip steps 3-4, create no directory, write no
     `HANDOFF.md`.
   - **Material** (any file write, decision, or PLAN step): ask for the mission
     directory, or create one per the Workspace rule above, then continue to step 3.
3. Read, in order: `HANDOFF.md` (current state) → `INSTRUCTION.md` (bottom lines,
   boundaries, open questions) → `PLAN.md` (if it exists).
4. Report current state and next step in one short message. If `PLAN.md` already
   authorizes that step, proceed; otherwise wait for the operator.

## Pipeline per mission
1. **Explore.** First session checks feasibility and resolves the open questions in
   `INSTRUCTION.md`. Write findings to `EXPLORATIONS.md`, summarize to the operator,
   then stop and wait for a decision.
2. **Plan.** Once the operator decides, record that decision verbatim in `PLAN.md`
   and write the executable plan. If exploration already made the answer unambiguous,
   say so explicitly in the plan instead of waiting.
3. **Execute.** Follow `PLAN.md`. At the end of every turn, update `HANDOFF.md` only
   if the state changed materially (see "When `HANDOFF.md` must be rewritten").


## Restrictions
- During exploration: READ-ONLY. You may read local files and make read-only API
  calls. You may NOT create, edit, delete, rename, or run anything that changes
  existing files, or call any API with side effects, unless the operator explicitly
  asks for that specific change.
- Writing inside `.localworkspace/` is always allowed; it is not an "existing asset".

## Coding Style
- Inline comments are short and factual: state only WHAT the code does, and only
  where it isn't obvious from the code itself. No why, no background, no restating
  the line.
