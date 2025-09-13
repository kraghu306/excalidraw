# Excalidraw Codebase Flow Documentation

## User Interaction Flow

This document describes the complete flow from user interaction to canvas update.

## 1. Drawing a Rectangle - Complete Flow

### Step 1: Tool Selection

```typescript
// User clicks rectangle tool
// packages/excalidraw/components/Toolbar.tsx
<ToolButton onClick={() => app.setActiveTool({ type: "rectangle" })} />;

// packages/excalidraw/components/App.tsx:2431
setActiveTool = (tool) => {
  this.setState({ activeTool: tool });
};
```

### Step 2: Pointer Down Event

```typescript
// packages/excalidraw/components/App.tsx:7456
handleCanvasPointerDown = (event) => {
  // 1. Convert coordinates
  const scenePointerCoords = viewportCoordsToSceneCoords(event);

  // 2. Check if creating new element
  if (activeTool.type === "rectangle") {
    // 3. Create new element
    const newElement = newElement({
      type: "rectangle",
      x: scenePointerCoords.x,
      y: scenePointerCoords.y,
      width: 0,
      height: 0,
      // ... style properties from appState
    });

    // 4. Update state
    this.setState({
      newElement,
      draggingElement: newElement,
    });

    // 5. Add to scene
    this.scene.insertElement(newElement);
  }
};
```

### Step 3: Pointer Move (Dragging)

```typescript
// packages/excalidraw/components/App.tsx:8234
handleCanvasPointerMove = (event) => {
  if (this.state.draggingElement) {
    // 1. Calculate new dimensions
    const newCoords = viewportCoordsToSceneCoords(event);
    const width = newCoords.x - draggingElement.x;
    const height = newCoords.y - draggingElement.y;

    // 2. Update element
    mutateElement(draggingElement, {
      width,
      height,
    });

    // 3. Trigger re-render
    this.triggerRender();
  }
};
```

### Step 4: Pointer Up (Finish Drawing)

```typescript
// packages/excalidraw/components/App.tsx:9012
handleCanvasPointerUp = (event) => {
  if (this.state.draggingElement) {
    // 1. Finalize element
    const finalElement = this.state.draggingElement;

    // 2. Add to history
    this.history.resumeRecording();

    // 3. Clear dragging state
    this.setState({
      draggingElement: null,
      newElement: null,
    });

    // 4. Switch to selection tool if not locked
    if (!activeTool.locked) {
      this.setActiveTool({ type: "selection" });
    }
  }
};
```

### Step 5: Rendering

```typescript
// packages/excalidraw/renderer/renderScene.ts
export const renderScene = (
  canvas: HTMLCanvasElement,
  rc: RoughCanvas,
  appState: AppState,
  elements: readonly NonDeletedExcalidrawElement[],
) => {
  // 1. Clear canvas
  context.clearRect(0, 0, canvas.width, canvas.height);

  // 2. Apply viewport transform
  context.save();
  context.scale(appState.zoom.value, appState.zoom.value);
  context.translate(-appState.scrollX, -appState.scrollY);

  // 3. Render each visible element
  elements.forEach((element) => {
    if (isElementInViewport(element, viewport)) {
      renderElement(element, rc, context, appState);
    }
  });

  context.restore();
};

// packages/excalidraw/renderer/renderElement.ts
const renderElement = (element, rc, context) => {
  switch (element.type) {
    case "rectangle":
      // Generate or get cached shape
      const shape =
        ShapeCache.get(element) ||
        rc.generator.rectangle(0, 0, element.width, element.height, {
          roughness: element.roughness,
        });

      // Draw shape
      rc.draw(shape);
      break;
  }
};
```

## 2. Text Editing Flow

### Step 1: Create or Edit Text

```typescript
// Double-click to edit or select text tool
// packages/excalidraw/components/App.tsx:7623
handleCanvasDoubleClick = (event) => {
  const hitElement = getElementAtPosition(coords);

  if (hitElement?.type === "text") {
    // Start editing existing text
    this.setState({
      editingTextElement: hitElement,
    });

    // Show text editor
    this.showTextEditor(hitElement);
  }
};
```

### Step 2: Text Editor Component

```typescript
// packages/excalidraw/components/TextEditor.tsx
const TextEditor = ({ element, onChange, onSubmit }) => {
  // Render contenteditable div
  return (
    <div
      contentEditable
      onInput={(e) => {
        // Update element text
        mutateElement(element, {
          text: e.currentTarget.textContent,
        });

        // Recalculate dimensions
        const dimensions = measureText(
          element.text,
          element.fontSize,
          element.fontFamily,
        );

        mutateElement(element, {
          width: dimensions.width,
          height: dimensions.height,
        });
      }}
      onBlur={onSubmit}
    />
  );
};
```

