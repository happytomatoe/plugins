---
name: subagent-adapter
description: 'Translate Cursor-oriented pstack delegation instructions to OMP (oh-my-pi).'
---

# OMP adaptation for pstack

This install runs inside OMP. Cursor's `Task`, `subagent_type`, cloud workers, and Cursor transcript paths do not exist here. OMP's subagent primitive is the built-in `task` tool.

## The primitive

`task` spawns subagents. Batch mode is on by default, so one call carries shared context and one item per child:

```json
{
  "context": "shared background rendered into every child's system prompt",
  "tasks": [
    {
      "agent": "scout",
      "task": "complete, self-contained assignment",
      "solutionSpace": "what is open, e.g. 'one fix: rename, names given'"
    }
  ]
}
```

- `agent` is the agent type. Bundled: `task` (default, general), `scout` (read-only), `reviewer`, `security-reviewer`, `sonic`. Custom agents live in project `.omp/agents/<name>.md` or user `~/.omp/agent/agents/<name>.md`, with frontmatter `name`, `description`, optional `model` (`@role` or `provider/model:level`) and `tools`. Omit `agent` for the general agent.
- `task` is the whole assignment. The child does not see this conversation.
- `solutionSpace` is required. It states how open-ended the work is; the thinking classifier reads it, not the task text.
- `context` exists only in batch mode. Flat single-spawn calls are accepted at runtime but not advertised.

Results and control:

| Need | Use |
| --- | --- |
| saved output | `agent://<id>` |
| transcript | `history://<id>` |
| inspect a running spawn | `proc://<id>` |
| cancel a job or agent | write `proc://<id>/kill` with empty content |
| message or revive an agent | write `agent://<id>` |
| block until background spawns finish | `wait` |

With `async.enabled=true` (the default), spawns run as background jobs and results auto-deliver to this session. There is no `run_in_background` flag: async is the default, and `wait` is how you block. Scripted fan-out uses the eval helpers `agent()` and `workpool()`.

## Substitutions

- **Investigative or reasoning work:** `task` with `agent: "scout"` for read-only work, `agent: "reviewer"` for review, the default agent for mixed work. Read `agent://<id>`.
- **Shell command, test, build, watcher:** `bash`, as an async job for long runs or a supervised service for watchers. Do not spawn an agent just to run a command.
- **Attested child-agent report:** no `bg_run_pi_attested`. Evidence is the child's artifact (`agent://<id>`) or transcript (`history://<id>`). Quote it; do not claim attestation.
- **Model choice:** per-call `model` works only when `task.perCallModel` is `on` or, for eval `agent()`/`workpool()`, `eval` (default `off`). Otherwise name the model in the agent file's frontmatter or `task.agentModelOverrides`, and map aliases through `modelRoles`.
- **Parallel fan-out:** one batch `task` call with one item per unit, or parallel `task` calls in one message. `task.maxConcurrency` (default 32) bounds it.
- **Fresh-eyes review:** a new `task` spawn with the explicit rubric. Never claim independent review from the same reasoning pass.
- **History:** `history://<id>`, `agent://<id>`, and eval `archive`. Pi's `session_search`/`session_read` do not exist here.
- **Interactive questions:** the `ask` tool exists. Use it only for irreversible or genuinely ambiguous choices.
- **Separate checkouts:** set `isolated: true` on an item only when `task.isolation.enabled` is true and plan mode is off. With `ovnfs` on PATH, isolated children use `ovnfs new`, not `git worktree add`.
- **Cursor built-ins:** replace with the closest local skill or the repository's own command. Never invent a command that is not installed.

## Enablement gates

`task` is absent from the tool list when the session cannot spawn. Check before promising a subagent:

- `task.maxRecursionDepth`: default 2. `0` disables subagents entirely; negative removes the cap. `omp config get task.maxRecursionDepth` reads it; `omp config set task.maxRecursionDepth 2` restores the default.
- Parent `spawns` policy: `*` allows all, an empty value denies all.
- Plan mode restricts children to read-only tools and blocks eval-defined tools and isolation.
- `task.disabledAgents` removes named agents.

## Caps

Child output truncates at 500k bytes or 5000 lines; the full text stays in `<id>.md`. Inline summaries carry about 11.5k characters, so longer results must be read through `agent://<id>`.
