# Sign Up / Sign In Troubleshooting Guide

## Understanding the Errors You're Seeing

### ❌ The Chrome Extension Error (NOT YOUR APP'S FAULT!)

The errors you're seeing:
```
Uncaught (in promise) TypeError: Error in invocation of tabs.get(integer tabId, function callback): 
Error at parameter 'tabId': Value must be at least 0.
```

**THIS IS NOT FROM YOUR APPLICATION!**

This error is coming from a **browser extension** you have installed that's trying to use Chrome's extension APIs incorrectly. Your AstroTalk Insight application does NOT use any Chrome extension APIs.

#### How to Identify the Problematic Extension:

1. Open Chrome DevTools (F12)
2. Look at the error stack trace - it shows `background.js:23:178626`
3. This is from an extension's background script
4. To find which extension:
   - Go to `chrome://extensions/`
   - Enable "Developer mode" (top right)
   - Look for extensions with background scripts
   - Disable them one by one to find the culprit

#### Common Extensions That Cause This:
- Ad blockers
- Tab managers
- Productivity extensions
- Screenshot tools
- Download managers

#### Quick Fix:
1. Test your app in **Incognito Mode** (extensions are usually disabled)
2. Or temporarily disable all extensions and re-enable them one by one

---

## ✅ Actual Deployment Issues to Check

### 1. Backend Status Check

**Your Backend:** https://astrotalkinsight.onrender.com

#### Test Backend Health:
```bash
curl https://astrotalkinsight.onrender.com/health
```

**Expected Response:**
```json
{"status": "healthy"}
```

If you get an error:
- ❌ Backend might be sleeping (Render free tier spins down after inactivity)
- ❌ Backend deployment failed
- ✅ Solution: Check https://dashboard.render.com

---

### 2. Frontend Environment Variables (CRITICAL!)

**Your Frontend:** https://astrotalkinsight.vercel.app

The frontend MUST have these environment variables set on Vercel:

| Variable | Value |
|----------|-------|
| `VITE_API_URL` | `https://astrotalkinsight.onrender.com` |
| `VITE_RAZORPAY_KEY_ID` | `rzp_test_SAOJ9udbL5iqeF` |

#### How to Check if Variables are Set:

1. **Option 1: Open deployed site**
   - Visit: https://astrotalkinsight.vercel.app
   - Open Console (F12)
   - Type: `import.meta.env`
   - Check if VITE_API_URL shows the correct backend URL

2. **Option 2: Check Vercel Dashboard**
   - Go to https://vercel.com/dashboard
   - Select your project
   - Settings → Environment Variables
   - Verify both variables exist

#### If Variables Are Missing:

**Add them via Vercel Dashboard:**

1. Go to https://vercel.com/dashboard
2. Click on your `astrotalkinsight` project
3. Navigate to **Settings** → **Environment Variables**
4. Click **Add Variable**
5. Add:
   ```
   Name: VITE_API_URL
   Value: https://astrotalkinsight.onrender.com
   Environments: ✅ Production ✅ Preview ✅ Development
   ```
6. Click **Add Variable** again
7. Add:
   ```
   Name: VITE_RAZORPAY_KEY_ID
   Value: rzp_test_SAOJ9udbL5iqeF
   Environments: ✅ Production ✅ Preview ✅ Development
   ```
8. Click **Save**
9. **IMPORTANT:** Trigger a new deployment:
   - Go to **Deployments** tab
   - Click the ⋯ menu on latest deployment
   - Select **Redeploy**

---

### 3. CORS Configuration

Your backend needs to allow requests from your frontend domain.

#### Current Allowed Origins (in Backend main.py):
```python
origins = [
    "https://astrotalkinsight.vercel.app",
    "https://astrotalkinsight.com",
    "http://localhost:5173",
    # ... etc
]
```

