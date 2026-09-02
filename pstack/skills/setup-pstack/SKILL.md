---
name: setup-pstack
description: Configure which models pstack uses per role. Detects your available models and writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack (pi)

**Pi environment:** there is no Task tool, no per-role subagent models, and
no `~/.cursor/rules/pstack-models.mdc`. All model-slug configuration in
pstack skills is inert here — every skill runs on the parent session model.

Nothing to configure. This skill only verifies the installation:

1. Check `~/.agents/skills/pstack/` exists and contains the skill dirs and
   `SUBAGENT-ADAPTER.md`.
2. Report the list of installed skills and remind that subagent-dependent
   steps run inline per the adapter.
