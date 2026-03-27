# View Rendering

## What Problem Does This Solve?

OpenMCT needs to display different visualizations for different types of domain objects. A telemetry point might be shown as a plot, table, or gauge. A display layout might be rendered as a flexible canvas. The View Rendering system matches objects to appropriate views and handles the component lifecycle.

## Available Operations

### View Provider Matching

- **Evaluate**: Determine which view providers can display an object
- **Prioritize**: Select the best provider when multiple match
- **Fallback**: Use default view when no specific provider matches

### View Lifecycle

- **Create**: Instantiate view component
- **Mount**: Attach view to DOM
- **Update**: Refresh view when object changes
- **Unmount**: Clean up when view is closed

### View Types

- **Primary Views**: Main object visualization (plot, table, imagery)
- **Toolbar Views**: Controls for the current view
- **Inspector Views**: Property panels for configuration

## Design Choices

### Provider Pattern with Priority

View providers declare:
- **Predicate**: Function that tests if provider can display object
- **Priority**: Numeric value for selection when multiple match
- **Component**: Vue component or factory for the view

Higher priority providers win when multiple match, allowing plugins to override default views.

### Vue Component-Based

Views are implemented as Vue components, enabling:
- Reactive data binding
- Component composition
- Lifecycle hooks for setup/teardown
- Scoped styling

### Lazy Loading

View components can be loaded asynchronously, reducing initial bundle size and improving startup time.

## Common Workflow Patterns

### Rendering an Object View

1. User navigates to an object (e.g., clicks in tree)
2. App requests views for object type
3. View registry evaluates all registered providers
4. Providers return priority if they can display object
5. Highest priority provider is selected
6. Vue component is created and mounted
7. Component subscribes to object changes
8. View updates reactively as object changes

### Switching Between Views

1. User clicks view switcher (plot → table)
2. Current view is unmounted and cleaned up
3. New view provider is selected
4. New component is created and mounted
4. Component initializes with same object
5. Subscriptions are re-established

### Multi-View Layouts

1. Display layout object references multiple child objects
2. For each child, appropriate view is selected
3. Views are mounted in layout's grid cells
4. All views share same time context
5. Views update independently but synchronize time

## Error Scenarios and Recovery

### No Matching Providers

- **Unknown Object Type**: No provider can display object
  - Recovery: Display object properties as JSON, suggest type installation

- **Disabled Plugins**: Required view provider plugin not installed
  - Recovery: Show placeholder, link to plugin documentation

### Component Errors

- **Mount Failure**: View component throws during creation
  - Recovery: Catch error, display error message, log to console

- **Runtime Error**: Component fails after mounting
  - Recovery: Error boundary catches, shows fallback UI, allows recovery

- **Memory Leak**: Component doesn't clean up subscriptions
  - Recovery: Force unmount, warning in console, monitor memory

### Performance Issues

- **Slow Rendering**: Complex view takes too long to render
  - Recovery: Show loading spinner, timeout and abort, suggest simpler view

- **Too Many Views**: Layout has excessive number of views
  - Recovery: Lazy render off-screen views, virtual scrolling, limit count

### Data Issues

- **Missing Data**: Object has no telemetry data
  - Recovery: Display "no data" message, suggest time range adjustment

- **Malformed Data**: Data doesn't match expected format
  - Recovery: Validate and sanitize, display warning, use default values

## View Provider Priority Guidelines

Priority values by use case:

- **100+**: Plugin-specific overrides (user wants custom view for type)
- **50-99**: Specialized views (imagery for image objects, plot for telemetry)
- **10-49**: General-purpose views (table can show anything with telemetry)
- **1-9**: Fallback views (object properties, JSON inspector)
- **0**: Disabled (provider should not be selected)

## Best Practices

### View Implementation

- Clean up subscriptions in unmount hooks
- Handle missing/malformed data gracefully
- Show loading states for async operations
- Use Vue reactivity for automatic updates
- Implement error boundaries

### Performance

- Virtualize long lists and tables
- Throttle/debounce high-frequency updates
- Lazy load large datasets
- Use web workers for heavy computation
- Profile render performance

### User Experience

- Preserve view state across navigation
- Remember user's view preferences
- Provide keyboard shortcuts
- Support responsive layouts
- Indicate when data is stale
