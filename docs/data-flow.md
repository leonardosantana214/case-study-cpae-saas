# Data Flow & Lifecycle — CPAE

## 1. Patient Intake & Onboarding Flow

```mermaid
sequenceDiagram
    autonumber
    actor Reception as Reception Staff
    participant Gateway as API Gateway
    participant PatientMod as Patient Service
    participant Audit as Audit Logger
    participant DB as PostgreSQL

    Reception->>Gateway: POST /api/v1/patients (Demographics, Guardian Data)
    Gateway->>Gateway: Validate input via class-validator DTO
    Gateway->>PatientMod: createPatient(dto, tenantId)
    PatientMod->>DB: INSERT INTO patients (tenant_id, name, contact, status)
    DB-->>PatientMod: Created Record (id: uuid)
    PatientMod->>Audit: Record Intake Action (staffId, patientId, timestamp)
    PatientMod-->>Gateway: Sanitized Patient Summary
    Gateway-->>Reception: HTTP 201 Created (Patient Card View)
```

## 2. Clinical Evolution & Document Attachment Pipeline

When a medical or psychological consultation concludes:
1. **Practitioner Drafts Evolution:** Clinical notes are entered via the web portal interface.
2. **Document Attachment (Optional):** Practitioner uploads diagnostic files or test score sheets.
3. **Stream to Object Storage:** File payload is streamed via multipart upload to MinIO into the tenant's isolated bucket path:
   ```
   tenants/{tenantId}/patients/{patientId}/records/{recordId}/{fileUuid}.pdf
   ```
4. **Relational Linkage:** Metadata, file size, storage key, and upload checksum are persisted in PostgreSQL.
5. **Report Generation (Async):** If a consolidated evaluation report is requested, a Bull job is dispatched:
   - Worker retrieves historical notes, demographic data, and standardized diagnostic test scores.
   - Worker renders the PDF document using `pdf-lib` / `docx` templates.
   - Worker uploads the compiled report back to MinIO and updates the record status to `READY`.
