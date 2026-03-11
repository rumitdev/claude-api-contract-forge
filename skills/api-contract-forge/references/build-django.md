# Django REST Framework — Build Templates

Use these templates when the project detection (Step 0.1) identifies Django REST Framework. Adapt all placeholder tokens to the actual resource name. Use snake_case per Python/Django conventions.

Generate: Model, Serializer, ViewSet, URL config, Permissions. Generate only the operations the user requested.

---

## File 1: Model — `{app}/models.py`

```python
from django.db import models
import uuid

class {Resource}(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(max_length=200)
    amount = models.DecimalField(max_digits=12, decimal_places=2)
    status = models.CharField(
        max_length=20,
        choices=[("draft", "Draft"), ("sent", "Sent"), ("paid", "Paid")],
        default="draft",
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ["-created_at"]

    def __str__(self):
        return self.title
```

---

## File 2: Serializer — `{app}/serializers.py`

```python
from rest_framework import serializers
from .models import {Resource}

class {Resource}Serializer(serializers.ModelSerializer):
    class Meta:
        model = {Resource}
        fields = ["id", "title", "amount", "status", "created_at", "updated_at"]
        read_only_fields = ["id", "created_at", "updated_at"]

class Create{Resource}Serializer(serializers.ModelSerializer):
    """Separate serializer for creation — allows different validation rules."""
    class Meta:
        model = {Resource}
        fields = ["title", "amount", "status"]

    def validate_amount(self, value):
        if value < 0:
            raise serializers.ValidationError("Amount must be non-negative.")
        return value
```

Using separate serializers for read vs. write operations gives you tighter control over which fields are accepted on input vs. returned on output. This prevents accidental field exposure.

---

## File 3: ViewSet — `{app}/views.py`

```python
from rest_framework import viewsets, permissions, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.filters import SearchFilter, OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend
from .models import {Resource}
from .serializers import {Resource}Serializer, Create{Resource}Serializer

class {Resource}ViewSet(viewsets.ModelViewSet):
    queryset = {Resource}.objects.all()
    serializer_class = {Resource}Serializer
    permission_classes = [permissions.IsAuthenticated]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ["status"]
    search_fields = ["title"]
    ordering_fields = ["created_at", "amount"]
    ordering = ["-created_at"]

    def get_serializer_class(self):
        if self.action == "create":
            return Create{Resource}Serializer
        return {Resource}Serializer

    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)
```

---

## File 4: URLs — `{app}/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import {Resource}ViewSet

router = DefaultRouter()
router.register(r"{resources}", {Resource}ViewSet)

urlpatterns = [
    path("", include(router.urls)),
]
```

Then register in the project's main `urls.py`:
```python
urlpatterns = [
    path("api/v1/", include("{app}.urls")),
]
```

---

## File 5: Migration

Run after creating the model:
```bash
python manage.py makemigrations {app}
python manage.py migrate
```

---

## Pagination (project-level)

If not already configured, add to `settings.py`:
```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 10,
}
```
