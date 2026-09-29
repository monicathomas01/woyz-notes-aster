# WOYZ Firebase deployment

This package uses Firebase Authentication and Cloud Firestore. Notes are stored at:

`users/{firebaseAuthUid}/notes/{noteId}`

The UID comes from the signed-in Firebase account. The included Firestore rules prevent one user from reading or changing another user's notes. The same account can remain signed in on multiple devices, and Firestore listeners synchronize changes in real time.

## Run locally

This is a static site with no build step. The pages load the Firebase SDK as ES modules, so they must be served over HTTP. Opening `index.html` directly from disk (`file://`) will not work.

From this directory, start any static server:

```bash
# Option A: Python (no install needed)
python3 -m http.server 8000

# Option B: Firebase CLI (uses firebase.json, including clean URLs)
firebase serve --only hosting --port 8000
```

Then open <http://localhost:8000> and sign in with an account from **Authentication > Users**. Firebase authorizes `localhost` by default, and the service worker also runs on `localhost`.

The local site still uses the live Firebase project from `firebase-config.js`, so any notes you create are real Firestore data.

## 1. Create and configure Firebase

1. Create a Firebase project.
2. Add a Web app in **Project settings > Your apps**.
3. Copy its configuration values into `firebase-config.js`.
4. Open **Authentication > Sign-in method** and enable **Email/Password**.
5. Create each doctor/user under **Authentication > Users**.
6. Create a Cloud Firestore database in production mode.

## 2. Deploy the Firestore rules

Install and authenticate the Firebase CLI, then run from this directory:

```bash
firebase login
cp .firebaserc.example .firebaserc
firebase deploy --only firestore:rules
```

Replace `YOUR_PROJECT_ID` in `.firebaserc` before deployment. You can alternatively paste `firestore.rules` into **Firestore Database > Rules** in the Firebase console and publish it there.

## 3. Publish the web files

For GitHub Pages, commit at least:

- `index.html`
- `user.html`
- `firebase-config.js`

In Firebase Authentication settings, add the GitHub Pages hostname (for example, `username.github.io`) to **Authorized domains**.

For Firebase Hosting, run:

```bash
firebase deploy --only hosting
```

## Data behavior

- The app opens on today's date after sign-in.
- Previous, next, and calendar-date controls load notes for that date.
- Each user sees only documents under their own UID.
- New drafts are immediately created in Firestore.
- Generated text is saved back to the draft.
- **Save Draft** finalizes and persists the note.
- Draft deletion removes the Firestore document.
- Real-time listeners allow the same user to see changes across multiple devices.

## Important

The Firebase web configuration is not a server secret. Access control depends on Firebase Authentication and the deployed `firestore.rules`. Never use permissive test rules in production.
