# ProjectCodefixer Implementation Tasks

## Scope

Ordered implementation plan for the project requirements and designs. This plan preserves the current Angular/FastAPI architecture and existing SQLite, ChromaDB, and local/Azure storage choices. Resolve the open questions in `requirements.md` before expanding scope to authentication, retention, new supported test runners, or other unapproved behavior.

## Tasks

### 1. Stabilize workflow API validation and run lifecycle

**Traceability:** [Req 1.1] [Req 1.2] [Req 1.4] [Req 1.5] [Req 5.1] [Req 5.2] [BE 1.1] [BE 1.2] [BE 1.3] [BE 1.4]

- Preserve the existing health and workflow routes and request fields.
- Verify and test rejection of empty error lists and lists longer than 10.
- Make the active-run check and run start atomic so simultaneous requests cannot both start a workflow.
- Define consistent accepted, already-running, and failed-start responses; keep the asynchronous response behavior.
- Ensure terminal workflow success and failure are distinguishable in progress events.

**Completion criteria / tests**

- API tests cover valid requests, both validation boundaries, health/storage-mode output, an already-active run, and simultaneous start attempts.
- Tests assert the start endpoint returns without waiting for the long-running workflow.
- Progress-stream tests assert the agreed terminal success/failure events and required stage-state messages.

### 2. Preserve indexing, retrieval, and analysis behavior

**Traceability:** [Req 2.1] [Req 2.2] [Req 2.3] [Req 2.4] [Req 2.5] [Req 2.6] [BE 2.1] [BE 2.2] [BE 2.3] [BE 2.4] [DB 1.3]

- Exercise clone and skip-clone paths, configured source extensions, ChromaDB collection creation, and dependency-graph generation.
- Exercise retrieval, graph expansion, and missing-context recovery before model analysis.
- Validate the analysis JSON shape and ensure unmatched errors produce an empty fixes list rather than unrelated code changes.
- Preserve the CLI paths for explicit errors, generic audit, and configured Jira ticket lookup; report Jira lookup failure or fallback clearly.
- Keep repository and model integrations configured through environment settings.

**Completion criteria / tests**

- Isolated tests use temporary repository fixtures and stub external Git, model, and Jira calls.
- Tests cover both successful and failed clone/index paths, small/full-context and retrieval behavior where applicable, valid and unmatched analysis results, and Jira unavailable behavior.
- Tests confirm the index uses the configured persistence path and collection name.

### 3. Verify fixes in an isolated sandbox and generate reports

**Traceability:** [Req 3.1] [Req 3.2] [Req 5.3] [BE 2.5] [BE 3.1] [BE 3.2] [FE 3.1] [DB 1.3]

- Ensure candidate fixes are applied and verification runs only in a run-specific copy of the indexed repository.
- Preserve before/after content, per-fix outcome, verification summary, and execution log for the report.
- Generate the HTML report with user- and model-supplied content escaped.
- Serve reports using the configured local or Azure report path.

**Completion criteria / tests**

- A sandbox test proves the indexed source fixture remains unchanged after a candidate fix is applied and verification runs.
- Report tests assert required report sections and verify HTML-sensitive input is escaped.
- Storage-mode tests verify local report serving and Azure report read/write delegation using stubs.

### 4. Align branch creation and remote push behavior

**Traceability:** [Req 1.3] [Req 3.5] [Req 4.2] [BE 2.6] [BE 4.1] [FE 1.2]

- Pass the requested `create_branch` and `push_on_finish` options through workflow orchestration without silently overriding them.
- Keep branch creation and push as separate decisions and report each attempted operation's result.
- Keep credentials out of source files, API responses, and logs.
- Do not expand public-repository authentication behavior until the corresponding open question in `requirements.md` is resolved.

**Completion criteria / tests**

- Tests cover all relevant option combinations and assert no branch/push command is invoked when disabled.
- Stubbed Git tests verify expected branch and push calls and their success/failure log outcomes.
- Log and API-response tests assert configured tokens are not exposed.

### 5. Preserve history and configured storage behavior

**Traceability:** [Req 3.6] [Req 4.1] [BE 1.5] [BE 3.3] [BE 3.4] [DB 1.1] [DB 1.2] [DB 1.3]

