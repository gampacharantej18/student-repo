# Deploy LearnPulse on Render

This repository is configured as a Render **Web Service**. The application is an Express server that serves the Vite frontend and the `/api/*` backend from one process.

## If using the included Blueprint

In Render, choose **New > Blueprint** and select this repository. Render will read `render.yaml`, use `learnpulse` as the root directory, install pnpm 10.15.0 through Corepack, build the frontend, start the Express server, and check `/api/health`.

## If creating the service manually

Use these exact settings:

- **Service type:** Web Service
- **Root Directory:** `learnpulse`
- **Runtime:** Node
- **Node version:** `22` (the repository includes `.node-version`)
- **Build Command:** `corepack enable && corepack prepare pnpm@10.15.0 --activate && pnpm install --frozen-lockfile && pnpm run build`
- **Start Command:** `pnpm start`
- **Health Check Path:** `/api/health`
- **Plan:** Free is suitable for testing; production use may need a paid plan because free services sleep when idle.

Do not use `yarn` as the build command. Do not set the root directory to the repository's outer folder when deploying manually; it must be `learnpulse`.

## Environment variables

Add the following in Render under **Environment > Environment Variables**. Values are project-specific and must come from your Firebase project:

```text
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
FIREBASE_STORAGE_BUCKET=
GROQ_API_KEY=
GROQ_MODEL=openai/gpt-oss-20b
```

For Firebase Admin SDK authentication, the safest Render approach is to create a **Secret File** named `firebase-service-account.json`, paste the Firebase service-account JSON into it, and set:

```text
GOOGLE_APPLICATION_CREDENTIALS=/etc/secrets/firebase-service-account.json
```

Do not commit the service-account JSON or put it in any `VITE_*` variable. Render supplies `PORT` automatically; do not hard-code a Render port.

## Firebase and deployment checks

1. Enable Email/Password authentication in Firebase Authentication.
2. Create Firestore and enable Firebase Storage if the application will use file uploads.
3. Confirm the Firebase project ID and storage bucket match across the browser and server variables.
4. Deploy the service and open `https://YOUR-SERVICE.onrender.com/api/health`. A healthy response confirms that the Node service is reachable.
5. Open the service URL in a browser and test registration/login.
6. Run the one-time admin bootstrap only from a trusted operator environment as described in `README.md`; never add bootstrap credentials to normal Render environment variables.

## Common failure fixes

- `yarn@pnpm@10.15.0` or Yarn 1 errors: the wrong old build command is being used. Replace it with the pnpm build command above.
- Frozen lockfile errors: deploy the supplied `pnpm-lock.yaml` together with `package.json`; do not delete or regenerate only one of them.
- `Cannot find module` at startup: verify that the service root is `learnpulse` and that the build completed successfully.
- Firebase Admin initialization warnings: verify the Secret File path and the project/storage variables. The frontend can load without Admin credentials, but protected API operations require them.
