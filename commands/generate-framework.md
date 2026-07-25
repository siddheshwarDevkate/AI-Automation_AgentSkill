# ==============================================================================
# COMMAND - GENERATE PLAYWRIGHT FRAMEWORK
# ==============================================================================

## PURPOSE

Generate a complete, production-ready Playwright automation framework for the
target application.

This command is the entry point of the framework generation process.

It is responsible only for invoking the Framework Generation Agent and passing
the user's request.

---

## INPUT

The user may provide one or more of the following:

- Application URL
- Login credentials
- Business requirements
- Test scenarios
- Existing automation framework
- Additional generation instructions

If required information is unavailable, the Agent should either discover it
during application exploration or request clarification from the user.

---

## EXECUTION

Invoke the **Framework Generation Agent**.

Pass the complete user request to the Agent without modification.

Use the configured browser automation tool (Playwright MCP in the current
implementation) to interact with the live application.

The Agent is responsible for:

- Understanding the application
- Planning the execution workflow
- Invoking Skills in the correct sequence
- Loading Skill dependencies
- Managing intermediate outputs
- Handling failures and recovery
- Performing framework validation
- Performing framework review
- Returning the completed framework

---

## EXPECTED OUTPUT

Generate a complete Playwright automation framework that is:

✓ Production Ready

✓ Modular

✓ Maintainable

✓ Reusable

✓ Scalable

✓ Consistent with the defined project architecture

---

## IMPORTANT

This command must **NOT**:

✗ Analyze the application

✗ Generate locators

✗ Generate Page Objects

✗ Generate Spec files

✗ Generate assertions

✗ Generate test data

✗ Perform validation

✗ Perform framework review

✗ Load Knowledge files directly

✗ Load Template files directly

All execution responsibilities belong to the Framework Generation Agent.

---

## END OF COMMAND