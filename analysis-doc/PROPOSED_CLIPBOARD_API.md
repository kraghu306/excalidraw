# Proposed Excalidraw Clipboard JSON API

## Overview

Currently, Excalidraw does **not** have a direct HTTP GET API that returns the `excalidraw/clipboard` JSON format. This document outlines how to create such an API.

## Current vs Proposed

### Current Scene API

```
GET /api/v2/scenes/{sceneId}
Response: Binary (compressed + encrypted)
```

### Proposed Clipboard API

```
GET /api/v2/scenes/{sceneId}/clipboard
Response: JSON (excalidraw/clipboard format)
```

## Implementation Plan

### 1. Backend API Endpoint

Add a new endpoint to the existing backend:

```typescript
// New endpoint alongside existing scene API
app.get("/api/v2/scenes/:sceneId/clipboard", async (req, res) => {
  try {
    const { sceneId } = req.params;
    const { key } = req.query; // Decryption key from URL hash

    // Get compressed data from existing storage
    const compressedData = await getSceneData(sceneId);

    // Decompress and decrypt
    const decrypted = await decryptData(compressedData, key);
    const sceneData = JSON.parse(decrypted);

    // Transform to clipboard format
    const clipboardData = {
      type: "excalidraw/clipboard",
      elements: sceneData.elements,
      files: sceneData.files,
    };

    res.json(clipboardData);
  } catch (error) {
    res.status(400).json({ error: "Invalid scene or key" });
  }
});
```

### 2. Frontend Integration

Update the sharing system to provide clipboard API URLs:

```typescript
// In excalidraw-app/data/index.ts
export const getClipboardApiUrl = (sceneId: string, encryptionKey: string) => {
  return `${BACKEND_V2_GET}${sceneId}/clipboard?key=${encryptionKey}`;
};

// Usage
const shareLink = getCollaborationLink({ roomId, roomKey });
const clipboardApiUrl = getClipboardApiUrl(roomId, roomKey);
```

### 3. URL Structure

**Share Link Format** (current):

```
https://excalidraw.com/#json=abc123,def456
```

**Clipboard API URL** (proposed):

```
https://backend.excalidraw.com/api/v2/scenes/abc123/clipboard?key=def456
```

## Code Implementation

### Backend Changes

#### 1. Add Route Handler

```typescript
// backend/routes/scenes.ts
import { Router } from "express";
import { getSceneData, decryptSceneData } from "../services/sceneService";

const router = Router();

// Existing route
router.get("/:sceneId", handleGetScene);

// New clipboard route
router.get("/:sceneId/clipboard", async (req, res) => {
  const { sceneId } = req.params;
  const { key } = req.query;

  if (!key) {
    return res.status(400).json({
      error: "Decryption key required",
    });
  }

  try {
    // Get compressed binary data
    const binaryData = await getSceneData(sceneId);

    // Decrypt and decompress
    const sceneJson = await decryptSceneData(binaryData, key as string);

    // Transform to clipboard format
    const clipboardFormat = transformToClipboardFormat(sceneJson);

    res.json(clipboardFormat);
  } catch (error) {
    res.status(404).json({
      error: "Scene not found or invalid key",
    });
  }
});

export default router;
```

#### 2. Transform Function

```typescript
// backend/services/transformService.ts
interface SceneData {
  elements: ExcalidrawElement[];
  appState: AppState;
  files?: BinaryFiles;
}

interface ClipboardData {
  type: "excalidraw/clipboard";
  elements: ExcalidrawElement[];
  files?: BinaryFiles;
}

export const transformToClipboardFormat = (
  sceneData: SceneData,
): ClipboardData => {
  return {
    type: "excalidraw/clipboard",
    elements: sceneData.elements.filter((el) => !el.isDeleted),
    files: sceneData.files,
  };
};
```

### Frontend Changes

#### 1. Share Dialog Update

```typescript
// excalidraw-app/components/ShareDialog.tsx
const ShareDialog = () => {
  const [shareData, setShareData] = useState(null);

  const handleCreateShare = async () => {
    const result = await exportToBackend(elements, appState, files);

    if (result.url) {
      const urlParts = new URL(result.url);
      const [sceneId, key] = urlParts.hash.substring(6).split(","); // Remove "#json="

      const shareInfo = {
        shareUrl: result.url,
        clipboardApiUrl: `${BACKEND_V2_GET}${sceneId}/clipboard?key=${key}`,
        sceneId,
        key,
      };

      setShareData(shareInfo);
    }
  };

  return (
    <div>
      {shareData && (
        <>
          <p>Share Link: {shareData.shareUrl}</p>
          <p>Clipboard API: {shareData.clipboardApiUrl}</p>
          <button onClick={() => copyToClipboard(shareData.clipboardApiUrl)}>
            Copy API URL
          </button>
        </>
      )}
    </div>
  );
};
```

