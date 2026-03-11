# Moira — Firestore Setup Guide

## Overview

Moira runs fully in-browser with no backend server. Cloud sync is provided by **Firebase Firestore** with **Anonymous Authentication**. Out of the box the app works in local-only mode; this guide gets you to a live, multi-user cloud-synced newsroom.

---

## 1. Create a Firebase Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and click **Add project**.
2. Give it a name (e.g. `moira-newsroom`). You can disable Google Analytics — it isn't used.
3. Once created, click the **Web** icon (`</>`) under "Get started by adding Firebase to your app".
4. Register a nickname (e.g. `moira-web`). Skip Firebase Hosting.
5. Firebase will show you a `firebaseConfig` object. **Copy the entire block** — you'll need it in step 4.

---

## 2. Enable Anonymous Authentication

1. In the Firebase console sidebar go to **Build → Authentication**.
2. Click **Get started**.
3. Under the **Sign-in method** tab, click **Anonymous** and toggle it **Enabled**. Save.

Every browser that opens Moira will silently sign in as an anonymous user. This is the only auth method the app uses unless you inject a custom token via `__initial_auth_token`.

---

## 3. Create the Firestore Database

1. In the sidebar go to **Build → Firestore Database**.
2. Click **Create database**.
3. Choose **Start in production mode** (you'll add proper rules below).
4. Pick a region close to your broadcast location (e.g. `europe-west2` for the UK).

---

## 4. Deploy Security Rules

Moira stores all data under a fixed path structure:

```
artifacts/{appId}/public/data/bulletins/{bulletinId}
artifacts/{appId}/public/data/rundown_stories/{storyId}
```

The `appId` defaults to `moira-default` (see step 5 to override it).

In the Firebase console go to **Firestore → Rules** and replace the default with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // All Moira data lives under artifacts/{appId}/public/...
    match /artifacts/{appId}/public/{document=**} {
      // Any signed-in user (including anonymous) can read and write
      allow read, write: if request.auth != null;
    }
  }
}
```

Click **Publish**.

> **Tighter option:** if you want to prevent anonymous users from writing (e.g. presenter-only clients), you can split `read` and `write` and gate writes on a custom claim. The anonymous-only setup above is sufficient for a trusted newsroom LAN.

---

## 5. Supply the Firebase Config to Moira

The app checks for its config in this order:

1. **`localStorage` key `moira_firebase_config`** — set via the in-app Settings UI (easiest for end users)
2. **`window.MOIRA_CONFIG`** — inject via a `<script>` tag in the HTML (best for a self-hosted deployment)
3. **Global `__firebase_config`** — a JSON string, used by Canvas/hosted wrapper environments

### Option A — In-app paste (no code change required)

1. Open Moira in your browser.
2. Click the **⚙ Settings** button (top-right).
3. Go to the **Connection** tab.
4. Paste the entire `firebaseConfig` block from step 1 into the textarea, or fill in the individual fields.
5. Click **Save & Connect**.

The config is stored in `localStorage` and will persist across sessions on that browser.

### Option B — Baked into the HTML (recommended for production)

Add a `<script>` block immediately before the closing `</head>` tag in `moira.html`:

```html
<script>
  window.MOIRA_CONFIG = {
    apiKey:            "AIza...",
    authDomain:        "your-project.firebaseapp.com",
    projectId:         "your-project",
    storageBucket:     "your-project.appspot.com",
    messagingSenderId: "123456789",
    appId:             "1:123...:web:abc..."
  };
</script>
```

### Option C — Override the app ID

By default all data is stored under `artifacts/moira-default/...`. To use a different namespace (useful for running multiple independent newsrooms from the same Firebase project), add:

```html
<script>
  const __app_id = 'studio-2';          // e.g. 'morning-show', 'studio-2'
  window.MOIRA_CONFIG = { ... };
</script>
```

Each unique `appId` gets completely isolated rundowns and bulletins in Firestore.

---

## 6. Verify the Connection

1. Open Moira in your browser with the config in place.
2. The status indicator in the header should turn green and show **Connected**.
3. In the Firebase console go to **Firestore → Data** and you should see:

```
artifacts/
  moira-default/         ← (or your custom appId)
    public/
      data/
        bulletins/
          bulletin-primary      ← created automatically on first load
        rundown_stories/
          <uuid>                ← the seed story
```

---

## 7. Rooms / Multiple Rundowns

Moira supports multiple concurrent rundowns ("rooms") within the same Firebase project and `appId`. Each room maps to a separate document in `bulletins/` and a scoped query in `rundown_stories/`.

| Room name entered by user | `bulletinId` stored in Firestore |
|--------------------------|----------------------------------|
| `Main` (default)         | `bulletin-primary`               |
| `Sport`                  | `bulletin-sport`                 |
| `Late News`              | `bulletin-late-news`             |

Users switch rooms via the room selector in the header. Staff identity resets on room switch by design.

---

## 8. Data Schema Reference

### `bulletins/{bulletinId}` document

Holds the rundown metadata and running order.

| Field | Type | Notes |
|---|---|---|
| `order` | `string[]` | Ordered array of story UUIDs + `"marker-line"` sentinel |
| `staff` | `object[]` | `{ id, name, role, lastSeen }` |
| `formats` | `string[]` | e.g. `["READ","OOV","CLIP",...]` |
| `roles` | `string[]` | e.g. `["Producer","Reporter","Presenter"]` |
| `activeStoryId` | `string\|null` | Story currently on air |
| `isTXMode` | `boolean` | Live transmission mode |
| `isRehearsalMode` | `boolean` | |
| `rundownLocked` | `boolean` | Prevents reordering by non-editors |
| `txStartedAt` | `number\|null` | Unix ms timestamp |
| `txTargetDuration` | `number\|null` | Total planned duration in seconds |

### `rundown_stories/{storyId}` documents

| Field | Type | Notes |
|---|---|---|
| `bulletinId` | `string` | Foreign key back to the parent bulletin |
| `slug` | `string` | Story headline |
| `script` | `string` | Presenter script body |
| `format` | `string` | e.g. `"READ"`, `"OOV"`, `"CLIP"` |
| `duration` | `number` | Seconds |
| `presenterId` | `string` | Staff `id` or `""` |
| `reporterId` | `string` | Staff `id` or `""` |
| `mediaFile` | `string` | Filename passed to OBS |
| `completed` | `boolean` | Story has aired |
| `lockedBySession` | `string\|null` | UUID of the editing session |
| `lockedByName` | `string\|null` | Display name of editor |
| `lockedAt` | `number\|null` | Unix ms — locks older than 10 s are considered stale |
| `rev` | `number` | Optimistic concurrency counter |

---

## 9. Indexes

Moira queries stories with `where("bulletinId", "==", ...)`. Firestore handles this with no composite index required (single-field queries are indexed automatically). No manual index configuration is needed.

---

## 10. Cost / Quota Notes

Moira uses real-time `onSnapshot` listeners. Each connected client holds one listener on the bulletin doc and one on the stories collection. For a typical newsroom team of 2–10 people the daily read count will be well within Firebase's free **Spark** tier (50k reads/day). If you run a large team or very frequent TX mode cycles, monitor usage in the Firebase console under **Usage** → **Firestore**.