class AnalyticalDatasetCreateSerializer(serializers.ModelSerializer):
    Serializer for creating analytical datasets.

    The GCS bucket is automatically generated via a post_save signal,
    so it is a read-only field. Only the name and project are required when creating a dataset.



class FileWithMetadataSerializer(serializers.ModelSerializer):
    Serializer for files with their metadata from the original file.


    def get_metadata_tags(self, obj):
        Get metadata tags from the original file.
        If this is a copied file, get metadata from the original file.
        Otherwise, get metadata from this file.

class AccessRequestFileSerializer(serializers.ModelSerializer):
    Serializer for AccessRequest related to a file in a dataset.



class AnalyticalDatasetDetailSerializer(serializers.ModelSerializer):
    Serializer for retrieving analytical dataset details.

        Whether the dataset has at least one PENDING access request.

        Uses the existing ``prefetch_related("access_requests")`` cache that
        ``AnalyticalDatasetViewSet._get_optimized_queryset`` sets up, so no
        extra query is issued per row.

    def _request_user_is_platform_admin(self) -> bool:
        Memoized platform-admin check.

        ``is_platform_admin`` issues a ``user.groups`` lookup. For
        ``many=True`` responses DRF reuses the same child serializer
        instance across rows, so caching the answer here avoids a
        per-row N+1.

    def get_access_requests(self, obj):
        Get access requests associated with this dataset.
        Uses prefetched access_requests to avoid N+1 queries.

        Visibility rules:
        - Platform admins: see every access request on the dataset (they
          can approve any request, so the dashboard must show them all).
        - Dataset creator: sees every access request on their dataset.
        - Everyone else: sees only access requests they are listed as an
          approver on (typically the file's lab director).

        # Use prefetched access_requests - .all() uses the prefetch cache

        if user and not self._request_user_is_platform_admin() and obj.created_by != user:
            # Note: This requires approvers to be prefetched if we want to avoid N+1
            # For now, we'll filter in Python since it's a small list
            access_requests = [ar for ar in access_requests if user in ar.approvers.all()]

        return AccessRequestFileSerializer(access_requests, many=True).data

    def get_file_count(self, obj):
        Get the number of files in this dataset.
        Uses prefetched files if available to avoid extra query.
        # Using len() on prefetched data avoids an extra COUNT query


class CopyFileToDatasetSerializer(serializers.Serializer):
    Serializer for copying a file to an analytical dataset.

    source_file_id = serializers.IntegerField(required=True)

    def validate_source_file_id(self, value):
        Validate that the source file exists.


class AnalyticalDatasetListSerializer(serializers.ModelSerializer):
    Lightweight serializer for listing analytical datasets.


    def get_file_count(self, obj):
        Get the number of files in this dataset.
        Uses prefetched files if available to avoid extra query.

        # Using len() on prefetched data avoids an extra COUNT query
        return len(obj.files.all())


    def get_has_pending_access_requests(self, obj) -> bool:
        Whether the dataset has at least one PENDING access request.

        Uses the existing ``prefetch_related("access_requests")`` cache that
        ``AnalyticalDatasetViewSet._get_optimized_queryset`` sets up, so no
        extra query is issued per row.

    def _request_user_is_platform_admin(self) -> bool:
        Memoized platform-admin check.

        ``is_platform_admin`` issues a ``user.groups`` lookup. For
        ``many=True`` responses DRF reuses the same child serializer
        instance across rows, so caching the answer here avoids a
        per-row N+1.

    def get_access_requests(self, obj):
        Nested access requests for the dashboard widget.

        Mirrors ``AnalyticalDatasetDetailSerializer.get_access_requests``:
        - Platform admins see every access request on the dataset.
        - Dataset creator sees every access request on their dataset.
        - Other users see only access requests they are listed as an
          approver on (typically the file's lab director).

        Reads from the ``Prefetch("access_requests", ...)`` cache, so no
        extra SQL is issued per row.

    def get_files(self, obj):
        Nested files for the dashboard widget.

        Reads from the ``Prefetch("files", ...)`` cache so the file rows
        themselves do not cost extra queries. ``FileWithMetadataSerializer``
        is reused to keep the shape identical to the dedicated files
        endpoint.


class RequestAccessToFilesSerializer(serializers.Serializer):
    Serializer for requesting access to files in an analytical dataset.


    def validate_files(self, value):
        Validate that all file IDs exist.


class DenyAccessRequestSerializer(serializers.Serializer):
    Serializer for denying access requests to an analytical dataset.



# ---------------------------------------------------------------------------
# Dynamic-dataset subscription serializers
# ---------------------------------------------------------------------------


class DatasetSubscriptionSerializer(serializers.ModelSerializer):
    """Read/write serializer for ``DatasetSubscription``.

    Validates the ``metadata_filters`` payload against the same rules used by
    the runtime evaluator, enforces a non-empty rule, and applies
    case-insensitive name uniqueness within the dataset.
    """



    def validate_metadata_filters(self, value):
        # Dual-write `key_id` alongside `key` so the runtime evaluator can
        # resolve via FK and rule matching survives any later `Key.name`
        # rename. Idempotent and tolerant of unresolvable names.

    def validate(self, attrs):
        # Empty-rule guard: at least one metadata condition or source type.
        # This must hold both at create and at update; we look at whatever
        # the merged state will be after applying ``attrs`` to the instance.

        # Case-insensitive uniqueness within the dataset.


class DatasetSubscriptionPreviewPayloadSerializer(serializers.Serializer):
    """Validates an unsaved subscription rule for the preview endpoint."""


    def validate_metadata_filters(self, value):
        # Preview is ephemeral but should still hand the evaluator a payload
        # in the same shape that the persisted serializer produces.


class DatasetSubscriptionPreviewFileSerializer(serializers.ModelSerializer):
    """Lightweight File representation used in subscription preview results."""

