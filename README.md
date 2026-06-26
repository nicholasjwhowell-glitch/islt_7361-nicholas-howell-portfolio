# Faith and Fury — Setup Guide

This module now saves uploaded images, audio, video, and citations to the **cloud**, so that everyone who opens the page link sees the same content — not just the person who uploaded it. That requires a free **Firebase** project as the "backend." GitHub Pages alone can't do this on its own, because it only serves static files and has no way to store anything new that visitors upload.

The good news: Firebase's free tier is built for exactly this kind of small, shared classroom project, and setup takes about 10 minutes, done once.

---

## Step 1 — Create a free Firebase project

1. Go to **https://console.firebase.google.com** and sign in with any Google account.
2. Click **Add project**, give it a name (e.g. `faith-and-fury`), and finish the setup wizard (you can skip Google Analytics).

## Step 2 — Turn on Firestore (for text/citations) and Storage (for files)

1. In the left sidebar, under **Build**, click **Firestore Database** → **Create database**. Choose any region close to you → **Start in test mode** → Enable.
2. Still under **Build**, click **Storage** → **Get started** → **Start in test mode** → Done.

> Test mode rules expire automatically after 30 days. Step 3 below replaces them with permanent rules, so don't skip it.

## Step 3 — Set permanent security rules

In **Firestore Database → Rules**, replace everything with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /faithFury/{document=**} {
      allow read, write: if true;
    }
  }
}
```
Click **Publish**.

In **Storage → Rules**, replace everything with:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /media/{fileName} {
      allow read, write: if true;
    }
  }
}
```
Click **Publish**.

These rules leave the module fully open (no login needed) so any student with the link can upload or update content — appropriate for a collaborative classroom tool. They only grant access to this project's specific data (the `faithFury` collection and `media/` folder), not anything else in your Firebase account.

**A note on safety:** the free "Spark" plan has no billing attached, so there's no risk of a surprise charge even with open rules — usage simply stops working if the free quota is ever reached, which is very unlikely for a single class project.

## Step 4 — Get your config and paste it into the page

1. In Firebase, click the **gear icon → Project settings**.
2. Scroll to **Your apps** → click the **Web** icon (`</>`) → give it a nickname → **Register app**.
3. Firebase shows you a `firebaseConfig` object. Copy it.
4. Open `index.html` in any text editor, find this block near the top of the `<script>` section, and paste your real values over the placeholders:

```js
const firebaseConfig = {
  apiKey: "PASTE_YOUR_API_KEY_HERE",
  authDomain: "PASTE_YOUR_PROJECT.firebaseapp.com",
  projectId: "PASTE_YOUR_PROJECT_ID",
  storageBucket: "PASTE_YOUR_PROJECT.appspot.com",
  messagingSenderId: "PASTE_YOUR_SENDER_ID",
  appId: "PASTE_YOUR_APP_ID"
};
```

> These values are safe to put directly in a public GitHub repo. Firebase web config keys are designed to be public — the security rules from Step 3 are what actually control access, not secrecy of this object.

Save the file. The orange warning banner at the top of the page will disappear once this is filled in correctly and the page is reloaded.

## Step 5 — Put it on GitHub Pages

1. Create a new repository on GitHub (public or private both work for Pages on a free personal account, though private repos need GitHub Pro for Pages).
2. Upload `index.html` to the root of the repository (drag-and-drop on the GitHub website works fine, or use `git push`).
3. Go to the repo's **Settings → Pages**.
4. Under **Source**, choose the branch (usually `main`) and folder `/ (root)` → **Save**.
5. GitHub will give you a live URL like `https://yourusername.github.io/your-repo-name/` within a minute or two.

That URL is what you share with students — anything anyone uploads or types there will now be saved and visible to everyone who opens it.

---

## How it works, in plain terms

- **Images, audio files, and video files** you upload go to Firebase Storage and get a public link, which is saved to a shared database so everyone's page shows the same one.
- **Citations, audio links, and YouTube links** are saved to a shared Firestore document the same way.
- The page checks for live updates automatically — if someone uploads a new image while you have the page open, you'll see it appear without needing to refresh.
- If Firebase isn't configured yet, the page still works for local editing/previewing, but uploads only show on your own screen and the orange banner stays visible as a reminder.

## Troubleshooting

- **Banner won't go away** — double check there are no leftover quotes or typos in the pasted config, and that you replaced every `PASTE_YOUR_...` value.
- **Uploads don't appear for other people** — open the browser console (F12 → Console tab) and look for a red error; it usually points to a Rules typo from Step 3.
- **It worked, then stopped after a month** — you're likely still on default test-mode rules. Redo Step 3 with the permanent rules above.

## Editing the content

Everything besides the Firebase config is in plain HTML, CSS, and JavaScript inside `index.html` — open it in any text editor (VS Code, Notepad++, even TextEdit) to change wording, colors, quiz questions, or layout. No build tools or installations are needed.
