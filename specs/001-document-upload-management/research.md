# Research: Document Upload & Management

This research artifact captures the key technical findings, existing architecture context, and open questions that will guide the implementation of the document upload and management feature.

## Existing Application Context

- **Framework**: ASP.NET Core 10 with Blazor Server.
- **Data layer**: `ContosoDashboard.Data.ApplicationDbContext` uses SQLite via EF Core.
- **Authentication/Authorization**:
  - Cookie-based auth for training (mock users/roles)
  - Roles defined: Employee, TeamLead, ProjectManager, Administrator
  - Authorization policies in `Program.cs` map these roles (Employee+ for general access, TeamLead for team, etc.)
- **Services**:
  - Services are registered in DI in `Program.cs` (e.g., `UserService`, `ProjectService`, `TaskService`).

## Key Implementation Considerations

### File Storage
- Must store files outside of `wwwroot` (security requirement).
- We will introduce an `IFileStorageService` interface with at least:
  - `Task<string> UploadAsync(Stream fileStream, string contentType, string targetPath)`
  - `Task<Stream> DownloadAsync(string storagePath)`
  - `Task<bool> DeleteAsync(string storagePath)`
  - `Task<string> GetUrlAsync(string storagePath)` (for future cloud use)
- Implement a `LocalFileStorageService` using `System.IO` and a configured base directory (e.g., `AppData/uploads`).
- Storage paths should be GUID-based and include user/project scoping (e.g., `{userId}/{projectId or "personal"}/{guid}{ext}`).

### Database Model
- Add new EF entities:
  - `Document` (metadata + storage path)
  - `DocumentShare` (links documents to users/teams)
  - `DocumentAudit` or `DocumentActivityLog` (records upload/download/delete/share)
- Ensure `DocumentId` is an `int`, and category is stored as `string`.

### Authorization Rules
- Document access should be gated by roles and ownership:
  - Owners can view/edit/delete their documents.
  - Team Leads can view/delete documents owned by their team members.
  - Project Managers can view/delete documents associated with their projects.
  - Administrators can view/delete everything.
- Sharing should grant access without changing ownership (ACL-style).

### UI/UX Approach
- Add a new top-level page or section for document management (e.g., `/documents`).
- Implement "My Documents" as a table with sorting/filtering/search.
- Add "Recent Documents" widget to the dashboard home page.
- Integrate document attachments into the task details page.

### Security & Validation
- Validate file type by extension and MIME type (whitelist PDF, Office formats, text, images).
- Reject files > 25 MB.
- Implement a stubbed antivirus scan (e.g., a function that always returns clean with a clear TODO marker).
- Ensure download/preview endpoints verify document access before serving.

## Open Questions

1. **Teams model**: How are teams represented in the current domain model? (Needed for enforcing TeamLead scope and sharing by team.)
2. **Notification system**: How should notifications be triggered? (existing `NotificationService` likely can be reused.)
3. **UI framework conventions**: Are there existing shared components for tables, dialogs, and toasts we should reuse?
4. **File size handling**: Is there an existing global upload size limit configuration we should align with? (likely in `appsettings.json`)
5. **Audit logging**: Should document activity logs go into the same DB table as other audit logs, or a new table?

## Next Steps (Phase 1)

- Finalize the data model in `data-model.md` (entities + relationships).
- Define API contracts/endpoints in `contracts/` (document controller routes, request/response shapes).
- Create a quickstart guide in `quickstart.md` for running the feature locally and testing uploads.
- Develop a phased task list in `tasks.md` (UI, backend, security, tests).
