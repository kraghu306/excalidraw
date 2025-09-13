# Excalidraw Folder Structure Guide

## Root Directory

```
excalidraw/
├── .github/                    # GitHub configuration
│   ├── workflows/             # CI/CD workflows
│   └── assets/                # GitHub assets
├── .husky/                     # Git hooks configuration
├── dev-docs/                   # Development documentation
├── examples/                   # Usage examples
├── excalidraw-app/            # Web application
├── firebase-project/          # Firebase configuration
├── packages/                  # Core library packages
├── public/                    # Static assets
├── scripts/                   # Build and utility scripts
└── [config files]             # Root configuration files
```

## Packages Directory (`/packages/`)

### `/packages/excalidraw/` - Main Library

```
excalidraw/
├── actions/                   # User actions/commands
│   ├── actionCanvas.tsx      # Canvas-related actions
│   ├── actionExport.tsx      # Export functionality
│   ├── actionHistory.tsx     # Undo/redo
│   ├── actionProperties.tsx  # Element properties
│   ├── actionTools.tsx       # Tool selection
│   ├── manager.tsx           # Action manager
│   ├── register.ts           # Action registry
│   └── types.ts              # Action types
├── components/                # React components
│   ├── App.tsx               # Main app component
│   ├── Canvas.tsx            # Canvas wrapper
│   ├── ContextMenu/          # Context menu
│   ├── Dialog/               # Dialog components
│   ├── Footer/               # Footer components
│   ├── Icons/                # Icon components
│   ├── LayerUI.tsx           # UI layer
│   ├── MainMenu/             # Main menu
│   ├── Properties/           # Properties panel
│   ├── Stats.tsx             # Statistics display
│   ├── Toolbar/              # Toolbar components
│   └── WelcomeScreen/        # Welcome screen
├── context/                   # React contexts
│   ├── ui-appState.tsx       # UI state context
│   └── tunnels.tsx           # React tunnels
├── css/                       # Stylesheets
│   ├── app.scss              # Main app styles
│   ├── styles.scss           # Component styles
│   └── theme.scss            # Theme definitions
├── data/                      # Data handling
│   ├── blob.ts               # Blob operations
│   ├── clipboard.ts          # Clipboard handling
│   ├── encryption.ts         # Encryption utilities
│   ├── filesystem.ts         # File system access
│   ├── json.ts               # JSON operations
│   ├── library.ts            # Library management
│   ├── reconcile.ts          # Data reconciliation
│   ├── restore.ts            # State restoration
│   └── types.ts              # Data types
├── fonts/                     # Font files and config
│   ├── fonts.css             # Font definitions
│   └── [font files]          # Actual font files
├── hooks/                     # React hooks
│   ├── useDevice.ts          # Device detection
│   ├── useHistory.ts         # History hook
│   ├── useLibrary.ts         # Library hook
│   └── useWindowSize.ts      # Window size hook
├── locales/                   # Internationalization
│   ├── [language codes]/     # Language files
│   └── index.ts              # Locale exports
├── renderer/                  # Canvas rendering
│   ├── renderElement.ts      # Element rendering
│   ├── renderScene.ts        # Scene rendering
│   └── roundRect.ts          # Shape utilities
├── scene/                     # Scene management
│   ├── Scene.ts              # Scene class
│   ├── selection.ts          # Selection logic
│   ├── scrollbars.ts         # Scrollbar handling
│   └── zoom.ts               # Zoom functionality
├── tests/                     # Test files
│   ├── __snapshots__/        # Test snapshots
│   ├── fixtures/             # Test fixtures
│   ├── helpers/              # Test helpers
│   └── [test files]          # Unit tests
├── wysiwyg/                   # Text editor
│   ├── index.tsx             # WYSIWYG editor
│   └── styles.scss           # Editor styles
├── analytics.ts               # Analytics tracking
├── animation-frame-handler.ts # RAF handler
├── appState.ts               # App state utilities
├── charts.ts                 # Chart functionality
├── clients.ts                # Client management
├── clipboard.ts              # Clipboard operations
├── constants.ts              # App constants
├── cursor.ts                 # Cursor handling
├── deburr.ts                 # Text normalization
├── editor-jotai.ts           # Jotai state
├── errors.ts                 # Error definitions
├── gesture.ts                # Gesture recognition
├── history.ts                # History management
├── i18n.ts                   # Internationalization
├── index.tsx                 # Main export
├── polyfill.ts               # Browser polyfills
├── snapping.ts               # Snap-to-grid logic
├── types.ts                  # TypeScript types
└── workers.ts                # Web workers
```

### `/packages/common/` - Shared Utilities

```
common/
├── src/
│   ├── arrayUtils.ts         # Array utilities
│   ├── browserUtils.ts       # Browser detection
│   ├── constants.ts          # Shared constants
│   ├── debounce.ts          # Debounce utility
│   ├── throttle.ts          # Throttle utility
│   ├── types.ts             # Common types
│   └── utils.ts             # General utilities
├── tests/                    # Unit tests
└── package.json             # Package config
```

### `/packages/element/` - Element System

```
element/
├── src/
│   ├── bounds.ts            # Bounding box logic
│   ├── collision.ts         # Collision detection
│   ├── dragElements.ts      # Drag handling
│   ├── elementProperties.ts # Element properties
│   ├── index.ts            # Main exports
│   ├── linearElementEditor.ts # Linear element editing
│   ├── mutateElement.ts    # Element mutations
│   ├── newElement.ts       # Element creation
│   ├── resizeElements.ts   # Resize logic
│   ├── rotateElements.ts   # Rotation logic
│   ├── textElement.ts      # Text handling
│   ├── transformElements.ts # Transformations
│   └── types.ts            # Element types
├── tests/                   # Unit tests
└── package.json            # Package config
```

