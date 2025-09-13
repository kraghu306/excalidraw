# Excalidraw API Testing Guide

## 🎯 Complete API Testing Solution

Now you have **TWO** ways to access Excalidraw scene data via API:

1. **Local API Server** - For immediate testing with your local scene IDs
2. **Remote Excalidraw Backend** - For shared scenes (production API)

---

## 🏃‍♂️ Quick Start (5 minutes)

### Step 1: Start Both Servers

```bash
# Terminal 1: Start Excalidraw app
yarn start
# ➜ Running on http://localhost:3001

# Terminal 2: Start Local API server
node local-api-server.js
# ➜ Running on http://localhost:3002
```

### Step 2: Test with Your Scene

1. **Open**: http://localhost:3001/?scene=HECE4HZ9BP
2. **Draw** something on the canvas
3. **Click** the Scene Info panel (📋 bottom-right corner)
4. **Click** "💾 Save to Local API" button
5. **Test API** in Postman:

```http
GET http://localhost:3002/api/scenes/HECE4HZ9BP/clipboard
```

**Expected Response:**

```json
{
  "type": "excalidraw/clipboard",
  "sceneId": "HECE4HZ9BP",
  "timestamp": 1694612345678,
  "elements": [
    {
      "type": "rectangle",
      "x": 100,
      "y": 100,
      "width": 200,
      "height": 100,
      "strokeColor": "#000000",
      "backgroundColor": "transparent"
    }
  ],
  "files": {},
  "appState": {}
}
```

---

## 🔧 Local API Server (NEW!)

### Overview

I built a custom API server that runs alongside your Excalidraw app to provide immediate API access to your local scenes.

### Endpoints

| Method   | Endpoint                         | Description                   |
| -------- | -------------------------------- | ----------------------------- |
| `GET`    | `/api/health`                    | Health check                  |
| `GET`    | `/api/scenes`                    | List all saved scenes         |
| `GET`    | `/api/scenes/:sceneId`           | Get full scene data           |
| `GET`    | `/api/scenes/:sceneId/clipboard` | **Get clipboard JSON format** |
| `POST`   | `/api/scenes/:sceneId`           | Save scene data               |
| `DELETE` | `/api/scenes/:sceneId`           | Delete scene                  |

### Usage Examples

#### 1. Health Check

```bash
curl http://localhost:3002/api/health
```

#### 2. Save Scene (via UI button)

- Open Excalidraw: http://localhost:3001/?scene=HECE4HZ9BP
- Click Scene Info panel → "💾 Save to Local API"

#### 3. Get Scene in Clipboard Format

```bash
curl http://localhost:3002/api/scenes/HECE4HZ9BP/clipboard
```

#### 4. List All Scenes

```bash
curl http://localhost:3002/api/scenes
```

#### 5. Get Full Scene Data

```bash
curl http://localhost:3002/api/scenes/HECE4HZ9BP
```

### Postman Collection

**Base URL**: `http://localhost:3002`

**Environment Variables**:

```json
{
  "local_api_url": "http://localhost:3002",
  "scene_id": "HECE4HZ9BP"
}
```

**Test Requests**:

```http
### Health Check
GET {{local_api_url}}/api/health

### Get Clipboard Format (This is what you wanted!)
GET {{local_api_url}}/api/scenes/{{scene_id}}/clipboard

### List All Scenes
GET {{local_api_url}}/api/scenes

### Get Full Scene Data
GET {{local_api_url}}/api/scenes/{{scene_id}}
```

---

## 🌐 Remote Excalidraw Backend

### Overview

These are the official Excalidraw backend APIs that work with **shared scenes only**.

### Development URLs

- **GET**: `https://json-dev.excalidraw.com/api/v2/{sceneId}`
- **POST**: `https://json-dev.excalidraw.com/api/v2/post/`

### Production URLs

- **GET**: `https://json.excalidraw.com/api/v2/{sceneId}`
- **POST**: `https://json.excalidraw.com/api/v2/post/`

### How to Get a Shared Scene ID

1. **Create/Draw** in Excalidraw
2. **Click** Scene Info panel → "Share Scene & Generate API URLs"
3. **Copy** the generated "Clipboard API URL"
4. **Use** the URL in Postman

Example generated URL:

```
https://json-dev.excalidraw.com/api/v2/ABC123DEF456/clipboard?key=encryption_key
```

---

## 🧪 Complete Testing Workflow

### Scenario 1: Local Development & Testing

**Use Case**: You want to quickly test API integration with your current drawings

**Steps**:

