# Firebase Setup Guide for Shared History

This guide will help you set up Firebase Firestore to enable shared history across all users of your Ruleta EOS application.

## Why Firebase?

Firebase Firestore provides:
- **Free tier**: Generous free quota (50K reads/day, 20K writes/day)
- **Real-time sync**: All users see updates instantly
- **No backend server**: Everything runs client-side
- **Easy setup**: Just a few configuration steps

## Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Add project" or select an existing project
3. Follow the setup wizard:
   - Enter a project name (e.g., "ruleta-eos")
   - Disable Google Analytics (optional, not needed for this)
   - Click "Create project"

## Step 2: Enable Firestore Database

1. In your Firebase project, click "Firestore Database" in the left sidebar
2. Click "Create database"
3. Choose "Start in test mode" (for now - you can secure it later)
4. Select a location closest to your users
5. Click "Enable"

## Step 3: Get Your Firebase Configuration

1. In Firebase Console, click the gear icon ⚙️ next to "Project Overview"
2. Select "Project settings"
3. Scroll down to "Your apps" section
4. Click the web icon `</>` to add a web app
5. Register your app (you can use any nickname, e.g., "Ruleta EOS Web")
6. Copy the `firebaseConfig` object that appears

## Step 4: Update Your HTML File

Open `index.html` and find this section (around line 7-20):

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Replace the placeholder values with your actual Firebase config values from Step 3.

## Step 5: Set Up Firestore Security Rules (Important!)

1. In Firebase Console, go to "Firestore Database" → "Rules"
2. Replace the default rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow read/write access to ruleta_sessions collection
    match /ruleta_sessions/{sessionId} {
      allow read, write: if true; // Public access for now
    }
  }
}
```

**Note**: These rules allow anyone to read/write. For production, you might want to add:
- Rate limiting
- Authentication requirements
- Data validation

## Step 6: Set Up Firestore Security Rules (CRITICAL!)

**⚠️ THIS IS THE MOST COMMON ISSUE!** If your app is still using local storage, it's likely because Firestore security rules are blocking access.

1. In Firebase Console, go to **"Firestore Database"** → **"Rules"** tab
2. Click **"Edit rules"**
3. Replace the default rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Allow read/write access to ruleta_sessions collection
    match /ruleta_sessions/{sessionId} {
      allow read, write: if true; // Public access for now
    }
  }
}
```

4. Click **"Publish"** to save the rules
5. Wait a few seconds for the rules to propagate

**Important**: The default rules in "test mode" expire after 30 days. Make sure you've set up permanent rules!

## Step 7: Test It Out

1. Save your `index.html` file
2. Open it in a browser (or deploy to GitHub Pages)
3. Open the browser console (F12 → Console tab)
4. Look for diagnostic messages - the app will automatically run a Firebase status check
5. Create a session and click "Guardar sesión"
6. Check the console for any errors
7. Open the app in another browser/device - you should see the same history!

### Manual Diagnostic

If you want to manually check Firebase status, open the browser console and run:
```javascript
checkFirebaseStatus()
```

This will show you:
- Whether Firebase is initialized
- Whether Firestore connection works
- Any permission errors
- Specific error codes and messages

## Troubleshooting

### Still seeing "Historial local" (Local history)?

**Most likely cause: Firestore Security Rules**

1. **Check browser console** - Look for errors like "permission-denied" or "PERMISSION DENIED"
2. **Verify Firestore Rules**:
   - Go to Firebase Console → Firestore Database → Rules
   - Make sure rules allow `read, write: if true` for `ruleta_sessions` collection
   - Click "Publish" after making changes
   - Wait 10-30 seconds for rules to propagate
3. **Run diagnostic**: Open browser console and type `checkFirebaseStatus()`
4. **Check Firestore is enabled**: Firebase Console → Firestore Database should show your database

### "Firebase not initialized" error
- Check that you've replaced all placeholder values in `firebaseConfig`
- Verify your Firebase project is active
- Check browser console for module loading errors
- Make sure you're using HTTPS (Firebase requires secure context)

### "Permission denied" error
- **This is the #1 issue!** Check your Firestore security rules
- Make sure rules allow read/write access to `ruleta_sessions`
- Rules must be published (not just saved)
- Wait a few seconds after publishing for rules to take effect

### History not syncing
- Check browser console for errors (F12 → Console)
- Verify Firestore is enabled in Firebase Console
- Check that the collection name matches: `ruleta_sessions`
- Run `checkFirebaseStatus()` in console for detailed diagnostics
- Make sure you're testing from different browsers/devices (not just different tabs)

### Still using local storage?
- The app automatically falls back to localStorage if Firebase isn't configured or has errors
- Check the status indicator in the History tab - it will show "Historial local" if Firebase isn't working
- Look at browser console for error messages
- Most common issue: **Firestore security rules blocking access**

## Alternative Solutions

If Firebase doesn't work for you, here are other options:

### Option 2: GitHub Gists API (Free, but rate-limited)
- Store history as a JSON file in a GitHub Gist
- Requires GitHub Personal Access Token
- Rate limit: 5,000 requests/hour

### Option 3: Serverless Functions + Database
- Use Vercel/Netlify Functions with a database
- More setup required but more control
- Free tiers available

### Option 4: Backend Server
- Set up your own server (Node.js, Python, etc.)
- Use PostgreSQL, MongoDB, or similar
- Requires hosting (Heroku, Railway, etc.)

## Need Help?

- Firebase Documentation: https://firebase.google.com/docs/firestore
- Firestore Security Rules: https://firebase.google.com/docs/firestore/security/get-started
