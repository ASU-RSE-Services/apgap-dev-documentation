
API views for analytical datasets.


class OptionalPageNumberPagination(PageNumberPagination):
    Only paginate if ?pagination=true is present.
    Otherwise return full list.



class AnalyticalDatasetViewSet(viewsets.ModelViewSet):

    ViewSet for managing analytical datasets.

    Provides CRUD operations for analytical datasets with the following endpoints:
    - POST /api/analytical_datasets/ - Create a new dataset
    - GET /api/analytical_datasets/ - List all datasets
    - GET /api/analytical_datasets/<id>/ - Retrieve a specific dataset with files
    - DELETE /api/analytical_datasets/<id>/ - Delete a dataset and all associated files
    - POST /api/analytical_datasets/<id>/copy_file/ - Copy a file to the dataset
    - GET /api/analytical_datasets/<id>/files/ - List files in the dataset (paginated)
    - DELETE /api/analytical_datasets/<dataset_id>/files/<file_id>/ - Delete a file from the dataset

    Only returns analytical datasets for projects in active labs.



    def _get_optimized_queryset(self, queryset):

        Apply select_related and prefetch_related optimizations to reduce N+1 queries.

    def get_queryset(self):
        
        Filter analytical datasets to only those the user has access to.
        Uses the User model's get_analytical_datasets() method for consistent permission filtering.

        Lab directors also get access to datasets with pending access requests
        for files in their labs, even if the dataset belongs to another lab's project.
        

    def get_serializer_class(self):  # NOQA: PLR0911
        
        Return the appropriate serializer based on the action.
        

    @action(detail=True, methods=["get"], url_path="build-log")
    def build_log(self, request, pk=None):
        
        Return the most recent captured GCS bucket creation error for the
        given analytical dataset. Empty string when there is no recorded error.
        

    @action(detail=False, methods=["get"], url_path="pending-approvals")
    def pending_approvals(self, request):
        
        Get all analytical datasets with pending access requests that the user can approve.

        Only returns datasets where the user has approval authority:
        - Platform admins: All datasets with pending requests in active labs
        - Lab directors: Only datasets with pending requests for files in their labs

        Only returns datasets in active labs.
        
        # Check if user is a platform admin
        is_platform_admin = (
        )

        if is_platform_admin:
            # Platform admins can approve all pending requests in active labs
        else:
            # Get active labs where user is a lab director

            if not user_director_lab_ids:
                # User is not a lab director of any active lab, return empty

            # Return datasets with pending requests for files in labs where user is director,
            # excluding datasets the user created themselves (they should not approve their own).


    def perform_create(self, serializer):
        
        Set the created_by field when creating a dataset.
        

    def destroy(self, request, *args, **kwargs):
        
        Delete an analytical dataset and all associated files.

        Only the user who created the dataset or a lab director of the lab
        that the dataset's project belongs to can delete the dataset.

        This will:
        - Delete all files in the dataset from GCS
        - Delete all file records from the database
        - Delete the dataset record
        - Clean up orphaned original files if applicable
        

    def _get_or_create_access_request(self, user, dataset, source_file):
        
        Get or create an access request for copying a file to a dataset.

        Returns the access request.
        

    @action(detail=True, methods=["post"], url_path="copy_file")
    def copy_file(self, request, pk=None):  # noqa: PLR0911
        
        Copy a file to this analytical dataset.

        First checks if the user's organization has auto-approval enabled.
        - If auto-approval: Creates an APPROVED access request and copies the file.
        - If no auto-approval: Creates a PENDING access request and returns without copying.


        Returns:
        - If auto-approved: The newly created file record (201)
        - If pending approval: Message indicating access request was created (200)
        

        try:
            if access_request.status != AccessRequestStatus.APPROVED:
                return Response(
                    {
                    },
                )

            # File is approved — check whether ALL requests for this dataset
            # are also approved. Files are only copied once every request is
            # approved (handled by the post_save signal).
            has_pending = (
            )

            if has_pending:
                return Response(
                )

            # All approved — the signal should have copied the file already

            # Edge case — copy it now


    @action(detail=True, methods=["get"], url_path="files")
    def files(self, request, pk=None):
        
        List files in this analytical dataset.

        Pagination and ?search= behave the same as /api/files/.
        
        # look up the dataset directly (no filter_queryset) so ?search= is not consumed

    def delete_file(self, request, pk=None, file_id=None):
        
        Delete a file from this analytical dataset.

        This deletes both the database record and the GCS blob.
        

    @action(detail=True, methods=["post"], url_path="approve")
    def approve(self, request, pk=None):
        
        Approve access requests for an analytical dataset.

        Platform admins can approve ALL pending access requests for the dataset.
        Lab directors can approve pending access requests for files in their active lab(s).
        

    @action(detail=True, methods=["post"], url_path="deny")
    def deny(self, request, pk=None):
        
        Deny access requests for an analytical dataset.
        Verifies that the user is a lab director and denies (rejects) all pending access requests
        for files in this dataset that belong to their active lab(s).


    @action(detail=True, methods=["get"], url_path="metadata_csv")
    def metadata_csv(self, request, pk=None):
        
        Export metadata for all files in the analytical dataset as CSV.

        Returns a CSV file download response with one row per file.
        

    @action(detail=True, methods=["post"], url_path="bulk-copy-files")
    def bulk_copy_files(self, request, pk=None):
        
        Bulk copy files to this analytical dataset.

        For each file ID provided, this endpoint will get or create an AccessRequest
        linking the file to this dataset.

        Uses a two-phase bulk approach:
        Phase 1 — bulk_create all new requests as PENDING (no post_save signals fire).
        Phase 2 — individually save auto-approvable requests to APPROVED, so the
        post_save signal sees accurate total counts and won't prematurely mark the
        dataset as APPROVED.

        # Phase 1: bulk_create all new access requests as PENDING.
        # bulk_create does not fire post_save signals, so no premature
        # dataset approval can occur.

        # Bulk fetch lab directors for all relevant labs in one query

        # Bulk create all approver records

        # Phase 2: Auto-approve eligible requests. Each individual save fires
        # the post_save signal which handles file copying and the dataset
        # approval check with accurate total counts.

    # ------------------------------------------------------------------
    # Dynamic-dataset subscriptions
    # ------------------------------------------------------------------

    def _check_subscription_write_permission(self, request, dataset):
        Subscription writes follow the same rule as deleting the dataset:
        platform admin OR creator of the dataset OR lab director of the
        dataset's lab.
        

    def _subscription_preview_queryset(self, request, dataset, *, metadata_filters, logic, source_type_ids):
        Build a PRIMARY-only queryset filtered by the rule, scoped to the
        files this user can see.

        For now this matches the data-catalog default: a non-admin user only
        sees PRIMARY files from labs they belong to, plus all PRIMARY files
        from the dataset's lab. Lab directors / platform admins see all
        PRIMARY files. ``allow_cross_lab`` does not affect what is shown in
        preview — it only affects what would be auto-copied vs. requested.
        

    def subscriptions(self, request, pk=None):
        List or create subscriptions for this dataset.


    @action(
        detail=True,
        methods=["get", "patch", "delete"],
        # Constrain sub_id to digits so the literal "preview" path is not
        # captured by this route (DRF/Django try patterns in declaration
        # order and the more permissive [^/.]+ would match "preview").
        url_path=r"subscriptions/(?P<sub_id>\d+)",
    )
    def subscription_detail(self, request, pk=None, sub_id=None):
        Retrieve / update / delete a single subscription rule.


    @action(
        detail=True,
        methods=["post"],
        url_path=r"subscriptions/(?P<sub_id>\d+)/preview",
    )
    def subscription_preview(self, request, pk=None, sub_id=None):
        Preview matches for an *existing* subscription rule.

        Returns a count of currently PRIMARY files that match the rule, plus
        the first ``sample_size`` results (default 5, max 50).


    @action(
        detail=True,
        methods=["post"],
        url_path="subscriptions/preview",
    )
    def subscription_preview_payload(self, request, pk=None):
        Preview matches for an *unsaved* subscription rule payload.

        Used by the UI's "Preview matches" button so users can sanity-check
        before saving.

