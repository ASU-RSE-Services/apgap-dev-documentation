# API Endpoints Reference

This document provides a comprehensive reference of all available API endpoints in the ASU APGAP backend.

**Base URL:** `/api/`

**Authentication:** All endpoints require authentication unless otherwise noted. Use JWT tokens obtained from the authentication endpoints.

---

## Authentication

**Base Path:** `/authentication/`

**POST `/authentication/login/`**

Login with email/password credentials.

**Request:**

```json
{
  "email": "user@example.com",
  "password": "your-password"
}
```

**Response:**

```json
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1...",
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1...",
  "user": {
    "pk": 1,
    "email": "user@example.com"
  }
}
```

**POST `/authentication/google/login/`**

Login with Google OAuth code.

**Request:**

```json
{
  "code": "google-oauth-authorization-code"
}
```

**POST `/authentication/token/refresh/`**

Refresh access token.

**Request:**

```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1..."
}
```

**Response:**

```json
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1..."
}
```

**POST `/authentication/token/verify/`**

Verify token validity.

**Request:**

```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1..."
}
```

---

## Users

**Base Path:** `/api/users/`

**GET `/api/users/`**

List all users.

**Response:**

```json
[
  {
    "id": 1,
    "email": "user@example.com",
    "name": "John Doe",
    "first_name": "John",
    "last_name": "Doe",
    "is_platform_admin": false,
    "is_data_analyst": false,
    "is_active": true,
    "organization": {
      "id": 1,
      "display_name": "Arizona State University"
    }
  }
]
```

**POST `/api/users/`**

Create a new user.

**Request:**

```json
{
  "email": "newuser@example.com",
  "first_name": "Jane",
  "last_name": "Smith",
  "organization": 1,
  "is_platform_admin": false,
  "is_data_analyst": false,
  "lab": 1,
  "is_lab_admin": false,
  "project": 1
}
```

**GET `/api/users/{id}/`**

Retrieve user details with permissions and memberships.

**Response:**

```json
{
  "id": 1,
  "email": "user@example.com",
  "name": "John Doe",
  "first_name": "John",
  "last_name": "Doe",
  "is_platform_admin": false,
  "is_data_analyst": false,
  "is_active": true,
  "is_azdhs_user": false,
  "organization": {
    "id": 1,
    "display_name": "Arizona State University"
  },
  "permissions": {
    "groups": ["Lab Director"],
    "permissions": ["labs.view_lab", "labs.change_lab"]
  },
  "lab_memberships": [
    {
      "id": 1,
      "display_name": "Genomics Lab",
      "role": "Lab Director",
      "projects": [
        {
          "id": 1,
          "display_name": "COVID-19 Sequencing",
          "role": "Bioinformatics User"
        }
      ]
    }
  ],
  "project_memberships": [
    {
      "id": 1,
      "display_name": "COVID-19 Sequencing",
      "role": "Bioinformatics User"
    }
  ]
}
```

**PATCH `/api/users/{id}/`**

Update user.

**Request:**

```json
{
  "first_name": "John",
  "last_name": "Smith",
  "organization": 2,
  "is_platform_admin": true
}
```

**DELETE `/api/users/{id}/`**

Soft delete (deactivate) user.

**GET `/api/users/me/`**

Get current authenticated user's details.

**POST `/api/users/{id}/reactivate/`**

Reactivate a deactivated user.

---

## Organizations

**Base Path:** `/api/organizations/`

**GET `/api/organizations/`**

List all organizations.

**Response:**

```json
[
  {
    "id": 1,
    "display_name": "Arizona State University",
    "active": true,
    "default_approve_analytical_dataset_requests": false
  }
]
```

**POST `/api/organizations/`**

Create a new organization.

**Request:**

```json
{
  "display_name": "New Organization",
  "default_approve_analytical_dataset_requests": false
}
```

**GET `/api/organizations/{id}/`**

Retrieve organization details.

**PATCH `/api/organizations/{id}/`**

Update organization settings.

**Request:**

```json
{
  "default_approve_analytical_dataset_requests": true
}
```

**DELETE `/api/organizations/{id}/`**

Soft delete organization (sets active=false). Fails if organization has users.

---

## Labs

**Base Path:** `/api/labs/`

**GET `/api/labs/`**

List labs accessible to user.

**Response:**

