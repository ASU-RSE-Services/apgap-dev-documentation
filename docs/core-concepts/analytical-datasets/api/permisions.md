class AnalyticalDatasetPermission(permissions.BasePermission):
    Permission class for analytical dataset operations.

    Permissions:
    - Create: Platform Admin, Lab Director, Bioinformatics User (for their projects)
    - List/Retrieve: Platform Admin, Lab Director, Bioinformatics User (for their projects)
    - Delete: Platform Admin, Lab Director (for their lab projects), Bioinformatics User (personal datasets only)
    - Copy File: Platform Admin, Lab Director, Bioinformatics User (for their projects)
    - Approve/Deny Requests: Platform Admin, Lab Director (for their lab data)

    def has_permission(self, request, view):  # noqa: PLR0911
        Check if user has permission to perform the action.

        # Platform Admin has full access

        # For list/retrieve, allow all authenticated users (filtering happens in get_queryset)

        # For create, check if user has project access
            # If no project specified, allow (will be filtered in serializer/view)

        # For copy_file and delete_file actions, defer to object permission

        # For other actions, defer to object permission

    def has_object_permission(self, request, view, obj):
        Check if user has permission to perform the action on a specific analytical dataset.

        # Platform Admin has full access

        # Check project access

        # Lab directors can retrieve/approve/deny datasets with pending requests
        # for files in their labs, even if they don't have access to the dataset's project
