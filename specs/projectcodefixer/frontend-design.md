# ProjectCodefixer Frontend Design

## Scope and existing structure

The browser client is a single-page Angular 19 application in `jpa-projectcodefixer-web`. `AppComponent` renders the repository/error form, backend status and storage mode, workflow progress and logs, the current report, and a slide-out run-history drawer. `RagService` calls the existing `/api` endpoints and opens the workflow log stream with Server-Sent Events.

This design preserves the current single-page layout and API boundary. It describes the current user-visible behavior and highlights existing mismatches; it does not introduce a new screen or service endpoint.

## Design statements

### Main workflow and input

- **[FE 1.1]** Keep repository URI and error descriptions as the primary workflow inputs. Continue splitting the error field into trimmed, non-empty entries before submission, and display request validation errors when no errors or more than 10 are supplied [Req 1.1] [Req 1.2] [BE 1.2].
- **[FE 1.2]** Preserve the existing controls for skipping cloning and choosing whether to push a fix branch. Ensure the branch-creation choice passed to the API reflects the UI state rather than being hard-coded. Do not imply that disabling push disables branch creation [Req 1.3] [BE 1.2] [BE 2.6].
- **[FE 1.3]** Show the backend connection state and active local/Azure mode in the page header using the health response. When the service is unavailable, keep the state visibly disconnected rather than presenting a successful connection [Req 1.5] [Req 4.1] [BE 1.1].
- **[FE 1.4]** Disable duplicate workflow submission while a request is being submitted or a run is active. If the API reports that another run is active or rejects the request, show an actionable error and return the controls to an appropriate state [Req 1.4] [Req 5.1] [BE 1.3].

### Progress, activity, and errors

- **[FE 2.1]** Keep the existing progress sequence for clone, index, analysis, report, and branch push. Render pending, active, completed, and failed states based on the progress event stream [Req 5.2] [BE 1.4] [BE 4.3].
- **[FE 2.2]** Display streamed log messages while the workflow runs, and stop the running state on a terminal success or failure event. A failed terminal outcome must not be displayed as a successful completion [Req 3.3] [Req 5.1] [Req 5.2] [BE 1.4] [BE 4.3].
- **[FE 2.3]** Render streamed log content as text, not executable markup. The current template inserts formatted log strings through `[innerHTML]`; the design must avoid interpreting repository-, model-, or process-originated content as HTML [Req 3.3] [BE 3.1] [BE 4.4].

### Reports and history

- **[FE 3.1]** When a report-ready event arrives, display the report in the existing embedded report area and expose the existing PDF-download action when PDF generation is available [Req 3.2] [Req 3.4] [BE 3.1] [BE 3.2] [DB 1.3].
- **[FE 3.2]** Preserve the history drawer, empty-history state, refresh action, and report-opening interaction. Display each run's identifier, timestamp, and storage mode; opening a run shall use its persisted report URL [Req 3.6] [BE 1.5] [BE 3.3] [DB 1.1] [DB 1.2].
- **[FE 3.3]** Refresh displayed history after a run reaches its terminal state and after a report becomes available, without presenting missing reports as valid report links [Req 3.6] [BE 1.5] [BE 3.3].

### Accessibility and interaction

- **[FE 4.1]** Keep visible labels for the repository URI, error list, and workflow options. Ensure progress state and backend connection state are conveyed by text/state as well as color or icons [Req 1.1] [Req 3.3] [Req 5.2].
- **[FE 4.2]** Ensure the history drawer, refresh/close controls, workflow submission, and report/PDF actions are operable using keyboard interaction and have accessible names consistent with their visible purpose [Req 3.3] [Req 3.4] [Req 3.6].
- **[FE 4.3]** Associate workflow validation and execution errors with the relevant form or progress region so they are perceivable without relying on the activity log alone [Req 1.2] [Req 3.3] [BE 4.3].

## Existing gaps and constraints

1. `create_branch` is currently sent as `true` regardless of component state; the browser exposes a push toggle but not an independent branch toggle [Req 1.3] [BE 2.6].
2. The browser currently starts code verification as part of every web workflow, while the CLI accepts a verification option [Req 3.1] [BE 2.5].
3. Workflow start errors are not handled by the current `startWorkflow()` subscription, so an API rejection can leave the view in an inaccurate loading state [Req 1.4] [Req 5.1] [BE 1.3].
4. The current history cards use clickable `div` elements rather than explicit buttons/links; keyboard activation and semantic action are not established [Req 3.6].
5. The PDF action currently logs to the browser console if the global PDF library is unavailable; the interface needs a user-visible unavailable/failure state [Req 3.4].
6. The report iframe URL is passed through a custom sanitizer. The report source and expected browser policy should remain limited to the configured application report route [Req 3.2] [BE 3.2].

## Traceability

All frontend design statements reference project requirements. Backend references point to the API, workflow, report, and error-handling design. Database references apply only to stored history and persisted report artifacts; the frontend introduces no new data model.
