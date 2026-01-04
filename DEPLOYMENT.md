# Deployment Guide: Skribally to Render and Vercel

This guide covers two deployment strategies for your Skribally application.

## Strategy 1: Full Stack on Render (Recommended for Socket.io)

This approach deploys both backend and frontend on Render, which is ideal for Socket.io applications.

### Prerequisites
- GitHub account
- Render account (sign up at https://render.com)
- Your code pushed to a GitHub repository

### Step 1: Prepare Your Repository

1. Ensure all your code is committed and pushed to GitHub:
   ```bash
   git add .
   git commit -m "Prepare for deployment"
   git push origin main
   ```

### Step 2: Deploy Backend on Render

1. **Go to Render Dashboard**: https://dashboard.render.com
2. **Click "New +"** → **"Web Service"**
3. **Connect your GitHub repository** (authorize Render if needed)
4. **Select your repository** (`skribally` or your repo name)
5. **Configure the service**:
   - **Name**: `skribally-backend` (or any name you prefer)
   - **Environment**: `Node`
   - **Build Command**: 
     ```
     npm install && npm install --prefix ./backend && npm install --prefix ./frontend && npm run build --prefix ./frontend
     ```
   - **Start Command**: 
     ```
     cd backend && node server.js
     ```
   - **Instance Type**: Choose based on your needs (Free tier available)

6. **Set Environment Variables** (in Render dashboard):
   - `NODE_ENV` = `production`
   - `PORT` = `10000` (Render will override this, but set it anyway)
   - `REACT_APP_NODE_ENV` = `production`

7. **Click "Create Web Service"**

8. **Wait for deployment** - Render will build and deploy your app. This may take 5-10 minutes.

9. **Note your Render URL** - It will be something like `https://skribally-backend.onrender.com`

### Step 3: Update Frontend Configuration

After deployment, update the frontend to use your Render backend URL:

1. **Update `frontend/src/views/PlayScreen.jsx`**:
   - Replace line 36 with your Render backend URL:
   ```javascript
   const ENDPOINT = "https://your-backend-name.onrender.com/";
   ```

2. **Commit and push**:
   ```bash
   git add .
   git commit -m "Update backend endpoint"
   git push origin main
   ```

3. **Render will automatically redeploy** with the new configuration.

### Step 4: Verify Deployment

1. Visit your Render URL: `https://your-backend-name.onrender.com`
2. The app should load and work correctly
3. Test multiplayer functionality by opening two browser tabs

---

## Strategy 2: Backend on Render + Frontend on Vercel

This approach separates frontend and backend, which can be better for scaling.

### Prerequisites
- GitHub account
- Render account
- Vercel account (sign up at https://vercel.com)

### Part A: Deploy Backend on Render

Follow **Step 2** from Strategy 1, but with these modifications:

1. **Build Command**: 
   ```
   npm install && npm install --prefix ./backend
   ```
   (Don't build frontend here)

2. **Start Command**: 
   ```
   cd backend && node server.js
   ```

3. **Environment Variables**:
   - `NODE_ENV` = `production`
   - `PORT` = `10000`
   - `CORS_ORIGIN` = `https://your-vercel-app.vercel.app` (you'll update this after Vercel deployment)

4. **Note your Render backend URL** (e.g., `https://skribally-backend.onrender.com`)

### Part B: Update Backend CORS Settings

1. **Update `backend/server.js`** to allow your Vercel domain:
   ```javascript
   const io = new Server(server, {
     pingTimeout: 60000,
     cors: {
       origin: process.env.CORS_ORIGIN || "https://your-vercel-app.vercel.app",
       credentials: true
     },
     connectionStateRecovery: {},
   });
   ```

2. **Also update Express CORS**:
   ```javascript
   app.use(cors({
     origin: process.env.CORS_ORIGIN || "https://your-vercel-app.vercel.app",
     credentials: true
   }));
   ```

### Part C: Deploy Frontend on Vercel

1. **Go to Vercel Dashboard**: https://vercel.com/dashboard
2. **Click "Add New..."** → **"Project"**
3. **Import your GitHub repository**
4. **Configure the project**:
   - **Framework Preset**: `Create React App`
   - **Root Directory**: `frontend`
   - **Build Command**: `npm run build` (should auto-detect)
   - **Output Directory**: `build` (should auto-detect)

5. **Set Environment Variables**:
   - `REACT_APP_NODE_ENV` = `production`
   - `REACT_APP_API_URL` = `https://your-backend-name.onrender.com`

6. **Click "Deploy"**

7. **Note your Vercel URL** (e.g., `https://skribally.vercel.app`)

### Part D: Update Frontend to Use Environment Variables

1. **Update `frontend/src/views/PlayScreen.jsx`**:
   ```javascript
   const ENDPOINT = process.env.REACT_APP_API_URL || "https://your-backend-name.onrender.com/";
   const ENDPOINT_LOCAL = "http://localhost:3001/";
   
   useEffect(() => {
     // ... existing code ...
     const newSocket = io.connect(
       process.env.REACT_APP_NODE_ENV === "production" ? ENDPOINT : ENDPOINT_LOCAL
     );
     // ... rest of code ...
   }, []);
   ```

2. **Update Render environment variable**:
   - Go back to Render dashboard
   - Update `CORS_ORIGIN` to your Vercel URL: `https://your-vercel-app.vercel.app`

3. **Redeploy both services**:
   - Push changes to GitHub (Vercel auto-deploys)
   - Trigger manual redeploy on Render

---

## Important Notes

### Render Free Tier Limitations
- Services spin down after 15 minutes of inactivity
- First request after spin-down may take 30-60 seconds
- Consider upgrading for production use

### Socket.io on Render
- Socket.io works well on Render
- Ensure CORS is properly configured
- Use WebSocket transport (default)

### Environment Variables
- Never commit `.env` files to Git
- Use platform-specific environment variable settings
- Update CORS origins when deploying to new domains

### Troubleshooting

**Issue: CORS errors**
- Solution: Check CORS configuration in `server.js` and ensure your frontend URL is whitelisted

**Issue: Socket.io connection fails**
- Solution: Verify the backend URL is correct and CORS allows your frontend origin

**Issue: Build fails on Render**
- Solution: Check build logs, ensure all dependencies are in `package.json`

**Issue: Frontend can't connect to backend**
- Solution: Verify environment variables are set correctly in both platforms

---

## Quick Reference

### Render Backend URL Format
```
https://your-service-name.onrender.com
```

### Vercel Frontend URL Format
```
https://your-project-name.vercel.app
```

### Environment Variables Checklist

**Render (Backend)**:
- `NODE_ENV` = `production`
- `PORT` = `10000`
- `CORS_ORIGIN` = Your Vercel URL (if using Strategy 2)

**Vercel (Frontend)**:
- `REACT_APP_NODE_ENV` = `production`
- `REACT_APP_API_URL` = Your Render backend URL

---

## Next Steps

1. Set up custom domains (optional)
2. Configure SSL certificates (automatic on both platforms)
3. Set up monitoring and logging
4. Consider database integration if needed
5. Set up CI/CD for automatic deployments

For more help, refer to:
- [Render Documentation](https://render.com/docs)
- [Vercel Documentation](https://vercel.com/docs)

