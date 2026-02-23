# Firebase Setup for DutchEZ Shared Bills (Free Tier)

This guide walks you through using **Firebase Firestore** so users can share a **single unique link**. Everyone who opens that link sees the same bill state and can add their own orders in real time, instead of passing one long URL with all data encoded.

---

## What You’ll Get

- **One short link** per bill (e.g. `yoursite.com?id=abc12`).
- **Real-time sync**: when someone adds a person, adds items, or changes orders, everyone with the link sees updates.
- **No login required** for guests (optional: you can add Auth later).
- **Free tier** is enough for personal/small-group use (50K reads, 20K writes per day).

---

## Step 1: Create a Firebase Project (Free)

1. Go to [Firebase Console](https://console.firebase.google.com/).
2. Click **Add project** (or **Create a project**).
3. Enter a name (e.g. `dutchez-bills`) and continue; disable Google Analytics if you want.
4. Click **Create project** and wait for it to finish.

---

## Step 2: Register Your App

1. In the project overview, click the **Web** icon (`</>`).
2. Register app nickname (e.g. `DutchEZ Web`).
3. **Do not** check Firebase Hosting for now (you can add it later for GitHub Pages).
4. Click **Register app** and copy the `firebaseConfig` object. You’ll paste it into your app in Step 5.
5. Click **Continue to console**.

---

## Step 3: Enable Firestore

1. In the left sidebar, open **Build → Firestore Database**.
2. Click **Create database**.
3. Choose **Start in test mode** (for development; we’ll tighten rules in Step 6).
4. Pick a location (e.g. `us-central1`) and **Enable**.

---

## Step 4: Get Your Config and SDK

1. In **Project settings** (gear icon), under **Your apps**, copy the `firebaseConfig` object.
2. In your `index.html`, add the Firebase SDK and init **before** your main script:

```html
  <!-- Add before your main <script> -->
  <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
  <script>
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT_ID.appspot.com",
      messagingSenderId: "123456789",
      appId: "YOUR_APP_ID"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.firestore();
  </script>
```

Replace the values with the ones from the Firebase Console.

---

## Step 5: Use a Unique Link (ID in URL)

**Idea:** One Firestore document per “bill session,” keyed by a short ID in the URL (e.g. `?id=abc12`).

1. **When creating a share:**
   - Generate a short unique ID (e.g. 6–8 alphanumeric characters).
   - Save your current `state` (people, restaurants, orders, sharedItems, tax, tip, etc.) to Firestore:
     - Collection: `bills`
     - Document ID: the short ID
     - Fields: e.g. `state`, `createdAt`, `updatedAt`
   - Show the user the link: `https://yoursite.com?id=abc12`.

2. **When opening a link with `?id=xxx`:**
   - Read the document `bills/xxx`.
   - If it exists, load `state` into your app and optionally **listen** for real-time updates with `onSnapshot`.
   - If it doesn’t exist, show an error or create a new bill (your choice).

3. **When anyone changes the bill:**
   - Update the same document (e.g. write the full `state` or only changed fields) so everyone with the link sees updates.

---

## Step 6: Firestore Security Rules (Free Tier Safe)

In Firestore → **Rules**, use something like this to allow read/write only for the `bills` collection (so random people can’t read/write the rest of your DB):

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /bills/{billId} {
      allow read, write: if true;
    }
  }
}
```

- **Development:** This is enough to get going; anyone with a bill ID can read/write that document.
- **Later:** You can restrict `write` to “only if request.auth != null” or add checks so only the creator can delete.

Publish the rules.

---

## Step 7: Optional – Add “Create share link” in the App

1. Add a button, e.g. **Create shared link** (or reuse “Copy Share Link” with a different mode).
2. On click:
   - Generate ID: e.g. `const id = Math.random().toString(36).slice(2, 10);`
   - Save to Firestore: `db.collection('bills').doc(id).set({ state: state, updatedAt: firebase.firestore.FieldValue.serverTimestamp() });`
   - Copy to clipboard: `window.location.origin + window.location.pathname + '?id=' + id`
   - Show toast: “Shared link copied!”
3. On page load, if URL has `?id=xxx`:
   - Load once: `db.collection('bills').doc(id).get().then(doc => { if (doc.exists) applyState(doc.data().state); });`
   - Optional real-time: `db.collection('bills').doc(id).onSnapshot(doc => { if (doc.exists) applyState(doc.data().state); });`

`applyState` should assign the loaded `state` into your app’s `state` and call `renderAll()` (and update inputs for tax/tip if needed).

---

## Step 8: Stay on Free Tier

- **Firestore free limits:** 50K reads, 20K writes, 20K deletes per day.
- Use **one document per bill** and update that document on each change (or debounce updates to reduce writes).
- Avoid writing on every keystroke; write when the user adds/removes an item or person, or when they click “Save” / “Copy share link.”

---

## Summary Checklist

| Step | Action |
|------|--------|
| 1 | Create Firebase project |
| 2 | Register web app, copy `firebaseConfig` |
| 3 | Enable Firestore (test mode first) |
| 4 | Add Firebase scripts + config to `index.html` |
| 5 | Use `?id=xxx` for unique links; read/write `bills/xxx` |
| 6 | Set Firestore rules for `bills/{billId}` |
| 7 | Add “Create shared link” and load/listen for `?id=` |
| 8 | Deploy and test; stay within free tier limits |

After this, users can share a single unique link and each person can add their own menu/orders with updates synced via Firebase.