```json
[
  {
    "id": 1,
    "display_name": "Genomics Lab",
    "description": "Main genomics research lab",
    "project_id": "gcp-project-123",
    "organization": {
      "id": 1,
      "display_name": "Arizona State University"
    },
    "projects": [
      {
        "id": 1,
        "display_name": "COVID-19 Sequencing",
        "status": "ACTIVE"
      }
    ]
  }
]
```

**POST `/api/labs/`**

Create a new lab.

**Request:**

```json
{
  "display_name": "New Lab",
  "description": "Lab description",
  "organization": 1,
  "lab_director_id": 1
}
```

**GET `/api/labs/{id}/members/`**

Get all lab members.

**Response:**

```json
{
  "lab_users": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "role": "Lab Director",
      "is_lab_admin": true,
      "organization": { "id": 1, "display_name": "ASU" }
    }
  ],
  "project_users": [
    {
      "id": 1,
      "user_id": 2,
      "user_name": "Jane Smith",
      "user_email": "jane@example.com",
      "project_id": 1,
      "project_name": "COVID-19 Sequencing",
      "permission_id": 3,
      "permission_name": "Bioinformatics User"
    }
  ]
}
```

**POST `/api/labs/{id}/assign-user/`**

Assign a user to the lab.

**Request:**

```json
{
  "user_id": 2,
  "is_lab_director": false
}
```

**POST `/api/labs/{id}/remove-user/`**

Remove a user from the lab.

**Request:**

```json
{
  "user_id": 2
}
```

---

## Projects

**Base Path:** `/api/projects/`

**GET `/api/projects/`**

List projects accessible to user.

**Query Parameters:**

- `lab` - Filter by lab ID
- `status` - Filter by status (ACTIVE, ARCHIVED)

**Response:**

```json
[
  {
    "id": 1,
    "display_name": "COVID-19 Sequencing",
    "lab": 1,
    "program": null,
    "active": true,
    "status": "ACTIVE",
    "description": "Project description",
    "gcs_bucket": "project-bucket-123",
    "output_bucket": "gs://output-bucket",
    "opt_out_of_seqera": false,
    "build_status": "SUCCESS"
  }
]
```

**POST `/api/projects/`**

Create a new project.

**Request:**

```json
{
  "display_name": "New Project",
  "lab": 1,
  "description": "Project description",
  "opt_out_of_seqera": false
}
```

**GET `/api/projects/{id}/resource-detail/`**

Get detailed project information.

**Response:**

```json
{
  "id": 1,
  "display_name": "COVID-19 Sequencing",
  "description": "Project description",
  "opt_out_of_seqera": false,
  "seqera_launchpad_url": "https://tower.nf/...",
  "vertex_notebook_uri": "https://notebooks.cloud.google.com/...",
  "output_bucket": "gs://output-bucket",
  "project_id": "gcp-project-id",
  "project_users": [...],
  "lab_directors": [...]
}
```

**POST `/api/projects/{id}/archive/`**

Archive a project (Lab Director only).

**POST `/api/projects/{id}/hard-delete/`**

Permanently delete a project (Platform Admin only).

**Request:**

```json
{
  "deletion_justification": "Reason for deletion (minimum 10 characters)"
}
```

**Project Users**

**Base Path:** `/api/project-users/`

**POST `/api/project-users/`**

Add user to project.

**Request:**

```json
{
  "user": 2,
  "project": 1,
  "permission_group": 3
}
```

**Response:**

```json
{
  "id": 1,
  "user": {
    "id": 2,
    "email": "jane@example.com",
    "name": "Jane Smith",
    "organization": "ASU"
  },
  "project": {
    "id": 1,
    "display_name": "COVID-19 Sequencing"
  },
  "permission_group": {
    "id": 3,
    "name": "Bioinformatics User"
  }
}
```

**PATCH `/api/project-users/{id}/`**

Update project user role.

**Request:**

```json
{
  "permission_group": 4
}
```

---

## Files

**Base Path:** `/api/files/`

**GET `/api/files/`**

List files with optional pagination.

**Query Parameters:**

- `lab` - Filter by lab ID
- `status` - Filter by status (DRAFT, PROCESSING, PRIMARY, ARCHIVED)
- `created_by` - Filter by creator ID
- `pagination` - Enable pagination (`true`)
- `page`, `page_size` - Pagination controls
- `search` - Search in gcs_file_path

**Response:**

