# Excalidraw API Reference Guide

## Table of Contents

1. [Overview: The API Ecosystem](#overview-the-api-ecosystem)
2. [HTTP/REST APIs (Postman-testable)](#httprest-apis-postman-testable)
3. [Core APIs](#core-apis)
4. [Element APIs](#element-apis)
5. [Scene APIs](#scene-apis)
6. [Action APIs](#action-apis)
7. [Rendering APIs](#rendering-apis)
8. [Collaboration APIs](#collaboration-apis)
9. [Export & Import APIs](#export--import-apis)
10. [Utility APIs](#utility-apis)
11. [Hook APIs](#hook-apis)
12. [Internal Communication Patterns](#internal-communication-patterns)

---

## Overview: The API Ecosystem

Excalidraw's API system is designed as a layered architecture where each layer provides specific capabilities to the layers above it. The APIs can be categorized into:

1. **Public APIs**: Exposed to consumers of the library
2. **Internal APIs**: Used within the codebase for component communication
3. **Utility APIs**: Helper functions available throughout the system
4. **Hook APIs**: React hooks for state and lifecycle management

```
API Layer Architecture:

┌─────────────────────────────────────┐
│         Public API Surface          │
│  (What library consumers use)       │
├─────────────────────────────────────┤
│         Component APIs              │
│  (Inter-component communication)    │
├─────────────────────────────────────┤
│         Core System APIs            │
│  (Element, Scene, Action systems)   │
├─────────────────────────────────────┤
│         Utility APIs                │
│  (Math, transforms, helpers)        │
└─────────────────────────────────────┘
```

---

## HTTP/REST APIs (Postman-testable)

**Yes, Excalidraw does have HTTP APIs that you can test with Postman!** These are the REST endpoints used by the web application for sharing, collaboration, and AI features.

### Overview of HTTP Endpoints

Excalidraw uses several HTTP APIs for different purposes:

```
HTTP API Architecture:

┌─────────────────────────────┐
│    Excalidraw Frontend      │
├─────────────────────────────┤
│                             │
├──► Scene Sharing API        │ ──► Backend V2 (GET/POST)
├──► File Storage API         │ ──► Firebase Storage
├──► AI Integration API       │ ──► AI Backend
└──► Collaboration WebSocket  │ ──► Socket.io Server
```

### 1. Scene Sharing API

**Base URLs**: Configured via environment variables

- GET: `VITE_APP_BACKEND_V2_GET_URL`
- POST: `VITE_APP_BACKEND_V2_POST_URL`

#### **POST** - Create Shareable Scene Link

**Endpoint**: `${BACKEND_V2_POST}`  
**Method**: POST  
**Content-Type**: `application/octet-stream`  
**Purpose**: Create a shareable link for an Excalidraw scene

**Request Body**: Compressed and encrypted scene data (binary)

```javascript
// Example request preparation (from code)
const payload = await compressData(
  new TextEncoder().encode(
    serializeAsJSON(elements, appState, files, "database"),
  ),
  { encryptionKey },
);

fetch(BACKEND_V2_POST, {
  method: "POST",
  body: payload.buffer, // Binary data
});
```

**Response Format**:

```json
{
  "id": "abc123def456",
  "success": true
}
```

**Error Responses**:

```json
{
  "error_class": "RequestTooLargeError",
  "message": "Payload too large"
}
```

**Postman Testing**:

```
POST {{BACKEND_V2_POST}}
Content-Type: application/octet-stream
Body: [Binary scene data - see encoding section below]
```

#### **GET** - Load Shared Scene

**Endpoint**: `${BACKEND_V2_GET}{sceneId}`  
**Method**: GET  
**Purpose**: Load a shared scene by ID

**URL Pattern**:

```
GET https://backend-url/scenes/abc123def456
```

**Response**: Compressed and encrypted scene data (binary)

**Postman Testing**:

```
GET {{BACKEND_V2_GET}}abc123def456
Accept: application/octet-stream
```

### 2. Firebase Storage API

**Purpose**: File storage for images and assets in shared scenes

#### **GET** - Download File

**Endpoint Pattern**:

```
https://firebasestorage.googleapis.com/v0/b/{storageBucket}/o/{path}%2F{fileId}?alt=media
```

**Method**: GET  
**Headers**: None required  
**Response**: File binary data

**Example URLs**:

```
https://firebasestorage.googleapis.com/v0/b/excalidraw-app.appspot.com/o/files%2FshareLinks%2Fabc123%2Fimage1.png?alt=media
```

**Postman Testing**:

```
GET https://firebasestorage.googleapis.com/v0/b/excalidraw-app.appspot.com/o/files%2FshareLinks%2F{{sceneId}}%2F{{fileId}}?alt=media
```

### 3. AI Integration API

**Base URL**: `VITE_APP_AI_BACKEND`  
**Purpose**: Convert diagrams to code using AI

#### **POST** - Generate Code from Diagram

**Endpoint**: `${AI_BACKEND}/v1/ai/diagram-to-code/generate`  
**Method**: POST  
**Content-Type**: `application/json`

**Request Body**:

```json
{
  "texts": ["Button", "Login Form", "Submit"],
  "image": "data:image/jpeg;base64,/9j/4AAQSkZJRgABA...",
  "theme": "light"
}
```

**Headers**:

```
Accept: application/json
Content-Type: application/json
```

**Success Response**:

```json
{
  "html": "<div>Generated HTML code here...</div>"
}
```

**Rate Limit Response** (429):

```json
{
  "statusCode": 429,
  "message": "Too many requests"
}
```

**Postman Testing**:

```
POST {{AI_BACKEND}}/v1/ai/diagram-to-code/generate
Content-Type: application/json

{
  "texts": ["Login", "Password", "Submit"],
  "image": "data:image/jpeg;base64,{{base64_image}}",
  "theme": "light"
}
```

### 4. WebSocket Collaboration API

**Connection**: Socket.io-based real-time communication  
**Purpose**: Live collaboration and presence

#### Connection Setup

```javascript
import io from "socket.io-client";
const socket = io("wss://your-collaboration-server.com");
```

#### Message Types

**Scene Updates**:

```json
{
  "type": "SCENE_UPDATE",
  "payload": {
    "elements": [...],
    "collaborator": {
      "socketId": "abc123",
      "username": "User1"
    }
  }
}
```

**Mouse Position**:

```json
{
  "type": "MOUSE_LOCATION",
  "payload": {
    "socketId": "abc123",
    "pointer": { "x": 100, "y": 200, "tool": "pointer" },
    "button": "down",
    "username": "User1"
  }
}
```

**User Status**:

```json
{
  "type": "IDLE_STATUS",
  "payload": {
    "socketId": "abc123",
    "userState": "active",
    "username": "User1"
  }
}
```

### Environment Variables

To test these APIs, you need the following environment variables:

```bash
# Scene sharing backend
VITE_APP_BACKEND_V2_GET_URL=https://backend.excalidraw.com/api/v2/scenes/
VITE_APP_BACKEND_V2_POST_URL=https://backend.excalidraw.com/api/v2/scenes

# Firebase configuration
VITE_APP_FIREBASE_CONFIG={"apiKey":"...","authDomain":"...","storageBucket":"..."}

# AI backend
VITE_APP_AI_BACKEND=https://ai.excalidraw.com

# Excalidraw+ landing page (for rate limit messages)
VITE_APP_PLUS_LP=https://plus.excalidraw.com
```

### Data Encoding/Decoding

#### Scene Data Format

Excalidraw scenes are encoded as:

1. **JSON Serialization**: Elements + AppState → JSON string
2. **Text Encoding**: JSON → UTF-8 bytes
3. **Compression**: Uses compression algorithm
4. **Encryption**: AES encryption with generated key
5. **Transport**: Binary payload over HTTP

#### Creating Test Data

To create valid test data for Postman:

```javascript
// Example scene data
const elements = [
  {
    type: "rectangle",
    x: 100,
    y: 100,
    width: 200,
    height: 100,
    strokeColor: "#000000",
    backgroundColor: "transparent",
    // ... other properties
  },
];

const appState = {
  viewBackgroundColor: "#ffffff",
  currentItemFontFamily: 1,
  // ... other properties
};

// This would need to be compressed and encrypted
const sceneData = { elements, appState, files: {} };
```

### Postman Collection Setup

Create a Postman collection with these endpoints:

#### Variables

```json
{
  "BACKEND_V2_GET": "https://backend.excalidraw.com/api/v2/scenes/",
  "BACKEND_V2_POST": "https://backend.excalidraw.com/api/v2/scenes",
  "AI_BACKEND": "https://ai.excalidraw.com",
  "FIREBASE_BUCKET": "excalidraw-app.appspot.com"
}
```

#### Test Requests

1. **Share Scene**:

   ```
   POST {{BACKEND_V2_POST}}
   Body: [Encoded scene data]
   ```

2. **Load Scene**:

   ```
   GET {{BACKEND_V2_GET}}{{scene_id}}
   ```

3. **AI Generation**:

   ```
   POST {{AI_BACKEND}}/v1/ai/diagram-to-code/generate
   Content-Type: application/json
   Body: {"texts": [...], "image": "...", "theme": "light"}
   ```

4. **Download File**:
   ```
   GET https://firebasestorage.googleapis.com/v0/b/{{FIREBASE_BUCKET}}/o/files%2FshareLinks%2F{{scene_id}}%2F{{file_id}}?alt=media
   ```

### Authentication & Security

- **Scene API**: No authentication required, security through encryption
- **Firebase Storage**: No authentication for public files
- **AI API**: May have rate limiting, no explicit auth in code
- **Collaboration**: WebSocket authentication handled by Socket.io

### Rate Limiting

- **AI API**: Has rate limiting (429 responses)
- **Scene API**: No explicit rate limiting visible in code
- **Firebase**: Google's standard rate limits apply

### CORS Considerations

These APIs are designed to be called from the Excalidraw frontend, so CORS headers should allow browser access. For Postman testing, CORS doesn't apply.

### Testing Workflow

1. **Start with Scene Creation**:
   - POST to scene sharing API
   - Get scene ID from response
2. **Test Scene Loading**:

   - GET using the scene ID
   - Verify compressed data response

3. **Test AI Features**:

   - POST with sample diagram data
   - Check for HTML response or rate limit

4. **Test File Downloads**:
   - Use Firebase Storage URLs
   - Test with known file IDs

### Missing: Clipboard JSON Format API

⚠️ **Currently Missing**: There is **NO direct HTTP GET API** that returns the `excalidraw/clipboard` JSON format.

The existing Scene API returns compressed/encrypted binary data, not the clipboard JSON format like:

```json
{
  "type": "excalidraw/clipboard",
  "elements": [...],
  "files": {...}
}
```

**📄 Proposed Solution**: See [PROPOSED_CLIPBOARD_API.md](./PROPOSED_CLIPBOARD_API.md) for a detailed implementation plan to add this capability.

---

## Core APIs

### 1. ExcalidrawAPI Instance

The main API object that provides programmatic control over Excalidraw:

```typescript
// How to obtain the API instance
import { Excalidraw } from "@excalidraw/excalidraw";

const MyApp = () => {
  const [excalidrawAPI, setExcalidrawAPI] = useState(null);

  return (
    <Excalidraw
      ref={(api) => setExcalidrawAPI(api)}
      // or
      excalidrawAPI={(api) => setExcalidrawAPI(api)}
    />
  );
};
```

#### ExcalidrawAPI Methods

```typescript
interface ExcalidrawAPI {
  // Scene Management
  updateScene(sceneData: {
    elements?: readonly ExcalidrawElement[];
    appState?: Partial<AppState>;
    collaborators?: Map<string, Collaborator>;
  }): void;

  resetScene(): void;

  getSceneElements(): readonly ExcalidrawElement[];
  getSceneElementsIncludingDeleted(): readonly ExcalidrawElement[];

  getAppState(): AppState;

  // File Operations
  getFiles(): BinaryFiles;
  addFiles(files: BinaryFileData[]): void;

  // History
  history: {
    clear(): void;
  };

  // Scrolling
  scrollToContent(
    target?: ExcalidrawElement | readonly ExcalidrawElement[],
    opts?: {
      fitToContent?: boolean;
      animate?: boolean;
      duration?: number;
    },
  ): void;

  // Library
  updateLibrary(opts: {
    libraryItems: LibraryItems;
    prompt?: boolean;
    merge?: boolean;
    defaultStatus?: "published" | "unpublished";
  }): Promise<LibraryItems>;

  // Canvas
  refresh(): void;
  setToast(toast: { message: string; duration?: number }): void;

  // Element Operations
  setActiveTool(tool: AppState["activeTool"]): void;
  setCursor(cursor: string): void;
  resetCursor(): void;

  // Export
  exportToBlob(opts?: ExportOpts): Promise<Blob>;
  exportToSvg(opts?: ExportOpts): Promise<SVGSVGElement>;
  exportToClipboard(opts?: ExportOpts): Promise<void>;
}
```

#### Usage Examples

```typescript
// Update scene with new elements
excalidrawAPI.updateScene({
  elements: [
    {
      type: "rectangle",
      x: 100,
      y: 100,
      width: 200,
      height: 100,
      // ... other properties
    },
  ],
  appState: {
    viewBackgroundColor: "#000000",
    currentItemFontFamily: 1,
  },
});

// Scroll to specific elements
const elementsToFocus = scene.getElements().slice(0, 3);
excalidrawAPI.scrollToContent(elementsToFocus, {
  fitToContent: true,
  animate: true,
  duration: 500,
});

// Export current scene
const blob = await excalidrawAPI.exportToBlob({
  mimeType: "image/png",
  quality: 0.95,
  exportBackground: true,
  exportPadding: 20,
});
```

### 2. App Component API

The internal API of the main App component:

```typescript
class App {
  // State Management
  setState(update: Partial<AppState>): void;
  setAppState(update: Partial<AppState>): void;

  // Element Operations
  addElementsFromPasteOrLibrary(opts: {
    elements: readonly ExcalidrawElement[];
    files: BinaryFiles | null;
    position: { x: number; y: number };
  }): void;

  // Tool Management
  setActiveTool(tool: AppState["activeTool"]): void;
  setOpenDialog(dialog: AppState["openDialog"]): void;

  // Canvas Operations
  togglePenMode(force?: boolean): void;
  toggleLock(force?: boolean): void;

  // Collaboration
  setCollaborators(collaborators: Map<string, Collaborator>): void;
  broadcastElements(elements: readonly ExcalidrawElement[]): void;

  // Event Handlers (internal)
  handleCanvasPointerDown(event: React.PointerEvent): void;
  handleCanvasPointerMove(event: React.PointerEvent): void;
  handleCanvasPointerUp(event: React.PointerEvent): void;
}
```

---

## Element APIs

### 3. Element Creation APIs

Located in `packages/element/src/newElement.ts`:

```typescript
// Create new elements
export const newElement = (
  opts: {
    type: ExcalidrawElement["type"];
    x: number;
    y: number;
    width?: number;
    height?: number;
    strokeColor?: string;
    backgroundColor?: string;
    fillStyle?: FillStyle;
    strokeWidth?: number;
    strokeStyle?: StrokeStyle;
    roughness?: number;
    opacity?: number;
    // ... other properties
  }
): NonDeleted<ExcalidrawElement>;

// Specialized creators
export const newTextElement = (
  opts: {
    x: number;
    y: number;
    text: string;
    fontSize?: number;
    fontFamily?: FontFamilyValues;
    textAlign?: TextAlign;
    verticalAlign?: VerticalAlign;
    containerId?: string | null;
  }
): NonDeleted<ExcalidrawTextElement>;

export const newLinearElement = (
  opts: {
    type: "arrow" | "line";
    x: number;
    y: number;
    points?: readonly Point[];
    startArrowhead?: Arrowhead;
    endArrowhead?: Arrowhead;
  }
): NonDeleted<ExcalidrawLinearElement>;

export const newImageElement = (
  opts: {
    x: number;
    y: number;
    width: number;
    height: number;
    fileId: FileId;
    status?: "pending" | "saved" | "error";
    scale?: [number, number];
  }
): NonDeleted<ExcalidrawImageElement>;
```

#### Usage in Components

```typescript
// Creating a rectangle in a custom tool
const handlePointerDown = (event: PointerEvent) => {
  const coords = viewportCoordsToSceneCoords(event, appState);

  const element = newElement({
    type: "rectangle",
    x: coords.x,
    y: coords.y,
    width: 0,
    height: 0,
    strokeColor: appState.currentItemStrokeColor,
    backgroundColor: appState.currentItemBackgroundColor,
    fillStyle: appState.currentItemFillStyle,
    strokeWidth: appState.currentItemStrokeWidth,
    roughness: appState.currentItemRoughness,
    opacity: appState.currentItemOpacity,
  });

  scene.insertElement(element);
};
```

### 4. Element Mutation APIs

Located in `packages/element/src/mutateElement.ts`:

```typescript
// Mutate single element
export const mutateElement = <T extends ExcalidrawElement>(
  element: T,
  updates: Partial<T>,
  informMutation = true
): T;

// Batch mutations
export const batchMutateElements = (
  elements: readonly ExcalidrawElement[],
  updates: (element: ExcalidrawElement) => Partial<ExcalidrawElement>
): void;

// Create new element with updates
export const newElementWith = <T extends ExcalidrawElement>(
  element: T,
  updates: Partial<T>
): T;
```

#### Internal Usage Pattern

```typescript
// Single mutation
mutateElement(element, {
  x: 100,
  y: 200,
  width: 300,
});

// Batch mutation for performance
batchMutateElements(selectedElements, (element) => ({
  strokeColor: "#ff0000",
  opacity: 0.5,
}));

// Immutable update
const updatedElement = newElementWith(element, {
  angle: Math.PI / 4,
});
```

### 5. Element Property APIs

```typescript
// Bound elements
export const isBindingElement = (
  element: ExcalidrawElement | null
): element is ExcalidrawBindableElement;

export const isBindableElement = (
  element: ExcalidrawElement | null
): element is ExcalidrawBindableElement;

export const bindOrUnbindLinearElements = (
  linearElements: readonly NonDeleted<ExcalidrawLinearElement>[],
  otherElements: readonly NonDeleted<ExcalidrawElement>[],
  app: App
): void;

// Text elements
export const refreshTextDimensions = (
  textElement: ExcalidrawTextElement,
  container?: ExcalidrawElement | null
): void;

export const wrapText = (
  text: string,
  font: FontString,
  maxWidth: number
): string;

// Transformation
export const transformElements = (
  elements: readonly NonDeleted<ExcalidrawElement>[],
  transformHandleType: TransformHandleType,
  pointerX: number,
  pointerY: number,
  centerX?: number,
  centerY?: number
): void;

export const rotateElements = (
  elements: readonly ExcalidrawElement[],
  angle: number,
  centerX: number,
  centerY: number
): readonly ExcalidrawElement[];
```

---

## Scene APIs

### 6. Scene Management APIs

Located in `packages/excalidraw/scene/Scene.ts`:

```typescript
class Scene {
  // Element management
  getElements(): readonly NonDeleted<ExcalidrawElement>[];
  getElementsIncludingDeleted(): readonly ExcalidrawElement[];
  getNonDeletedElements(): readonly NonDeleted<ExcalidrawElement>[];

  getElement<T extends ExcalidrawElement>(id: string): T | null;

  // Element manipulation
  insertElement(element: ExcalidrawElement): void;
  insertElements(elements: readonly ExcalidrawElement[]): void;

  replaceAllElements(elements: readonly ExcalidrawElement[]): void;

  // Selection
  getSelectedElements(opts?: {
    selectedElementIds: AppState["selectedElementIds"];
    includeBoundTextElement?: boolean;
    includeElementsInFrames?: boolean;
  }): readonly NonDeleted<ExcalidrawElement>[];

  // Z-index operations
  moveToTop(elements: readonly ExcalidrawElement[]): void;
  moveToBottom(elements: readonly ExcalidrawElement[]): void;
  moveOneLayer(
    elements: readonly ExcalidrawElement[],
    direction: "up" | "down",
  ): void;
}
```

#### Scene Utilities

```typescript
// Get common bounds
export const getCommonBounds = (
  elements: readonly ExcalidrawElement[]
): [minX: number, minY: number, maxX: number, maxY: number];

// Element visibility
export const isElementInViewport = (
  element: ExcalidrawElement,
  viewportBounds: {
    minX: number;
    minY: number;
    maxX: number;
    maxY: number;
  },
  zoom: Zoom
): boolean;

// Hit testing
export const getElementAtPosition = (
  elements: readonly NonDeleted<ExcalidrawElement>[],
  x: number,
  y: number,
  zoom: Zoom
): NonDeleted<ExcalidrawElement> | null;

export const getElementsAtPosition = (
  elements: readonly NonDeleted<ExcalidrawElement>[],
  x: number,
  y: number
): NonDeleted<ExcalidrawElement>[];
```

---

## Action APIs

### 7. Action System APIs

Located in `packages/excalidraw/actions/`:

```typescript
// Action definition interface
interface Action {
  name: string;
  trackEvent?: TrackEventConfig;
  predicate?: (
    elements: readonly ExcalidrawElement[],
    appState: AppState,
    props: ExcalidrawProps,
    app: AppClassProperties,
  ) => boolean;
  perform: (
    elements: readonly ExcalidrawElement[],
    appState: AppState,
    value: any,
    app: AppClassProperties,
  ) => ActionResult;
  PanelComponent?: React.FC<{
    elements: readonly ExcalidrawElement[];
    appState: AppState;
    updateData: (data: any) => void;
  }>;
  keyTest?: (
    event: KeyboardEvent,
    appState: AppState,
    elements: readonly ExcalidrawElement[],
  ) => boolean;
}

// Action result
interface ActionResult {
  elements?: readonly ExcalidrawElement[];
  appState?: Partial<AppState>;
  files?: BinaryFiles;
  commitToHistory?: boolean;
  syncHistory?: boolean;
  replaceFiles?: boolean;
  multiElement?: NonDeleted<ExcalidrawLinearElement> | null;
}
```

#### Registering Custom Actions

```typescript
// Custom action example
const customAction: Action = {
  name: "customAction",
  perform: (elements, appState) => {
    // Custom logic
    const newElements = elements.map((el) => ({
      ...el,
      strokeColor: "#ff0000",
    }));

    return {
      elements: newElements,
      appState: {
        ...appState,
        selectedElementIds: {},
      },
      commitToHistory: true,
    };
  },
  keyTest: (event) => event.key === "k" && event.ctrlKey,
};

// Register action
actionManager.registerAction(customAction);
```

### 8. Action Manager API

```typescript
class ActionManager {
  // Action registration
  registerAction(action: Action): void;

  // Action execution
  executeAction(
    action: Action,
    source: ActionSource,
    value?: any,
  ): ActionResult | false;

  // Render action UI
  renderAction(name: string): React.ReactElement | null;

  // Check if action is enabled
  isActionEnabled(action: Action): boolean;

  // Get action by name
  getAction(name: string): Action | undefined;

  // Handle keyboard events
  handleKeyDown(event: KeyboardEvent): boolean;
}
```

---

## Rendering APIs

### 9. Canvas Rendering APIs

Located in `packages/excalidraw/renderer/`:

```typescript
// Main rendering function
export const renderScene = (
  canvas: HTMLCanvasElement | null,
  rc: RoughCanvas | null,
  {
    elements,
    appState,
    scale,
    renderConfig
  }: {
    elements: readonly NonDeleted<ExcalidrawElement>[];
    appState: StaticCanvasAppState;
    scale?: number;
    renderConfig?: RenderConfig;
  }
): void;

// Interactive layer rendering
export const renderInteractiveScene = (
  canvas: HTMLCanvasElement | null,
  context: CanvasRenderingContext2D | null,
  {
    elements,
    appState,
    selectionElement,
    scale,
    renderConfig
  }: {
    elements: readonly NonDeleted<ExcalidrawElement>[];
    appState: InteractiveCanvasAppState;
    selectionElement?: NonDeleted<ExcalidrawElement>;
    scale?: number;
    renderConfig?: RenderConfig;
  }
): void;

// Element-specific rendering
export const renderElement = (
  element: NonDeleted<ExcalidrawElement>,
  rc: RoughCanvas,
  context: CanvasRenderingContext2D,
  appState: StaticCanvasAppState,
  renderConfig?: RenderConfig
): void;
```

#### Rendering Utilities

```typescript
// Shape generation
export const generateElementShape = (
  element: NonDeleted<ExcalidrawElement>,
  generator: RoughGenerator
): Drawable | null;

// Path generation for freedraw
export const getFreeDrawPath = (
  element: ExcalidrawFreeDrawElement
): Path2D;

// Text rendering
export const renderText = (
  element: ExcalidrawTextElement,
  context: CanvasRenderingContext2D,
  appState: AppState
): void;

// Selection rendering
export const renderSelectionBorder = (
  elements: readonly NonDeleted<ExcalidrawElement>[],
  context: CanvasRenderingContext2D,
  appState: InteractiveCanvasAppState
): void;
```

---

## Collaboration APIs

### 10. Collaboration System APIs

Located in `excalidraw-app/collab/`:

```typescript
// Portal API for WebSocket communication
class Portal {
  socket: SocketIOClient.Socket | null;
  roomId: string | null;
  roomKey: string | null;

  // Connection management
  connect(roomId: string, roomKey: string): Promise<void>;
  disconnect(): void;

  // Broadcasting
  broadcastElements(elements: readonly ExcalidrawElement[]): void;
  broadcastMouseLocation(pointer: CollaboratorPointer): void;
  broadcastIdleChange(userState: UserIdleState): void;
  broadcastVisibleSceneBounds(bounds: VisibleSceneBounds): void;

  // Event handling
  on(event: string, handler: Function): void;
  off(event: string, handler: Function): void;
}

// Collaboration component API
interface CollabAPI {
  // Portal management
  portal: Portal;

  // User management
  getCollaborators(): Map<string, Collaborator>;
  setCollaborators(collaborators: Map<string, Collaborator>): void;

  // Synchronization
  syncElements(elements: readonly ExcalidrawElement[]): void;
  handleRemoteSceneUpdate(elements: readonly ExcalidrawElement[]): void;

  // Room management
  createRoom(): Promise<{ roomId: string; roomKey: string }>;
  joinRoom(roomId: string): Promise<void>;
  leaveRoom(): void;
}
```

#### Reconciliation API

```typescript
// Element reconciliation for collaboration
export const reconcileElements = (
  localElements: readonly ExcalidrawElement[],
  remoteElements: readonly ExcalidrawElement[],
  appState: AppState
): readonly ExcalidrawElement[];

// Version comparison
export const shouldUseRemoteElement = (
  localElement: ExcalidrawElement,
  remoteElement: ExcalidrawElement
): boolean;
```

---

## Export & Import APIs

### 11. Data Export APIs

Located in `packages/excalidraw/data/`:

```typescript
// Export to various formats
export const exportToBlob = async (opts: {
  elements: readonly NonDeleted<ExcalidrawElement>[];
  appState: AppState;
  files: BinaryFiles | null;
  mimeType?: "image/png" | "image/svg+xml" | "image/jpeg";
  quality?: number;
  exportPadding?: number;
  maxWidthOrHeight?: number;
}): Promise<Blob>;

export const exportToSvg = async (opts: {
  elements: readonly NonDeleted<ExcalidrawElement>[];
  appState: AppState;
  files: BinaryFiles | null;
  exportPadding?: number;
  renderEmbeddables?: boolean;
}): Promise<SVGSVGElement>;

export const exportToCanvas = async (opts: {
  elements: readonly NonDeleted<ExcalidrawElement>[];
  appState: AppState;
  files: BinaryFiles | null;
  exportPadding?: number;
  maxWidthOrHeight?: number;
}): Promise<HTMLCanvasElement>;

// JSON export
export const serializeAsJSON = (
  elements: readonly ExcalidrawElement[],
  appState: Partial<AppState>,
  files: BinaryFiles | null,
  libraryItems?: LibraryItems
): string;

export const serializeLibraryAsJSON = (
  libraryItems: LibraryItems
): string;
```

### 12. Data Import APIs

```typescript
// Import from various sources
export const loadFromBlob = async (
  blob: Blob,
  localAppState?: Partial<AppState> | null
): Promise<{
  elements: readonly ExcalidrawElement[];
  appState: Partial<AppState>;
  files: BinaryFiles;
}>;

export const loadFromJSON = async (
  json: string,
  localAppState?: Partial<AppState> | null
): Promise<{
  elements: readonly ExcalidrawElement[];
  appState: Partial<AppState>;
  files: BinaryFiles;
}>;

// Restore data
export const restore = (
  data: ImportedDataState,
  localAppState?: Partial<AppState> | null,
  localElements?: readonly ExcalidrawElement[] | null
): RestoredDataState;

// Load library
export const loadLibraryFromBlob = async (
  blob: Blob
): Promise<LibraryItems>;
```

---

## Utility APIs

### 13. Coordinate System APIs

Located in `packages/excalidraw/coords.ts`:

```typescript
// Coordinate transformations
export const viewportCoordsToSceneCoords = (
  coords: { clientX: number; clientY: number },
  appState: AppState
): { x: number; y: number };

export const sceneCoordsToViewportCoords = (
  coords: { x: number; y: number },
  appState: AppState
): { x: number; y: number };

// Grid snapping
export const getGridPoint = (
  x: number,
  y: number,
  gridSize: number | null
): [number, number];

export const snapToGrid = (
  value: number,
  gridSize: number
): number;
```

### 14. Math APIs

Located in `packages/math/`:

```typescript
// Vector operations
export const vectorFromPoint = (
  point: Point
): Vector;

export const vectorAdd = (
  a: Vector,
  b: Vector
): Vector;

export const vectorSubtract = (
  a: Vector,
  b: Vector
): Vector;

export const vectorScale = (
  v: Vector,
  scalar: number
): Vector;

export const vectorDot = (
  a: Vector,
  b: Vector
): number;

export const vectorNormalize = (
  v: Vector
): Vector;

// Point operations
export const pointFrom = (
  x: number,
  y: number
): Point;

export const pointDistance = (
  a: Point,
  b: Point
): number;

export const pointRotateRads = (
  point: Point,
  center: Point,
  angle: number
): Point;

// Line operations
export const lineLength = (
  line: Line
): number;

export const lineIntersection = (
  line1: Line,
  line2: Line
): Point | null;

// Angle operations
export const normalizeAngle = (
  angle: number
): number;

export const angleBetweenPoints = (
  p1: Point,
  p2: Point
): number;
```

### 15. Browser APIs

Located in `packages/common/`:

```typescript
// Browser detection
export const isSafari = (): boolean;
export const isFirefox = (): boolean;
export const isChrome = (): boolean;
export const isMobile = (): boolean;
export const isIOS = (): boolean;
export const isAndroid = (): boolean;

// Feature detection
export const supportsResizeObserver = (): boolean;
export const supportsClipboardAPI = (): boolean;
export const supportsFileSystemAPI = (): boolean;

// DOM utilities
export const querySelector = <T extends Element>(
  selector: string,
  container?: Element | Document
): T | null;

export const addEventListener = (
  target: EventTarget,
  type: string,
  listener: EventListener,
  options?: AddEventListenerOptions
): () => void;
```

---

## Hook APIs

### 16. React Hooks

Located in `packages/excalidraw/hooks/`:

```typescript
// Device detection
export const useDevice = (): {
  isSmScreen: boolean;
  isMdScreen: boolean;
  isLgScreen: boolean;
  isMobile: boolean;
  isTablet: boolean;
  isDesktop: boolean;
  canDeviceFitSidebar: boolean;
};

// Library management
export const useLibrary = (opts?: {
  excalidrawAPI: ExcalidrawAPI;
  getInitialLibraryItems?: () => LibraryItems;
}): {
  library: Library;
  libraryItems: LibraryItems;
  isLoading: boolean;
  isLibraryOpen: boolean;
  setIsLibraryOpen: (open: boolean) => void;
};

// History
export const useHistory = (): {
  isUndoStackEmpty: boolean;
  isRedoStackEmpty: boolean;
};

// Window size
export const useWindowSize = (): {
  width: number;
  height: number;
};

// Animation frame
export const useAnimationFrame = (
  callback: (deltaTime: number) => void,
  deps?: React.DependencyList
): void;

// Local storage
export const useLocalStorage = <T>(
  key: string,
  defaultValue: T
): [T, (value: T) => void];
```

---

## Internal Communication Patterns

### 17. Event System

Excalidraw uses an internal event system for decoupled communication:

```typescript
// Event emitter
class Emitter {
  on(event: string, handler: Function): () => void;
  off(event: string, handler: Function): void;
  emit(event: string, ...args: any[]): void;
  once(event: string, handler: Function): void;
}

// Global events
const globalEvents = new Emitter();

// Usage example
globalEvents.on("elements.change", (elements) => {
  console.log("Elements changed:", elements);
});

globalEvents.emit("elements.change", updatedElements);
```

### 18. Store Communication

Using Jotai for cross-component state:

```typescript
// Define atoms
export const selectedElementsAtom = atom<readonly ExcalidrawElement[]>([]);
export const appStateAtom = atom<AppState>(getDefaultAppState());

// Use in components
const SelectedElementsPanel = () => {
  const [selectedElements] = useAtom(selectedElementsAtom);

  return <div>Selected: {selectedElements.length} elements</div>;
};

// Update from anywhere
const updateSelection = useSetAtom(selectedElementsAtom);
updateSelection(newSelection);
```

### 19. Message Passing

For WebWorker and iframe communication:

```typescript
// Worker communication
const worker = new Worker("processor.worker.js");

worker.postMessage({
  type: "PROCESS_ELEMENTS",
  payload: { elements },
});

worker.onmessage = (event) => {
  if (event.data.type === "ELEMENTS_PROCESSED") {
    const processed = event.data.payload;
    // Handle processed elements
  }
};

// iframe embedding communication
window.parent.postMessage(
  {
    type: "EXCALIDRAW_EXPORT",
    data: {
      elements,
      appState,
      files,
    },
  },
  "*",
);
```

---

## API Usage Patterns

### Complete Example: Custom Tool Implementation

```typescript
// Define a custom star tool
const StarTool = {
  name: "star",

  // Create star element
  createElement: (x: number, y: number, size: number) => {
    const points = generateStarPoints(x, y, size);

    return newLinearElement({
      type: "line",
      x,
      y,
      points,
      strokeColor: appState.currentItemStrokeColor,
      backgroundColor: appState.currentItemBackgroundColor,
      fillStyle: appState.currentItemFillStyle,
      strokeWidth: appState.currentItemStrokeWidth,
      roughness: appState.currentItemRoughness,
      opacity: appState.currentItemOpacity,
    });
  },

  // Handle pointer events
  handlePointerDown: (event: PointerEvent, app: App) => {
    const coords = viewportCoordsToSceneCoords(event, app.state);
    const element = StarTool.createElement(coords.x, coords.y, 0);

    app.scene.insertElement(element);
    app.setState({
      draggingElement: element,
      editingElement: element,
    });
  },

  handlePointerMove: (event: PointerEvent, app: App) => {
    if (!app.state.draggingElement) return;

    const coords = viewportCoordsToSceneCoords(event, app.state);
    const size = pointDistance(
      { x: app.state.draggingElement.x, y: app.state.draggingElement.y },
      coords,
    );

    const points = generateStarPoints(
      app.state.draggingElement.x,
      app.state.draggingElement.y,
      size,
    );

    mutateElement(app.state.draggingElement, { points });
  },

  handlePointerUp: (event: PointerEvent, app: App) => {
    if (!app.state.draggingElement) return;

    app.actionManager.executeAction(actionFinalize);
    app.setState({
      draggingElement: null,
      editingElement: null,
      activeTool: { type: "selection", customType: null },
    });
  },
};

// Register and use the tool
const MyDrawingApp = () => {
  const [excalidrawAPI, setExcalidrawAPI] = useState(null);

  const handleToolClick = () => {
    excalidrawAPI?.setActiveTool({
      type: "custom",
      customType: "star",
    });
  };

  return (
    <>
      <button onClick={handleToolClick}>Star Tool</button>
      <Excalidraw
        excalidrawAPI={setExcalidrawAPI}
        onPointerDown={(event, appState) => {
          if (appState.activeTool.customType === "star") {
            StarTool.handlePointerDown(event, excalidrawAPI);
          }
        }}
        onPointerMove={(event, appState) => {
          if (appState.activeTool.customType === "star") {
            StarTool.handlePointerMove(event, excalidrawAPI);
          }
        }}
        onPointerUp={(event, appState) => {
          if (appState.activeTool.customType === "star") {
            StarTool.handlePointerUp(event, excalidrawAPI);
          }
        }}
      />
    </>
  );
};
```

### Integration Example: Collaborative Backend

```typescript
// Setting up collaboration with custom backend
const CollaborativeExcalidraw = () => {
  const [excalidrawAPI, setExcalidrawAPI] = useState(null);
  const [collaborators, setCollaborators] = useState(new Map());
  const socket = useRef(null);

  useEffect(() => {
    if (!excalidrawAPI) return;

    // Connect to WebSocket server
    socket.current = io("wss://your-server.com");

    // Handle incoming updates
    socket.current.on("scene-update", (data) => {
      const { elements, collaborator } = data;

      // Reconcile elements
      const reconciledElements = reconcileElements(
        excalidrawAPI.getSceneElements(),
        elements,
        excalidrawAPI.getAppState(),
      );

      // Update scene
      excalidrawAPI.updateScene({
        elements: reconciledElements,
      });

      // Update collaborators
      setCollaborators((prev) => {
        const next = new Map(prev);
        next.set(collaborator.id, collaborator);
        return next;
      });
    });

    // Broadcast local changes
    const handleChange = (elements, appState) => {
      socket.current.emit("scene-update", {
        elements: elements.filter((el) => !el.isDeleted),
        appState: {
          viewBackgroundColor: appState.viewBackgroundColor,
        },
      });
    };

    return () => {
      socket.current?.disconnect();
    };
  }, [excalidrawAPI]);

  return (
    <Excalidraw
      excalidrawAPI={setExcalidrawAPI}
      isCollaborating={true}
      collaborators={collaborators}
      onChange={(elements, appState) => {
        // Broadcast changes
        socket.current?.emit("update", { elements, appState });
      }}
      onPointerUpdate={(pointer) => {
        // Broadcast pointer position
        socket.current?.emit("pointer", pointer);
      }}
    />
  );
};
```

---

## API Best Practices

### 1. Performance Considerations

```typescript
// ❌ Bad: Multiple individual updates
elements.forEach((element) => {
  mutateElement(element, { strokeColor: "#000" });
});

// ✅ Good: Batch updates
batchMutateElements(elements, () => ({
  strokeColor: "#000",
}));

// ❌ Bad: Frequent API calls
const animate = () => {
  excalidrawAPI.updateScene({ appState: { zoom: zoom + 0.01 } });
  requestAnimationFrame(animate);
};

// ✅ Good: Use built-in animations
excalidrawAPI.scrollToContent(elements, {
  animate: true,
  duration: 500,
});
```

### 2. State Management

```typescript
// ❌ Bad: Direct state mutation
appState.selectedElementIds[element.id] = true;

// ✅ Good: Immutable updates
excalidrawAPI.updateScene({
  appState: {
    selectedElementIds: {
      ...appState.selectedElementIds,
      [element.id]: true,
    },
  },
});
```

### 3. Error Handling

```typescript
// Always handle API errors gracefully
try {
  const blob = await excalidrawAPI.exportToBlob({
    mimeType: "image/png",
    quality: 0.95,
  });
  // Process blob
} catch (error) {
  console.error("Export failed:", error);
  excalidrawAPI.setToast({
    message: "Export failed. Please try again.",
    duration: 5000,
  });
}
```

---

This comprehensive API reference provides a complete understanding of how different parts of Excalidraw communicate and interact. Each API is designed with specific responsibilities, ensuring clean architecture and maintainability throughout the system.