## 3. Selection and Transformation Flow

### Step 1: Element Selection

```typescript
// packages/excalidraw/components/App.tsx:7512
handleSelection = (event) => {
  const hitElement = getElementAtPosition(coords);

  if (hitElement) {
    // Single selection
    this.setState({
      selectedElementIds: { [hitElement.id]: true },
    });
  } else {
    // Start selection box
    this.setState({
      selectionElement: {
        type: "selection",
        x: coords.x,
        y: coords.y,
        width: 0,
        height: 0,
      },
    });
  }
};
```

### Step 2: Show Transform Handles

```typescript
// packages/excalidraw/renderer/renderInteractiveScene.ts
const renderTransformHandles = (element, context, appState) => {
  const handles = getTransformHandles(element, appState.zoom);

  handles.forEach((handle) => {
    // Draw resize handles
    context.fillRect(
      handle.x - HANDLE_SIZE / 2,
      handle.y - HANDLE_SIZE / 2,
      HANDLE_SIZE,
      HANDLE_SIZE,
    );
  });

  // Draw rotation handle
  if (shouldShowRotationHandle(element)) {
    renderRotationHandle(element, context);
  }
};
```

### Step 3: Transform Element

```typescript
// packages/excalidraw/components/App.tsx:8345
handleElementTransform = (event) => {
  const handle = this.state.resizingElement?.handle;

  if (handle) {
    // Calculate new dimensions
    const newBounds = calculateNewBounds(
      element,
      handle,
      pointerCoords,
      shouldMaintainAspectRatio(event),
    );

    // Update element
    mutateElement(element, newBounds);

    // Update bound elements
    updateBoundElements(element);
  }
};
```

## 4. Collaboration Flow

### Step 1: Local Change

```typescript
// packages/excalidraw/components/App.tsx
// Any element mutation triggers this
mutateElement(element, updates) {
  // 1. Update element
  element = { ...element, ...updates };

  // 2. Increment version
  element.version++;
  element.versionNonce = randomInteger();

  // 3. Broadcast if collaborating
  if (this.props.isCollaborating) {
    this.broadcastElements([element]);
  }
}
```

### Step 2: Broadcast Changes

```typescript
// excalidraw-app/collab/Collab.tsx:423
broadcastElements = (elements) => {
  const message = {
    type: "SCENE_UPDATE",
    elements: elements.map((el) => ({
      ...el,
      // Include version info for conflict resolution
      version: el.version,
      versionNonce: el.versionNonce,
    })),
  };

  this.portal.socket.emit("message", message);
};
```

### Step 3: Receive Remote Changes

```typescript
// excalidraw-app/collab/Collab.tsx:567
handleRemoteSceneUpdate = (remoteElements) => {
  // 1. Reconcile with local elements
  const reconciledElements = reconcileElements(
    this.getSceneElements(),
    remoteElements,
    this.excalidrawAPI.getAppState(),
  );

  // 2. Update scene
  this.excalidrawAPI.updateScene({
    elements: reconciledElements,
  });
};
```

### Step 4: Conflict Resolution

```typescript
// packages/excalidraw/data/reconcile.ts
const reconcileElements = (local, remote) => {
  const resolved = new Map();

  // Process each element
  [...local, ...remote].forEach((element) => {
    const existing = resolved.get(element.id);

    if (!existing || shouldUseRemoteElement(existing, element)) {
      resolved.set(element.id, element);
    }
  });

  return Array.from(resolved.values());
};

const shouldUseRemoteElement = (local, remote) => {
  // Higher version wins
  if (remote.version > local.version) return true;
  if (remote.version < local.version) return false;

  // Same version: higher versionNonce wins
  return remote.versionNonce > local.versionNonce;
};
```

## 5. Export Flow

### Step 1: Trigger Export

```typescript
// packages/excalidraw/actions/actionExport.tsx
export const actionExportImage = {
  perform: async (elements, appState) => {
    // 1. Prepare export config
    const config = {
      elements,
      appState,
      files: this.files,
      exportPadding: 10,
      maxWidthOrHeight: DEFAULT_MAX_IMAGE_WIDTH_OR_HEIGHT,
    };

    // 2. Export based on type
    const blob = await exportToBlob(config);

    // 3. Download or copy
    await fileSave(blob, {
      fileName: `excalidraw-${Date.now()}.png`,
    });
  },
};
```

### Step 2: Canvas Export

