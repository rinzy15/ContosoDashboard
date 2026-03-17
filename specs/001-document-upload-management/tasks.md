# Tasks: Document Upload & Management

**Input**: Design documents from `/specs/001-document-upload-management/`
**Prerequisites**: `plan.md` (required), `spec.md` (required for user stories), `data-model.md`, `contracts/`

> **Constitution Compliance**: Tasks must align with the project constitution (`.specify/memory/constitution.md`) and support the principles defined therein.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Prepare the existing ContosoDashboard project for the Document Upload & Management feature.

- [ ] T001 Update `appsettings.json` to include a configuration section for document storage (e.g., `DocumentStorage:BasePath`).
- [ ] T002 Add a `Documents` entry into the existing `NavMenu.razor` (or equivalent) to surface the feature.
- [ ] T003 [P] Add a new `Pages/Documents.razor` page and supporting components (`DocumentsList`, `DocumentUpload`, etc.) (initial scaffolding).
- [ ] T004 [P] Add a shared UI component for file upload progress (e.g., `Shared/FileUploadProgress.razor`).

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Implement core data model, storage abstraction, authorization helpers, and basic services.

**⚠️ CRITICAL**: No user story work can begin until foundational artifacts exist and are wired in.

- [ ] T005 Create `Document`, `DocumentShare`, and `DocumentActivityLog` models in `ContosoDashboard/Models/` (per `data-model.md`).
- [ ] T006 Add `DbSet<Document> Documents`, `DbSet<DocumentShare> DocumentShares`, and `DbSet<DocumentActivityLog> DocumentActivityLogs` to `ApplicationDbContext` and configure relationships/constraints.
- [ ] T007 Create `IFileStorageService` abstraction in `Services/IFileStorageService.cs`.
- [ ] T008 Implement `LocalFileStorageService` in `Services/LocalFileStorageService.cs` that stores files outside `wwwroot` using GUID-based filenames and validates against path traversal.
- [ ] T009 Register `IFileStorageService` and `DocumentService` (see below) in `Program.cs`.
- [ ] T010 Create `DocumentService` in `Services/DocumentService.cs` (and interface `IDocumentService`) responsible for:
  - Validating upload (size, type whitelist, required metadata)
  - Saving metadata to the database
  - Invoking `IFileStorageService` for file persistence
  - Enforcing access control rules (owner / team lead / project manager / shared users)
  - Recording audit events to `DocumentActivityLog`
- [ ] T011 Add storage configuration and ensure the app creates the storage directory on startup if missing.
- [ ] T012 Add a small “virus scan stub” helper that can be swapped later (e.g., `IVirusScanService`) and use it in `DocumentService` to mark files as scanned.
- [ ] T013 Add or update EF Core migrations for the new entities and ensure the migration is applied on startup (or document how to apply it).

**Checkpoint**: New data model, storage abstraction, and core service exist and can be unit tested.

---

## Phase 3: User Story 1 - Upload a document (Priority: P1) 🎯 MVP

**Goal**: Allow authenticated users to upload documents with required metadata and receive feedback on success/failure.

**Independent Test**: A user can upload a file, see it in their "My Documents" list, and download it again.

### Implementation for User Story 1

- [ ] T014 [P] [US1] Implement `DocumentsController` (or Blazor API service) to accept uploads, validate file type/size, and return success/failure messages.
- [ ] T015 [P] [US1] Implement `Pages/Documents.razor` UI for file selection, metadata entry (title, category, tags, description, project), and upload progress.
- [ ] T016 [P] [US1] Add client-side validation for required fields and supported file types (PDF, DOC/DOCX, XLS/XLSX, PPT/PPTX, TXT, JPG/JPEG, PNG).
- [ ] T017 [P] [US1] Ensure uploads are stored via `IFileStorageService` outside `wwwroot` and metadata is stored in the database (`Document` table).
- [ ] T018 [P] [US1] Add server-side validation and clear error messages for file size (>25 MB) and unsupported file types.
- [ ] T019 [P] [US1] Record an upload event in `DocumentActivityLog` (action: `Upload`).

**Checkpoint**: Users can upload files, and uploads are persisted with metadata and audit logs.

---

## Phase 4: User Story 2 - Browse and search documents (Priority: P1)

**Goal**: Provide a "My Documents" browser with sorting, filtering, and search.

**Independent Test**: A user can open "My Documents", filter/sort results, and execute a search that returns only authorized documents.

### Implementation for User Story 2

- [ ] T020 [P] [US2] Implement list/query API in `DocumentService` to return documents accessible by the requesting user, with paging, sorting, and filtering (category, project, date range).
- [ ] T021 [P] [US2] Implement search API in `DocumentService` that searches title, description, tags, uploader name, and project name (within authorization scope).
- [ ] T022 [P] [US2] Update `Pages/Documents.razor` to display a sortable table with columns: Title, Category, Upload Date, File Size, Project, Owner (if shared).
- [ ] T023 [P] [US2] Add UI controls to filter by category/project, sort columns, and run free-text search.
- [ ] T024 [P] [US2] Ensure list and search results only include documents the user is authorized to view (owned/shared/in-scope).

