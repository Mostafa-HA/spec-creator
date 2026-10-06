# ProjectCodefixer Backend Design

## Scope and architecture

The backend is a Python service implemented with FastAPI in `jpa-projectcodefixer-service`. `src.main` mounts the API router under `/api`, initializes SQLite run history at startup, and serves local reports as static files or proxies report requests to Azure Blob Storage. `src.api.router` provides health, workflow-run, log-stream, and history routes. `src.api.workflow` orchestrates indexing, analysis, reports, history, and optional Git push. The CLI path is `rag_scraper.rag_indexer` followed by `query/rag_querier.py`.

The flow below describes the existing architecture and current behavior. It does not introduce a new API or service boundary.

## Design statements

### API contract and validation

- **[BE 1.1]** Preserve `GET /api/health`, returning the service status and configured storage mode [Req 1.5] [Req 4.1].
- **[BE 1.2]** Preserve `POST /api/workflow/run` with a validated `StartRequest` containing `git_uri`, `errors`, `create_branch`, `skip_cloning`, and `push_on_finish`. Require at least one error and reject more than 10 errors through the Pydantic validator [Req 1.1] [Req 1.2] [Req 1.3].
- **[BE 1.3]** Reject a workflow start while a run is active; otherwise acknowledge the request and execute the workflow asynchronously so the HTTP request does not wait for indexing and analysis [Req 1.4] [Req 5.1].
- **[BE 1.4]** Preserve `GET /api/workflow/logs` as a Server-Sent Events stream of log, progress, report-ready, and final-state messages. Keep its event vocabulary compatible with the Angular client's progress and report handling [Req 3.3] [Req 5.1] [Req 5.2].
- **[BE 1.5]** Preserve `GET /api/workflow/history` as the history read surface, returning the 50 most recent persisted runs [Req 3.6] [DB 1.1] [DB 1.2].

### Workflow orchestration and integrations

- **[BE 2.1]** `start_workflow_task` shall use the submitted repository URI and options to invoke the indexer, then invoke the query/fix pipeline with the submitted errors. The indexer and query services remain separate Python processes launched from the service root, with their output forwarded to the workflow log stream [Req 2.1] [Req 2.2] [Req 2.3] [Req 2.4] [Req 5.2].
- **[BE 2.2]** The indexer shall build the configured ChromaDB code collection and dependency graph; retrieval shall use the index and available graph/file recovery before analysis. Do not define a new indexing or retrieval API [Req 2.2] [Req 2.4] [DB 1.3].
- **[BE 2.3]** The analysis service shall call the configured Azure OpenAI deployment with the repository context and error description, then validate/consume the JSON result shape specified by the analysis prompt: analysis text and fixes containing target file, function, explanation, and corrected code. An error that cannot be mapped to retrieved code shall not be attributed to an unrelated function [Req 2.5] [Req 2.6].
- **[BE 2.4]** The CLI shall continue to support custom error analysis, generic audit, and Jira ticket lookup. Jira access shall remain optional configuration for CLI ticket mode; Jira failures shall be surfaced as an unavailable-ticket/fallback outcome rather than treated as a successfully retrieved ticket [Req 2.3] [Req 4.2].
- **[BE 2.5]** When verification is enabled, the query pipeline shall copy the target codebase into a run-specific sandbox, apply candidate fixes there, execute the current verification behavior, and capture before/after code and execution results for reporting. The indexed source tree shall not be the verification write target [Req 3.1] [Req 5.3].
- **[BE 2.6]** When branch creation and push are enabled, use the existing Git service to create a fix branch and push it to the submitted repository URI using configured credentials. Report the branch operation outcome in the workflow log [Req 1.3] [Req 3.5] [Req 4.2].

### Reports, history, and persistence

- **[BE 3.1]** The report service shall generate an HTML report containing the submitted errors, fix statuses, before/after code, verification summary, and execution log. Escape user- or model-provided text before embedding it in HTML [Req 3.2].
- **[BE 3.2]** In local mode, expose generated reports under `/api/reports/`; in Azure mode, read reports from the configured output container through the existing report route [Req 3.2] [Req 4.1] [DB 1.3].
- **[BE 3.3]** Persist run history through `src.api.history` in SQLite, retaining the existing `run_history` schema and startup initialization. Return the newest 50 entries and do not introduce migrations under this design [Req 3.6] [DB 1.1] [DB 1.2].
- **[BE 3.4]** Continue to use ChromaDB for the code index and the configured local/Azure storage implementation for generated assets and reports; no additional persistence model or database is introduced [Req 2.2] [Req 4.1] [DB 1.3].

### Configuration, authorization, and error behavior

- **[BE 4.1]** Load Azure OpenAI, Git, Jira, and Azure-storage settings from environment configuration; credentials shall not be added to source files or returned in API responses [Req 4.2].
- **[BE 4.2]** No authentication or authorization mechanism is established by the inspected API. This design does not claim that workflow, logs, history, or reports are access-controlled; access-control requirements remain an unresolved product/security decision [Req 4.2].
- **[BE 4.3]** Surface validation errors through normal FastAPI request validation and report operational failures through workflow error logs and failed progress stages. Do not report a failed indexing, analysis, report, or push operation as a successful stage [Req 1.2] [Req 3.3] [Req 3.5] [Req 5.2].
- **[BE 4.4]** Do not expose Git tokens, Azure credentials, or model API keys in streamed output or persisted history. The current workflow logs submitted repository URIs and errors; whether these inputs require redaction is an open security question in `requirements.md` [Req 4.2].

## Known behavior gaps to resolve during implementation planning

1. `process_active` is a module-level flag, while the run route checks and starts a thread separately; the current check/start sequence does not establish an atomic single-run lock across simultaneous requests [Req 1.4].
2. The browser workflow currently requires `GIT_AUTH_TOKEN` before starting, even for a repository that may be public, while the README describes the token as needed for private repositories [Req 2.1] [Req 4.2].
3. The workflow invokes the CLI with `--run-code` unconditionally, so the UI does not currently make verification optional even though the CLI supports an option [Req 3.1].
4. The browser sends `create_branch: true` unconditionally, although the API model accepts that option [Req 1.3] [Req 3.5].
5. Workflow exceptions can currently emit a final-completed progress event without an explicit final-failed state. Failure-state details should be aligned with the progress requirement [Req 5.2].
6. Workflow run reporting locates the latest matching report rather than carrying an explicit run identifier from the query process into report discovery. Correct run/report association should be preserved if concurrent runs are supported in a future requirement [Req 3.2] [Req 3.6].

## Traceability

All design statements reference project requirements. Persistence statements additionally reference the no-schema-change decisions in `database-design.md`. No new database entities, endpoints, integrations, or deployment components are proposed.
