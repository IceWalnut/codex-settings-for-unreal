# Global Codex Engineering Rules

These rules apply to all workspaces unless a repository AGENTS.md overrides them.

## Language
- Always respond in Chinese (Simplified), unless the user explicitly asks otherwise.

## Rule entrypoint
Before starting any task, read and follow (in this order):
1) ~/.codex/rules/workflow.md
2) ~/.codex/rules/coding.md
3) ~/.codex/rules/unreal.md (only if the repository involves Unreal Engine)

If there is a conflict:
- This AGENTS.md overrides any other global files under ~/.codex/rules/.
- A repository AGENTS.md may override these rules for that repo.

## Workflow (high level)
- Search before editing. Prefer `rg` / `rg --files` for speed.
- Make minimal, localized diffs. Avoid unrelated refactors.
- Group related changes into one coherent patch. Avoid one-line micro-edits.
- After code changes, provide a clear validation plan (build/test/run commands).

## Code implementation principles (high level)
1. Variable naming:
   - Avoid 1–2 letter variable names like `g`, `ok`, `tmp`.
   - Exception: conventional loop indices (`i`, `j`, `k`) and very small scopes are allowed.
   - Prefer descriptive names that reflect intent and units (e.g., `timeoutMs`, `assetPath`).

2. Strict error handling:
   - Do not swallow errors with broad `try/catch`.
   - If catching, handle the error meaningfully or rethrow with context.
   - Prefer returning structured errors/results over silent failures.

3. Type safety:
   - Avoid `as any` (TypeScript) or equivalent type escapes.
   - Prefer proper types, type guards, and narrowings.
   - If an unsafe cast is unavoidable, isolate it and explain why.

4. Batch editing:
   - Finish related edits together (code + tests + docs if needed).
   - Do not split a single logical change into many tiny commits/patches.

## Editing constraints
- Default to ASCII in new content unless the file already uses Unicode.
- Do not reformat entire files unless explicitly requested or required by the change.
- Add comments only where the logic is non-obvious.

## Communication
- When proposing changes, include:
  - What you will change (short plan)
  - The patch/diff (or file-level edits)
  - How to verify (commands/steps)

## Enforcement reminder
- In PuerTS TypeScript code, do not use `as any` or broad `try/catch` as shortcuts.
- Prefer explicit checks and fail-fast behavior.

