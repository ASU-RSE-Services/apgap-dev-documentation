# Arizona State University Pathogen Genomics Analytics Platform Django Backend

## Local Development Environment

### Prerequisites

- Python 3.12
- Docker and Docker Compose
- Git
- Pre-commit


### Environment Secrets

The application requires several secret environment variables to function properly. These should be stored in:

```
.envs/.local/.secrets
```

Secret values can be found in the Secret Manager for the application deployment. Required values for local development include:

- `GOOGLE_OAUTH_CLIENT_ID`
- `GOOGLE_OAUTH_CLIENT_SECRET`
- `GOOGLE_OAUTH_REDIRECT_URL` - **Must be set to `postmessage` for local development** (see note below)
- `SENDGRID_API_KEY`
- `SEQERA_API_TOKEN`
- `SEQERA_ORGANIZATION_ID`

**Important:** The `GOOGLE_OAUTH_REDIRECT_URL` must be set to `postmessage` when running locally with a frontend that uses Google OAuth's popup-based authentication flow. This tells Google's OAuth to return the authorization code via JavaScript `postMessage` instead of redirecting to a URL.

Example `.envs/.local/.secrets` file:

```bash
GOOGLE_OAUTH_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_OAUTH_CLIENT_SECRET=your-client-secret
GOOGLE_OAUTH_REDIRECT_URL=postmessage
SENDGRID_API_KEY=your-sendgrid-key
SEQERA_API_TOKEN=your-seqera-token
SEQERA_ORGANIZATION_ID=your-seqera-org-id
```

Never commit these secrets to the repository. The `.envs/.local/.secrets` file is included in `.gitignore`.

Non-sensitive environment variables for local development are located in:

```
.envs/.local/
```

### Service Account Keyfile

The application uses a service account to interact with Google Cloud Services:

- Service account: `dev-app-backend@asu-ap-gap-test-2360.iam.gserviceaccount.com`
- This account is permissioned to interact with GCS and run Module deployments on project creation
- The keyfile should be placed at: `secrets/keyfile_DEV.json`
- This file is also included in `.gitignore` and should never be committed to the repository

### Domain Whitelist Setup

The application validates user email addresses against a whitelist of approved domains. **All users, including superusers, must have an email domain in the whitelist before they can be created.** You must add at least one domain to the whitelist before creating any users.

#### Option 1: Via Django Shell (Recommended for Initial Setup)

Since you need to whitelist a domain before creating a superuser, use the Django shell for initial setup:

```bash
docker compose -f docker-compose.local.yml run django python manage.py shell_plus
```

Then in the shell:

```python
from asu_apgap.domain_whitelist.models import DomainWhitelist

# Add your email domain (replace with your actual domain)
DomainWhitelist.objects.create(domain="asu.edu", description="Arizona State University")

# Or add multiple domains
domains_to_add = [
    ("asu.edu", "Arizona State University"),
    ("azdhs.gov", "Arizona Department of Health Services"),
    ("tgen.org", "Translational Genomics Research Institute"),
    ("gmail.com", "Gmail (for testing)"),
]
for domain, description in domains_to_add:
    DomainWhitelist.objects.get_or_create(domain=domain, defaults={"description": description})
```

Exit the shell with `exit()`, then proceed to create your superuser.

#### Option 2: Via Django Admin (After Superuser Exists)

Once you have a superuser:

1. Access the admin interface at **http://localhost:8000/admin**
2. Log in with your superuser credentials
3. Navigate to **Domain Whitelist** in the sidebar
4. Click **Add Domain Whitelist**
5. Enter the domain (e.g., `asu.edu`, `gmail.com`, `example.com`)
6. Optionally add a description
7. Click **Save**

#### Option 3: Via API (Requires Authentication)

```bash
curl -X POST http://localhost:8000/api/domain-whitelist/ \
  -H "Authorization: Bearer your-access-token" \
  -H "Content-Type: application/json" \
  -d '{"domain": "example.edu", "description": "Example University"}'
```
### GCP Interaction Settings

The application includes settings to control interactions with Google Cloud Platform:

```python
# From config/settings/local.py
ENABLE_CLOUD_BUILD_TRIGGER = False  # Controls whether  local actions can trigger Cloud Build pipelines
ENABLE_GCP_INTERACTIONS = False     # Controls whether the app will interact with GCP services
```

These settings are defaulted to `False` in local development to prevent accidental triggering of cloud resources. Only enable these if you specifically need to test GCP integrations locally.

### Making Authenticated API Requests

To make authenticated requests to the API during local development, you'll need to obtain an access token. Here's an example using Postman:

```javascript
// Postman pre-request script for authentication
const postRequest = {
  url: pm.environment.get("url") + "/authentication/login/",
  method: "POST",
  header: {
    "Content-Type": "application/json",
  },
  body: {
    mode: "raw",
    raw: JSON.stringify({
      email: pm.environment.get("email"),
      password: pm.environment.get("password"),
    }),
  },
};

pm.sendRequest(postRequest, function (err, response) {
  const responseJSON = response.json();
  pm.environment.set("access_token", responseJSON.access);
});
```

