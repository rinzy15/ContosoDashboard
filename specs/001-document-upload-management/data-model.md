# Data Model: Document Upload & Management

This document defines the key entities, their attributes, and relationships needed to support document upload, sharing, access control, and auditing.

## Entities

### Document
Represents a stored file and its metadata.

- **DocumentId** `(int, PK)` — Surrogate key, consistent with existing integer keys.
- **Title** `(string, required)` — User-provided title.
- **Description** `(string, optional)` — User-provided description.
- **Category** `(string, required)` — One of: `Project Documents`, `Team Resources`, `Personal Files`, `Reports`, `Presentations`, `Other`.
- **Tags** `(string, optional)` — Comma-separated list or JSON array of tags for search.
- **ProjectId** `(int?, FK)` — Optional reference to `Project` (nullable for personal documents).
- **UploaderId** `(int, FK)` — Reference to the `User` who uploaded the document.
- **UploadDateUtc** `(DateTime, required)` — When the document was uploaded.
- **FileSizeBytes** `(long, required)` — Size of the stored file.
- **MimeType** `(string, required, max 255)` — MIME type for the file (e.g. `application/pdf`).
- **StoragePath** `(string, required)` — GUID-based path used by storage service.
- **OriginalFileName** `(string, required)` — The original filename provided by the user (for UI display).
- **IsDeleted** `(bool, required)` — Soft-delete flag (optional, if desired for audit/recovery).

### DocumentShare
Represents an explicit share grant to a user or team.

- **DocumentShareId** `(int, PK)`
- **DocumentId** `(int, FK)`
- **SharedWithUserId** `(int?, FK)` — Nullable: grant to a specific user.
- **SharedWithTeamId** `(int?, FK)` — Nullable: grant to a team (based on existing team model).
- **SharedByUserId** `(int, FK)` — The user who created the share.
- **SharedAtUtc** `(DateTime, required)` — Timestamp of when the share was created.
- **ExpiresAtUtc** `(DateTime?, optional)` — Optional expiration date for the share.

_Rules:_
- Either `SharedWithUserId` or `SharedWithTeamId` must be non-null (not both null).
- A unique constraint can be applied per (DocumentId, SharedWithUserId, SharedWithTeamId) to prevent duplicates.

### DocumentActivityLog
Records document-related actions for auditing and reporting.

- **DocumentActivityLogId** `(int, PK)`
- **DocumentId** `(int, FK)`
- **UserId** `(int, FK)` — User performing the action.
- **Action** `(string, required)` — e.g., `Upload`, `Download`, `Delete`, `Share`, `Unshare`, `Preview`, `MetadataEdit`.
- **ActionDetails** `(string, optional)` — Free-form details (e.g., target user/team for share, old/new values for metadata edits).
- **TimestampUtc** `(DateTime, required)`
- **SourceIp** `(string, optional)` — (if available) for audit.

## Relationships

- `Document (UploaderId) -> User` (many-to-one)
- `Document (ProjectId) -> Project` (many-to-one, optional)
- `DocumentShare (DocumentId) -> Document` (many-to-one)
- `DocumentShare (SharedWithUserId) -> User` (many-to-one, optional)
- `DocumentShare (SharedWithTeamId) -> Team` (many-to-one, optional)
- `DocumentActivityLog (DocumentId) -> Document` (many-to-one)
- `DocumentActivityLog (UserId) -> User` (many-to-one)

## Notes
- The model intentionally uses integer primary keys to align with existing User and Project keys.
- `Category` is stored as text to keep the model simple and flexible.
- A separate `File` table is not needed since the file content is stored in the filesystem and referenced via `StoragePath`.
- Future cloud migration: `StoragePath` can be a blob name; `IFileStorageService` will resolve how to interpret it.
