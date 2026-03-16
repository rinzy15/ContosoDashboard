# Quickstart: Document Upload & Management Feature

This quickstart guide explains how to run the ContosoDashboard app locally and test the new document upload and management feature.

## Prerequisites

- .NET 10 SDK installed
- Windows (recommended, but Blazor Server is cross-platform)
- A terminal/PowerShell session in the repo root (`ContosoDashboard`)

## Run the App

1. Open a terminal in the repository root:
   ```powershell
   cd "D:\GitHubSpec kit\ContosoDashboard"
   ```
2. Restore and build the project:
   ```powershell
   dotnet restore
   dotnet build
   ```
3. Run the app:
   ```powershell
   dotnet run --project ContosoDashboard\ContosoDashboard.csproj
   ```
4. Open a browser and go to:
   - `https://localhost:5001` (HTTPS)
   - `http://localhost:5000` (HTTP)

## Accessing the Feature

- Sign in using the mock auth flow (the app currently uses cookie-based mock authentication).
- Navigate to the **Documents** section (this will be added as part of the feature). If not visible, verify the route `/documents`.

## Uploading a Document

1. On the Documents page, click **Upload**.
2. Select one or more supported files (PDF, DOCX, XLSX, PPTX, TXT, JPG, PNG).
3. Provide required metadata: **Title**, **Category**.
4. Optionally add a description, tags, and associate the document with a project.
5. Submit and confirm that the upload completes and the document appears in your list.

## Verifying Storage

- Uploaded files are stored in a local directory outside `wwwroot` (e.g., `AppData/uploads`).
- Document metadata is stored in the SQLite database (`Data/ApplicationDbContext`).

## Testing Permissions

- As the document owner, verify you can view, download, and delete your document.
- As a Team Lead or Project Manager, verify you can view/download/delete documents within your scope.
- Verify that unauthorized users cannot access shared documents.

## Debugging Tips

- If the app fails to start, ensure the SQLite database file is writable and not locked.
- Enable detailed logging by setting `Logging:LogLevel:Default` to `Debug` in `appsettings.Development.json`.

## Next Steps

Once the feature is implemented, you can validate it via automated tests (unit + integration) and by exercising:
- Search/filter/sort functionality on the My Documents page
- Document sharing with other users/teams
- Document preview (PDF/image)
- Audit logs for upload/download/delete events