Then in your subsequent requests, use the access token in the Authorization header:

```
Authorization: Bearer {{access_token}}
```

For curl or other HTTP clients, first obtain the token:

```bash
curl -X POST http://localhost:8000/authentication/login/ \
  -H "Content-Type: application/json" \
  -d '{"email":"your-email@example.com","password":"your-password"}'
```

Then use the returned access token in subsequent requests:

```bash
curl -X GET http://localhost:8000/api/endpoint/ \
  -H "Authorization: Bearer your-access-token"
```

### Running Migrations

Ensure your database schema is up-to-date by running migrations:

```bash
docker compose -f docker-compose.local.yml run django python manage.py migrate
```

### Making Migrations

After modifying models, create new migration files with:

```bash
docker compose -f docker-compose.local.yml run django python manage.py makemigrations
```

### Building Containers

Build the Docker containers for local development:

```bash
docker compose -f docker-compose.local.yml build
```

### Running Containers

Start the application with:

```bash
docker compose -f docker-compose.local.yml up
```

The application will be available at **http://localhost:8000**.

### Running Interactive Shell

Access the Django shell for interactive debugging and development:

```bash
docker compose -f docker-compose.local.yml run django python manage.py shell_plus
```

## Administration

### Creating a Django Superuser

To create an administrator account with full access to the platform:

```bash
docker compose -f docker-compose.local.yml run django python manage.py createsuperuser
```

Follow the prompts to set email and password.

