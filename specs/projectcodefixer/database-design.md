# ProjectCodefixer Database Design

## Scope

This design covers persistence required by the project requirements. The inspected implementation uses SQLite for workflow history and ChromaDB for code-index persistence. No new product data requirements were identified that require extending either store.

## Existing persistence

### Workflow history

The service initializes a SQLite database at the configured `DB_PATH` and creates `run_history` if it does not exist.

| Field | Type | Constraints / behavior |
| --- | --- | --- |
| `id` | `INTEGER` | Primary key; autoincrement |
| `run_id` | `TEXT` | Not null |
| `timestamp` | `TEXT` | Not null; stored as a formatted local timestamp |
| `storage_type` | `TEXT` | Not null; current values are `local` and `azure` |
| `report_url` | `TEXT` | Not null; uses `N/A` when no report URL is available |
| `errors` | `TEXT` | Nullable; submitted error descriptions are joined with commas |
| `status` | `TEXT` | Defaults to `completed`; current insertion relies on this default |

History is read newest-first and limited to 50 records. The inspected write path records a run after its report lookup and before its optional push; the schema currently has no distinct run lifecycle or per-error records.

### Code index and artifacts

ChromaDB persists the code index at the configured local path or Azure Files mount, according to storage mode. The indexer replaces the named `code_index` collection when building an index. Run outputs and assets are stored locally or in Azure Blob Storage as configured. These stores contain indexed source chunks and generated artifacts, not relational workflow records.

## Design statements

- **[DB 1.1]** No database schema change is required for the requirements. The existing `run_history` fields already support the required run identifier, timestamp, storage mode, report URL, submitted errors, and status [Req 3.6].
- **[DB 1.2]** Preserve the existing SQLite history table and its current 50-record read limit; do not add migrations, indexes, or retention behavior without a requirement that needs them [Req 3.6].
- **[DB 1.3]** Preserve ChromaDB and the configured local/Azure artifact storage for code indexes, generated reports, and related assets; no new persistence entities are introduced [Req 2.2] [Req 3.2] [Req 4.1].

## Migration and lifecycle

No migration is proposed. Startup continues to create the `run_history` table if absent. Existing records remain unchanged. Code-index replacement and artifact retention behavior remain as currently implemented.

## Known data-model limitations

The current schema records history as a single row per run. It does not establish a normalized error table, explicit run-state transitions, per-stage results, or a retention/deletion policy. The requirements do not define behavior that requires those changes. Open questions in `requirements.md` concerning failed-run history, report retention, and lifecycle/security policy should be resolved before changing this model.
