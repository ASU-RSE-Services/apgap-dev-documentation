# Database Entity Relationship Diagram

This document provides a comprehensive view of the ASU APGAP database schema.

## Visual Diagram

![Entity Relationship Diagram](/images/apgap_erd.png)

## Quick Reference

| Domain         | Models                                                                                                                                                    |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Organization   | `Organization`, `Lab`, `Project`, `Program`                                                                                                               |
| Users          | `User`, `LabUser`, `ProjectUser`                                                                                                                          |
| Files          | `File`, `Upload`, `BatchUpload`                                                                                                                           |
| Datasets       | `Dataset`, `DatasetFile`, `DatasetProjectAssignment`, `AnalyticalDataset`                                                                                 |
| Metadata       | `Key`, `Value`, `MetadataTag`, `FileMetadataTag`, `DatasetMetadataTag`, `SourceType`, `MetadataTemplate`, `MetadataTemplateOption`, `MetadataRequirement` |
| Access Control | `AccessRequest`, `AccessRequestApprover`                                                                                                                  |
| Audit          | `Deletion`, `ArchiveRequest`                                                                                                                              |
| Notifications  | `Notification`, `UserNotificationPreference`                                                                                                              |
| Other          | `DomainWhitelist`, `SeqeraWorkspace`                                                                                                                      |

---

## Model Details

### Organization

| Field                                       | Type    | Constraints | Description                   |
| ------------------------------------------- | ------- | ----------- | ----------------------------- |
| id                                          | int     | PK          | Primary key                   |
| display_name                                | string  | UNIQUE      | Organization name             |
| default_approve_analytical_dataset_requests | boolean |             | Auto-approve dataset requests |
| active                                      | boolean |             | Soft delete flag              |

### Lab

| Field           | Type     | Constraints       | Description                     |
| --------------- | -------- | ----------------- | ------------------------------- |
| id              | int      | PK                | Primary key                     |
| organization_id | int      | FK → Organization | Parent organization             |
| display_name    | string   |                   | Lab name                        |
| description     | text     |                   | Lab description                 |
| active          | boolean  |                   | Active status                   |
| gcs_bucket      | string   |                   | GCS bucket for lab files        |
| project_prefix  | string   | UNIQUE            | Unique prefix for GCP resources |
| build_status    | string   |                   | Cloud Build deployment status   |
| created_by_id   | int      | FK → User         | User who created the lab        |
| created_at      | datetime |                   | Creation timestamp              |
| gcp_project_id  | string   |                   | Cached GCP project ID           |

### LabUser

| Field               | Type    | Constraints | Description             |
| ------------------- | ------- | ----------- | ----------------------- |
| id                  | int     | PK          | Primary key             |
| lab_id              | int     | FK → Lab    | Lab reference           |
| user_id             | int     | FK → User   | User reference          |
| permission_group_id | int     | FK → Group  | Django permission group |
| is_lab_admin        | boolean |             | Lab director flag       |

**Unique Constraint**: (lab_id, user_id)

### Program

| Field        | Type   | Constraints | Description  |
| ------------ | ------ | ----------- | ------------ |
| id           | int    | PK          | Primary key  |
| display_name | string | UNIQUE      | Program name |

### Project

| Field                  | Type     | Constraints  | Description                 |
| ---------------------- | -------- | ------------ | --------------------------- |
| id                     | int      | PK           | Primary key                 |
| lab_id                 | int      | FK → Lab     | Parent lab                  |
| program_id             | int      | FK → Program | Optional program            |
| display_name           | string   |              | Project name                |
| active                 | boolean  |              | Active status               |
| status                 | string   |              | ACTIVE, ARCHIVED            |
| gcs_bucket             | string   |              | GCS bucket for project      |
| project_prefix         | string   | UNIQUE       | Unique prefix               |
| created_by_id          | int      | FK → User    | Creator                     |
| deleted_by_id          | int      | FK → User    | Who deleted                 |
| archived_by_id         | int      | FK → User    | Who archived                |
| archived_at            | datetime |              | Archive timestamp           |
| deleted_at             | datetime |              | Deletion timestamp          |
| deletion_justification | text     |              | Deletion reason             |
| description            | text     |              | Project description         |
| build_status           | string   |              | Cloud Build status          |
| opt_out_of_seqera      | boolean  |              | Skip Seqera workspace       |
| seqera_workspace_id    | string   |              | Seqera workspace ID         |
| seqera_compute_env_id  | string   |              | Seqera compute env ID       |
| seqera_credentials_id  | string   |              | Seqera credentials ID       |
| gcp_project_id         | string   |              | Cached GCP project ID       |
| seqera_output_bucket   | string   |              | Cached Seqera output bucket |
| vertex_notebook_uri    | string   |              | Cached Vertex notebook URI  |

