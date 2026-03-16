# Requirements: Document Upload & Management

This document captures the functional requirements for the Document Upload and Management feature, derived from the stakeholder requirements and aligned with the project’s spec-driven development workflow.

## Goal
Provide a centralized, secure, searchable place where Contoso employees can upload, store, share, and manage work-related documents within the ContosoDashboard application.

## Core Functional Requirements

### 1. Document Upload
- Users must be able to select one or more files and upload them with required metadata.
- Supported file types:
  - PDF, DOC/DOCX, XLS/XLSX, PPT/PPTX, TXT, JPG/JPEG, PNG
- Maximum file size: **25 MB per file**.
- Users must see upload progress and receive success/error messages.
- Required metadata: Title, Category (predefined list).
- Optional metadata: Description, Tags, Associated Project.
- System must capture: upload date/time, uploader, file size, MIME type (<=255 chars).
- Files must be stored outside `wwwroot` and served via authorized endpoints.
- File paths must be generated using a GUID-based pattern to prevent collisions and path traversal.

### 2. Validation & Security
- Reject unsupported file types with clear error messages.
- Reject files > 25 MB with clear error messages.
- Scan uploads for malware/viruses (training stub acceptable; must be clearly marked).
- Prevent path traversal; never use user-supplied filenames in storage paths.
- Enforce authorization on all document operations (view, download, delete, share).

### 3. Browsing & Search
- Provide a “My Documents” view for users to list their own documents.
- Display: Title, Category, Upload Date, File Size, Associated Project.
- Support sorting by Title, Upload Date, Category, File Size.
- Support filtering by Category, Project, Date Range.
- Support searching by Title, Description, Tags, Uploader Name, and Project.
- Search results must only include documents the user is authorized to access.

### 4. Download, Preview & Access
- Users must be able to download any document they have access to.
- Support in-browser preview for PDF and image files.
- Provide secure endpoints to serve files from storage (not direct static access).

### 5. Metadata Editing & Versioning
- Document owners may edit metadata (Title, Description, Category, Tags).
- Document owners may replace the underlying file; old files must be removed.
- Changes must be reflected immediately in lists/search results.

### 6. Deletion & Sharing
- Document owners may delete their documents (with confirmation).
- Project Managers may delete any document within their projects.
- Deletion must remove both metadata and physical file(s).
- Users may share documents with other users or teams.
- Receivers of shared documents should get an in-app notification.
- Shared documents must appear in a “Shared with Me” view.

### 7. Integration Points
- Tasks: allow attaching documents to tasks and uploading from task detail pages.
- Dashboard: show “Recent Documents” widget and include document counts in summary cards.
- Notifications: send notifications when documents are shared or added to a user’s project.

### 8. Auditing & Reporting
- Log document-related activities: upload, download, delete, share.
- Support reports for:
  - Top document types by volume
  - Most active uploaders
  - Document access patterns

## Non-Functional Requirements
- Must work offline without external cloud services (training environment requirement).
- Upload must complete within 30 seconds for 25 MB files.
- List pages must load within 2 seconds for up to 500 documents.
- Search must return results within 2 seconds.
- Preview must load within 3 seconds for supported formats.

## Technical Constraints
- Use local filesystem storage (outside `wwwroot`) in training mode.
- Implement `IFileStorageService` abstraction for future cloud migration (Azure Blob, etc.).
- Database: `DocumentId` must be an integer (consistent with existing keys).
- Database: `Category` must be stored as text (not integer enum).
- The feature must fit within current application architecture without large rewrites.
- Must comply with existing mock authentication model.

---

*This requirements document is intended to be referenced by the spec (`spec.md`) and implementation plan (`plan.md`).*