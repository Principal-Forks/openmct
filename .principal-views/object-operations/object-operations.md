# Object Operations

## What Problem Does This Solve?

OpenMCT manages domain objects (telemetry points, displays, folders, etc.) that need to be persisted and retrieved across sessions. The Object API provides a unified interface for creating, reading, updating, and deleting these objects while abstracting away the underlying storage mechanism (CouchDB, local storage, etc.).

## Available Operations

- **Create**: Create new domain objects with unique identifiers
- **Read**: Load existing objects by identifier
- **Update**: Modify and save changes to existing objects
- **Delete**: Remove objects from persistence
- **Search**: Find objects by criteria or type

## Design Choices

### Provider Pattern

The Object API uses a provider pattern where different persistence backends (CouchDB, LocalStorage, in-memory) can be registered and handle objects in different namespaces. This allows:

- Multiple storage backends to coexist
- Plugin-based extensibility
- Namespace isolation for different object types

### Asynchronous Operations

All object operations are asynchronous to support remote storage backends and ensure the UI remains responsive during I/O operations.

### Identifier-Based Routing

Objects are identified by a compound key `{namespace, key}` which determines which persistence provider handles the operation.

## Common Workflow Patterns

### Creating a New Object

1. User initiates creation (e.g., "Create New Display Layout")
2. Application calls `openmct.objects.save(domainObject)`
3. Object API determines appropriate provider based on namespace
4. Provider persists the object
5. Success/failure callback notifies application

### Updating an Existing Object

1. User modifies an object (e.g., changes display layout)
2. Application calls `openmct.objects.save(modifiedObject)`
3. Provider updates existing record
4. Observers are notified of changes

### Loading Object on Navigation

1. User navigates to an object (e.g., clicks in tree)
2. Application calls `openmct.objects.get(identifier)`
3. Provider retrieves object data
4. Object is rendered in the view

## Error Scenarios and Recovery

### Save Failures

- **Network Error**: Provider fails to reach remote storage
  - Recovery: Retry with exponential backoff, queue for later

- **Validation Error**: Object fails schema validation
  - Recovery: Show error to user, prevent save

- **Permission Error**: User lacks write permissions
  - Recovery: Display error message, revert to read-only mode

### Load Failures

- **Object Not Found**: Requested identifier doesn't exist
  - Recovery: Display "not found" page, suggest navigation

- **Network Error**: Cannot reach storage backend
  - Recovery: Show cached version if available, retry in background

### Conflict Resolution

When multiple users modify the same object:
- Last write wins (simple case)
- Plugins can implement custom conflict resolution
- Optimistic locking with version checks (advanced)
