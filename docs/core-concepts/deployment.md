
# Deployment

### Cloud Build

- Google Cloud Build pipelines handle application deployment
- The development environment is deployed to the `development` namespace in GKE
- The deployment process:
  1. Builds a container image
  2. Uses Helm to apply changes to Kubernetes manifests
  3. Deploys the updated application code

### Helm Chart

- The application uses Helm for Kubernetes deployments
- Helm is used for the Development deployment of the application.
- Helm charts define the application's GKE resources.
- Configuration files are located in the `helm` directory
- More details are in the README.md in the `helm` directory.

### Container Startup Process

The application uses an entrypoint script and start script to initialize the container.

#### Entrypoint Script (`compose/production/django/entrypoint`)

The entrypoint script runs first and handles database connectivity:

1. Sets PostgreSQL connection defaults
2. Constructs the `DATABASE_URL` environment variable
3. Waits for PostgreSQL to become available using `wait-for-it`
4. Passes control to the start script

#### Start Script

**Production** (`compose/production/django/start`):

1. `collectstatic` - Collects static files for serving
2. `migrate` - Applies database migrations
3. `ensure_adhs_organization` - Creates the default ADHS organization if it doesn't exist
4. `setup_permission_groups` - Creates/updates permission groups and their associated permissions
5. Starts Gunicorn WSGI server on port 8000

**Local Development** (`compose/local/django/start`):

1. `migrate` - Applies database migrations
2. `ensure_adhs_organization` - Creates the default ADHS organization
3. `setup_permission_groups` - Creates/updates permission groups
4. Starts Django development server with `runserver_plus` on port 8000

#### Key Management Commands

| Command                    | Description                                                                                                 |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `ensure_adhs_organization` | Creates the default ADHS organization required for the platform                                             |
| `setup_permission_groups`  | Creates all permission groups (Platform Admin, Lab Director, etc.) with their associated Django permissions |
| `seed_metadata_templates`  | Seeds/updates metadata templates, keys, values, and source types for file metadata                          |

### Metadata Template Seeding

The application uses a configurable metadata template system for tagging files and datasets with structured metadata. The `seed_metadata_templates` management command populates and updates the database with predefined metadata schemas.

#### Running the Command

```bash
docker compose -f docker-compose.local.yml run django python manage.py seed_metadata_templates
```

#### Data Model Overview

The metadata system consists of several interconnected models:

| Model                    | Location                                    | Purpose                                                                      |
| ------------------------ | ------------------------------------------- | ---------------------------------------------------------------------------- |
| `Key`                    | `asu_apgap/metadatatags/models.py`          | Predefined metadata field names with data types (TEXT, NUMBER, DATE, SELECT) |
| `Value`                  | `asu_apgap/metadatatags/models.py`          | Predefined values that can be assigned to keys (e.g., dropdown options)      |
| `SourceType`             | `asu_apgap/metadata_requirements/models.py` | Sample source categories (Human Host, Water Sample, etc.)                    |
| `MetadataTemplate`       | `asu_apgap/metadata_requirements/models.py` | Defines which keys apply to which source types, with requirement rules       |
| `MetadataTemplateOption` | `asu_apgap/metadata_requirements/models.py` | Links predefined values to templates for select/multi-select fields          |

#### How the Seeding Works

The management command (`asu_apgap/metadatatags/management/commands/seed_metadata_templates.py`) performs the following operations in a single atomic transaction:

1. **Source Types Creation**: Creates 12 predefined source types (Human Host, Companion Animal Host, Wildlife Host, Vectors, Livestock AG Animal Host, Air, Produce AG, Food Product, Surface, Soil Sample, Water Sample, Wastewater Sample)

2. **Keys & Values Collection**: Collects all unique metadata keys and values from two data structures:
   - `ALL_SEQUENCES`: Core metadata fields that apply to **all** source types (e.g., Sample ID, Pathogen name, Date Collected, Sequencing instrument)
   - `SOURCE_TYPE_METADATA`: Source-type-specific fields (e.g., "Biospecimen type" for Human Host, "Water source" for Water Sample)

3. **Bulk Key Creation**: Creates `Key` objects with normalized names (uppercase, trimmed) and appropriate data types mapped from field types:
   - `text`, `text_field`, `text_input` → `TEXT`
   - `select`, `multi_select` → `SELECT`
   - `date`, `time` → `DATE`
   - `number` → `NUMBER`

4. **Bulk Value Creation**: Creates `Value` objects for all predefined dropdown/select options

5. **Template Creation**: Creates `MetadataTemplate` records that define:
   - Which key applies to which source type (or `None` for core templates)
   - Whether the field is required
   - Whether multiple values can be selected
   - Display sort order
   - The UI field type (select, multi_select, text, etc.)

6. **Template Options Creation**: Links predefined `Value` objects to their corresponding `MetadataTemplate` records for select/multi-select fields

#### Data Structure Definition

The metadata schemas are defined as Python dictionaries in the management command file:

```python
# Core fields applied to ALL sequences (source_type=None)
ALL_SEQUENCES = {
    "keys": [
        {"name": "Sample ID", "required": True, "type": "text", "values": []},
        {"name": "Pathogen/organism name", "required": True, "type": "multi_select", "values": ["SARS-CoV-2", "Influenza", ...]},
        {"name": "Date Collected", "required": True, "type": "date", "values": []},
        # ... more fields
    ]
}

# Source-type specific metadata requirements
SOURCE_TYPE_METADATA = {
    "HUMAN_HOST": {
        "keys": [
            {"name": "Biospecimen type", "required": True, "type": "select", "values": ["Nasopharyngeal swab", "Saliva", ...]},
            {"name": "Age (in years)", "required": False, "type": "number", "values": []},
            # ... more fields
        ]
    },
    "WATER_SAMPLE": {
        "keys": [
            {"name": "Water source", "required": True, "type": "select", "values": ["Municipal tap", "Irrigation line", ...]},
            {"name": "pH", "required": True, "type": "number", "values": []},
            # ... more fields
        ]
    },
    # ... more source types
}
```

#### Idempotent Behavior

The command is designed to be run multiple times safely:

- Existing records are updated if their properties have changed
- New records are created only if they don't exist
- Uses `ignore_conflicts=True` on bulk creates to handle race conditions
- All operations are wrapped in a database transaction

#### Modifying Metadata Templates

To add or modify metadata templates:

1. Edit the `ALL_SEQUENCES` dictionary for core fields that apply to all source types
2. Edit the `SOURCE_TYPE_METADATA` dictionary for source-type-specific fields
3. Run the management command to apply changes:
   ```bash
   docker compose -f docker-compose.local.yml run django python manage.py seed_metadata_templates
   ```

The command will output a summary of created/updated records upon completion.