#### 2. API Helper Functions

```typescript
// excalidraw-app/data/clipboardApi.ts
export const fetchClipboardData = async (
  sceneId: string,
  encryptionKey: string,
): Promise<ClipboardData> => {
  const url = `${BACKEND_V2_GET}${sceneId}/clipboard?key=${encryptionKey}`;

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Failed to fetch clipboard data: ${response.statusText}`);
  }

  return response.json();
};

export const getClipboardApiUrl = (
  sceneId: string,
  encryptionKey: string,
): string => {
  return `${BACKEND_V2_GET}${sceneId}/clipboard?key=${encryptionKey}`;
};
```

## Postman Testing

### 1. Test Requests

```
GET {{BACKEND_V2_GET}}abc123/clipboard?key=def456
Accept: application/json
```

### 2. Expected Response

```json
{
  "type": "excalidraw/clipboard",
  "elements": [
    {
      "type": "text",
      "version": 378,
      "versionNonce": 1889672898,
      "index": "a0",
      "isDeleted": false,
      "id": "bGuhWaI-Sv7HPwcFrKSxt",
      "fillStyle": "hachure",
      "strokeWidth": 1,
      "strokeStyle": "solid",
      "roughness": 1,
      "opacity": 100,
      "angle": 0,
      "x": -266.3845618234136,
      "y": 117.90624511034216,
      "strokeColor": "#1e1e1e",
      "backgroundColor": "#ffffff",
      "width": 212.07591247558594,
      "height": 45,
      "seed": 1180574373,
      "groupIds": [],
      "frameId": null,
      "roundness": null,
      "boundElements": [],
      "updated": 1718641764588,
      "link": null,
      "locked": false,
      "fontSize": 36,
      "fontFamily": 1,
      "text": "Web Crawler",
      "textAlign": "left",
      "verticalAlign": "top",
      "containerId": null,
      "originalText": "Web Crawler",
      "autoResize": true,
      "lineHeight": 1.25
    }
    // ... more elements
  ],
  "files": {
    // Binary file data if any images
  }
}
```

### 3. Error Responses

```json
// Missing key
{
  "error": "Decryption key required"
}

// Invalid scene or key
{
  "error": "Scene not found or invalid key"
}
```

## Security Considerations

### 1. Rate Limiting

```typescript
// Add rate limiting to prevent abuse
import rateLimit from "express-rate-limit";

const clipboardApiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: "Too many requests, please try again later",
});

router.get("/:sceneId/clipboard", clipboardApiLimiter, handleClipboardApi);
```

### 2. Key Validation

```typescript
const validateEncryptionKey = (key: string): boolean => {
  // Validate key format and length
  return /^[a-zA-Z0-9_-]{22}$/.test(key);
};
```

### 3. CORS Headers

```typescript
// Allow cross-origin requests for API usage
app.use(
  "/api/v2/scenes/:sceneId/clipboard",
  cors({
    origin: true,
    methods: ["GET"],
    allowedHeaders: ["Content-Type", "Authorization"],
  }),
);
```

## Benefits

1. **Direct JSON Access**: No need to decompress/decrypt on client
2. **API Integration**: Easy integration with other tools
3. **Postman Testing**: Direct HTTP testing capability
4. **Clipboard Format**: Exact format for copy/paste operations
5. **Backward Compatible**: Doesn't break existing scene API

## Migration Path

1. **Phase 1**: Add new endpoint alongside existing API
2. **Phase 2**: Update share dialog to show both URLs
3. **Phase 3**: Update documentation
4. **Phase 4**: Monitor usage and optimize

## Alternative: Client-Side Transformation

If backend changes aren't feasible, create a client-side service:

```typescript
// Transform existing binary API response to clipboard format
export const getClipboardDataFromScene = async (
  sceneId: string,
  encryptionKey: string,
): Promise<ClipboardData> => {
  // Use existing binary API
  const binaryData = await fetch(`${BACKEND_V2_GET}${sceneId}`);
  const buffer = await binaryData.arrayBuffer();

  // Decrypt and decompress (using existing functions)
  const sceneData = await decryptAndDecompress(buffer, encryptionKey);

  // Transform to clipboard format
  return {
    type: "excalidraw/clipboard",
    elements: sceneData.elements,
    files: sceneData.files,
  };
};
```

## Conclusion

The clipboard API would provide direct access to Excalidraw diagrams in the exact format you need, making integration and testing much easier. The implementation builds on existing infrastructure while adding the specific JSON format capability.