**Important:** You must add your email domain to the whitelist before creating a superuser. See [Domain Whitelist Setup](#domain-whitelist-setup).

### Using Administrative Panel

- Access the admin interface at **http://localhost:8000/admin**
- Log in with your superuser credentials
- Manage users, permissions, and application data
- **Key admin sections:**
  - **Users**: Manage user accounts and permissions
  - **Domain Whitelist**: Manage allowed email domains for user registration
  - **Organizations**: Manage organizations
  - **Labs**: Manage laboratories and lab memberships
  - **Projects**: Manage projects and project memberships

## User Management

### Permissioning System

The application uses a role-based access control (RBAC) model with the following predefined roles:

| Role                    | Scope         | Description                                                                                                                                        |
| ----------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **PLATFORM_ADMIN**      | User level    | Full system access and configuration privileges within their organization. Can access the data catalog. No access to datasets or Seqera resources. |
| **LAB_DIRECTOR**        | Lab level     | Manages laboratory projects, team members, and datasets. Can browse the data catalog.                                                              |
| **LAB_COLLABORATOR**    | Lab level     | Can view lab resources, upload files, and browse the data catalog.                                                                                 |
| **BIOINFORMATICS_USER** | Project level | Manages and accesses project-specific datasets and Seqera resources. Can browse the data catalog.                                                  |
| **DATA_ANALYST**        | User level    | Can browse the data catalog only.                                                                                                                  |

#### Special Permissions

- **AZDHS** users have elevated privileges that allow them to assign datasets directly to their own projects without requiring approval from the dataset owner.

These roles are automatically created when the Django container is initialized via the management command:

```bash
python manage.py setup_permission_groups
```

Role definitions can be found in `asu_apgap/utils/permissions.py` and the management command is located at `asu_apgap/users/management/commands/setup_permission_groups.py`.

## Database Structure

The application uses Django ORM with PostgreSQL. The database schema consists of 30+ models organized into logical domain areas.

![Entity Relationship Diagram](/images/apgap_erd.png)

For detailed field definitions, see [Database Entity Relationship Diagram](/core-concepts/database).

## API Endpoints

The application exposes a RESTful API with endpoints organized by resource type.

### Interactive Documentation

When the application is running locally, interactive API documentation is available at:

- **Swagger UI:** http://localhost:8000/api/docs/
- **OpenAPI Schema:** http://localhost:8000/api/schema/

To access these endpoints, ensure the local development containers are running:

```bash
docker compose -f docker-compose.local.yml up
```

### Sphinx Documentation

Additional project documentation is available via Sphinx. To build and view the docs:

1. **Build and run the docs container:**

```bash
docker compose -f docker-compose.docs.yml up --build
```

2. **Access the documentation at:** http://localhost:9000

The docs container will watch for changes and automatically rebuild when source files are modified.

### Main Endpoint Categories

| Category            | Base Path                                            | Description                                 |
| ------------------- | ---------------------------------------------------- | ------------------------------------------- |
| Authentication      | `/authentication/`                                   | Login, logout, OAuth, JWT tokens            |
| Users               | `/api/users/`                                        | User management and profiles                |
| Organizations       | `/api/organizations/`                                | Organization management                     |
| Labs                | `/api/labs/`                                         | Lab management and membership               |
| Projects            | `/api/projects/`                                     | Project management and archival             |
| Files               | `/api/files/`                                        | File management and metadata                |
| Uploads             | `/api/uploads/`                                      | GUI and batch file uploads                  |
| Datasets            | `/api/datasets/`                                     | Data catalog (searchable datasets)          |
| Analytical Datasets | `/api/analytical-datasets/`                          | Project-specific datasets with file copying |
| Access Requests     | `/api/access-requests/`                              | File access request workflow                |
| Notifications       | `/api/notifications/`                                | User notifications and preferences          |
| Metadata            | `/api/metadata-tags/`, `/api/metadata-requirements/` | Metadata tags and templates                 |
| Domain Whitelist    | `/api/domain-whitelist/`                             | Allowed email domains                       |
| Deletions           | `/api/deletions/`                                    | Archive requests and deletion audit         |

For complete endpoint documentation with request/response examples, see [API ENDPOINTS REFERENCE](/core-concepts/api_endpoints/).

## Quality Assurance

### Running Tests

Execute the test suite to ensure application functionality:

```bash
docker compose -f docker-compose.local.yml run django pytest
```

### Linting and Code Quality

Maintain code quality standards:

- Ensure pre-commit is installed in your development environment
- Pre-commit hooks automatically run linting against files on commit
- Run linting manually against all files with:

```bash
pre-commit run --all-files
```

## Development Workflow

### Branching Strategy

- Development work should branch from and merge back to the `development` branch
- Future plans include dedicated `uat` and `main` branches for UAT and production environments
- Currently, only the `development` environment has an associated deployment

### CI/CD Pipeline

- GitHub Actions automates testing pipelines
- The workflow configuration is in `.github/workflows/ci.yml`
- Tests run in a Docker Compose environment to execute unit tests
- CI checks are triggered on pull requests to `development`, `uat`, and `main` branches

### Semantic Versioning

This repository uses [Python Semantic Release](https://python-semantic-release.readthedocs.io/) to automatically determine and publish version numbers based on commit messages. The workflow runs automatically when changes are merged into `main`.

#### How it works

1. When a pull request is merged into `main`, the `Semantic Release` GitHub Actions workflow runs
2. It analyzes all commit messages since the last release to determine the next version number
3. It updates the version in `pyproject.toml` and `asu_apgap/__init__.py`, commits the change, creates a git tag, and publishes a GitHub Release

#### Commit prefix conventions

The version bump is determined by the **prefix** of the commit message:

| Prefix                                                             | Example                         | Version bump              |
| ------------------------------------------------------------------ | ------------------------------- | ------------------------- |
| `feat:`                                                            | `feat: add new API endpoint`    | **Minor** (0.1.0 → 0.2.0) |
| `fix:`                                                             | `fix: correct pagination bug`   | **Patch** (0.1.0 → 0.1.1) |
| `perf:`                                                            | `perf: optimize database query` | **Patch** (0.1.0 → 0.1.1) |
| `feat!:` or `BREAKING CHANGE:`                                     | `feat!: redesign auth flow`     | **Major** (0.1.0 → 1.0.0) |
| `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `test:` | `chore: update dependencies`    | No version bump           |

#### Breaking changes

A breaking change can be indicated in two ways:

- Add `!` after the type: `feat!: remove deprecated endpoint`
- Add a `BREAKING CHANGE:` footer to any commit:

  ```
  feat: redesign authentication

  BREAKING CHANGE: The /auth/login endpoint now requires a JSON body instead of form data.
  ```

#### Scopes (optional)

Scopes can be added in parentheses after the prefix to provide additional context — they do not affect the version bump:

```
feat(users): add bulk user import
fix(labs): resolve membership duplication issue
```

## Deployment

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

## Troubleshooting

### Common Issues

#### "Domain 'x' is not allowed" Error

This error occurs when trying to create a user with an email domain that isn't in the whitelist.

**Solution:** Add the domain to the whitelist via Django admin or shell. See [Domain Whitelist Setup](#domain-whitelist-setup).

#### Google OAuth Not Working

1. Verify `GOOGLE_OAUTH_REDIRECT_URL` is set to `postmessage` in `.envs/.local/.secrets`
2. Ensure `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET` are correctly set
3. Verify the OAuth client in Google Cloud Console has `http://localhost:8000` as an authorized JavaScript origin
4. Check that `http://localhost:4200` (or your frontend URL) is listed as an authorized redirect URI

#### Database Migration Errors

If you encounter migration errors, try:

```bash
# Reset and recreate the database
docker compose -f docker-compose.local.yml down -v
docker compose -f docker-compose.local.yml up -d postgres
docker compose -f docker-compose.local.yml run django python manage.py migrate
```

#### Permission Denied Errors

Permission groups are automatically created when the Django container initializes. If you encounter permission issues:

1. Verify the user has the correct role assigned
2. Check the user's organization and lab memberships
3. Review the permissions documentation in `asu_apgap/utils/permissions.py`

#### Container Won't Start

1. Check Docker logs: `docker compose -f docker-compose.local.yml logs django`
2. Ensure all required environment files exist in `.envs/.local/`
3. Verify the secrets file is properly formatted (no trailing whitespace, proper line endings)