### ProjectUser

| Field               | Type | Constraints  | Description       |
| ------------------- | ---- | ------------ | ----------------- |
| id                  | int  | PK           | Primary key       |
| project_id          | int  | FK → Project | Project reference |
| user_id             | int  | FK → User    | User reference    |
| permission_group_id | int  | FK → Group   | Permission group  |

**Unique Constraint**: (project_id, user_id)

### User

| Field             | Type    | Constraints       | Description         |
| ----------------- | ------- | ----------------- | ------------------- |
| id                | int     | PK                | Primary key         |
| name              | string  |                   | Full name           |
| email             | string  | UNIQUE            | Email address       |
| organization_id   | int     | FK → Organization | User's organization |
| is_platform_admin | boolean |                   | Platform admin flag |
| is_data_analyst   | boolean |                   | Data analyst flag   |
| is_active         | boolean |                   | Active status       |
| created_by_id     | int     | FK → User         | Who created         |
| deleted_by_id     | int     | FK → User         | Who deleted         |
| reactivated_by_id | int     | FK → User         | Who reactivated     |

### File

| Field                     | Type     | Constraints            | Description                                                                            |
| ------------------------- | -------- | ---------------------- | -------------------------------------------------------------------------------------- |
| id                        | int      | PK                     | Primary key                                                                            |
| lab_id                    | int      | FK → Lab               | Lab that owns the file                                                                 |
| gcs_file_path             | string   |                        | GCS path                                                                               |
| file_type                 | string   |                        | File extension/type                                                                    |
| file_size                 | bigint   |                        | Size in bytes                                                                          |
| status                    | string   |                        | DRAFT, PRIMARY, WAITING, PROCESSING, PII_DETECTED, UPLOADED, FAILED, ARCHIVED, DELETED |
| created_by_id             | int      | FK → User              | Uploader                                                                               |
| created_at                | datetime |                        | Upload timestamp                                                                       |
| updated_at                | datetime |                        | Last update                                                                            |
| source_type_id            | int      | FK → SourceType        | Sample source type                                                                     |
| original_file_id          | int      | FK → File              | Original if this is a copy                                                             |
| analytical_dataset_id     | int      | FK → AnalyticalDataset | If copied to dataset                                                                   |
| description               | text     |                        | File description                                                                       |
| external_storage_location | string   |                        | Archive location (e.g., NCBI SRA)                                                      |
| external_storage_type     | string   |                        | NCBI_SRA, OTHER                                                                        |
| archived_at               | datetime |                        | Archive timestamp                                                                      |
| deletion_reason           | text     |                        | Reason for deletion                                                                    |

### Upload

| Field           | Type     | Constraints      | Description                    |
| --------------- | -------- | ---------------- | ------------------------------ |
| id              | int      | PK               | Primary key                    |
| user_id         | int      | FK → User        | Uploader                       |
| lab_id          | int      | FK → Lab         | Target lab                     |
| file_id         | int      | FK → File        | Associated file                |
| filename        | string   |                  | Original filename              |
| content_type    | string   |                  | MIME type                      |
| gcs_filepath    | string   |                  | GCS path                       |
| ingest_uuid     | uuid     |                  | Unique ingest identifier       |
| time_to_live    | int      |                  | TTL in hours                   |
| created_at      | datetime |                  | Upload start time              |
| status          | string   |                  | CREATED, SUCCEEDED, FAILED     |
| file_size       | bigint   |                  | Size in bytes                  |
| batch_upload_id | int      | FK → BatchUpload | Parent batch (if batch upload) |

### BatchUpload

| Field                             | Type     | Constraints | Description             |
| --------------------------------- | -------- | ----------- | ----------------------- |
| id                                | int      | PK          | Primary key             |
| user_id                           | int      | FK → User   | Creator                 |
| lab_id                            | int      | FK → Lab    | Target lab              |
| batch_service_account_secret_name | string   |             | Secret Manager path     |
| batch_service_account_email       | string   |             | Service account email   |
| ingest_bucket                     | string   |             | Temporary ingest bucket |
| kind                              | string   |             | CLI, BASESPACE          |
| time_to_live                      | int      |             | TTL in hours            |
| description                       | text     |             | Batch description       |
| created_at                        | datetime |             | Creation time           |

### Dataset

