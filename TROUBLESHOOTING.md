# Troubleshooting Guide for Plato's Cave

## Common Hosting Issues and Solutions

### Issue 1: Socket.io 403 Forbidden Error

**Symptom:** You see this error in the browser console:
```
Request URL: http://localhost:5000/socket.io/?EIO=4&transport=polling&t=hgyv0ga6
Status Code: 403 Forbidden
```

**Root Causes:**
1. Backend server is not running
2. Backend server is not listening on the correct host/port
3. Missing dependencies or configuration issues

**Solutions:**

#### Step 1: Verify Backend is Running

Make sure you've followed all backend setup steps:

```bash
cd backend
uv sync
source .venv/bin/activate
python server.py
```

You should see:
```
[SERVER] Starting Flask-SocketIO server on http://0.0.0.0:5000
[SERVER] CORS enabled for all origins
[SERVER] Using eventlet for async operations
```

#### Step 2: Check if Port 5000 is Available

```bash
# On Linux/Mac:
lsof -i :5000

# On Windows:
netstat -ano | findstr :5000
```

If another process is using port 5000, either kill that process or change the port in:
- `backend/server.py` (line 701)
- `frontend/src/pages/index.tsx` (line 54)

#### Step 3: Verify Backend Dependencies

```bash
cd backend
uv sync
```

Make sure all required packages are installed:
- flask>=3.0.0
- flask-socketio>=5.3.0
- flask-cors>=4.0.0
- eventlet>=0.36.1

#### Step 4: Test Backend Directly

Open a browser and navigate to:
```
http://localhost:5000/socket.io/
```

You should see a Socket.io response (not a 403 error).

#### Step 5: Check Firewall Settings

Ensure your firewall allows connections to port 5000:

```bash
# On Linux with ufw:
sudo ufw allow 5000

# On Mac:
# System Preferences > Security & Privacy > Firewall > Firewall Options
```

---

### Issue 2: "Attention Is All You Need" Object in Console

**Symptom:** You see this object in the browser console:
```javascript
{
  url: "Attention Is All You Need",
  agentAggressiveness: 5,
  evidenceThreshold: 0.8,
  credibility: 1,
  // ... more properties
}
```

**Explanation:** This is NOT an error! This is demo data from a typewriter animation showing example research paper titles. It's completely harmless and part of the normal UI behavior.

**Location:** `frontend/src/components/FileUploader.tsx:125-134`

If you want to disable the console logging, you can add this to your browser console:
```javascript
console.log = () => {}; // Disable all console.log (not recommended for debugging)
```

---

### Issue 3: WebSocket Connection Keeps Disconnecting

**Symptom:** Socket connects briefly then disconnects

**Solutions:**

1. **Check Backend Logs** - Look for disconnect messages in the backend terminal
2. **Increase Timeouts** - Already configured in the latest version:
   - `ping_timeout: 300` (5 minutes)
   - `ping_interval: 25` (25 seconds)
3. **Check Network Stability** - Ensure stable connection between frontend and backend

---

### Issue 4: CORS Errors

**Symptom:** Browser console shows CORS policy errors

**Solution:** CORS is already enabled in the backend (`backend/server.py:16-22`):
```python
CORS(app, resources={r"/*": {"origins": "*"}})
socketio = SocketIO(app, cors_allowed_origins="*", ...)
```

If you still see CORS errors:
1. Clear browser cache
2. Try a different browser
3. Disable browser extensions that might block requests

---

## Proper Startup Sequence

You need **THREE** separate terminal sessions:

### Terminal 1: Docker (Browser Automation)
```bash
docker compose -f docker-compose.browser.yaml up --build remote-browser
```

### Terminal 2: Backend Server
```bash
cd backend
source .venv/bin/activate
python server.py
```

Wait until you see:
```
[SERVER] Starting Flask-SocketIO server on http://0.0.0.0:5000
```

### Terminal 3: Frontend
```bash
cd frontend
npm install
gatsby develop
```

Wait until you see:
```
You can now view frontend in the browser.
http://localhost:8000/
```

Now open your browser and go to `http://localhost:8000`

---

## Debugging Tips

### Enable Verbose Logging

**Backend:** Already has extensive logging with `[SERVER DEBUG]` prefixes

**Frontend:** Open browser DevTools (F12) and check:
- Console tab - for JavaScript errors
- Network tab - for failed requests
- Application tab > WebSocket - for Socket.io connection status

### Common Debug Commands

```bash
# Check if backend is responding
curl http://localhost:5000/socket.io/

# Check frontend build
cd frontend && gatsby clean && gatsby develop

# Restart all services
# Kill all terminals (Ctrl+C) then restart in order: Docker -> Backend -> Frontend
```

---

## Still Having Issues?

1. **Check the Logs:**
   - Backend: Look for error messages in the terminal running `python server.py`
   - Frontend: Open browser DevTools and check Console tab
   - Docker: Check `docker logs` for browser container

2. **Verify Setup:**
   - Python version: >= 3.11
   - Node version: >= 18.x
   - Docker: Running and accessible

3. **Clean Install:**
   ```bash
   # Backend
   cd backend
   rm -rf .venv
   uv sync

   # Frontend
   cd frontend
   rm -rf node_modules .cache public
   npm install
   ```

4. **Environment Variables:**
   Make sure you have a `.env` file in the `backend` directory with:
   ```
   BROWSER_USE_API_KEY="bu_YOURKEY"
   ```

---

## Fixed Issues

The following issues have been fixed in the latest version:

- ✅ Socket.io client now has proper configuration with reconnection logic
- ✅ Socket.io client has error handlers for `connect_error` and `error` events
- ✅ Backend server now binds to `0.0.0.0` instead of `127.0.0.1`
- ✅ Backend server includes `allow_unsafe_werkzeug=True` for development
- ✅ Improved logging for both client and server connections
- ✅ Socket.io transports explicitly set to `['polling', 'websocket']`

---

## Contact

If you're still experiencing issues, please:
1. Check existing GitHub issues
2. Create a new issue with:
   - Error messages from browser console
   - Error messages from backend terminal
   - Your operating system and versions (Python, Node, Docker)
   - Steps to reproduce the issue
