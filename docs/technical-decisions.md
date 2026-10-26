# Technical Decisions & Trade-Offs — CPAE

## Decision 1: NestJS over plain Express or Fastify

- **Context:** The domain has many business rules: scheduling conflicts, agenda transfer, payout rules, audit trails, clinical record versioning.
- **Decision:** NestJS with TypeScript for module-based dependency injection, validation pipes (`class-validator`) and conventions that keep a 34-module monolith navigable.
- **Trade-off:** More boilerplate than Express, in exchange for maintainability and testability.

## Decision 2: Prisma ORM over TypeORM or raw SQL

- **Context:** Fast iteration with type safety between schema and API (79 models).
- **Decision:** Prisma with migrations, plus a Client extension (`$extends`) that injects the `clinicUnitId` filter into unit-owned models.
- **Trade-off:** Generated types prevent schema drift. The unit filter only covers models listed in the extension, so new unit-owned models must be added to that list explicitly, and administrative bypass paths need care.

## Decision 3: Storage with local-disk fallback, MinIO optional

- **Context:** Clinical media (photos, attachments, scanned documents) must stay private and the system must run even when object storage is not configured.
- **Decision:** `StorageService` uses MinIO when available and falls back automatically to private local disk. Some clinical uploads (attachments, activity photos) are still written directly to `uploads/` on the container volume.
- **Trade-off:** Simple operations and no hard dependency on MinIO, but two storage paths to maintain and no per-unit buckets. Presigned read URLs currently last 24 hours; shortening this is on the hardening roadmap. Consolidating uploads behind `StorageService` is a natural next step.

## Decision 4: Bull queues scoped to the migration pipeline

- **Context:** Importing legacy Google Drive material and running OCR on it is slow and must not block API requests.
- **Decision:** `@nestjs/bull` on Redis for the `drive-import` and `ocr-processing` queues. Audit writes stay synchronous so a record exists when the request completes.
- **Trade-off:** Redis is a required dependency; it also backs Luna session memory. The global rate limiter uses in-memory storage rather than Redis.

## Decision 5: Self-hosted LLM inference through Ollama

- **Context:** The assistant handles clinical context and should avoid sending prompts to external LLM APIs.
- **Decision:** Luna calls Ollama running as a container in the production compose stack (defaults `gemma3:4b` and `qwen2.5:1.5b`), with guardrails, sandboxed tools, redaction and a usage budget.
- **Trade-off:** Small local models limit answer quality, which is why the design leans on routing, grounding and golden tests. OCR still uses Google Cloud Vision, a third-party service.