```json
[
  {
    "id": 1,
    "lab": 1,
    "lab_name": "Genomics Lab",
    "gcs_file_path": "lab-1/sample.fastq",
    "file_type": "fastq",
    "file_size": 1048576,
    "status": "PRIMARY",
    "description": null,
    "created_by": 1,
    "created_by_name": "John Doe",
    "created_at": "2024-01-15T10:30:00Z",
    "source_type": 1,
    "source_type_name": "Illumina",
    "batch_upload_id": null,
    "batch_upload_name": null,
    "copied_to_datasets": []
  }
]
```

**POST `/api/files/`**

Create file record or upload file.

**Request (upload with file):**

```
Content-Type: multipart/form-data

file: (binary file data)
lab: 1
```

**Request (metadata only):**

```json
{
  "lab": 1,
  "gcs_file_path": "path/to/file.txt",
  "file_type": "txt",
  "file_size": 1024
}
```

**GET `/api/files/{id}/`**

Retrieve file details.

**Response:**

```json
{
  "id": 1,
  "lab": { "id": 1, "display_name": "Genomics Lab", ... },
  "gcs_file_path": "lab-1/sample.fastq",
  "gcs_url": "gs://bucket/lab-1/sample.fastq",
  "file_type": "fastq",
  "file_size": 1048576,
  "status": "PRIMARY",
  "description": null,
  "created_by": 1,
  "created_by_email": "john@example.com",
  "created_by_name": "John Doe",
  "meets_retention_requirement": true,
  "metadata_tags": [
    { "key": "sample_id", "value": "S001", "data_type": "TEXT" }
  ],
  "copied_to_datasets": []
}
```

**DELETE `/api/files/{id}/`**

Delete file (requires justification).

**Request:**

```json
{
  "justification": "Reason for deletion (minimum 10 characters)"
}
```

**POST `/api/files/{id}/associate-metadata/`**

Set metadata tags for file (replaces existing).

**Request:**

```json
[
  { "key": "sample_id", "value": "S001", "source_type": 1 },
  { "key": "platforms", "value": ["illumina", "pacbio"], "source_type": 1 }
]
```

**Response:**

```json
{
  "file_id": 1,
  "metadata_set": [
    {
      "key": "sample_id",
      "value": "S001",
      "key_created": false,
      "value_created": false
    }
  ],
  "added": 2,
  "removed": 1,
  "kept": 0,
  "total": 2
}
```

**GET `/api/files/{id}/metadata-tags/`**

Get all metadata tags for file.

**Response:**

```json
{
  "file_id": 1,
  "metadata_tags": [
    {
      "key": "sample_id",
      "value": "S001",
      "data_type": "TEXT",
      "source_type": 1,
      "source_type_name": "Illumina",
      "is_required": true,
      "sort_order": 1
    }
  ],
  "count": 1
}
```

**POST `/api/files/{id}/set-primary/`**

Promote file to PRIMARY status.

---

## Uploads

**Base Path:** `/api/uploads/`

**GUI Uploads**

**POST `/api/uploads/`**

Create GUI upload (returns signed URL).

**Request:**

```json
{
  "filename": "sample.fastq",
  "content_type": "application/octet-stream",
  "lab": 1,
  "time_to_live": 3600,
  "file_size": 1048576
}
```

**Response:**

```json
{
  "id": 1,
  "filename": "sample.fastq",
  "content_type": "application/octet-stream",
  "lab": 1,
  "time_to_live": 3600,
  "file_size": 1048576,
  "upload_url": "https://storage.googleapis.com/...",
  "ingest_uuid": "550e8400-e29b-41d4-a716-446655440000",
  "created_at": "2024-01-15T10:30:00Z",
  "file": 1,
  "status": "PENDING"
}
```

**Batch Uploads**

**GET `/api/uploads/batch-uploads/`**

List batch uploads.

**Response:**

```json
[
  {
    "id": 1,
    "user": "john@example.com",
    "lab": "Genomics Lab",
    "ingest_bucket": "batch-upload-lab1-20240115-abc123",
    "kind": "FASTQ",
    "time_to_live": 86400,
    "description": "Batch upload for sequencing data",
    "created_at": "2024-01-15T10:30:00Z",
    "expires_at": "2024-01-16T10:30:00Z",
    "is_expired": false,
    "file_count": 5
  }
]
```

**POST `/api/uploads/batch-uploads/`**

