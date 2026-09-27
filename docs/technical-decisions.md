# Technical Decisions & Trade-Offs — CPAE

## Decision 1: NestJS over Plain Express or Fastify

- **Context:** The clinical domain encompasses numerous complex business rules (scheduling conflict algorithms, recurring sessions, HIPAA/LGPD audit trails, medical document indexing).
- **Decision:** Selected NestJS with TypeScript for its robust module dependency injection system, built-in validation pipes (`class-validator`), and architecture convention.
- **Trade-off:** Slightly higher boilerplate compared to Express, but significantly superior maintainability, testability, and refactoring confidence across an evolving domain.

## Decision 2: Prisma ORM over TypeORM or Raw SQL

- **Context:** The team needed rapid feature iteration combined with absolute type safety between database schema and API controllers.
- **Decision:** Implemented Prisma ORM with automated migrations.
- **Trade-off:**
  - *Advantage:* Generated TypeScript types prevent runtime schema mismatches.
  - *Limitation:* Complex reporting queries (e.g., monthly practitioner occupancy vs revenue aggregates) occasionally required raw SQL execution (`prisma.$queryRaw`) due to Prisma's join aggregation limitations.

## Decision 3: MinIO S3 Object Storage over Relational Bytea / Local Filesystem

- **Context:** Medical consultations generate large binary files: neuropsychological assessment scoresheets, speech therapy video clips, and clinical summary PDFs.
- **Decision:** Deployed MinIO as a containerized, S3-compatible object store.
- **Trade-off:**
  - *Advantage:* Zero storage impact on PostgreSQL; transparent migration path to AWS S3 or Cloudflare R2 if cloud scaling is required without modifying application code.
  - *Trade-off:* Requires managing MinIO bucket credentials and configuring presigned URL token lifecycles.

## Decision 4: Asynchronous Bull Queues with Redis

- **Context:** Generating comprehensive clinical synthesis documents involving multi-page PDF rendering and OCR extraction blocked the Node.js event loop during peak hours.
- **Decision:** Integrated `@nestjs/bull` powered by Redis to handle background workers for document generation, notification dispatches, and periodic schedule consistency checks.
- **Trade-off:** Added Redis infrastructure dependency, which was already beneficial for rate limiting and session caching.
