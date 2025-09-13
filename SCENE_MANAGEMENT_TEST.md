# Scene Management Implementation Test Plan

## ✅ Implementation Complete!

### What Was Implemented

1. **URL-based Scene ID Management** ✅

   - Every scene now gets a unique ID in the URL automatically
   - Format: `localhost:3000?scene=abcd1234ef`
   - Scene ID persists across page refreshes

2. **Scene Info Component** ✅

   - Displays in bottom-right corner
   - Shows current scene ID
   - Shows element count
   - Collapsible/expandable interface

3. **Features Added** ✅
   - **Copy Scene ID**: Click to copy the scene ID
   - **Share Scene**: Generate shareable link with encryption
   - **Clipboard API URL**: Get the API endpoint for the scene
   - **Copy as Clipboard JSON**: Copy current scene in clipboard format
   - **Download JSON**: Download scene as JSON file

## How to Test

### 1. Start the Development Server

```bash
yarn start
```

### 2. Open Excalidraw

Navigate to: `http://localhost:3000`

### 3. Check URL

You should see a scene ID in the URL:

```
http://localhost:3000?scene=xY3kL9mN2p
```

### 4. Scene Info Panel

Look at the **bottom-right corner** of the screen. You'll see:

```
📋 Scene: xY3kL9mN2p
```

Click on it to expand and see:

- Scene ID field with Copy button
- Element count
- Share button (to generate API URLs)
- Copy/Download options

### 5. Test Features

#### A. Copy Scene ID

1. Click "Copy" next to Scene ID
2. Paste somewhere to verify

#### B. Share Scene

1. Draw some elements
2. Click "Share Scene & Generate API URLs"
3. You'll get:
   - Share URL (for browser sharing)
   - Clipboard API URL (for Postman/API access)

#### C. Get Clipboard JSON

1. Draw some shapes
2. Click "Copy as Clipboard JSON"
3. Paste in a text editor to see the JSON format:

```json
{
  "type": "excalidraw/clipboard",
  "sceneId": "xY3kL9mN2p",
  "timestamp": 1234567890,
  "elements": [...]
}
```

#### D. Download JSON

1. Click "Download JSON"
2. File will be saved as `excalidraw-scene-{sceneId}.json`

## What You Can Now Do

### 1. **Scene Always Has an ID**

- Every drawing session has a unique identifier
- No more anonymous/unidentifiable scenes
- ID visible in URL for easy reference

### 2. **Easy Sharing**

- Share button generates permanent links
- Get both browser URL and API endpoint
- API URL can be used in Postman (once backend implements the endpoint)

### 3. **Export in Clipboard Format**

- Get the exact JSON format you requested
- Includes all elements and metadata
- Can be used for integration with other tools

### 4. **Persistent Scene Reference**

- Scene ID stays in URL
- Bookmark specific scenes
- Share scene references with others

## Files Created/Modified

### New Files:

1. `excalidraw-app/data/sceneManagement.ts` - Scene ID management utilities
2. `excalidraw-app/components/SceneInfo.tsx` - UI component
3. `excalidraw-app/components/SceneInfo.scss` - Styles

### Modified Files:

1. `excalidraw-app/App.tsx` - Integration of scene management

## Next Steps

To complete the full API integration:

1. **Backend Implementation** (if you have access):

   - Implement the `/api/v2/scenes/{sceneId}/clipboard` endpoint
   - Return unencrypted JSON in clipboard format

2. **Testing with Postman**:

   - Once backend is ready, use the Clipboard API URL
   - Make GET request to fetch scene JSON

3. **Enhancements** (optional):
   - Add scene name/title editing
   - Scene history/versioning
   - Quick scene switching
   - Scene search functionality

## Troubleshooting

If the Scene Info panel doesn't appear:

1. Check browser console for errors
2. Ensure you're on `localhost:3000` (not production)
3. Try refreshing the page
4. Check that JavaScript is enabled

## Success Criteria ✅

- [x] Scene ID appears in URL automatically
- [x] Scene Info panel shows in bottom-right
- [x] Can copy scene ID
- [x] Can share scene and get API URLs
- [x] Can export as clipboard JSON
- [x] Scene ID persists on refresh
- [x] TypeScript compilation passes

The implementation is complete and ready to use!
