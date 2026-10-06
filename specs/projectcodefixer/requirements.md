# ProjectCodefixer Requirements

## Status and scope

Project-level requirements for the existing ProjectCodefixer application, based on the checked-in README, Angular client, and Python service at `C:\workspace\projectcodefixer`. These statements describe the product behavior to preserve and clarify; they are not a proposal to add unimplemented capabilities.

The project provides a browser workflow and Python command-line tools for indexing a Git repository, analyzing supplied errors or a generic audit request, generating candidate fixes, optionally verifying them in a sandbox, and presenting reports and run history.

## Functional requirements

### Workflow input and execution

- **[Req 1.1]** The application shall allow a user to submit a Git repository URI and one or more error descriptions to analyze.
- **[Req 1.2]** The workflow API shall reject requests with no errors or more than 10 errors.
- **[Req 1.3]** The workflow shall accept options to skip cloning, create a fix branch, and push the branch after the run. The browser client currently requests branch creation and exposes skip-cloning and auto-push controls.
- **[Req 1.4]** The workflow shall report an already-running execution rather than starting a second concurrent run.
- **[Req 1.5]** The application shall expose a health endpoint that reports service status and the active local or Azure storage mode.

### Indexing, retrieval, and analysis

- **[Req 2.1]** Before analysis, the service shall clone the requested repository and branch when cloning is enabled, or use the existing local codebase when cloning is skipped.
- **[Req 2.2]** The indexer shall parse configured source-file extensions and create a persistent code index and dependency graph for retrieval.
- **[Req 2.3]** The service shall support analysis from supplied error descriptions and a generic code-quality audit. The command-line interface shall also support fetching an error description from a configured Jira ticket.
- **[Req 2.4]** The service shall retrieve relevant code context for each error and expand or recover context when applicable before requesting analysis.
- **[Req 2.5]** The analysis result shall be valid JSON with an overall analysis and a list of fixes. Each fix shall identify its file and function, explain the change, and provide corrected function code. If no relevant function is identified, the fixes list shall be empty.
- **[Req 2.6]** Generated changes shall be limited to functions related to the reported error, shall preserve unrelated functions, and shall preserve existing literal-string content as directed by the analysis prompt.

### Verification and delivery

- **[Req 3.1]** When automated verification is requested, the service shall copy the indexed repository into a per-run sandbox, apply candidate fixes there, and run verification without applying those changes directly to the indexed source.
- **[Req 3.2]** The service shall create an HTML report containing the submitted errors, analysis/fix status, before-and-after code, verification summary, and execution log.
- **[Req 3.3]** The browser client shall display workflow progress and log messages while a run is executing, and shall display the report when it is available.
- **[Req 3.4]** The browser client shall provide a PDF download action for the displayed report when its PDF-generation dependency is available.
- **[Req 3.5]** When branch creation and push are enabled, the service shall create a fix branch and attempt to push it to the configured remote; the workflow shall report the outcome.
- **[Req 3.6]** The service shall record completed workflow history, including run identifier, timestamp, storage mode, report URL, and submitted errors. The history endpoint shall return the most recent 50 records, and the browser client shall let users open a report from history.

### Storage and configuration

- **[Req 4.1]** The service shall support local storage and Azure storage modes for the code index and generated assets or reports, subject to the platform-specific storage behavior configured by the service.
- **[Req 4.2]** The service shall read model, Git, Jira, and Azure-storage configuration from environment settings rather than requiring credentials in source code.

## Non-functional requirements

- **[Req 5.1]** Workflow execution shall not block the initial API response; the API shall acknowledge a run and stream subsequent log and progress events to connected clients.
- **[Req 5.2]** A run shall expose progress for its clone, index, query, report, and branch stages, including failure states where a stage reports failure.
- **[Req 5.3]** Repository indexing and fix verification shall operate on filesystem data rooted in the configured service/repository paths so generated changes and test execution can be isolated from the application source.

## Acceptance criteria

1. A valid workflow request starts one run; a request with an empty error list, more than 10 errors, or a concurrently active run is rejected or reported as such.
2. The client can determine backend health and storage mode, observe run logs/progress, and display a report URL emitted for the run.
3. For a repository that can be cloned and indexed with valid configuration, analysis returns the documented JSON shape; an unmatched error produces no proposed fixes.
4. When verification is enabled, candidate edits and verification output are produced in a run-specific sandbox and are included in the run report.
5. A completed run is available from history with its report link, and the history response is limited to the 50 newest entries.
6. Local and Azure storage configurations route supported persistent artifacts to the selected storage behavior.

## Out of scope

- Adding new code-analysis or remediation capabilities beyond the existing RAG, graph-context, model-analysis, and verification workflow.
- Defining new authentication, authorization, tenancy, or user-account behavior; none is established by the inspected application flow.
- Replacing Angular, FastAPI, ChromaDB, SQLite, Azure OpenAI, Jira, or Azure Storage.
- Mandating a particular deployment topology beyond the local and Azure behavior already represented in configuration and launch scripts.

## Assumptions and open questions

1. The requested specification covers the complete existing application, including its browser client and command-line entry points, rather than one isolated feature.
2. The root README describes intended capabilities, but some setup instructions and paths do not match the checked-in split frontend/service layout. Which source should be authoritative where they differ: running code or README?
3. The workflow currently requires `GIT_AUTH_TOKEN` in its web service path even though documentation describes it as optional for public repositories. Should public repositories be supported without a token?
4. The browser UI exposes a push toggle but always sends `create_branch: true`; should users be able to disable branch creation independently from pushing?
5. Should Jira ticket analysis be available in the browser, or is the existing command-line-only entry point sufficient?
6. Verification behavior and supported project languages/test commands are not fully specified. Should verification remain the current Python-oriented sandbox behavior, or should the product support repository-specific test commands?
7. The inspected web workflow creates and pushes Git branches but does not clearly expose a review/approval checkpoint before push. Is human approval required before any remote write?
8. Requirements for secret redaction, access control, retention/deletion of code and reports, and handling of untrusted repositories need product/security decisions before they can be specified as guarantees.
