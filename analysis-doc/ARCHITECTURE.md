# Excalidraw Architecture Documentation

## Table of Contents

1. [Project Structure](#project-structure)
2. [Core Packages](#core-packages)
3. [Initialization Flow](#initialization-flow)
4. [Element System](#element-system)
5. [Rendering Pipeline](#rendering-pipeline)
6. [State Management](#state-management)
7. [Action System](#action-system)
8. [Collaboration Architecture](#collaboration-architecture)
9. [Key Components](#key-components)
10. [Development Guidelines](#development-guidelines)

## Project Structure

Excalidraw is organized as a monorepo using Yarn workspaces with the following structure:

```
excalidraw/
├── packages/                    # Core library packages
│   ├── excalidraw/             # Main React component library (@excalidraw/excalidraw)
│   ├── common/                 # Shared utilities and constants
│   ├── element/                # Element types and operations
│   ├── math/                   # Mathematical utilities for geometry
│   └── utils/                  # General utility functions
├── excalidraw-app/             # Web application (excalidraw.com)
├── examples/                   # Integration examples
├── dev-docs/                   # Development documentation
└── scripts/                    # Build and utility scripts
```

## Core Packages

### @excalidraw/excalidraw

The main React component library that provides the Excalidraw editor as a reusable component.

**Key Files:**

- `index.tsx` - Main export and component wrapper
- `components/App.tsx` - Core application component (class-based)
- `types.ts` - TypeScript definitions for all public APIs
- `actions/` - Action system for handling user operations
- `components/` - UI components
- `scene/` - Scene management and rendering
- `renderer/` - Canvas rendering logic

### @excalidraw/common

Shared utilities, constants, and helper functions used across packages.

**Key Exports:**

- Constants (colors, keycodes, defaults)
- Utility functions (throttle, debounce, etc.)
- Type utilities
- Browser detection utilities

### @excalidraw/element

Core element system defining all drawable shapes and their operations.

**Key Components:**

- Element type definitions
- Element creation factories
- Element transformation utilities
- Bounding box calculations
- Element rendering logic

### @excalidraw/math

Mathematical utilities for geometry calculations.

**Key Functions:**

- Vector operations
- Point transformations
- Distance calculations
- Angle computations
- Collision detection

### @excalidraw/utils

Additional utility functions and helpers.

## Initialization Flow

### 1. Component Mounting

```typescript
// Entry point: packages/excalidraw/index.tsx
ExcalidrawBase
  └── EditorJotaiProvider (Jotai state management)
      └── InitializeApp (Polyfills, i18n, theme)
          └── App (Main application component)
```

### 2. App Initialization (packages/excalidraw/components/App.tsx:571)

The App component handles:

- Canvas setup
- Event listener registration
- State initialization
- Action manager setup
- Rendering loop initialization

### 3. State Setup

- Initial app state creation
- Element store initialization
- History system setup
- Collaboration setup (if enabled)

## Element System

### Element Type Hierarchy

```typescript
interface ExcalidrawElement {
  id: string;
  type: ElementType;
  x;
  y: number;
  width;
  height: number;
  angle: number;
  strokeColor: string;
  backgroundColor: string;
  fillStyle: FillStyle;
  strokeWidth: number;
  strokeStyle: StrokeStyle;
  roughness: number;
  opacity: number;
  version: number;
  versionNonce: number;
  isDeleted: boolean;
  groupIds: GroupId[];
  frameId: string | null;
  boundElements: BoundElement[];
  link: string | null;
  locked: boolean;
  // ... additional properties
}
```

### Element Types

- **Shapes**: rectangle, diamond, ellipse
- **Lines**: arrow, line
- **Drawing**: freedraw
- **Text**: text elements with font support
- **Media**: image, embeddable (iframe)
- **Containers**: frame, magicframe

### Element Creation Flow

1. Tool selection triggers `newElement()` factory
2. Element initialized with default properties
3. Element added to scene during pointer interaction
4. Version and index assigned for collaboration

## Rendering Pipeline

### Multi-Canvas Architecture

Excalidraw uses multiple canvas layers for performance:

```
┌─────────────────────────────┐
│     Interactive Canvas      │ <- UI overlays, selections
├─────────────────────────────┤
│      Static Canvas          │ <- Main element rendering
├─────────────────────────────┤
│     Background Pattern      │ <- Grid/dots pattern
└─────────────────────────────┘
```

### Rendering Process

1. **Scene Calculation**

   - Viewport determination
   - Element visibility culling
   - Z-index ordering

2. **Element Rendering** (packages/excalidraw/renderer/renderElement.ts)

   - Shape generation (RoughJS for hand-drawn look)
   - Canvas caching for complex elements
   - Text rendering with font handling
   - Image loading and scaling

3. **Performance Optimizations**
   - Element-level canvas caching
   - Dirty region tracking
   - RAF-throttled updates
   - Viewport-based culling

### Render Loop

```typescript
// Main render cycle
renderScene() {
  // Static canvas - elements
  renderStaticScene(canvas, {
    elements: visibleElements,
    appState,
    scale,
    renderConfig
  });

  // Interactive canvas - UI
  renderInteractiveScene(canvas, {
    appState,
    elements,
    selectionElement,
    handles,
    cursors
  });
}
```

## State Management

### Jotai-Based Architecture

Excalidraw uses Jotai for reactive state management:

```typescript
// Core stores
editorJotaiStore; // Editor-specific state
appJotaiStore; // Application-level state
```

### AppState Structure

```typescript
interface AppState {
  // Canvas state
  zoom: Zoom;
  scrollX: number;
  scrollY: number;
  width: number;
  height: number;

  // Tool state
  activeTool: ActiveTool;
  currentItemStrokeColor: string;
  currentItemBackgroundColor: string;
  currentItemFillStyle: FillStyle;
  // ... other style properties

  // Selection state
  selectedElementIds: { [id: string]: true };
  selectedGroupIds: { [id: string]: true };
  editingTextElement: ExcalidrawElement | null;

  // UI state
  openMenu: "canvas" | "shape" | null;
  showWelcomeScreen: boolean;
  zenModeEnabled: boolean;
  gridModeEnabled: boolean;

  // Collaboration
  collaborators: Map<string, Collaborator>;

  // ... many more properties
}
```

### State Update Flow

1. User action triggers event handler
2. Action dispatched through ActionManager
3. Action performs state calculations
4. New state returned and applied
5. React re-render triggered
6. Canvas updated if needed

## Action System

### Action Architecture

Actions are command objects that encapsulate operations:

```typescript
interface Action {
  name: string;
  perform: (elements, appState, value, app) => ActionResult;
  keyTest?: (event: KeyboardEvent) => boolean;
  contextItemLabel?: string;
  trackEvent?: TrackEventConfig;
}
```

### Key Actions

**Element Operations:**

- `actionDeleteSelected` - Delete selected elements
- `actionDuplicateSelection` - Duplicate elements
- `actionGroup` - Group elements
- `actionBringToFront` - Z-index manipulation

**Canvas Operations:**

- `actionZoomIn/Out` - Zoom control
- `actionResetZoom` - Reset viewport
- `actionToggleGridMode` - Toggle grid

**Tool Actions:**

- `actionChangeTool` - Switch active tool
- `actionChangeStroke` - Modify stroke properties
- `actionChangeBackground` - Modify fill

### Action Registration

Actions are registered in `packages/excalidraw/actions/register.ts`:

```typescript
export const actions = {
  deleteSelected: actionDeleteSelected,
  duplicate: actionDuplicateSelection,
  // ... all other actions
};
```

## Collaboration Architecture

### WebSocket Protocol

Real-time collaboration uses WebSocket for synchronization:

1. **Connection Management**

   - Auto-reconnection logic
   - Heartbeat/keepalive
   - Offline detection

2. **Message Types**

   ```typescript
   type CollabMessage =
     | { type: "SCENE_UPDATE"; elements: Element[] }
     | { type: "POINTER_UPDATE"; pointer: Pointer }
     | { type: "USER_VISIBLE_SCENE_BOUNDS"; bounds: Bounds };
   ```

3. **Conflict Resolution**
   - Version vectors (version + versionNonce)
   - Last-write-wins with version comparison
   - Fractional indexing for ordering

### Synchronization Flow

```
Local Change → Broadcast → Remote Receive → Reconcile → Update
     ↓                           ↓
  Optimistic              Version Check
   Update                & Merge Logic
```

### Data Persistence

- **Firebase Backend**: Scene and file storage
- **Encryption**: Client-side encryption for privacy
- **Local Storage**: Offline fallback

## Key Components

### Main Components

**App.tsx (packages/excalidraw/components/App.tsx)**

- Core application component
- Handles all user interactions
- Manages rendering loop
- Coordinates between subsystems

**Canvas Components**

- `InteractiveCanvas.tsx` - Handles user interactions
- `StaticCanvas.tsx` - Renders elements
- `LayerUI.tsx` - UI overlay components

**UI Components**

- `Toolbar.tsx` - Tool selection
- `Properties.tsx` - Element properties panel
- `Menu.tsx` - Application menu
- `Dialog.tsx` - Modal dialogs

### Component Hierarchy

```
App
├── LayerUI
│   ├── Toolbar
│   ├── Properties Panel
│   ├── Menu
│   └── Dialogs
├── Canvas Container
│   ├── Static Canvas
│   ├── Interactive Canvas
│   └── Background Pattern
└── Footer
    └── Status Bar
```

## Development Guidelines

### Code Organization

- **Feature-based structure**: Group related code together
- **Type safety**: Use TypeScript strictly
- **Component isolation**: Keep components focused
- **State management**: Use Jotai atoms for shared state

### Performance Considerations

1. **Memoization**: Use React.memo for expensive components
2. **Canvas caching**: Cache rendered elements
3. **Throttling**: Throttle expensive operations
4. **Lazy loading**: Load features on demand

### Testing Strategy

- **Unit tests**: Test utilities and pure functions
- **Integration tests**: Test component interactions
- **Visual regression**: Test rendering output
- **E2E tests**: Test critical user flows

### Adding New Features

1. **New Element Type**

   - Add type definition in `element/types.ts`
   - Create factory in `element/newElement.ts`
   - Add rendering logic in `renderer/renderElement.ts`
   - Register tool in action system

2. **New Action**

   - Create action file in `actions/`
   - Implement perform function
   - Register in `actions/register.ts`
   - Add keyboard shortcut if needed

3. **New UI Component**
   - Create component in `components/`
   - Add to LayerUI if needed
   - Connect to app state via props or Jotai

### Build System

- **Vite**: Development and production builds
- **ESBuild**: Package compilation
- **TypeScript**: Type checking
- **Vitest**: Testing framework

### Commands

```bash
# Development
yarn start              # Start dev server
yarn test              # Run tests
yarn test:typecheck    # TypeScript checking
yarn fix               # Auto-fix linting

# Building
yarn build             # Build all packages
yarn build:packages    # Build library packages
yarn build:app        # Build web application
```

## Architecture Decisions

### Why Class Component for App?

The main App component uses a class component for:

- Complex lifecycle management
- Performance optimizations
- Historical reasons (migration in progress)

### Why Multiple Canvas Layers?

- Performance: Avoid redrawing static elements
- Separation of concerns: UI vs content
- Smooth interactions: UI updates don't block rendering

### Why Jotai for State?

- Fine-grained reactivity
- Better performance than Context
- Cleaner than prop drilling
- Good TypeScript support

### Why RoughJS?

- Unique hand-drawn aesthetic
- Consistent cross-browser rendering
- Good performance with caching
- Customizable appearance

## Future Considerations

- Migration to functional components
- WebAssembly for performance-critical paths
- Plugin system for extensibility
- Enhanced mobile experience
- Improved accessibility features
