# OMP adaptation of pstack

This checkout adapts pstack skills for OMP (oh-my-pi). Cursor-specific
delegation, transcript paths, model files, and built-ins are replaced by the
OMP adapter in [SUBAGENT-ADAPTER.md](./SUBAGENT-ADAPTER.md).

Subagents use the built-in `task` tool (batch `{context, tasks[]}`), with
`agent://<id>` for output, `history://<id>` for transcripts, and `bash` for
commands. Model selection comes from agent frontmatter or `task.agentModelOverrides`;
per-call `model` requires `task.perCallModel`. Skills that need multiple
perspectives run separate `task` spawns with isolated output rather than
assuming Cursor cloud workers.
