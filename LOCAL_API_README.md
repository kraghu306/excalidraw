# 🚀 Excalidraw Local API Server

## Quick Start

The fastest way to test Excalidraw APIs with your local scene data:

### 1. One-Command Startup

```bash
./start-with-api.sh
```

This starts:

- **Excalidraw App**: http://localhost:3001
- **Local API Server**: http://localhost:3002

### 2. Save & Test a Scene

1. **Open**: http://localhost:3001/?scene=YOUR_SCENE_ID
2. **Draw** something
3. **Click** Scene Info panel (📋 bottom-right)
4. **Click** "💾 Save to Local API"
5. **Test** in Postman or curl:

```bash
# Get clipboard format (the format you wanted!)
curl http://localhost:3002/api/scenes/YOUR_SCENE_ID/clipboard

# List all saved scenes
curl http://localhost:3002/api/scenes

# Health check
curl http://localhost:3002/api/health
```

## 🎯 The Answer to Your Original Question

**Question**: _"do we have a get api for the diagram say do a get to get the diagram to excalidraw json which is in the format of {type: 'excalidraw/clipboard', elements: [...]}"_

**Answer**: **YES!** 🎉

```bash
GET http://localhost:3002/api/scenes/{sceneId}/clipboard
```

**Returns exactly**:

```json
{
  "type": "excalidraw/clipboard",
  "sceneId": "YOUR_SCENE_ID",
  "timestamp": 1694612345678,
  "elements": [...],
  "files": {...},
  "appState": {...}
}
```

## 📡 All Available Endpoints

| Endpoint                    | Method | Description                   |
| --------------------------- | ------ | ----------------------------- |
| `/api/health`               | GET    | Server health check           |
| `/api/scenes`               | GET    | List all scenes               |
| `/api/scenes/:id`           | GET    | Get full scene data           |
| `/api/scenes/:id/clipboard` | GET    | **Get clipboard JSON format** |
| `/api/scenes/:id`           | POST   | Save scene data               |
| `/api/scenes/:id`           | DELETE | Delete scene                  |

## 🔧 Manual Setup (Alternative)

If you prefer to start servers manually:

```bash
# Terminal 1: Excalidraw App
yarn start

# Terminal 2: Local API Server
node local-api-server.js
```

## 📁 File Structure

- `local-api-server.js` - The API server code
- `start-with-api.sh` - One-command startup script
- `.local-scenes/` - Where scene data is stored (auto-created)

## 🧪 Testing Examples

### Basic Testing

```bash
# Test the API is working
curl http://localhost:3002/api/health

# Create a test scene via API
curl -X POST http://localhost:3002/api/scenes/TEST123 \
  -H "Content-Type: application/json" \
  -d '{"elements": [{"type": "rectangle", "x": 0, "y": 0, "width": 100, "height": 100}]}'

# Get it back in clipboard format
curl http://localhost:3002/api/scenes/TEST123/clipboard
```

### Postman Collection

Import this into Postman:

**Environment Variables**:

- `api_base_url`: `http://localhost:3002`
- `scene_id`: `YOUR_SCENE_ID`

**Requests**:

```
GET {{api_base_url}}/api/health
GET {{api_base_url}}/api/scenes
GET {{api_base_url}}/api/scenes/{{scene_id}}/clipboard
```

---

**🎉 You now have complete API access to your Excalidraw scenes in the exact format you requested!**