#### Test CORS:
```bash
curl -H "Origin: https://astrotalkinsight.vercel.app" \
     -H "Access-Control-Request-Method: POST" \
     -H "Access-Control-Request-Headers: Content-Type" \
     -X OPTIONS \
     https://astrotalkinsight.onrender.com/signup
```

**Expected:** Should return CORS headers in response

---

### 4. MongoDB Connection

Your backend uses MongoDB. Check if the connection is working:

#### Check Backend Logs:
1. Go to https://dashboard.render.com
2. Select your `astrotalkinsight` service
3. Click **Logs** tab
4. Look for:
   - ✅ "Connected to MongoDB"
   - ❌ "Failed to connect to MongoDB"
   - ❌ Any authentication errors

#### MongoDB Connection String:
Make sure your `.env` on Render has:
```
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/astrotalk?retryWrites=true&w=majority
```

---

### 5. Network Tab Debugging

**When testing Sign Up / Sign In on deployed site:**

1. Open DevTools (F12)
2. Go to **Network** tab
3. Try to sign up
4. Look for the request to `/signup`

#### What to Check:

**Request URL:**
- Should be: `https://astrotalkinsight.onrender.com/signup`
- NOT: `http://localhost:8000/signup`
- If it's localhost → Environment variables not set on Vercel!

**Status Code:**
- `200 OK` ✅ Working!
- `400 Bad Request` → Check request body format
- `404 Not Found` → Wrong API URL
- `500 Internal Server Error` → Backend error (check Render logs)
- `CORS error` → Backend CORS configuration issue
- `Failed to fetch` → Backend is down or unreachable

**Response:**
- Check the response body for error details

---

## 🔧 Complete Diagnosis Steps

### Step 1: Use the Test Tool

I've created a comprehensive test tool for you at:
```
d:\client\deployment-test.html
```

**How to use it:**

1. **Test Locally First:**
   ```bash
   # Open the file in your browser
   start d:\client\deployment-test.html
   ```
   This tests if the backend is responsive

2. **Test from Deployed Site:**
   - Upload `deployment-test.html` to your Vercel deployment
   - Or use any static file hosting
   - Open it from the same origin as your app

### Step 2: Check Each Component

Run these checks in order:

#### ✅ Check 1: Backend Health
```bash
curl https://astrotalkinsight.onrender.com/health
```
Expected: `{"status": "healthy"}`

#### ✅ Check 2: Frontend Env Vars
1. Open: https://astrotalkinsight.vercel.app
2. Console: `import.meta.env.VITE_API_URL`
3. Should show: `"https://astrotalkinsight.onrender.com"`

#### ✅ Check 3: API Connection
```bash
curl -X POST https://astrotalkinsight.onrender.com/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"testpass123","full_name":"Test User"}'
```

#### ✅ Check 4: CORS from Browser
1. Go to: https://astrotalkinsight.vercel.app
2. Open Console
3. Run:
```javascript
fetch('https://astrotalkinsight.onrender.com/health')
  .then(r => r.json())
  .then(console.log)
  .catch(console.error)
```

---

## 🎯 Common Issues & Solutions

### Issue 1: "Unable to connect to server"

**Symptoms:**
- Sign up button loading forever
- Error: "Unable to connect to server"

**Causes & Solutions:**

1. **Backend is sleeping (Render free tier)**
   - Solution: Visit backend URL to wake it up
   - It takes ~1 minute to spin up

2. **Wrong API URL**
   - Check: Console shows `undefined` for `import.meta.env.VITE_API_URL`
   - Solution: Add environment variables to Vercel (see above)

3. **Backend is down**
   - Check: https://astrotalkinsight.onrender.com/health
   - Solution: Check Render dashboard for deployment errors

### Issue 2: CORS Error

**Symptoms:**
- Console shows: "CORS policy: No 'Access-Control-Allow-Origin' header"

