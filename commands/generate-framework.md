# ==============================================================================
# COMMAND - GENERATE PLAYWRIGHT FRAMEWORK
# ==============================================================================

## PURPOSE
Entry point for generating a production-ready Playwright automation framework from a supplied Test Case file. This command's only job is to trigger the Framework Generation Agent — it owns no logic of its own.

## EXECUTION
Invoke the Framework Generation Agent (`agent/agent.md`). Pass the complete user request to it without modification, interpretation, or filtering.

## BOUNDARIES
This command performs no parsing, analysis, generation, validation, or review itself, and does not load Knowledge or Template files directly. Every responsibility — required inputs, expected output, Skill sequencing, failure handling — is owned entirely by `agent/agent.md`. Refer there for anything beyond "what triggers the pipeline."

# ==============================================================================
# END OF COMMAND
# ==============================================================================
