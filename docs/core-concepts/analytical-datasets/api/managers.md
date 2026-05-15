class AnalyticalDatasetQuerySet(models.QuerySet):
    Custom QuerySet for AnalyticalDataset model.

    def in_active_labs(self):
        Filter analytical datasets to only include those in active labs.

    def for_creator(self, user):
        Filter analytical datasets created by the specified user.

    def for_project(self, project):
        Filter analytical datasets belonging to the specified project.

    def for_lab(self, lab):
        Filter analytical datasets belonging to the specified lab.

    def pending_approval(self):
        Filter analytical datasets with pending approval status.


class AnalyticalDatasetManager(models.Manager):
    Custom Manager for AnalyticalDataset model.

    def in_active_labs(self):
        Filter analytical datasets to only include those in active labs.

    def for_creator(self, user):
        Filter analytical datasets created by the specified user.

    def for_project(self, project):
        Filter analytical datasets belonging to the specified project.

    def for_lab(self, lab):
        Filter analytical datasets belonging to the specified lab.

    def pending_approval(self):
        Filter analytical datasets with pending approval status.