Create batch upload.

**Request:**

```json
{
  "lab": 1,
  "kind": "FASTQ",
  "time_to_live": 86400,
  "description": "Batch upload for sequencing data"
}
```

**Response:**

```json
{
  "id": 1,
  "lab": 1,
  "ingest_bucket": "batch-upload-lab1-20240115-abc123",
  "kind": "FASTQ",
  "time_to_live": 86400,
  "description": "Batch upload for sequencing data"
}
```

**GET `/api/uploads/batch-uploads/{id}/download_service_account_key/`**

Download service account keyfile.

**Query Parameters:**

- `as_file` - Return as downloadable file (`true`) or JSON response

---

## Datasets (Data Catalog)

**Base Path:** `/api/datasets/`

**GET `/api/datasets/`**

List searchable datasets.

**Query Parameters:**

- `type` - Filter by type (GUI, BATCH)
- `owning_project` - Filter by owning project ID
- `assigned_projects` - Filter by assigned project IDs
- `metadata_tags` - Filter by metadata tag IDs
- `metadata_tag_match_partial` - `true` for OR, `false` for AND (default)
- `search` - Search in display_name
- `pagination`, `page`, `page_size` - Pagination controls

**Response:**

```json
[
  {
    "id": 1,
    "display_name": "COVID Samples Dataset",
    "description": "Dataset description",
    "type": "GUI",
    "searchable": true,
    "owning_project": { "id": 1, "display_name": "COVID-19 Sequencing" },
    "assigned_projects": [{ "id": 2, "display_name": "Analysis Project" }],
    "file_count": 50,
    "total_file_size": 52428800,
    "metadata_tags": [
      {
        "id": 1,
        "metadata_tag": { "id": 1, "key": "organism", "value": "SARS-CoV-2" }
      }
    ]
  }
]
```

**POST `/api/datasets/`**

Create a new dataset.

**Request:**

```json
{
  "owning_project": 1,
  "display_name": "New Dataset",
  "description": "Dataset description",
  "type": "GUI",
  "searchable": true,
  "metadata_tags": [1, 2, 3]
}
```

**POST `/api/datasets/{id}/request-access/`**

Request access to dataset for a project.

**Request:**

```json
{
  "project_id": 2,
  "message": "We need access for our analysis project"
}
```

---

## Analytical Datasets

**Base Path:** `/api/analytical-datasets/`

**GET `/api/analytical-datasets/`**

List analytical datasets.

**Response:**

```json
[
  {
    "id": 1,
    "name": "Analysis Dataset 1",
    "date": "2024-01-15T10:30:00Z",
    "status": "PENDING",
    "gcs_bucket": "analytical-dataset-bucket",
    "project": 1,
    "lab_name": "Genomics Lab",
    "project_name": "COVID-19 Sequencing",
    "created_by": 1,
    "created_by_email": "john@example.com",
    "created_by_name": "John Doe",
    "file_count": 10
  }
]
```

**POST `/api/analytical-datasets/`**

Create analytical dataset.

**Request:**

```json
{
  "name": "New Analysis Dataset",
  "project": 1,
  "description": "Dataset for analysis"
}
```

**GET `/api/analytical-datasets/{id}/`**

Retrieve analytical dataset with files.

**Response:**

```json
{
  "id": 1,
  "name": "Analysis Dataset 1",
  "description": "Dataset description",
  "gcs_bucket": "analytical-dataset-bucket",
  "lab": 1,
  "lab_name": "Genomics Lab",
  "project": 1,
  "project_name": "COVID-19 Sequencing",
  "email": "john@example.com",
  "created_by": "John Doe",
  "created_by_name": "John Doe",
  "file_count": 10,
  "files": [
    {
      "id": 1,
      "gcs_file_path": "file.fastq",
      "file_type": "fastq",
      "file_size": 1048576,
      "status": "PRIMARY",
      "original_file_id": 5,
      "original_file_path": "lab-1/original.fastq",
      "lab_name": "Genomics Lab",
      "metadata_tags": [...]
    }
  ],
  "access_requests": [
    {
      "id": 1,
      "name": "sample.fastq",
      "lab_name": "Other Lab",
      "status": "PENDING",
      "denied_by": null,
      "denial_reason": null
    }
  ]
}
```

**POST `/api/analytical-datasets/{id}/copy_file/`**

