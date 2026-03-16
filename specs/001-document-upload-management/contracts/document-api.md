# API Contracts: Document Upload & Management

This document defines the key API endpoints, request/response shapes, and authorization requirements for the Document Upload & Management feature. All endpoints are expected to be implemented in the Blazor Server app (either as Razor Pages handlers or minimal API endpoints).

## Auth & Authorization

- All endpoints require authentication.
- Authorization is role-based and/or ownership-based:
  - Document owners can manage their documents.
  - Team Leads can manage documents owned by their team members.
  - Project Managers can manage documents associated with their projects.
  - Administrators can manage all documents.

## Endpoint: Upload Document

**POST** `/api/documents`

**Requires:** `Employee` role (any authenticated user)

**Request (multipart/form-data)**:
- `file` (file, required)
- `title` (string, required)
- `category` (string, required)
- `description` (string, optional)
- `tags` (string, optional) - comma-separated
- `projectId` (int, optional)

**Response (201 Created)**
```json
{
  "documentId": 123,
  "title": "Example",
  "category": "Project Documents",
  "uploadDateUtc": "2026-03-16T12:34:56Z",
  "fileSizeBytes": 12345,
  "mimeType": "application/pdf"
}
```

## Endpoint: List My Documents

**GET** `/api/documents/mine`

**Requires:** `Employee` role

**Query Parameters (optional)**:
- `category` (string)
- `projectId` (int)
- `search` (string) - matches title/description/tags
- `sortBy` (string) - `title`, `uploadDate`, `category`, `size`
- `sortDir` (string) - `asc` or `desc`
- `page` (int)
- `pageSize` (int)

**Response**
```json
{
  "items": [
    {
      "documentId": 123,
      "title": "Example",
      "category": "Project Documents",
      "uploadDateUtc": "2026-03-16T12:34:56Z",
      "fileSizeBytes": 12345,
      "projectId": 42,
      "uploaderId": 7,
      "mimeType": "application/pdf"
    }
  ],
  "total": 42
}
```

## Endpoint: Get Document Details

**GET** `/api/documents/{documentId}`

**Requires:** Authorized access to the document (owner/team/project/admin)

**Response**
```json
{
  "documentId": 123,
  "title": "Example",
  "description": "...",
  "category": "Project Documents",
  "tags": ["budget","2026"],
  "uploadDateUtc": "2026-03-16T12:34:56Z",
  "fileSizeBytes": 12345,
  "mimeType": "application/pdf",
  "projectId": 42,
  "uploaderId": 7,
  "sharedWith": [
    { "userId": 12 },
    { "teamId": 3 }
  ]
}
```

## Endpoint: Download Document

**GET** `/api/documents/{documentId}/download`

**Requires:** Authorized access to the document.

**Response**
- Binary stream with appropriate `Content-Type` and `Content-Disposition` headers.

## Endpoint: Preview Document

**GET** `/api/documents/{documentId}/preview`

**Requires:** Authorized access to the document.

**Response**
- Binary stream (PDF/image) with `Content-Type` set to the document's MIME type.

## Endpoint: Update Document Metadata

**PATCH** `/api/documents/{documentId}`

**Requires:** Document owner (or authorized role per policy)

**Request**
```json
{
  "title": "Updated Title",
  "description": "Updated",
  "category": "Reports",
  "tags": ["finance"],
  "projectId": 42
}
```

**Response**
- `204 No Content` on success.

## Endpoint: Replace Document File

**POST** `/api/documents/{documentId}/replace`

**Requires:** Document owner (or authorized role per policy)

**Request (multipart/form-data)**
- `file` (file, required)

**Response**
- `204 No Content` on success.

## Endpoint: Delete Document

**DELETE** `/api/documents/{documentId}`

**Requires:** Document owner, Team Lead (team scope), Project Manager (project scope), or Administrator.

**Response**
- `204 No Content` on success.

## Endpoint: Share Document

**POST** `/api/documents/{documentId}/share`

**Requires:** Document owner (or authorized role per policy)

**Request**
```json
{
  "sharedWithUserId": 12,
  "sharedWithTeamId": null,
  "expiresAtUtc": "2026-04-16T12:00:00Z"
}
```

**Response**
- `204 No Content` on success.

## Endpoint: List Shared With Me

**GET** `/api/documents/shared-with-me`

**Requires:** `Employee` role

**Response**
```json
{
  "items": [
    {
      "documentId": 123,
      "title": "Example",
      "sharedByUserId": 7,
      "sharedAtUtc": "2026-03-16T12:34:56Z"
    }
  ],
  "total": 12
}
```
