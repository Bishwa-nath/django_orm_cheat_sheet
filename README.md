# Django ORM Cheat Sheet

### Import model(s) before making queries
```python
from .models import ModelName, OtherModule, Project
from django.db.models import Count, Sum, Avg, Max, Min, Q, F
```
### 1. Retrieving Objects
``` python
# Get all objects
queryset = ModelName.objects.all()

# Get first / last / latest
first_item = ModelName.objects.first()
last_item = ModelName.objects.last()
latest_item = ModelName.objects.latest('created_at')  # requires DateTimeField

# Get single object (raises DoesNotExist if not found)
item = ModelName.objects.get(attribute='value')

# Get or create
item, created = ModelName.objects.get_or_create(attribute='value')

# Update or create
item, created = ModelName.objects.update_or_create(
    attribute='value',
    defaults={'another_field': 'new_value'}
)
```

### 2. Filtering & Excluding
```python
# Basic filter
queryset = ModelName.objects.filter(attribute='value')

# Exclude
queryset = ModelName.objects.exclude(attribute='value')

# Multiple filters (AND condition)
queryset = ModelName.objects.filter(attribute1='v1', attribute2='v2')

# OR condition using Q
queryset = ModelName.objects.filter(Q(attribute1='v1') | Q(attribute2='v2'))

# Field lookups
queryset = ModelName.objects.filter(
    attribute__exact='value',
    attribute__iexact='value',   # case-insensitive
    attribute__contains='val',
    attribute__icontains='val',
    attribute__startswith='val',
    attribute__istartswith='val',
    attribute__endswith='val',
    attribute__iendswith='val',
    attribute__in=['v1', 'v2', 'v3'],
    attribute__gt=10,
    attribute__gte=10,
    attribute__lt=100,
    attribute__lte=100,
    attribute__range=(10, 50),
    attribute__isnull=True
)
```
### 3. Ordering & Limiting
```python
# Ascending order
queryset = ModelName.objects.order_by('attribute')

# Descending order
queryset = ModelName.objects.order_by('-attribute')

# Multiple fields
queryset = ModelName.objects.order_by('field1', '-field2')

# Limit results
queryset = ModelName.objects.all()[:10]
```
### 4. Creating & Saving
```python
# Create directly
item = ModelName.objects.create(attribute='value')

# Create instance then save
item = ModelName(attribute='value')
item.save()

# Bulk create
items = [
    ModelName(attribute='v1'),
    ModelName(attribute='v2'),
]
ModelName.objects.bulk_create(items)
```
### 5. Updating
```python
# Update single object
item = ModelName.objects.get(pk=1)
item.attribute = "new_value"
item.save()

# Bulk update using QuerySet
ModelName.objects.filter(attribute='old').update(attribute='new')

# bulk_update (requires a list of objects)
items = list(ModelName.objects.filter(attribute='v1'))
for obj in items:
    obj.attribute = "updated"
ModelName.objects.bulk_update(items, ['attribute'])

# F expressions (update based on current value)
ModelName.objects.update(counter=F('counter') + 1)
```
### 6. Deleting
```python
# Delete single object
item = ModelName.objects.get(pk=1)
item.delete()

# Bulk delete
ModelName.objects.filter(attribute='value').delete()

# Delete all
ModelName.objects.all().delete()
```
### 7. Aggregations & Annotations
```python
# Aggregations
total = ModelName.objects.aggregate(Sum('amount'))
average = ModelName.objects.aggregate(Avg('amount'))
maximum = ModelName.objects.aggregate(Max('amount'))
minimum = ModelName.objects.aggregate(Min('amount'))
count = ModelName.objects.aggregate(Count('id'))

# Annotations (add computed fields)
queryset = ModelName.objects.annotate(order_count=Count('orders'))

# Group by (values + annotate)
queryset = ModelName.objects.values('category').annotate(total=Count('id'))
```
### 8. Relationships
```python
# One-to-Many (ForeignKey reverse lookup)
parent = ModelName.objects.first()
children = parent.childmodel_set.all()

# Many-to-Many
item = ModelName.objects.first()
related_items = item.relationshipname.all()

# Add to ManyToMany
other = OtherModule.objects.create(attribute='value')
item.relationshipname.add(other)

# Remove from ManyToMany
item.relationshipname.remove(other)

# Clear ManyToMany
item.relationshipname.clear()

# Prefetch / Select Related (performance optimization)
queryset = ModelName.objects.select_related('fk_field')   # for ForeignKey
queryset = ModelName.objects.prefetch_related('m2m_field')  # for ManyToMany
```
### 9. Raw Queries
```python
# Raw SQL query
items = ModelName.objects.raw("SELECT * FROM app_modelname WHERE attribute = %s", ['value'])
```
### 10. Extra Helpers
```python
# Count items
count = ModelName.objects.count()

# Exists check
exists = ModelName.objects.filter(attribute='value').exists()

# Distinct values
values = ModelName.objects.values_list('attribute', flat=True).distinct()

# Only select certain fields
queryset = ModelName.objects.only('field1', 'field2')

# Defer certain fields
queryset = ModelName.objects.defer('large_field')
```

### Extra tips:
| Method          | Return Type             | If No Match              | If Multiple Matches                | Common Use Case                                             |
| --------------- | ----------------------- | ------------------------ | ---------------------------------- | ----------------------------------------------------------- |
| **`.get()`**    | Single object           | ❌ Raises `DoesNotExist`  | ❌ Raises `MultipleObjectsReturned` | When you expect exactly **one** object (e.g., `id`, `slug`) |
| **`.filter()`** | QuerySet (0+ objects)   | ✅ Returns empty QuerySet | ✅ Returns all matching objects     | When you want **all matches**, or you’re unsure how many    |
| **`.first()`**  | Single object or `None` | ✅ Returns `None`         | ✅ Returns the **first** match only | When you want just **one object safely**, without errors    |