```typescript
// packages/excalidraw/data/blob.ts
export const exportToCanvas = async ({
  elements,
  appState,
  files,
  exportPadding,
}) => {
  // 1. Calculate bounds
  const bounds = getCommonBounds(elements);

  // 2. Create export canvas
  const canvas = document.createElement("canvas");
  canvas.width = bounds.width + exportPadding * 2;
  canvas.height = bounds.height + exportPadding * 2;

  // 3. Render to canvas
  renderStaticScene({
    canvas,
    elements,
    appState: {
      ...appState,
      // Center view on elements
      scrollX: bounds.minX - exportPadding,
      scrollY: bounds.minY - exportPadding,
      zoom: { value: 1 },
    },
    renderConfig: {
      canvasBackgroundColor: appState.exportBackground
        ? appState.viewBackgroundColor
        : "transparent",
    },
  });

  return canvas;
};
```

## 6. History (Undo/Redo) Flow

### Step 1: Record Changes

```typescript
// packages/excalidraw/history.ts
class History {
  record(elements: Element[], appState: AppState) {
    // 1. Create snapshot
    const snapshot = {
      elements: deepCopyElements(elements),
      appState: cleanAppStateForExport(appState),
    };

    // 2. Add to stack
    this.stack = this.stack.slice(0, this.pointer + 1);
    this.stack.push(snapshot);
    this.pointer++;

    // 3. Limit stack size
    if (this.stack.length > MAX_STACK_SIZE) {
      this.stack.shift();
      this.pointer--;
    }
  }
}
```

### Step 2: Undo Operation

```typescript
// packages/excalidraw/actions/actionHistory.tsx
export const actionUndo = {
  perform: (elements, appState, _, app) => {
    const previousState = app.history.undo();

    if (previousState) {
      return {
        elements: previousState.elements,
        appState: {
          ...appState,
          ...previousState.appState,
        },
        commitToHistory: false,
      };
    }
  },
};
```

## 7. Library Flow

### Step 1: Add to Library

```typescript
// packages/excalidraw/actions/actionAddToLibrary.ts
export const actionAddToLibrary = {
  perform: async (elements, appState) => {
    // 1. Get selected elements
    const selectedElements = getSelectedElements(elements, appState);

    // 2. Create library item
    const libraryItem = {
      id: randomId(),
      status: "unpublished",
      elements: deepCopyElements(selectedElements),
      created: Date.now(),
    };

    // 3. Save to library
    await library.addLibraryItem(libraryItem);

    // 4. Update UI
    return {
      appState: {
        ...appState,
        showLibrary: true,
      },
    };
  },
};
```

## Key Patterns

### 1. Immutable Updates

```typescript
// Never mutate directly
// ❌ Wrong
element.x = 100;

// ✅ Correct
mutateElement(element, { x: 100 });
```

### 2. Batched Updates

```typescript
// Group related updates
this.setState((prevState) => ({
  selectedElementIds: newSelection,
  selectedGroupIds: newGroups,
  editingGroupId: null,
}));
```

### 3. Performance Optimization

```typescript
// Use RAF for smooth updates
this.rafID = requestAnimationFrame(() => {
  this.renderScene();
});

// Throttle expensive operations
this.handlePointerMove = throttle(this.handlePointerMoveImpl, THROTTLE_DELAY);
```

### 4. Coordinate Systems

```typescript
// Always convert between coordinate systems
const sceneCoords = viewportCoordsToSceneCoords(event, appState);
const viewportCoords = sceneCoordsToViewportCoords(point, appState);
```

## File Reference Guide

### Core Files

- `packages/excalidraw/index.tsx` - Main export
- `packages/excalidraw/components/App.tsx` - Core app logic
- `packages/excalidraw/types.ts` - Type definitions
- `packages/excalidraw/constants.ts` - App constants

### Element System

- `packages/element/src/types.ts` - Element types
- `packages/element/src/newElement.ts` - Element creation
- `packages/element/src/mutateElement.ts` - Element updates
- `packages/element/src/bounds.ts` - Bounding box calculations

### Rendering

- `packages/excalidraw/renderer/renderScene.ts` - Main render
- `packages/excalidraw/renderer/renderElement.ts` - Element rendering
- `packages/excalidraw/scene/Scene.ts` - Scene management

### Actions

- `packages/excalidraw/actions/` - All user actions
- `packages/excalidraw/actions/manager.tsx` - Action manager
- `packages/excalidraw/actions/register.ts` - Action registry

### Collaboration

- `excalidraw-app/collab/` - Collaboration implementation
- `packages/excalidraw/data/reconcile.ts` - Conflict resolution

### State Management

- `packages/excalidraw/appState.ts` - State utilities
- `packages/excalidraw/history.ts` - Undo/redo
- `packages/excalidraw/editor-jotai.ts` - Jotai atoms
