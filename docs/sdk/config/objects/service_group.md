# Service Group Configuration Object

## Table of Contents

1. [Overview](#overview)
2. [Core Methods](#core-methods)
3. [Service Group Model Attributes](#service-group-model-attributes)
4. [Exceptions](#exceptions)
5. [Basic Configuration](#basic-configuration)
6. [Usage Examples](#usage-examples)
    - [Creating Service Groups](#creating-service-groups)
    - [Retrieving Service Groups](#retrieving-service-groups)
    - [Updating Service Groups](#updating-service-groups)
    - [Listing Service Groups](#listing-service-groups)
    - [Filtering Responses](#filtering-responses)
    - [Controlling Pagination with max_limit](#controlling-pagination-with-max_limit)
    - [Deleting Service Groups](#deleting-service-groups)
7. [Managing Configuration Changes](#managing-configuration-changes)
    - [Performing Commits](#performing-commits)
    - [Monitoring Jobs](#monitoring-jobs)
8. [Error Handling](#error-handling)
9. [Best Practices](#best-practices)
10. [Full Script Examples](#full-script-examples)
11. [Related Models](#related-models)

## Overview

The `ServiceGroup` class provides functionality to manage service groups in Palo Alto Networks' Strata Cloud Manager.
This
class inherits from `BaseObject` and provides methods for creating, retrieving, updating, and deleting service groups
that can be used to organize and manage collections of services for security policies and NAT rules.

## Core Methods

| Method     | Description                      | Parameters                               | Return Type                       |
|------------|----------------------------------|------------------------------------------|-----------------------------------|
| `create()` | Creates a new service group      | `data: Dict[str, Any]`                   | `ServiceGroupResponseModel`       |
| `get()`    | Retrieves a group by ID          | `object_id: str`                         | `ServiceGroupResponseModel`       |
| `update()` | Updates an existing group        | `service_group: ServiceGroupUpdateModel` | `ServiceGroupResponseModel`       |
| `delete()` | Deletes a group                  | `object_id: str`                         | `None`                            |
| `list()`   | Lists groups with filtering      | `folder: str`, `**filters`               | `List[ServiceGroupResponseModel]` |
| `fetch()`  | Gets group by name and container | `name: str`, `folder: str`               | `ServiceGroupResponseModel`       |

## Service Group Model Attributes

| Attribute | Type      | Required | Description                                 |
|-----------|-----------|----------|---------------------------------------------|
| `name`    | str       | Yes      | Name of group (max 63 chars)                |
| `id`      | UUID      | Yes*     | Unique identifier (*response only)          |
| `members` | List[str] | Yes      | List of service members                     |
| `tag`     | List[str] | No       | List of tags (max 64 chars each)            |
| `folder`  | str       | Yes**    | Folder location (**one container required)  |
| `snippet` | str       | Yes**    | Snippet location (**one container required) |
| `device`  | str       | Yes**    | Device location (**one container required)  |

## Exceptions

| Exception                    | HTTP Code | Description                        |
|------------------------------|-----------|------------------------------------|
| `InvalidObjectError`         | 400       | Invalid group data or format       |
| `MissingQueryParameterError` | 400       | Missing required parameters        |
| `NameNotUniqueError`         | 409       | Group name already exists          |
| `ObjectNotPresentError`      | 404       | Group not found                    |
| `ReferenceNotZeroError`      | 409       | Group still referenced by policies |
| `AuthenticationError`        | 401       | Authentication failed              |
| `ServerError`                | 500       | Internal server error              |

## Basic Configuration

```python
from scm.client import Scm

# Initialize client
client = Scm(
    client_id="your_client_id",
    client_secret="your_client_secret",
    tsg_id="your_tsg_id"
)

# Access service groups directly through the client
# No need to initialize a separate ServiceGroup object
```

## Usage Examples

### Creating Service Groups

```python
# Basic service group configuration
basic_group = {
    "name": "web-services",
    "members": ["HTTP", "HTTPS"],
    "folder": "Texas",
    "tag": ["Web"]
}

# Create basic group using the client
basic_group_obj = client.service_group.create(basic_group)

# Extended service group configuration
extended_group = {
    "name": "app-services",
    "members": ["HTTP", "HTTPS", "SSH", "FTP"],
    "folder": "Texas",
    "tag": ["Application", "Production"]
}

# Create extended group
extended_group_obj = client.service_group.create(extended_group)
```

### Retrieving Service Groups

```python
# Fetch by name and folder
group = client.service_group.fetch(name="web-services", folder="Texas")
print(f"Found group: {group.name}")

# Get by ID
group_by_id = client.service_group.get(group.id)
print(f"Retrieved group: {group_by_id.name}")
print(f"Members: {', '.join(group_by_id.members)}")
```

### Updating Service Groups

```python
# Fetch existing group
existing_group = client.service_group.fetch(name="web-services", folder="Texas")

# Update members
existing_group.members = ["HTTP", "HTTPS", "HTTP-8080"]
existing_group.tag = ["Web", "Updated"]

# Perform update
updated_group = client.service_group.update(existing_group)
```

### Listing Service Groups

```python
# List with direct filter parameters
filtered_groups = client.service_group.list(
    folder='Texas',
    values=['HTTP', 'HTTPS'],
    tags=['Production']
)

# Process results
for group in filtered_groups:
    print(f"Name: {group.name}")
    print(f"Members: {', '.join(group.members)}")

# Define filter parameters as dictionary
list_params = {
    "folder": "Texas",
    "values": ["SSH", "FTP"],
    "tags": ["Application"]
}

# List with filters as kwargs
filtered_groups = client.service_group.list(**list_params)
```

### Filtering Responses

The `list()` method supports additional parameters to refine your query results even further. Alongside basic filters
(like `types`, `values`, and `tags`), you can leverage the `exact_match`, `exclude_folders`, `exclude_snippets`, and
`exclude_devices` parameters to control which objects are included or excluded after the initial API response is fetched.

**Parameters:**

- `exact_match (bool)`: When `True`, only objects defined exactly in the specified container (`folder`, `snippet`, or `device`) are returned. Inherited or propagated objects are filtered out.
- `exclude_folders (List[str])`: Provide a list of folder names that you do not want included in the results.
- `exclude_snippets (List[str])`: Provide a list of snippet values to exclude from the results.
- `exclude_devices (List[str])`: Provide a list of device values to exclude from the results.

**Examples:**

```python
# Only return service_groups defined exactly in 'Texas'
exact_service_groups = service_groups.list(
   folder='Texas',
   exact_match=True
)

for app in exact_service_groups:
   print(f"Exact match: {app.name} in {app.folder}")

# Exclude all service_groups from the 'All' folder
no_all_service_groups = service_groups.list(
   folder='Texas',
   exclude_folders=['All']
)

for app in no_all_service_groups:
   assert app.folder != 'All'
   print(f"Filtered out 'All': {app.name}")

# Exclude service_groups that come from 'default' snippet
no_default_snippet = service_groups.list(
   folder='Texas',
   exclude_snippets=['default']
)

for app in no_default_snippet:
   assert app.snippet != 'default'
   print(f"Filtered out 'default' snippet: {app.name}")

# Exclude service_groups associated with 'DeviceA'
no_deviceA = service_groups.list(
   folder='Texas',
   exclude_devices=['DeviceA']
)

for app in no_deviceA:
   assert app.device != 'DeviceA'
   print(f"Filtered out 'DeviceA': {app.name}")

# Combine exact_match with multiple exclusions
combined_filters = service_groups.list(
   folder='Texas',
   exact_match=True,
   exclude_folders=['All'],
   exclude_snippets=['default'],
   exclude_devices=['DeviceA']
)

for app in combined_filters:
   print(f"Combined filters result: {app.name} in {app.folder}")
```

### Controlling Pagination with max_limit

The SDK supports pagination through the `max_limit` parameter, which defines how many objects are retrieved per API call. By default, `max_limit` is set to 2500. The API itself imposes a maximum allowed value of 5000. If you set `max_limit` higher than 5000, it will be capped to the API's maximum. The `list()` method will continue to iterate through all objects until all results have been retrieved. Adjusting `max_limit` can help manage retrieval performance and memory usage when working with large datasets.

```python
# Initialize the client with a custom max_limit for service groups
# This will retrieve up to 4321 objects per API call, up to the API limit of 5000.
client = Scm(
    client_id="your_client_id",
    client_secret="your_client_secret",
    tsg_id="your_tsg_id",
    service_group_max_limit=4321
)

# Now when we call list(), it will use the specified max_limit for each request
# while auto-paginating through all available objects.
all_groups = client.service_group.list(folder='Texas')

# 'all_groups' contains all objects from 'Texas', fetched in chunks of up to 4321 at a time.
```

### Deleting Service Groups

```python
# Delete by ID
group_id = "123e4567-e89b-12d3-a456-426655440000"
client.service_group.delete(group_id)
```

## Managing Configuration Changes

### Performing Commits

```python
# Prepare commit parameters
commit_params = {
    "folders": ["Texas"],
    "description": "Updated service groups",
    "sync": True,
    "timeout": 300  # 5 minute timeout
}

# Commit the changes directly using the client
result = client.commit(**commit_params)

print(f"Commit job ID: {result.job_id}")
```

### Monitoring Jobs

```python
# Get status of specific job using the client
job_status = client.get_job_status(result.job_id)
print(f"Job status: {job_status.data[0].status_str}")

# List recent jobs using the client
recent_jobs = client.list_jobs(limit=10)
for job in recent_jobs.data:
    print(f"Job {job.id}: {job.type_str} - {job.status_str}")
```

## Error Handling

```python
from scm.exceptions import (
    InvalidObjectError,
    MissingQueryParameterError,
    NameNotUniqueError,
    ObjectNotPresentError,
    ReferenceNotZeroError
)

try:
    # Create group configuration
    group_config = {
        "name": "test-group",
        "members": ["HTTP", "HTTPS"],
        "folder": "Texas",
        "tag": ["Test"]
    }

    # Create the group using the client
    new_group = client.service_group.create(group_config)

    # Commit changes using the client
    result = client.commit(
        folders=["Texas"],
        description="Added test group",
        sync=True
    )

    # Check job status using the client
    status = client.get_job_status(result.job_id)

except InvalidObjectError as e:
    print(f"Invalid group data: {e.message}")
except NameNotUniqueError as e:
    print(f"Group name already exists: {e.message}")
except ObjectNotPresentError as e:
    print(f"Group not found: {e.message}")
except ReferenceNotZeroError as e:
    print(f"Group still in use: {e.message}")
except MissingQueryParameterError as e:
    print(f"Missing parameter: {e.message}")
```

## Best Practices

1. **Member Management**
    - Use descriptive member names
    - Keep member lists organized
    - Document member purposes
    - Validate member existence
    - Monitor member changes

2. **Container Management**
    - Always specify exactly one container (folder, snippet, or device)
    - Use consistent container names
    - Validate container existence
    - Group related service groups

3. **Client Usage**
    - Use the unified client interface (`client.service_group`) for simpler code
    - Perform commits directly on the client (`client.commit()`)
    - Monitor jobs using client methods (`client.get_job_status()`, `client.list_jobs()`)
    - Initialize the client once and reuse across different object types

4. **Performance**
    - Use appropriate pagination
    - Cache frequently accessed groups
    - Implement proper retry logic
    - Monitor group sizes

5. **Security**
    - Follow least privilege principle
    - Validate input data
    - Use secure connection settings
    - Implement proper authentication
    - Monitor policy references

## Full Script Examples

Refer to
the [service_group.py example](https://github.com/cdot65/pan-scm-sdk/blob/main/examples/scm/config/objects/service_group.py).

## Related Models

- [ServiceGroupCreateModel](../../models/objects/service_group_models.md#Overview)
- [ServiceGroupUpdateModel](../../models/objects/service_group_models.md#Overview)
- [ServiceGroupResponseModel](../../models/objects/service_group_models.md#Overview)