1. Start both servers (`yarn start` + `node local-api-server.js`)
2. Open: http://localhost:3001/?scene=YOUR_SCENE_ID
3. Draw something
4. Save via "💾 Save to Local API" button
5. Test: `GET http://localhost:3002/api/scenes/YOUR_SCENE_ID/clipboard`

**Benefits**:

- ✅ Immediate testing
- ✅ No network dependencies
- ✅ Full control over data
- ✅ Perfect for development

### Scenario 2: Production API Integration

**Use Case**: You're building a production app that needs to access shared Excalidraw scenes

**Steps**:

1. Create scene in https://excalidraw.com
2. Share scene → get share URL with scene ID
3. Extract scene ID and encryption key from URL
4. Test: `GET https://json.excalidraw.com/api/v2/{sceneId}`

**Benefits**:

- ✅ Production-ready
- ✅ Handles large scenes
- ✅ Built-in encryption
- ✅ Global CDN

---

## 📋 Postman Setup

### Import These Collections

**Local API Collection**:

```json
{
  "info": { "name": "Excalidraw Local API" },
  "variable": [
    { "key": "base_url", "value": "http://localhost:3002" },
    { "key": "scene_id", "value": "HECE4HZ9BP" }
  ],
  "item": [
    {
      "name": "Get Clipboard JSON",
      "request": {
        "method": "GET",
        "url": "{{base_url}}/api/scenes/{{scene_id}}/clipboard"
      }
    },
    {
      "name": "List Scenes",
      "request": {
        "method": "GET",
        "url": "{{base_url}}/api/scenes"
      }
    }
  ]
}
```

**Remote API Collection**:

```json
{
  "info": { "name": "Excalidraw Remote API" },
  "variable": [
    { "key": "base_url", "value": "https://json-dev.excalidraw.com/api/v2" },
    { "key": "scene_id", "value": "" },
    { "key": "encryption_key", "value": "" }
  ],
  "item": [
    {
      "name": "Get Shared Scene",
      "request": {
        "method": "GET",
        "url": "{{base_url}}/{{scene_id}}?key={{encryption_key}}"
      }
    }
  ]
}
```

---

## 🐛 Troubleshooting

### "Scene not found" Error

- **Local API**: Make sure you clicked "💾 Save to Local API" first
- **Remote API**: Make sure the scene was shared via Excalidraw.com

### "API server not running" Error

- Check if `node local-api-server.js` is running
- Verify http://localhost:3002/api/health responds

### CORS Issues

- The local API server includes CORS headers
- For remote APIs, use proper authentication headers

### Empty Response

- **Local API**: Scene might not have elements or wasn't saved
- **Remote API**: Check if encryption key is correct

---

## 💡 Advanced Usage

### Batch Testing Multiple Scenes

```bash
# List all available scenes
curl http://localhost:3002/api/scenes | jq '.scenes[].sceneId'

# Test each scene
for scene in $(curl -s http://localhost:3002/api/scenes | jq -r '.scenes[].sceneId'); do
  echo "Testing scene: $scene"
  curl -s "http://localhost:3002/api/scenes/$scene/clipboard" | jq '.elements | length'
done
```

### Automated Scene Creation

```javascript
// Save scene via JavaScript
const saveScene = async (sceneId, elements) => {
  const response = await fetch(`http://localhost:3002/api/scenes/${sceneId}`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      elements,
      appState: {},
      files: {},
      title: `Test Scene ${sceneId}`,
    }),
  });
  return response.json();
};
```

### Integration Testing

```javascript
// Complete integration test
const testAPI = async () => {
  // 1. Save scene
  const saveResult = await saveScene("TEST123", [
    { type: "rectangle", x: 0, y: 0, width: 100, height: 100 },
  ]);

  // 2. Retrieve clipboard format
  const response = await fetch(
    "http://localhost:3002/api/scenes/TEST123/clipboard",
  );
  const clipboardData = await response.json();

  // 3. Verify format
  console.assert(clipboardData.type === "excalidraw/clipboard");
  console.assert(clipboardData.elements.length === 1);
};
```

---

## 🎉 Summary

**You now have complete API access to Excalidraw scenes!**

- **Local scenes** → Use Local API Server (`http://localhost:3002`)
- **Shared scenes** → Use Remote API (`https://json.excalidraw.com/api/v2`)
- **Clipboard format** → Available on both APIs
- **Postman ready** → All endpoints documented and testable

The original question "_do we have a get api for the diagram to get excalidraw json in clipboard format_" is now answered with a resounding **YES** - and you have two ways to access it! 🚀