### `/packages/math/` - Mathematical Utilities

```
math/
├── src/
│   ├── angle.ts            # Angle calculations
│   ├── collision.ts        # Collision math
│   ├── distance.ts         # Distance calculations
│   ├── index.ts           # Main exports
│   ├── line.ts            # Line operations
│   ├── point.ts           # Point operations
│   ├── polygon.ts         # Polygon operations
│   ├── triangle.ts        # Triangle operations
│   ├── types.ts           # Math types
│   └── vector.ts          # Vector operations
├── tests/                  # Unit tests
└── package.json           # Package config
```

### `/packages/utils/` - Utility Functions

```
utils/
├── src/
│   ├── export.ts          # Export utilities
│   ├── image.ts           # Image processing
│   ├── index.ts          # Main exports
│   ├── library.ts        # Library utilities
│   └── types.ts          # Utility types
├── tests/                 # Unit tests
└── package.json          # Package config
```

## Application Directory (`/excalidraw-app/`)

```
excalidraw-app/
├── app-language/          # App-specific i18n
│   └── LanguageList.tsx  # Language selector
├── collab/               # Collaboration features
│   ├── Collab.tsx       # Main collab component
│   ├── Portal.tsx       # WebSocket portal
│   ├── RoomDialog.tsx   # Room creation dialog
│   └── reconcile.ts     # Conflict resolution
├── components/          # App components
│   ├── AppFooter.tsx    # App footer
│   ├── AppMenu.tsx      # App menu
│   ├── AppToolbar.tsx   # App toolbar
│   ├── AppWelcomeScreen.tsx # Welcome screen
│   ├── GitHubCorner.tsx # GitHub link
│   └── ShareDialog.tsx  # Share dialog
├── data/                # App data handling
│   ├── FileManager.ts   # File management
│   ├── LocalData.ts     # Local storage
│   └── firebase.ts      # Firebase integration
├── share/              # Share functionality
│   └── ShareIcons.tsx  # Share icons
├── tests/              # App tests
├── App.tsx            # Main app component
├── index.html         # HTML template
├── index.tsx          # App entry point
├── package.json       # App dependencies
└── vite.config.ts     # Vite configuration
```

## Examples Directory (`/examples/`)

```
examples/
├── with-nextjs/           # Next.js integration
│   ├── pages/            # Next.js pages
│   ├── public/           # Static assets
│   └── package.json      # Dependencies
└── with-script-in-browser/ # Browser script example
    ├── index.html        # HTML example
    └── package.json      # Dependencies
```

## Scripts Directory (`/scripts/`)

```
scripts/
├── build-locales-coverage.js  # i18n coverage
├── build-node.js              # Node build
├── build-version.js           # Version generation
├── buildBase.js               # Base build script
├── buildPackage.js            # Package build
├── buildUtils.js              # Build utilities
├── release.js                 # Release script
├── updateChangelog.js         # Changelog update
└── wasm/                      # WASM utilities
```

## Configuration Files (Root)

```
├── .eslintignore             # ESLint ignore
├── .eslintrc.js             # ESLint config
├── .gitignore               # Git ignore
├── .prettierignore          # Prettier ignore
├── .prettierrc.json         # Prettier config
├── CHANGELOG.md             # Version changelog
├── LICENSE                  # MIT license
├── README.md               # Project readme
├── package.json            # Root package.json
├── tsconfig.json           # TypeScript config
├── vitest.config.mts       # Vitest config
└── yarn.lock              # Yarn lockfile
```

## Key File Locations

### Entry Points

- Library: `packages/excalidraw/index.tsx`
- App: `excalidraw-app/index.tsx`
- Types: `packages/excalidraw/types.ts`

### Core Components

- App Component: `packages/excalidraw/components/App.tsx`
- Canvas: `packages/excalidraw/components/Canvas.tsx`
- Layer UI: `packages/excalidraw/components/LayerUI.tsx`

### State Management

- App State: `packages/excalidraw/appState.ts`
- Jotai Store: `packages/excalidraw/editor-jotai.ts`
- History: `packages/excalidraw/history.ts`

### Rendering

- Scene Renderer: `packages/excalidraw/renderer/renderScene.ts`
- Element Renderer: `packages/excalidraw/renderer/renderElement.ts`

### Actions

- Action Manager: `packages/excalidraw/actions/manager.tsx`
- Action Registry: `packages/excalidraw/actions/register.ts`

### Collaboration

- Collab Component: `excalidraw-app/collab/Collab.tsx`
- WebSocket Portal: `excalidraw-app/collab/Portal.tsx`

### Data Handling

- JSON Export: `packages/excalidraw/data/json.ts`
- Image Export: `packages/excalidraw/data/blob.ts`
- Library: `packages/excalidraw/data/library.ts`

## Development Tips

### Finding Features

1. **Actions**: Look in `packages/excalidraw/actions/`
2. **UI Components**: Check `packages/excalidraw/components/`
3. **Element Logic**: See `packages/element/src/`
4. **Math Operations**: Find in `packages/math/src/`
5. **Rendering**: Explore `packages/excalidraw/renderer/`

### Common Modifications

- **Add new tool**: Update `types.ts`, create action, add to toolbar
- **New element type**: Modify element package, add renderer
- **UI changes**: Edit components and styles in components folder
- **Export formats**: Modify `data/blob.ts` and `data/json.ts`
- **Keyboard shortcuts**: Update action definitions in actions folder

### Testing Locations

- Unit tests: Next to source files as `*.test.ts`
- Integration tests: In `tests/` directories
- Snapshots: In `__snapshots__/` folders
- Fixtures: In `tests/fixtures/`
