# Moira — Firebase & Firestore Setup Guide

This guide will help you set up the "Invisible Server" that allows your reporters to collaborate in a real-time digital newsroom. Once configured, they can participate using any machine connected to the internet.

---

## 1. Create a Firebase Project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com) and sign in with your Google account.
2. Click **Add project**.
3. Give it a name (e.g. `moira-newsroom`).
4. Disable Google Analytics if you don't need it — it has no effect on Moira.
5. Click **Create project** and wait for it to provision.

---

## 2. Register a Web App

1. From your project dashboard, click the **Web** icon (`</>`) to add a web app.
2. Give the app a nickname (e.g. `moira`).
3. Do **not** enable Firebase Hosting unless you intend to use it.
4. Click **Register app**.
5. Firebase will display your config object — it looks like this:

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

6. **Copy this now!** — you'll need it in [Step 6](#6-connect-moira-to-firebase).

---

## 3. Enable Anonymous Authentication

Moira signs users in anonymously on load — no login screen required. Each browser session gets a unique anonymous identity that Firestore uses for access control.

1. In the Firebase Console, go to **Build → Authentication**.
2. Click **Get started**.
3. Under the **Sign-in method** tab, find **Anonymous** and click it.
4. Toggle it **Enabled** and click **Save**.

---

## 4. Create the Firestore Database

1. Go to **Build → Firestore Database**.
2. Click **Create database**.
3. Choose **Start in production mode** — you will set up your own rules in the next step.
4. Select a region close to your location (e.g. `europe-west2` for London).
5. Click **Enable**.

---

## 5. Set Firestore Security Rules

Moira uses three collections. All require authenticated read/write access. Replace the default rules with the following:

1. In the Firebase Console, go to **Firestore Database → Rules**.
2. Replace the entire contents with:

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

> **Why these paths?** Moira stores data under `artifacts/{appId}/public/data/` where `appId` defaults to `moira-default`. The bulletin and stories sit at the same level. Messages are a sub-collection under each bulletin, so they are scoped to the correct room automatically.

---

## 6. Connect Moira to Firebase

Moira reads its Firebase config from `localStorage`. The easiest way to set it is through the built-in Settings panel:

1. Open `moira.html` in Chrome or Edge.
2. Click the **gear icon** (⚙) in the top-right header.
3. The **Connection** tab is shown by default.
4. Paste your full `firebaseConfig` object from Step 2 into the paste field — Moira will extract the individual values automatically, or fill in the four fields manually:
   - **apiKey**
   - **authDomain**
   - **projectId**
   - **storageBucket**
5. Click **Save & Reconnect**. The page will reload and connect.

Once connected, the header badge will change from **Local Mode** to **Synced**.

> **Alternative — hard-code the config:** If you are deploying Moira from a server and want the config baked in without any setup step, add this before the closing `</head>` tag:
> ```html
> <script>
>   window.MOIRA_CONFIG = {
>     apiKey: "...",
>     authDomain: "...",
>     projectId: "...",
>     storageBucket: "..."
>   };
> </script>
> ```
> Moira checks for `window.MOIRA_CONFIG` before falling back to `localStorage`.

---

## 7. Configure Message TTL (Recommended)

Producer alert messages are written to the `messages` sub-collection. They are deliberately short-lived — connected clients receive them as toasts, and they should not accumulate indefinitely.

Set up a Firestore TTL policy to auto-delete them:

1. In the Firebase Console, go to **Firestore Database → TTL policies**.
2. Click **Add TTL policy**.
3. Set:
   - **Collection group:** `messages`
   - **Field:** `timestamp`
4. Click **Save**.

Firebase will automatically delete message documents approximately 10 minutes after their `timestamp` value. There is no charge for TTL deletions.

> Note: TTL deletion is eventual — documents may persist slightly beyond the expiry time, but they will never be delivered to new clients because the listener filters on `timestamp > SESSION_START`.

---

## 8. Verify the Data Structure

After loading Moira with a valid config and clicking around, you should see the following structure appear in **Firestore Database → Data**:

```
artifacts/
  moira-default/
    public/
      data/
        bulletins/
          bulletin-primary/          ← default "Main" room
            order: [...]
            staff: [...]
            isTXMode: false
            ...
            messages/                ← producer alert sub-collection
              {messageId}/
                senderName: "..."
                text: "..."
                timestamp: ...
        rundown_stories/
          {storyId}/
            bulletinId: "bulletin-primary"
            slug: "TOP STORY"
            script: "..."
            duration: 90
            ...
```

Each named room creates a new bulletin document (e.g. `bulletin-6pm-news`, `bulletin-10pm-news`). Stories are stored flat in `rundown_stories` and are filtered by `bulletinId` on query.

---

## 9. Multi-Room Setup

Moira supports multiple simultaneous newsrooms, each with their own rundown, staff list, and TX state. No additional Firebase configuration is required.

- The **Main** room maps to `bulletin-primary`.
- Any other room name is sanitised to a slug, e.g. `6PM News` → `bulletin-6pm-news`.
- Rooms are created automatically the first time a user joins them.
- Switch rooms using the **Newsroom** input and **Join** button in the header.
- Staff identity is cleared on room switch — users must check-in again in the new room.

---

## 10. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Badge shows **Local Mode** after config save | Config not saved or page didn't reload | Open Settings → Connection, re-paste config, click Save & Reconnect |
| Badge shows **Sync Error** | Invalid config values or network issue | Check `apiKey` and `projectId` are correct in Settings |
| `Missing or insufficient permissions` in console | Security rules not published | Re-check Step 5 and click Publish |
| Messages not arriving on other devices | `messages` sub-collection rule missing | Ensure the nested `match /messages/{messageId}` block is in your rules |
| Stories not appearing after room switch | User checked into old room | Re-check-in via the Staff button after switching rooms |
| `bulletin-primary` not created | Firebase connected but app hasn't seeded yet | Reload the page — Moira creates the bulletin automatically on first connect |
