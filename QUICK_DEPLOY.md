# Quick Deployment Checklist

## Option 1: Everything on Render (Easiest)

### Steps:
1. ✅ Push code to GitHub
2. ✅ Go to https://dashboard.render.com
3. ✅ New → Web Service → Connect GitHub repo
4. ✅ Configure:
   - **Build Command**: `npm install && npm install --prefix ./backend && npm install --prefix ./frontend && npm run build --prefix ./frontend`
   - **Start Command**: `cd backend && node server.js`
   - **Environment**: Node
5. ✅ Add Environment Variables:
   - `NODE_ENV` = `production`
   - `PORT` = `10000`
6. ✅ Deploy and wait (~5-10 min)
7. ✅ Update `frontend/src/views/PlayScreen.jsx` line 36 with your Render URL
8. ✅ Push changes → Auto-redeploy

**Done!** Your app is live at `https://your-service.onrender.com`

---

## Option 2: Backend (Render) + Frontend (Vercel)

### Backend on Render:
1. ✅ Same as Option 1, but:
   - **Build Command**: `npm install && npm install --prefix ./backend`
   - **Start Command**: `cd backend && node server.js`
   - Add `CORS_ORIGIN` env var (update after Vercel deploy)

### Frontend on Vercel:
1. ✅ Go to https://vercel.com/dashboard
2. ✅ New Project → Import GitHub repo
3. ✅ Configure:
   - **Root Directory**: `frontend`
   - **Framework**: Create React App
4. ✅ Add Environment Variables:
   - `REACT_APP_NODE_ENV` = `production`
   - `REACT_APP_API_URL` = `https://your-backend.onrender.com`
5. ✅ Deploy
6. ✅ Update Render `CORS_ORIGIN` with Vercel URL
7. ✅ Redeploy Render service

**Done!** Frontend: `https://your-app.vercel.app` | Backend: `https://your-backend.onrender.com`

---

## Environment Variables Reference

### Render (Backend):
```
NODE_ENV=production
PORT=10000
CORS_ORIGIN=https://your-frontend.vercel.app  (only if using Option 2)
```

### Vercel (Frontend):
```
REACT_APP_NODE_ENV=production
REACT_APP_API_URL=https://your-backend.onrender.com
```

---

## Common Issues

**CORS Error?**
- Check `CORS_ORIGIN` matches your frontend URL exactly
- Ensure no trailing slashes in URLs

**Socket.io not connecting?**
- Verify backend URL in `PlayScreen.jsx`
- Check CORS settings in `server.js`

**Build failing?**
- Check Render/Vercel build logs
- Ensure all dependencies in `package.json`

