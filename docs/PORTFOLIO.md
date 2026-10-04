# Portfolio engineering guide

## Problem and architecture

Titanzero is an exploratory Laravel/nWidart module archive for AI-assisted business workflows. The source is a real PHP module, but its relationship to the canonical Workforce platform is not established. The module separates AI declarations, action handlers, contracts, persistence, HTTP/controllers, services, evaluation, Filament pages, and health/repair concerns.

Useful code map:

- `module.json` and `module.manifest.json` — module/provider metadata, Filament paths, health checks, repair recipe, and declared AI manifest references.
- `AI/Guardrails/guardrails.json` — tenant-scoped input/output screening declaration, blocked terms, permission-check and action-log requirements.
- `AI/Citations/citation.schema.json` — structured citation fields and presentation limits.
- `manifests/ai_tools.json` — declared query, document-proposal, agent-run, and signal-ingest tools.
- `Services/Guardrails/BlockedTermGuardrailService.php` — runtime screening implementation.
- `Evaluation/AgentEvaluator.php` and `Jobs/RunAgentEvaluationJob.php` — response scoring and queued evaluation path.
- `Services/Security/CrossTenantGuard.php` and `Traits/CompanyScoped.php` — explicit company-boundary checks and model scoping.
- `Http/`, `Filament/`, `Resources/`, `Wizards/` — host-facing and operator-facing surfaces.

The notable design choice is separating recommendation/evaluation from authority: tool declarations and guardrails are data-driven, while company checks and persistence are explicit PHP paths.

## Implemented intelligence

The evaluator records task completion, hallucination flag, tool accuracy, response latency, and a weighted composite. The tests show that scores persist with `company_id`, that hallucination signals reduce the composite, and that scores remain separately queryable by company.

The guardrail tests exercise clean input/output, case sensitivity, multi-word blocked terms, custom handlers, manifest loading, and fail-open behaviour for a missing custom handler. That last behaviour is an important limitation to understand; a missing handler does not prove strong enforcement.

The cross-tenant tests cover same-company success, mismatched-company rejection, missing/null `company_id`, collection checks, and object records. These are valuable negative tests, not proof that every route, job, cache, file, or export path is tenant-safe.

## Quickstart and verification

The checked-in `composer.json` declares PSR-4 autoloading for `Modules\TitanZero\` and the `orhanerday/open-ai` dependency:

```bash
composer install
```

This is a module, not a standalone application. A Laravel host/module loader is required for `module_path()`, database migrations, Filament, `RefreshDatabase`, and the `Tests\TestCase` feature tests. No root PHPUnit configuration or test script was verified, so a host command such as `php artisan test` must be supplied by the integrating application rather than invented here.

## Evidence and known gaps

- Unit and feature test files are present under `Tests/Unit` and `Tests/Feature`, but no pass result was observed in this review.
- `module.json` points at `AI/Retrieval/retrieval.policy.json`; the referenced file was not available on the inspected default branch. Retrieval should therefore be treated as a declared-but-unverified capability.
- The two metadata files are not identical: `module.json` declares AI manifest links and two providers, while `module.manifest.json` carries health/repair metadata and one provider. A host should define which manifest is authoritative before release.
- `TitanZero_V1.9_Fixed.zip`, pass-labelled README files, and other historical material were preserved. No deletion was justified without a provenance and reference check.

## Portfolio status

Keep this repository as an exploratory module/source snapshot until lineage, host integration, licenses, and clean-checkout verification are established. It is not described here as production-ready or as the canonical Workforce implementation.
