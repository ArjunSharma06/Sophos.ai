# Production Deployment Guide

## Problem
YouTube processing works locally but fails in production because the **backend Express server is not deployed**.

## Architecture
- **Frontend**: Next.js on Vercel (`https://sophos7.vercel.app`)
- **Backend**: Express server (needs to be deployed to Render)
- **Database**: Supabase

## Step-by-Step Deployment

### 1. Deploy Backend to Render

#### Option A: Using Render Dashboard (Recommended)

1. **Go to Render Dashboard**: https://dashboard.render.com/
2. **Create New Web Service**:
   - Click "New +" → "Web Service"
   - Connect your GitHub repository
   - Select the repository: `Sophos.ai`

3. **Configure Build Settings**:
   - **Name**: `sophos-backend` (or any name you prefer)
   - **Root Directory**: `SophosBackEnd`
   - **Runtime**: `Node`
   - **Build Command**: `npm install && npm run build`
   - **Start Command**: `npm start`
   - **Instance Type**: Free (or paid if needed)

4. **Set Environment Variables**:
   Click "Environment" tab and add these variables:
   
   ```
   NODE_ENV=production
   PORT=3001
   SUPABASE_URL=<your-supabase-project-url>
   SUPABASE_ANON_KEY=<your-supabase-anon-key>
   SUPABASE_SERVICE_ROLE_KEY=<your-supabase-service-role-key>
   GROQ_API_KEY=<your-groq-api-key>
   ```

   **Where to find these values:**
   - Supabase credentials: Supabase Dashboard → Project Settings → API
   - GROQ API Key: Your GROQ account dashboard

5. **Deploy**:
   - Click "Create Web Service"
   - Wait for deployment to complete (5-10 minutes)
   - **Copy the service URL** (e.g., `https://sophos-backend-xyz.onrender.com`)

#### Option B: Using render.yaml (Automatic)

1. The `render.yaml` file is already created in `SophosBackEnd/render.yaml`
2. In Render Dashboard:
   - Go to "Blueprint" → "New Blueprint Instance"
   - Connect your repository
   - Render will automatically detect the `render.yaml` file
   - Set the environment variables manually (they're marked as `sync: false` for security)

### 2. Update Vercel Environment Variables

1. **Go to Vercel Dashboard**: https://vercel.com/dashboard
2. **Select your project**: `Sophos.ai`
3. **Go to Settings** → **Environment Variables**
4. **Add/Update this variable**:
   
   ```
   NEXT_PUBLIC_API_BASE_URL=https://sophos-backend-xyz.onrender.com
   ```
   
   Replace `sophos-backend-xyz.onrender.com` with your actual Render backend URL.

5. **Important**: Set this for all environments:
   - ✅ Production
   - ✅ Preview
   - ✅ Development (optional)

6. **Redeploy Frontend**:
   - Go to "Deployments" tab
   - Click "..." on the latest deployment
   - Click "Redeploy"
   - This ensures the new environment variable is picked up

### 3. Verify CORS Settings

The backend already has CORS configured for your Vercel URL. Verify in `SophosBackEnd/src/index.ts`:

```typescript
const whitelist = [
  'https://sophos7.vercel.app', // ✅ Your Vercel URL
  'http://localhost:3000'
];
```

If your Vercel URL is different, update this whitelist.

### 4. Test Production Deployment

1. **Wait for both deployments to complete**:
   - Backend on Render (check Render dashboard)
   - Frontend on Vercel (check Vercel dashboard)

2. **Test the YouTube feature**:
   - Go to `https://sophos7.vercel.app`
   - Log in
   - Try processing a YouTube URL
   - Check browser console for errors

3. **Check Backend Logs**:
   - Go to Render Dashboard → Your Service → Logs
   - Look for incoming requests and any errors

## Troubleshooting

### Issue: Still getting 404 errors

**Check:**
1. Backend is deployed and running on Render
2. `NEXT_PUBLIC_API_BASE_URL` is set correctly in Vercel
3. Frontend has been redeployed after setting the environment variable

**Debug:**
```bash
# Check what URL the frontend is using
# Open browser console on production site and run:
console.log(process.env.NEXT_PUBLIC_API_BASE_URL)
```

### Issue: CORS errors

**Check:**
1. Your Vercel URL is in the CORS whitelist in `SophosBackEnd/src/index.ts`
2. Backend logs show CORS errors

**Fix:**
Update the whitelist and redeploy backend:
```typescript
const whitelist = [
  'https://your-actual-vercel-url.vercel.app',
  'http://localhost:3000'
];
```

### Issue: Authentication errors

**Check:**
1. All Supabase environment variables are set correctly on Render
2. The same Supabase project is used for both frontend and backend

**Verify:**
- Frontend uses: `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- Backend uses: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY`

### Issue: Backend crashes on startup

**Check Render logs for:**
- Missing environment variables
- Build errors
- Port binding issues

**Common fixes:**
- Ensure all required environment variables are set
- Check that `package.json` has correct build and start scripts
- Verify Node.js version compatibility

## Environment Variables Checklist

### Render (Backend)
- [ ] `NODE_ENV=production`
- [ ] `PORT=3001`
- [ ] `SUPABASE_URL`
- [ ] `SUPABASE_ANON_KEY`
- [ ] `SUPABASE_SERVICE_ROLE_KEY`
- [ ] `GROQ_API_KEY`

### Vercel (Frontend)
- [ ] `NEXT_PUBLIC_API_BASE_URL` (your Render backend URL)
- [ ] `NEXT_PUBLIC_SUPABASE_URL`
- [ ] `NEXT_PUBLIC_SUPABASE_ANON_KEY`

## Quick Reference

**Backend URL format**: `https://sophos-backend-xyz.onrender.com`
**Frontend URL**: `https://sophos7.vercel.app`
**API endpoint**: `https://sophos-backend-xyz.onrender.com/api/youtube/process`

## Next Steps

After deployment:
1. Test all features (YouTube, PDF upload, chat, quiz)
2. Monitor Render logs for errors
3. Set up monitoring/alerts (optional)
4. Consider upgrading Render instance if performance is slow (free tier has cold starts)
