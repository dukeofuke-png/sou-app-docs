# Starting All Servers

**IMPORTANT:** Both servers are now configured with `.env` files to use the correct ports automatically:
- Backend: `http://localhost:3002`
- Frontend: `http://localhost:3000`

You have **4 easy options** to start and keep servers running:

---

## Option 1: Keep-Alive Monitor (RECOMMENDED - Auto-Restart)

This will start both servers AND automatically restart them if they crash:

```bash
./keep-servers-alive.sh
```

This script:
- ✅ Automatically starts both servers
- ✅ Monitors them every 10 seconds
- ✅ Restarts crashed servers automatically
- ✅ Clears ports if they're stuck
- Press **Ctrl+C** to stop monitoring

**View logs while running:**
```bash
# In separate terminals:
tail -f logs/backend.log
tail -f logs/frontend.log
```

---

## Option 2: NPM Script (Simple Parallel Start)

From the root directory (`/Users/matthew/Documents/SOU App`):

```bash
npm start
```

This will start both servers in parallel with color-coded output.
Press **Ctrl+C** once to stop both servers.

---

## Option 3: Bash Script (Background with Logs)

From the root directory:

```bash
./start-all.sh
```

This script:
- Clears ports automatically (no more "port in use" errors!)
- Starts both servers in the background
- Saves logs to `logs/backend.log` and `logs/frontend.log`
- Shows you the PIDs and URLs
- Press **Ctrl+C** to stop all servers

---

## Option 4: Manual (Two Terminals)

**Terminal 1 - Backend:**
```bash
cd /Users/matthew/Documents/SOU\ App/materials-server
npm start
```

**Terminal 2 - Frontend:**
```bash
cd /Users/matthew/Documents/SOU\ App/sou-song-browser
npm start
```

---

## Environment Variables (Configured for You)

Both servers now have `.env` files configured:

**Backend** (`materials-server/.env`):
- `PORT=3002`
- `FRONTEND_URL=http://localhost:3000`
- `MATERIALS_PATH=` (Google Drive path for PDFs)

**Frontend** (`sou-song-browser/.env`):
- `PORT=3000`
- `REACT_APP_API_URL=http://localhost:3002`
- `REACT_APP_MATERIALS_URL=http://localhost:3002`
- `BROWSER=none` (won't auto-open browser)

**These settings ensure the correct ports are always used!**

---

## Quick Access URLs

Once servers are running:

- **Frontend:** http://localhost:3000
- **Backend API:** http://localhost:3002
- **Admin Dashboard:** http://localhost:3000/admin
- **Search & Add Songs:** http://localhost:3000/admin (click "Search & Add Songs")
- **Bulk Add Songs:** http://localhost:3000/admin (click "Bulk Add Songs")
- **Manage Database:** http://localhost:3000/admin (click "Manage SOU Database")

---

## Troubleshooting

### ⚠️ Ports Already in Use

The `start-all.sh` script now **automatically clears ports**. But if you need to do it manually:

```bash
# Kill backend (port 3002)
lsof -ti:3002 | xargs kill -9

# Kill frontend (port 3000)
lsof -ti:3000 | xargs kill -9
```

### ❌ Servers Won't Start

**Reinstall dependencies:**
```bash
cd materials-server && npm install
cd ../sou-song-browser && npm install
```

### ⚠️ Backend Won't Connect

**Check if backend is running:**
```bash
curl http://localhost:3002/health
```

Should return: `{"status":"ok",...}`

### 🔄 Admin Dashboard Shows Zero Songs

This means the frontend is using the wrong port. The `.env` file fixes this, but if it happens:

1. Check that `sou-song-browser/.env` exists
2. Restart frontend with `npm start` (it will read `.env`)
3. Hard refresh browser (Cmd+Shift+R or Ctrl+Shift+R)

### 📄 PDF Buttons Don't Work

This means materials URL is wrong. The `.env` file fixes this, but if it happens:

1. Check that `sou-song-browser/.env` has `REACT_APP_MATERIALS_URL=http://localhost:3002`
2. Check that `materials-server/.env` has correct `MATERIALS_PATH` (Google Drive)
3. Restart frontend to load new environment variables

---

## Development Tips

### Watch Logs in Real-time

If using the bash script (`./start-all.sh`), you can watch logs:

```bash
# Watch backend logs
tail -f logs/backend.log

# Watch frontend logs
tail -f logs/frontend.log
```

### Restart Just One Server

If you need to restart just the backend or frontend:

```bash
# Kill backend only
lsof -ti:3002 | xargs kill -9
cd materials-server && npm start

# Kill frontend only
lsof -ti:3000 | xargs kill -9
cd sou-song-browser && npm start
```

---

## First Time Setup

If this is your first time running the app:

```bash
# Install all dependencies
cd /Users/matthew/Documents/SOU\ App
npm install
cd materials-server && npm install
cd ../sou-song-browser && npm install

# Then start servers
cd ..
npm start
```

---

## Status Check

**Are servers running?**

```bash
# Check processes
ps aux | grep node

# Check ports
lsof -i :3000  # Frontend
lsof -i :3002  # Backend
```

**Test endpoints:**

```bash
# Backend health check
curl http://localhost:3002/health

# Frontend (should return HTML)
curl http://localhost:3000
```

---

## Need Help?

- **Backend logs:** `logs/backend.log` or terminal output
- **Frontend logs:** `logs/frontend.log` or terminal output
- **Documentation:** See `FIELD_MAPPING_ARCHITECTURE.md` and `ADMIN_BACKEND_INTEGRATION_COMPLETE.md`