| Field                             | Type    | Constraints  | Description              |
| --------------------------------- | ------- | ------------ | ------------------------ |
| id                                | int     | PK           | Primary key              |
| owning_project_id                 | int     | FK → Project | Owner project            |
| searchable                        | boolean |              | Visible in catalog       |
| type                              | string  |              | batch, single_gui_upload |
| created_by_id                     | int     | FK → User    | Creator                  |
| batch_ingest_bucket               | string  |              | Batch ingest bucket      |
| target_gcs_bucket                 | string  |              | Target GCS bucket        |
| display_name                      | string  |              | Dataset name             |
| description                       | text    |              | Dataset description      |
| batch_service_account_email       | string  |              | Batch SA email           |
| batch_service_account_secret_name | string  |              | Batch SA secret          |
| seqera_data_link_id               | string  |              | Seqera data link ID      |

### DatasetFile

| Field         | Type   | Constraints  | Description                                |
| ------------- | ------ | ------------ | ------------------------------------------ |
| id            | int    | PK           | Primary key                                |
| dataset_id    | int    | FK → Dataset | Parent dataset                             |
| gcs_file_path | string |              | GCS path                                   |
| file_type     | string |              | File type                                  |
| file_size     | bigint |              | Size in bytes                              |
| status        | string |              | PROCESSING, PII_DETECTED, UPLOADED, FAILED |
| created_by_id | int    | FK → User    | Uploader                                   |

### DatasetProjectAssignment

| Field               | Type     | Constraints  | Description                          |
| ------------------- | -------- | ------------ | ------------------------------------ |
| id                  | int      | PK           | Primary key                          |
| dataset_id          | int      | FK → Dataset | Dataset                              |
| project_id          | int      | FK → Project | Assigned project                     |
| seqera_data_link_id | string   |              | Seqera data link for this assignment |
| created_at          | datetime |              | Assignment time                      |
| updated_at          | datetime |              | Last update                          |

**Unique Constraint**: (dataset_id, project_id)

### AnalyticalDataset

| Field               | Type     | Constraints  | Description               |
| ------------------- | -------- | ------------ | ------------------------- |
| id                  | int      | PK           | Primary key               |
| name                | string   |              | Dataset name              |
| gcs_bucket          | string   |              | GCS bucket (auto-created) |
| project_id          | int      | FK → Project | Parent project            |
| created_by_id       | int      | FK → User    | Creator                   |
| created_at          | datetime |              | Creation time             |
| updated_at          | datetime |              | Last update               |
| approval_status     | string   |              | PENDING, APPROVED, DENIED |
| description         | text     |              | Description               |
| seqera_data_link_id | string   |              | Seqera data link ID       |

### SourceType

| Field      | Type   | Constraints | Description      |
| ---------- | ------ | ----------- | ---------------- |
| id         | int    | PK          | Primary key      |
| name       | string |             | Source type name |
| sort_order | int    |             | Display order    |

### Key

| Field     | Type   | Constraints | Description                |
| --------- | ------ | ----------- | -------------------------- |
| id        | int    | PK          | Primary key                |
| name      | string | UNIQUE      | Key name (uppercase)       |
| data_type | string |             | TEXT, NUMBER, DATE, SELECT |

### Value

| Field | Type   | Constraints | Description            |
| ----- | ------ | ----------- | ---------------------- |
| id    | int    | PK          | Primary key            |
| name  | string | UNIQUE      | Value name (uppercase) |

### MetadataTag

| Field    | Type | Constraints | Description    |
| -------- | ---- | ----------- | -------------- |
| id       | int  | PK          | Primary key    |
| key_id   | int  | FK → Key    | Metadata key   |
| value_id | int  | FK → Value  | Metadata value |

**Unique Constraint**: (key_id, value_id)

### FileMetadataTag

| Field           | Type | Constraints      | Description         |
| --------------- | ---- | ---------------- | ------------------- |
| id              | int  | PK               | Primary key         |
| file_id         | int  | FK → File        | Tagged file         |
| metadata_tag_id | int  | FK → MetadataTag | Applied tag         |
| source_type_id  | int  | FK → SourceType  | Source type context |

**Unique Constraint**: (file_id, metadata_tag_id, source_type_id)

### DatasetMetadataTag

| Field           | Type | Constraints      | Description    |
| --------------- | ---- | ---------------- | -------------- |
| id              | int  | PK               | Primary key    |
| dataset_id      | int  | FK → Dataset     | Tagged dataset |
| metadata_tag_id | int  | FK → MetadataTag | Applied tag    |

**Unique Constraint**: (dataset_id, metadata_tag_id)