Copy file to dataset.

**Request:**

```json
{
  "source_file_id": 5
}
```

**Response (if auto-approved):**

```json
{
  "id": 10,
  "gcs_file_path": "copied-file.fastq",
  "file_type": "fastq",
  "file_size": 1048576,
  "status": "PRIMARY",
  "original_file_id": 5,
  "metadata_tags": [...]
}
```

**Response (if pending approval):**

```json
{
  "message": "Access request has been created and is pending approval.",
  "access_request_id": 1,
  "status": "PENDING"
}
```

**POST `/api/analytical-datasets/{id}/approve/`**

Approve pending access requests (Lab Director).

**Response:**

```json
{
  "status": "approved",
  "approved_count": 3
}
```

**POST `/api/analytical-datasets/{id}/deny/`**

Deny pending access requests (Lab Director).

**Request:**

```json
{
  "denial_reason": "Data cannot be shared due to IRB restrictions"
}
```

**Response:**

```json
{
  "status": "denied",
  "denied_count": 3
}
```

**POST `/api/analytical-datasets/{id}/request_access_to_files/`**

Request access to multiple files.

**Request:**

```json
{
  "files": [1, 2, 3, 4, 5]
}
```

**Response:**

```json
{
  "created": [{ "id": 1, "file_id": 1, "status": "PENDING" }],
  "existing": [{ "id": 2, "file_id": 2, "status": "APPROVED" }]
}
```

---

## Access Requests

**Base Path:** `/api/access-requests/`

**GET `/api/access-requests/`**

List access requests.

**Response:**

```json
[
  {
    "id": 1,
    "user": 2,
    "user_email": "jane@example.com",
    "user_name": "Jane Smith",
    "analytical_dataset": 1,
    "analytical_dataset_name": "Analysis Dataset 1",
    "file": 5,
    "file_name": "sample.fastq",
    "status": "PENDING",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
  }
]
```

**POST `/api/access-requests/`**

Create access request.

**Request:**

```json
{
  "user": 2,
  "analytical_dataset": 1,
  "file": 5
}
```

**POST `/api/access-requests/{id}/approve/`**

Approve access request.

**POST `/api/access-requests/{id}/reject/`**

Reject access request.

**GET `/api/access-requests/my-requests/`**

Get current user's access requests.

**GET `/api/access-requests/pending-approvals/`**

Get pending requests for approval.

---

## Notifications

**Base Path:** `/api/notifications/`

**GET `/api/notifications/`**

List notifications for current user.

**Response:**

```json
{
  "count": 10,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "title": "Access Request Approved",
      "details": "Your access request has been approved",
      "status": "info",
      "notification": "ACCESS_REQUEST_APPROVED",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

**GET `/api/notifications/preferences/`**

Get notification preferences.

**Response:**

```json
[
  {
    "notification": "ACCESS_REQUEST",
    "email_enabled": true
  }
]
```

**PUT `/api/notifications/preferences/`**

Update notification preferences (batch).

**Request:**

```json
[
  {
    "notification": "ACCESS_REQUEST",
    "email_enabled": false
  }
]
```

---

## Metadata Tags

**Base Path:** `/api/metadata-tags/`

**GET `/api/metadata-tags/`**

List metadata tags.

**Query Parameters:**

- `key` - Filter by key name
- `value` - Filter by value name

**Response:**

```json
[
  {
    "id": 1,
    "key": { "id": 1, "name": "organism", "data_type": "TEXT" },
    "value": { "id": 1, "name": "SARS-CoV-2" }
  }
]
```

---

## Metadata Requirements

**Base Path:** `/api/metadata-requirements/`

**Source Types**

**GET `/api/metadata-requirements/source-types/`**

List source types.

**Response:**

```json
[
  {
    "id": 1,
    "name": "Illumina",
    "sort_order": 1
  }
]
```

**POST `/api/metadata-requirements/source-types/`**

Create source type.

**Request:**

```json
{
  "name": "Oxford Nanopore",
  "sort_order": 2
}
```

**Templates**

**GET `/api/metadata-requirements/templates/`**

List metadata templates.

**Query Parameters:**

- `source_type` - Filter by source type ID
- `is_required` - Filter by required status
- `is_core` - Filter by core status

**Response:**

```json
[
  {
    "id": 1,
    "key": { "id": 1, "name": "sample_id", "data_type": "TEXT" },
    "source_type": { "id": 1, "name": "Illumina" },
    "is_required": true,
    "is_core": true,
    "sort_order": 1,
    "template_options": [{ "id": 1, "value": { "id": 1, "name": "option1" } }]
  }
]
```

**POST `/api/metadata-requirements/templates/`**

Create template.

**Request:**

```json
{
  "key": "new_field",
  "source_type": 1,
  "is_required": false,
  "is_core": false,
  "sort_order": 10,
  "data_type": "SELECT",
  "options": ["option1", "option2"]
}
```

**GET `/api/metadata-requirements/templates/download_csv/`**

Download CSV template.

**Query Parameters:**

- `source_type` - Source type ID
- `include_info_row` - Include requirement info (`true`/`false`)

**POST `/api/metadata-requirements/templates/upload_csv/`**

Upload CSV with metadata.

**Request:**

```
Content-Type: multipart/form-data