**Solution:**
1. Your frontend origin needs to be in backend's allowed origins
2. Check `Backend/main.py` line 14-27
3. Add your Vercel URL if missing:
   ```python
   origins = [
       "https://astrotalkinsight.vercel.app",
       "https://your-custom-domain.com",  # Add this if you have one
       # ...
   ]
   ```
4. Push changes to trigger Render redeploy

### Issue 3: "Email already registered"

**Symptoms:**
- Sign up fails with "Email already registered"

**This is actually WORKING!** ✅
- The error means authentication is working
- The email you're using already exists
- Solution: Use a different email OR use the login form instead

### Issue 4: MongoDB Connection Error

**Symptoms:**
- Sign up/Login returns 500 error
- Backend logs show: "Failed to connect to MongoDB"

**Solution:**
1. Check Render environment variables
2. Verify `MONGO_URI` is set correctly
3. Check MongoDB Atlas:
   - Network Access: Allow connections from `0.0.0.0/0`
   - Database User: Correct username/password

---

## 🚀 Quick Fix Checklist

Run through this checklist:

- [ ] Backend is responding at https://astrotalkinsight.onrender.com/health
- [ ] Vercel has `VITE_API_URL` environment variable set
- [ ] Vercel has `VITE_RAZORPAY_KEY_ID` environment variable set
- [ ] Redeployed Vercel after adding environment variables
- [ ] MongoDB connection is working (check Render logs)
- [ ] CORS allows your Vercel URL
- [ ] Tested in Incognito mode (to rule out extension interference)
- [ ] Network tab shows requests going to correct backend URL

---

## 🧪 Manual Testing Steps

### Test Signup Flow:

1. Open: https://astrotalkinsight.vercel.app
2. Open DevTools (F12) → Network tab
3. Click "Sign In" button in navbar
4. Switch to "Sign Up" tab
5. Fill in:
   - Name: Test User
   - Email: test{random}@example.com (use unique email)
   - Password: testpass123
6. Click "Sign Up"
7. **Check Network Tab:**
   - Look for POST request to `/signup`
   - Request URL should be: `https://astrotalkinsight.onrender.com/signup`
   - Status should be: `200 OK`
   - Response should have user data

### Test Login Flow:

1. After successful signup, try logging in with same credentials
2. Should get access token
3. Should see user name in navbar

---

## 📞 If Still Not Working

If you've checked everything above and it's still not working:

1. **Open deployment-test.html** and run all tests
2. **Share these details:**
   - Test results from deployment-test.html
   - Network tab screenshot showing the failed request
   - Response body of the failed request
   - Browser console errors (excluding the extension error!)
   - Render logs from the time of the request

3. **Check specific places:**
   - Vercel deployment logs
   - Render deployment logs
   - MongoDB Atlas logs (if accessible)

---

## ✅ Success Indicators

You'll know everything is working when:

1. ✅ `deployment-test.html` shows all green checks
2. ✅ Sign up creates a new user successfully
3. ✅ Login returns an access token
4. ✅ User's name appears in the navbar after login
5. ✅ "Get Started" form works and generates PDF report
6. ✅ Payment flow works (if testing Razorpay)

---

## 🔍 Advanced Debugging

### Enable Detailed Console Logging:

Add this to your browser console on the deployed site:

```javascript
// Override fetch to log all API calls
const originalFetch = window.fetch;
window.fetch = function(...args) {
    console.log('API Call:', args[0], args[1]);
    return originalFetch.apply(this, args)
        .then(response => {
            console.log('API Response:', response.status, response.url);
            return response;
        })
        .catch(error => {
            console.error('API Error:', error);
            throw error;
        });
};
```

This will log every API call and help you see exactly what's happening.

---

## Summary

**The Chrome extension error is NOT your app's problem!** Test in Incognito mode to verify.

**Most likely issue:** Environment variables not set on Vercel. Follow the steps above to fix it.

**Use the test tool:** `deployment-test.html` to diagnose all issues automatically.
