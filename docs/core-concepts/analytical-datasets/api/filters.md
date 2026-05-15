class AnalyticalDatasetFilter(django_filters.FilterSet):
    Filter for AnalyticalDataset model.


    def filter_queryset(self, queryset):
        Override to handle lab parameter even when empty
        # Check for lab parameter first, before calling super()
        request_params = self.request.query_params if hasattr(self.request, "query_params") else self.request.GET

        if "lab" in request_params:
            # Lab parameter is present, apply the filter
            queryset = self.filter_by_labs(queryset, "lab", None)

        # Apply other filters
        return super().filter_queryset(queryset)

    def _get_request_params(self):
        Get request parameters (DRF query_params or Django GET).

    def filter_exclude_pending(self, queryset, name, value):
        Exclude analytical datasets with PENDING status when value is True.

    def filter_has_pending_access_requests(self, queryset, name, value):
        Filter datasets by whether they have at least one pending AccessRequest.

        ``Exists`` plus ``OuterRef`` is preferred over
        ``.filter(access_requests__status=...).distinct()`` because it composes
        cleanly with the existing ``_get_optimized_queryset`` ``Prefetch``
        without forcing a JOIN duplication, and it returns each matching
        dataset exactly once without needing ``.distinct()``.


    def _parse_lab_ids(self, lab_ids_raw):
        Parse lab IDs from raw string values, handling comma-separated values.

    def filter_by_labs(self, queryset, name, value):
        Filter by lab IDs (supports multiple values via repeated parameters or comma-separated).

        # Check if lab parameter was explicitly provided (even if empty)

        # If no lab parameter provided at all, return queryset unchanged

        # If lab parameter was provided but list is empty, return empty queryset

        # Check if all values are empty strings

        # Parse lab IDs from raw values

        # If no valid lab IDs found but parameter was provided, return empty queryset
        # This handles the case where lab= was sent with no value or only empty strings