### MetadataRequirement

| Field          | Type | Constraints     | Description                  |
| -------------- | ---- | --------------- | ---------------------------- |
| id             | int  | PK              | Primary key                  |
| key_id         | int  | FK → Key        | Required key                 |
| source_type_id | int  | FK → SourceType | For source type (null = all) |

**Unique Constraint**: (key_id, source_type_id)

### MetadataTemplate

| Field           | Type    | Constraints     | Description           |
| --------------- | ------- | --------------- | --------------------- |
| id              | int     | PK              | Primary key           |
| key_id          | int     | FK → Key        | Template key          |
| name            | string  |                 | Template name         |
| source_type_id  | int     | FK → SourceType | For source type       |
| is_required     | boolean |                 | Required field        |
| multiple_select | boolean |                 | Allow multiple values |
| is_core         | boolean |                 | Applies to all types  |
| sort_order      | int     |                 | Display order         |
| value_type      | string  |                 | Value type hint       |

**Unique Constraint**: (key_id, source_type_id)

### MetadataTemplateOption

| Field                | Type | Constraints           | Description     |
| -------------------- | ---- | --------------------- | --------------- |
| id                   | int  | PK                    | Primary key     |
| metadata_template_id | int  | FK → MetadataTemplate | Parent template |
| value_id             | int  | FK → Value            | Option value    |
| sort_order           | int  |                       | Display order   |

**Unique Constraint**: (metadata_template_id, value_id)

### AccessRequest

| Field                 | Type     | Constraints            | Description                 |
| --------------------- | -------- | ---------------------- | --------------------------- |
| id                    | int      | PK                     | Primary key                 |
| user_id               | int      | FK → User              | Requester                   |
| analytical_dataset_id | int      | FK → AnalyticalDataset | Target dataset              |
| file_id               | int      | FK → File              | Target file                 |
| status                | string   |                        | PENDING, APPROVED, REJECTED |
| denied_by_id          | int      | FK → User              | Who denied                  |
| denial_reason         | text     |                        | Denial reason               |
| created_at            | datetime |                        | Request time                |
| updated_at            | datetime |                        | Last update                 |

### AccessRequestApprover

| Field             | Type     | Constraints        | Description    |
| ----------------- | -------- | ------------------ | -------------- |
| id                | int      | PK                 | Primary key    |
| access_request_id | int      | FK → AccessRequest | Parent request |
| approver_id       | int      | FK → User          | Approver user  |
| created_at        | datetime |                    | Approval time  |
| updated_at        | datetime |                    | Last update    |

### Notification

| Field                     | Type     | Constraints      | Description                           |
| ------------------------- | -------- | ---------------- | ------------------------------------- |
| id                        | int      | PK               | Primary key                           |
| title                     | string   |                  | Notification title                    |
| status                    | string   |                  | info, warning, error, failed, success |
| created_at                | datetime |                  | Creation time                         |
| details                   | text     |                  | Notification body                     |
| content_type_id           | int      | FK → ContentType | Generic FK type                       |
| object_id                 | int      |                  | Generic FK ID                         |
| enable_email_notification | boolean  |                  | Send email                            |
| csv_attachment            | text     |                  | CSV content for email                 |
| notification              | string   |                  | Notification type enum                |

**M2M Relations**: email_recipients → User, inapp_recipients → User

### UserNotificationPreference

| Field         | Type     | Constraints | Description       |
| ------------- | -------- | ----------- | ----------------- |
| id            | int      | PK          | Primary key       |
| user_id       | int      | FK → User   | User              |
| notification  | string   |             | Notification type |
| email_enabled | boolean  |             | Email preference  |
| created_at    | datetime |             | Creation time     |
| updated_at    | datetime |             | Last update       |

**Unique Constraint**: (user_id, notification)

### Deletion

| Field                 | Type     | Constraints      | Description           |
| --------------------- | -------- | ---------------- | --------------------- |
| id                    | int      | PK               | Primary key           |
| content_type_id       | int      | FK → ContentType | Deleted object type   |
| object_id             | int      |                  | Deleted object ID     |
| object_representation | string   |                  | String representation |
| object_model_name     | string   |                  | Model name            |
| justification         | text     |                  | Deletion reason       |
| deleted_by_id         | int      | FK → User        | Who deleted           |
| deleted_at            | datetime |                  | Deletion time         |
| additional_data       | json     |                  | Extra metadata        |

### ArchiveRequest

