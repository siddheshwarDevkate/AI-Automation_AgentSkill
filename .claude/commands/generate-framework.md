---
description: Generate a production-ready Playwright framework from the Test Case file in .claude/input/
---

# ==============================================================================
# COMMAND - GENERATE PLAYWRIGHT FRAMEWORK
# ==============================================================================

## PURPOSE
Entry point for generating a production-ready Playwright automation framework from a supplied Test Case file. This command's only job is to trigger the Framework Generation Agent — it owns no logic of its own.

## EXECUTION
Resolve the resource root — `.claude/` if it exists in the project root, otherwise `.opencode/` — then read and follow `<resource-root>/agent/agent.md`. Pass the complete user request to it without modification, interpretation, or filtering.

The Agent's first action is its **Intake Gate**: it asks the user for the application URL, the credentials and exact login steps, and confirmation that the Test Case file is in `<resource-root>/input/` as CSV. Do not pre-empt, answer, or skip those questions on the user's behalf.

## BOUNDARIES
This command performs no parsing, analysis, generation, validation, or review itself, and does not load Knowledge or Template files directly. Every responsibility — the Intake Gate, required inputs, expected output, Skill sequencing, failure handling — is owned entirely by `<resource-root>/agent/agent.md`. Refer there for anything beyond "what triggers the pipeline."

# ==============================================================================
# END OF COMMAND
# ==============================================================================