csv_file: (CSV file)
lab: 1
source_type: 1
validate_only: false
```

**Response:**

```json
{
  "success": true,
  "success_count": 10,
  "error_count": 0,
  "processed_files": ["file1.fastq", "file2.fastq"],
  "message": "Successfully processed 10 files",
  "errors": [],
  "warnings": []
}
```

---

## Domain Whitelist

**Base Path:** `/api/domain-whitelist/`

**GET `/api/domain-whitelist/`**

List whitelisted domains.

**Response:**

```json
[
  {
    "id": 1,
    "domain": "asu.edu",
    "description": "Arizona State University",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
]
```

**POST `/api/domain-whitelist/`**

Add domain to whitelist.

**Request:**

```json
{
  "domain": "example.edu",
  "description": "Example University"
}
```

---

## Deletions

**Base Path:** `/api/deletions/`

**Archive Requests**

**GET `/api/deletions/archive-requests/`**

List archive requests.

**Response:**

```json
[
  {
    "id": 1,
    "status": "PENDING",
    "justification": "Data no longer needed",
    "external_storage_location": "https://archive.example.com/file123",
    "retention_requirement_met": true,
    "requested_by": { "id": 1, "email": "john@example.com" },
    "reviewed_by": null,
    "reviewed_at": null,
    "created_at": "2024-01-15T10:30:00Z"
  }
]
```

**POST `/api/deletions/archive-requests/`**

Create archive request.

**Request:**

```json
{
  "content_type": 15,
  "object_id": 1,
  "justification": "Data no longer needed for research",
  "external_storage_location": "https://archive.example.com/file123"
}
```

**POST `/api/deletions/archive-requests/{id}/approve/`**

Approve archive request.

**POST `/api/deletions/archive-requests/{id}/deny/`**

Deny archive request.

**Request:**

```json
{
  "denial_reason": "Retention period not met"
}
```

---

## API Documentation

| Method | Endpoint       | Description                |
| ------ | -------------- | -------------------------- |
| GET    | `/api/schema/` | OpenAPI schema (JSON/YAML) |
| GET    | `/api/docs/`   | Swagger UI documentation   |

---

## Common Response Codes

| Code | Description                      |
| ---- | -------------------------------- |
| 200  | Success                          |
| 201  | Created                          |
| 204  | No Content (successful delete)   |
| 207  | Multi-Status (partial success)   |
| 400  | Bad Request (validation error)   |
| 401  | Unauthorized (not authenticated) |
| 403  | Forbidden (no permission)        |
| 404  | Not Found                        |
| 500  | Internal Server Error            |

## Pagination

Many list endpoints support optional pagination:

- Add `?pagination=true` to enable pagination
- Use `?page=N` to specify page number
- Use `?page_size=N` to specify items per page (max 100)

When pagination is enabled, responses include:

```json
{
  "count": 100,
  "next": "http://api/endpoint/?pagination=true&page=2",
  "previous": null,
  "results": [...]
}
```

## Filtering and Search

Most list endpoints support:

- **Filtering:** Use query parameters matching field names (e.g., `?status=PENDING`)
- **Search:** Use `?search=term` to search across multiple fields
- **Ordering:** Use `?ordering=field` or `?ordering=-field` for descending

## Error Responses

Error responses follow this format:

```json
{
  "error": "Error message",
  "detail": "Detailed description or field-specific errors"
}
```

Field validation errors:

```json
{
  "field_name": ["Error message for this field"]
}
```
