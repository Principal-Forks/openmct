# OpenMCT Architecture

## Overview

Open MCT (Open Mission Control Technologies) is NASA's next-generation mission control framework. It's a web-based platform for building mission control applications for spacecraft, satellites, and other complex systems.

## What Problem Does It Solve?

OpenMCT addresses the challenge of building flexible, extensible mission control applications. Traditional mission control systems are monolithic and difficult to customize. OpenMCT provides:

- **Flexibility**: Plugin-based architecture allows teams to add custom features
- **Reusability**: Core framework can be adapted for different missions
- **Modern UX**: Web-based interface with real-time data visualization
- **Extensibility**: Every aspect can be extended through well-defined APIs

## Core Architecture Patterns

### 1. Plugin-First Design

Nearly all functionality in OpenMCT is implemented as plugins. The core MCT class provides minimal features - just the plugin system and core APIs. Features like plots, tables, layouts, and telemetry processing are all separate plugins.

**Why This Design?**
- Teams can enable only the features they need
- Custom plugins can be developed without modifying core code
- Plugins can be shared across different OpenMCT deployments
- Easier testing and maintenance

### 2. Registry Pattern

OpenMCT uses registries to manage extensibility:
- **ViewRegistry**: Manages visualizations for domain objects
- **TypeRegistry**: Defines domain object types
- **ToolbarRegistry**: Provides toolbar actions
- **InspectorRegistry**: Controls inspector panel views

Plugins register providers with these registries, and OpenMCT queries them at runtime to determine what views, types, and actions are available.

### 3. API-Driven Extension

The MCT core exposes 18+ APIs that plugins use to extend functionality:
- **Object API**: Load, save, and search domain objects
- **Telemetry API**: Request and format telemetry data
- **Composition API**: Define object hierarchies
- **Time API**: Manage time bounds and clocks
- **Actions API**: Register context menu actions

## Key Operations

### Plugin Installation
```javascript
openmct.install(customPlugin(options));
```

Plugins are functions that receive the openmct instance and register providers, types, views, etc.

### Domain Object Operations
- Create: `openmct.objects.save()`
- Read: `openmct.objects.get(identifier)`
- Update: `openmct.objects.save()`
- Delete: `openmct.objects.delete(identifier)`
- Search: Query composition providers

### Telemetry Workflow
1. Plugin registers telemetry provider with `openmct.telemetry.addProvider()`
2. View requests telemetry via `openmct.telemetry.request()`
3. Provider returns data based on time bounds
4. View subscribes to real-time updates via `openmct.telemetry.subscribe()`

### View Rendering
1. User navigates to domain object
2. OpenMCT queries ViewRegistry for matching providers
3. Provider with highest priority is selected
4. Provider's `view()` method creates Vue component
5. Component is mounted in the object frame

## Design Choices

### Why Vue 3?
Vue provides reactive components and composables that simplify state management and UI updates, especially important for real-time telemetry display.

### Why EventEmitter Pattern?
Core systems (MCT, Selection, Router, APIs) extend EventEmitter3 to enable loose coupling. Components can react to changes without tight dependencies.

### Why Separate Registries?
Instead of one monolithic registry, specialized registries (View, Type, Toolbar) provide type safety and clearer separation of concerns.

### Why Provider Pattern?
Providers with `appliesTo()` and priority allow multiple plugins to compete for rendering the same object type, enabling customization and override behavior.

## Common Workflow Patterns

### Adding a New Visualization
1. Create plugin function
2. Register view provider with ViewRegistry
3. Implement `canView(domainObject)` to match object types
4. Implement `view(domainObject)` to return Vue component
5. Component uses Telemetry API to fetch/subscribe to data
6. Install plugin with `openmct.install()`

### Creating a New Object Type
1. Register type with `openmct.types.addType()`
2. Define type metadata (name, description, icon)
3. Register composition provider if object has children
4. Register view provider to visualize the type
5. Add creation action to create button

### Implementing Persistence
1. Create plugin implementing persistence interface
2. Register with `openmct.objects.addProvider()`
3. Implement `get()`, `create()`, `update()`, `delete()` methods
4. Methods can talk to CouchDB, REST APIs, localStorage, etc.

## Error Scenarios & Recovery

### Telemetry Provider Failure
- Telemetry API catches errors from providers
- Falls back to next provider in chain
- Emits error notification to user

### Object Loading Failure
- Object API tries all registered providers
- Returns error if none succeed
- UI shows "not found" message
- Navigation falls back to root

### Plugin Installation Error
- MCT catches errors during plugin install
- Logs error to console
- Continues loading other plugins
- Failed plugin features are unavailable but app still functions

### Time Conductor Issues
- Invalid time bounds are rejected
- Falls back to default bounds
- Clock failures fall back to local system time

## Key Components

### MCT Core
Central hub that:
- Initializes all APIs
- Creates all registries
- Installs default plugins (20+)
- Bootstraps Vue application
- Manages application lifecycle

### API Layer
18+ modules providing extension points:
- Object management
- Telemetry data
- Time management
- User actions
- Forms and validation
- Notifications
- Status tracking

### Plugin System
70+ built-in plugins including:
- **Visualizations**: Plot, Table, Imagery, Gauge
- **Layouts**: DisplayLayout, FlexibleLayout, Tabs
- **Telemetry**: TimeConductor, Correlation, Mean
- **Actions**: Export, Import, Move, Remove
- **Persistence**: CouchDB, LocalStorage
- **Themes**: Darkmatter, Espresso, Snow

### UI Layer
Vue 3 components and layout:
- App shell with browse bar
- Object tree navigation
- Router for URL-based navigation
- Inspector panel
- Toolbar system

## External Dependencies

- **Vue.js 3**: Component framework and reactivity
- **D3.js**: Data visualization (plots, charts)
- **EventEmitter3**: Event system
- **Moment.js**: Time/date handling
- **Lodash**: Utility functions
- **PouchDB/Nano**: Optional CouchDB integration

## Summary

OpenMCT's architecture prioritizes extensibility and flexibility through a plugin-first design, registry pattern, and comprehensive APIs. This enables teams to build custom mission control applications by composing plugins and implementing providers, without modifying the core framework.