| Field                     | Type     | Constraints      | Description               |
| ------------------------- | -------- | ---------------- | ------------------------- |
| id                        | int      | PK               | Primary key               |
| content_type_id           | int      | FK → ContentType | Object type               |
| object_id                 | int      |                  | Object ID                 |
| requested_by_id           | int      | FK → User        | Requester                 |
| status                    | string   |                  | PENDING, APPROVED, DENIED |
| justification             | text     |                  | Archive reason            |
| external_storage_location | string   |                  | External storage URL      |
| external_storage_type     | string   |                  | NCBI_SRA, OTHER           |
| retention_requirement_met | boolean  |                  | Met retention period      |
| reviewed_by_id            | int      | FK → User        | Reviewer                  |
| reviewed_at               | datetime |                  | Review time               |
| denial_reason             | text     |                  | Denial reason             |
| created_at                | datetime |                  | Request time              |
| updated_at                | datetime |                  | Last update               |

### DomainWhitelist

| Field       | Type     | Constraints | Description        |
| ----------- | -------- | ----------- | ------------------ |
| id          | int      | PK          | Primary key        |
| domain      | string   | UNIQUE      | Allowed domain     |
| description | string   |             | Domain description |
| created_at  | datetime |             | Creation time      |
| updated_at  | datetime |             | Last update        |

### SeqeraWorkspace

| Field      | Type   | Constraints  | Description     |
| ---------- | ------ | ------------ | --------------- |
| id         | int    | PK           | Primary key     |
| project_id | int    | FK → Project | Parent project  |
| name       | string |              | Workspace name  |
| owner      | string |              | Workspace owner |

---

## Relationship Summary

### One-to-Many Relationships

| Parent            | Child                  | Foreign Key                                 |
| ----------------- | ---------------------- | ------------------------------------------- |
| Organization      | Lab                    | lab.organization_id                         |
| Organization      | User                   | user.organization_id                        |
| Lab               | Project                | project.lab_id                              |
| Lab               | File                   | file.lab_id                                 |
| Lab               | LabUser                | labuser.lab_id                              |
| Lab               | Upload                 | upload.lab_id                               |
| Lab               | BatchUpload            | batchupload.lab_id                          |
| Program           | Project                | project.program_id                          |
| Project           | ProjectUser            | projectuser.project_id                      |
| Project           | AnalyticalDataset      | analyticaldataset.project_id                |
| Project           | Dataset                | dataset.owning_project_id                   |
| Project           | SeqeraWorkspace        | seqeraworkspace.project_id                  |
| User              | LabUser                | labuser.user_id                             |
| User              | ProjectUser            | projectuser.user_id                         |
| User              | Upload                 | upload.user_id                              |
| User              | BatchUpload            | batchupload.user_id                         |
| User              | File                   | file.created_by_id                          |
| User              | AccessRequest          | accessrequest.user_id                       |
| File              | Upload                 | upload.file_id                              |
| File              | File                   | file.original_file_id (copies)              |
| File              | FileMetadataTag        | filemetadatatag.file_id                     |
| File              | AccessRequest          | accessrequest.file_id                       |
| BatchUpload       | Upload                 | upload.batch_upload_id                      |
| Dataset           | DatasetFile            | datasetfile.dataset_id                      |
| AnalyticalDataset | File                   | file.analytical_dataset_id                  |
| AnalyticalDataset | AccessRequest          | accessrequest.analytical_dataset_id         |
| SourceType        | File                   | file.source_type_id                         |
| SourceType        | MetadataTemplate       | metadatatemplate.source_type_id             |
| Key               | MetadataTag            | metadatatag.key_id                          |
| Key               | MetadataTemplate       | metadatatemplate.key_id                     |
| Value             | MetadataTag            | metadatatag.value_id                        |
| MetadataTag       | FileMetadataTag        | filemetadatatag.metadata_tag_id             |
| MetadataTag       | DatasetMetadataTag     | datasetmetadatatag.metadata_tag_id          |
| MetadataTemplate  | MetadataTemplateOption | metadatatemplateoption.metadata_template_id |
| AccessRequest     | AccessRequestApprover  | accessrequestapprover.access_request_id     |

### Many-to-Many Relationships

| Model A       | Model B          | Through Table                 |
| ------------- | ---------------- | ----------------------------- |
| Dataset       | Project          | DatasetProjectAssignment      |
| MetadataTag   | Dataset          | DatasetMetadataTag            |
| MetadataTag   | File             | FileMetadataTag               |
| Notification  | User (email)     | notification_email_recipients |
| Notification  | User (inapp)     | notification_inapp_recipients |
| AccessRequest | User (approvers) | AccessRequestApprover         |
