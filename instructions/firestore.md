# Moira — Firebase & Firestore Setup Guide

This guide will help you set up the online database that allows your reporters to collaborate in a real-time digital newsroom. Once configured, they can participate using any machine connected to the internet.

## 1. Create a Firebase Project

![[Screenshot 2026-04-13 at 09.20.53.png]]


1. Go to [https://console.firebase.google.com](https://console.firebase.google.com) and sign in with your Google account.
2. Click **Add project** or Click the big "Get started..." option. 
3. Give it a name (e.g. `moira-newsroom`).

![[Pasted image 20260413092330.png]]

4. If prompted, Disable Gemini in Firebase/AI assistance
5. Disable Google Analytics/*Google Analytics for your Firebase project* if you don't need it — it has no effect on Moira.
6. Click **Create project** and wait for it to provision.
7. When ready, press **Continue.**

![[Pasted image 20260413092550.png]]


## 2. Register a Web App

![[Pasted image 20260413094455.png]]

1. From your project dashboard, click the **+Add App** button.
![[Screenshot 2026-04-13 at 09.45.12.png]]

2. Select the web option (</>)
3. Give the app a nickname (e.g. `moira`).
4. Do **not** enable Firebase Hosting
5. Click **Register app**.

![[Pasted image 20260413094646.png]]


5. When prompted, select **Use a Script tag**

![[Pasted image 20260413095046.png]]
6. Firebase will display a config script. The important part is the "You web app's Firebase configuration". It will look something like the code below.
   
```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

6. **Copy this now!** and save it somewhere — you'll need it in [Step 6](#6-connect-moira-to-firebase).
7. Click **Continue to console**

## 3. Enable Anonymous Authentication

![[Screenshot 2026-04-13 at 09.56.28.png]]

Moira signs users in anonymously on load — no login screen required. Each browser session gets a unique anonymous identity that Firestore uses for access control.

1. In the Firebase Console, from the Sidebar, select  **Security → Authentication**.
2. Click **Get started**.

![[Screenshot 2026-04-13 at 09.58.16.png]]


3. Under the **Sign-in method** tab, find **Anonymous** and select it
4. Toggle it **Enabled** and click **Save**.

![[Screenshot 2026-04-13 at 09.58.49.png]]


## 4. Create the Firestore Database

![[Screenshot 2026-04-13 at 10.01.42.png]]

1. From the Sidebar, select **Database & Storage → Firestore**
2. Select Standard Edition and click **Next**
3. Select an appropriate **Location** for your database e.g. `europe-west2(london)` and click **Next.**
4. Choose **Start in production mode** — you will set up your own rules in the next step.
5. Select a region close to your location (e.g. `europe-west2` for London).
6. Click **Create**.
7.
---

## 5. Set Firestore Security Rules
![[Screenshot 2026-04-13 at 10.28.33.png]]
Moira uses three *collections* to store information about the newsdays. Setting rules for the collections give the correct read/write access for moira to function. Replace the default rules with the following:

1. In the Firebase Console, go to **Firestore Database → Rules**.
2. Replace the default rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Bulletin documents — rundown metadata, order, staff, TX state
    match /artifacts/{appId}/public/data/bulletins/{bulletinId} {
      allow read, write: if request.auth != null;

      // Messages sub-collection — cross-device producer alerts (volatile)
      match /messages/{messageId} {
        allow read, write: if request.auth != null;
      }
    }

    // Story documents — scripts, formats, durations, lock state
    match /artifacts/{appId}/public/data/rundown_stories/{storyId} {
      allow read, write: if request.auth != null;
    }

  }
}
```

3. Click **Publish**.

---

## 6. Connect Moira to Firebase

Moira reads its Firebase config from `localStorage`. The easiest way to set it is through the built-in Settings panel:

1. Open `moira.html` in Chrome or Edge.
2. Click the **gear icon** in the top-right header.
3. The **Connection** tab is shown by default.
4. Paste your full `firebaseConfig` object from Step 2 into the paste field — Moira will extract the individual values automatically, or fill in the four fields manually:
   - **apiKey**
   - **authDomain**
   - **projectId**
   - **storageBucket**
5. Click **Save & Reconnect**. The page will reload and connect.

Once connected, the header badge will change from **Local Mode** to **Synced**.