**Checkpoint**: Users can browse and search documents they are allowed to access.

---

## Phase 5: User Story 3 - Download and preview documents (Priority: P2)

**Goal**: Allow users to download documents or preview supported formats (PDF/images) while enforcing authorization.

**Independent Test**: A user can click a document and either download it or view it inline for supported formats.

### Implementation for User Story 3

- [ ] T025 [P] [US3] Implement secure download endpoint (e.g., `/api/documents/{id}/download`) that checks access control and streams the file via `IFileStorageService`.
- [ ] T026 [P] [US3] Implement preview endpoint (e.g., `/api/documents/{id}/preview`) that streams PDF/image content with the correct MIME type.
- [ ] T027 [P] [US3] Add UI actions/buttons in `Documents.razor` for Download and Preview.
- [ ] T028 [P] [US3] Record `Download` and `Preview` actions in `DocumentActivityLog`.

**Checkpoint**: Users can securely download and preview allowed documents.

---

## Phase 6: User Story 4 - Edit metadata and replace file (Priority: P2)

**Goal**: Provide a way for document owners to update metadata and replace the stored file.

**Independent Test**: A user can edit a document's metadata and replace the file; changes persist and are reflected in the list.

### Implementation for User Story 4

- [ ] T029 [US4] Add an "Edit" action in the document list that opens an edit form for title, category, tags, description, and project.
- [ ] T030 [US4] Implement backend update method in `DocumentService` to update metadata and validate inputs.
- [ ] T031 [US4] Implement file replacement workflow in `DocumentService`:
  - Replace the stored file via `IFileStorageService`,
  - Update metadata (file size, MIME type, storage path, original filename),
  - Remove the old file safely.
- [ ] T032 [US4] Record metadata edits and file replacements in `DocumentActivityLog` (actions: `MetadataEdit`, `ReplaceFile`).
- [ ] T033 [US4] Ensure authorization checks enforce that only owners (or authorized roles) can edit/replace.

**Checkpoint**: Owners can edit metadata and swap files, with audit logs recorded.

---

## Phase 7: User Story 5 - Delete and share documents (Priority: P3)

**Goal**: Allow owners (and scoped managers) to delete documents and share them with other users.

**Independent Test**: A user can delete a document they own (and scoped managers can delete within their scope) and share a document; recipients see the document in a "Shared with Me" view and receive a notification.

### Implementation for User Story 5

- [ ] T034 [US5] Implement delete endpoint in `DocumentService` and API, enforcing authorization rules (owner + team leads/project managers).
- [ ] T035 [US5] Ensure delete removes both metadata and the file from storage (or marks it deleted) and records `Delete` activity in `DocumentActivityLog`.
- [ ] T036 [US5] Add sharing UI in `Documents.razor` to select a user or team and create a share grant.
- [ ] T037 [US5] Implement `DocumentShare` persistence and enforce access control based on shares.
- [ ] T038 [US5] Add a "Shared with Me" view/filter in the Documents page.
- [ ] T039 [US5] Send an in-app notification to the recipient when a document is shared (e.g., via `NotificationService`).
- [ ] T040 [US5] Ensure unauthorized access attempts to shared/owned documents are blocked and logged.

**Checkpoint**: Document deletion and sharing work, and collaborators see shared documents and notifications.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories.

- [ ] T041 [P] Add unit tests for `DocumentService` covering upload validation, access control, and metadata updates.
- [ ] T042 [P] Add integration tests (if feasible) for key user flows: upload→list→download.
- [ ] T043 [P] Add UI tests or manual test checklist for file size/type validation, preview functionality, and sharing.
- [ ] T044 [P] Ensure all new endpoints and pages are covered by route authorization (Blazor `@attribute [Authorize]` / endpoint policies).
- [ ] T045 [P] Add logging/telemetry for failures (storage errors, unauthorized access) using existing logging conventions.
- [ ] T046 [P] Add documentation updates to `quickstart.md` and any README sections to explain how to use the new feature.
- [ ] T047 [P] Verify compliance with security requirements (no path traversal, no storing files under `wwwroot`, sanitized inputs).
- [ ] T048 [P] Ensure feature works in development and production configurations (appsettings, file paths).

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phases 3-7)**: Depend on Foundational phase completion
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2)
- **User Story 2 (P1)**: Can start after Foundational (Phase 2)
- **User Story 3 (P2)**: Can start after Foundational (Phase 2)
- **User Story 4 (P2)**: Can start after Foundational (Phase 2)
- **User Story 5 (P3)**: Can start after Foundational (Phase 2)

### Within Each User Story

- Models before services
- Services before endpoints
- Core implementation before integration

### Parallel Opportunities

- Tasks marked [P] can be done in parallel when staffing allows
- Different user stories can be implemented in parallel once foundational tasks are complete
- Tests can be written in parallel with implementation work where possible
