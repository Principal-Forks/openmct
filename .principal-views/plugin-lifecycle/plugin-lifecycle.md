# Plugin Lifecycle

## What Problem Does This Solve?

OpenMCT is designed to be extended with custom functionality. The plugin system provides a structured way to add new features without modifying core code. Plugins can register domain object types, visualization views, telemetry providers, user actions, and more.

## Available Operations

### Plugin Installation

- **Install**: Register a plugin with the OpenMCT instance
- **Initialize**: Execute plugin setup code
- **Configure**: Pass options to customize plugin behavior

### Plugin Registration Points

Plugins can register:
- **Object Types**: New kinds of domain objects (e.g., custom displays)
- **View Providers**: Custom visualizations for objects
- **Telemetry Providers**: New data sources
- **Actions**: User commands and context menu items
- **Toolbar Providers**: Custom toolbar controls
- **Inspector Views**: Property panels for objects

## Design Choices

### Functional Plugin API

Plugins are functions that receive the OpenMCT API instance:

```javascript
function MyPlugin(openmct) {
  // Register types, views, providers, etc.
}

openmct.install(MyPlugin);
```

This approach:
- Keeps plugins self-contained
- Makes dependencies explicit
- Enables plugin composition

### Install-time Registration

All plugin registrations happen during installation (before app start) rather than lazily. This:
- Ensures deterministic initialization order
- Simplifies dependency resolution
- Enables static analysis of available features

### Registry Pattern

Plugins register components into well-defined registries (types, views, providers). The core queries these registries when needed, enabling loose coupling.

## Common Workflow Patterns

### Installing a Visualization Plugin

1. Application imports plugin function
2. Calls `openmct.install(VisualizationPlugin)`
3. Plugin registers object type (e.g., "example.gauge")
4. Plugin registers view provider for that type
5. Plugin registers inspector views if needed
6. Installation completes
7. App starts, new type is available in creation menu

### Installing a Telemetry Provider

1. Application installs plugin with provider options
2. Plugin receives configuration (URLs, credentials, etc.)
3. Plugin registers telemetry provider for specific object types
4. Plugin optionally registers format mappers
5. When telemetry is requested, provider is queried

### Plugin Composition

Multiple plugins can work together:
1. Plugin A registers a custom object type
2. Plugin B registers a view provider for that type
3. Plugin C adds actions applicable to that type
4. All plugins are installed independently
5. Core orchestrates their interactions

## Error Scenarios and Recovery

### Installation Failures

- **Missing Dependencies**: Plugin requires API not available
  - Recovery: Log error, skip plugin installation, continue with others

- **Duplicate Registration**: Plugin tries to register already-registered type
  - Recovery: Log warning, use first registration, ignore duplicate

- **Invalid Configuration**: Plugin receives malformed options
  - Recovery: Use defaults, log warning, install with reduced functionality

### Runtime Errors

- **View Provider Failure**: Custom view throws exception during rendering
  - Recovery: Catch error, display fallback view, log to console

- **Telemetry Provider Error**: Provider fails to fetch data
  - Recovery: Retry with backoff, use cached data, display error to user

### Version Compatibility

- **API Changes**: Plugin built for older OpenMCT version
  - Recovery: Version check on install, feature detection, graceful degradation

- **Breaking Changes**: Plugin uses removed API
  - Recovery: Console warning, disable incompatible features, suggest update

## Best Practices

### Plugin Design

- Keep plugins focused on single responsibility
- Avoid assumptions about other plugins
- Provide sensible defaults for all options
- Document required vs optional configuration

### Error Handling

- Never let plugin errors crash the application
- Log errors clearly for debugging
- Provide fallbacks for critical functionality
- Validate configuration before using it

### Testing

- Test plugins in isolation
- Test combinations with other plugins
- Test with different configurations
- Test error scenarios and recovery
