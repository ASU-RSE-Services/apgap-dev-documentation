# Administration

### Creating a Django Superuser

To create an administrator account with full access to the platform:

```bash
docker compose -f docker-compose.local.yml run django python manage.py createsuperuser
```

Follow the prompts to set email and password.

**Important:** You must add your email domain to the whitelist before creating a superuser. See [Domain Whitelist Setup](/core-concepts/quickstart_guide/#domain-whitelist-setup).

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

