# Security for DutchEZ (Firebase & public GitHub)

This app uses Firebase in the browser. Here’s how to keep things secure when the repo is public.

---

## 1. Don’t commit your real Firebase config

- **`firebase-config.js`** is in **`.gitignore`** — it is not committed.
- **`firebase-config.example.js`** is committed and contains only placeholders.
- **Before you publish:** Copy the example to `firebase-config.js`, fill in your real Firebase config, and **never commit** `firebase-config.js`. Your clone/deploy will have the real file locally or in your deploy step; the public repo will not.

If someone clones the repo, they get no keys; they must create their own Firebase project and `firebase-config.js` from the example. **For GitHub Pages:** if you don’t use a build step, you’ll need to add `firebase-config.js` to the repo for the live site to use Firebase — in that case, **restrict your API key** to your GitHub Pages domain (e.g. `https://tlt.github.io/*`) so the key is only usable from your site.

---

## 2. Restrict your Firebase API key by domain

Firebase **API keys for web are meant to be used in client code**. They identify your project; they do **not** by themselves grant access. Access is controlled by **Firestore Security Rules**. You still should restrict where the key can be used:

1. Open [Google Cloud Console](https://console.cloud.google.com/) and select the same project as your Firebase app.
2. Go to **APIs & Services → Credentials**.
3. Under **API Keys**, click your **Browser key** (the one used by your Firebase web app).
4. Under **Application restrictions**:
   - Choose **HTTP referrers (web sites)**.
   - Add your allowed origins, for example:
     - `https://yourusername.github.io/*`
     - `https://tlt.github.io/*`
     - `http://localhost:*` (for local dev)
5. Save.

After this, the key will only work when the request comes from those origins. If someone copies your key from a built site, they cannot use it from another domain.

---

## 3. Firestore Security Rules

Security is enforced in **Firestore → Rules**, not by hiding the API key.

**Basic (permissive, for shared links):**

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

This allows anyone who knows a `billId` to read/write that document (by design, for “share link” collaboration).

**Tighter (optional):** You can limit document size and ID format to reduce abuse:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /bills/{billId} {
      allow read, write: if billId.matches('^[a-z0-9]{6,20}$');
    }
  }
}
```

Only document IDs that look like your app’s IDs (e.g. 6–20 alphanumeric) are allowed. Adjust the regex if your IDs differ.

---

## 4. Summary

| What | Action |
|------|--------|
| **Repo** | Keep `firebase-config.js` out of the repo (use `.gitignore` and the example file). |
| **API key** | Restrict the Browser API key to your site’s domains in Google Cloud Console. |
| **Access control** | Rely on Firestore rules; optionally restrict `bills` document IDs. |

The API key will still be visible in the browser for your deployed site (that’s normal for Firebase web apps). Restricting it by referrer and using Firestore rules is what keeps usage under your control when the repo is public.
