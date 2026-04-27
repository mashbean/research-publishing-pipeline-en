# Automation Contract

## Goal

Once a research article job starts, it automatically enters active reminder mode. It exits only when the job is closed or has clearly failed.

## Start Semantics

When the user starts a research article task, the system should update `memory/automation-state.json` immediately after creating the job:

- `activeWork = true`
- `mode = "active"`
- `taskTitle = <job id or article title>`
- `taskNote = <task description>`
- `nextDeliverable = <nearest minimum deliverable>`
- `lastMeaningfulProgressAt = now`
- `lastUpdated = now`

## While Running

- The active checker continues every 10 minutes.
- If there is no new progress, perform a safe self-push before reporting.
- If the 30-minute stall watchdog determines the task is stuck, escalate with a blocked reason and recovery action.

## End Semantics

When the job enters any of the following states, update `memory/automation-state.json` and exit active mode:

- `verified`
- `publish-failed`
- `verification-failed`
- `blocked`
- explicit human closure

On exit, write:

- `activeWork = false`
- `mode = "idle"`
- empty `taskTitle`, `taskNote`, and `nextDeliverable`
- preserve `lastDeliverable`
- update `lastDeliverableAt`, `lastMeaningfulProgressAt`, and `lastUpdated`

## Acceptance Target

Once the user starts the article job, it keeps moving through the workflow and maintains 10-minute active reminders until completion. The user should not need to add repeated `please continue` prompts.