- Preserve SQLite `run_history` schema and startup table initialization; add no migration for this scope.
- Persist completed run history with run identifier, timestamp, storage mode, report URL, and submitted errors.
- Preserve newest-first history reads capped at 50 records.
- Exercise ChromaDB and report/asset persistence using local and configured Azure behavior.

**Completion criteria / tests**

- SQLite tests verify the existing schema, completed-run writes, ordering, and 50-record limit.
- Storage tests verify the configured mode is used for code index and report/asset persistence.
- Existing history records remain readable without schema migration.

### 6. Complete the browser workflow form and request feedback

**Traceability:** [Req 1.1] [Req 1.2] [Req 1.3] [Req 1.4] [Req 1.5] [Req 3.3] [Req 5.1] [FE 1.1] [FE 1.2] [FE 1.3] [FE 1.4] [FE 4.1] [FE 4.3] [BE 1.1] [BE 1.2] [BE 1.3]

- Preserve repository URI, error-list, skip-cloning, and push controls; expose an independent branch-creation choice so the request reflects the user's selection.
- Validate error count in the UI consistently with the API and present API validation or already-running errors to the user.
- Handle workflow-start request failures so the loading state is cleared and the user can retry.
- Display backend connection and storage mode from health checks; prevent duplicate submissions during an active run.

**Completion criteria / tests**

- Angular component tests cover valid input, empty and over-limit errors, option payload values, loading-state transitions, API rejection, and health state.
- Keyboard users can reach and operate every input and option; visible labels identify controls and errors.

### 7. Render live progress and logs safely

**Traceability:** [Req 3.3] [Req 5.1] [Req 5.2] [FE 2.1] [FE 2.2] [FE 2.3] [FE 4.1] [FE 4.3] [BE 1.4] [BE 4.3] [BE 4.4]

- Preserve the existing stage order and map progress events to pending, active, completed, and failed display states.
- Distinguish terminal failure from completion and stop the loading state on either outcome.
- Render streamed log content as text rather than injecting repository-, model-, or process-originated strings as HTML.
- Present execution errors in an accessible status area as well as in the activity log.

**Completion criteria / tests**

- Component tests cover each progress state, terminal success/failure, stream errors, and loading-state completion.
- A log containing HTML/script-like text is rendered literally and does not create DOM elements or execute markup.
- Status is understandable from text/accessibility semantics without relying only on color or icons.

### 8. Complete report, PDF, and history interactions

**Traceability:** [Req 3.2] [Req 3.4] [Req 3.6] [FE 3.1] [FE 3.2] [FE 3.3] [FE 4.2] [BE 1.5] [BE 3.1] [BE 3.2] [BE 3.3] [DB 1.1] [DB 1.2] [DB 1.3]

- Display the report when the workflow signals it is ready and retain the PDF action when the dependency is available.
- Show a user-visible state if PDF generation is unavailable or fails.
- Preserve history refresh, empty state, and opening of persisted report URLs; do not offer missing report URLs as valid actions.
- Use semantic keyboard-operable controls for history entries and report/PDF actions.
- Refresh history when report availability and workflow completion events occur.

**Completion criteria / tests**

- Component tests cover report-ready events, unavailable/failed PDF generation, empty and populated history, refresh, and report opening.
- Keyboard and accessibility tests confirm names, focusability, and activation for history and report actions.
- Report URLs are taken from the selected history record and are not replaced with another run's URL.

### 9. Reconcile setup documentation and run the validation suite

**Traceability:** [Req 2.1] [Req 2.2] [Req 4.2] [Req 5.1] [BE 2.1] [BE 2.2] [BE 4.1] [FE 1.1]

- Update the root README to match the checked-in split Angular/Python service layout, actual setup commands, and existing configuration files.
- Document required versus optional settings consistently with implemented workflow behavior; do not claim unsupported paths or integrations.
- Run targeted backend tests and the Angular test/build commands available in the repository; report external-dependency or configuration limitations explicitly.

**Completion criteria / tests**

- A fresh developer can follow the documented setup to install frontend/backend dependencies, configure environment variables, and launch the two services.
- The repository's available Python test commands and Angular test/build scripts complete successfully, or documented external prerequisites are reported as blockers.
