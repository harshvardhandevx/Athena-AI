# Athena AI

Athena AI is a browser-based learning assistant that explains concepts in three difficulty levels:

- Child
- Beginner
- Expert

It supports Google sign-in, persistent chat history in Firestore, optional image or document context, and in-app YouTube video embedding for topic reinforcement.

## Current architecture

This repository is a static frontend app:

- No Node build step
- No backend server in this repo
- Runs directly in the browser using native ES modules

Main files:

- index.html: App shell, UI layout, Tailwind CDN usage, modal/overlay markup
- script.js: App logic (auth, Firestore history, Groq requests, YouTube integration, attachment parsing)
- style.css: Additional stylesheet in repo (not currently linked in index.html)
- config.example.js: Template for local runtime keys

## Features implemented

- Difficulty-aware explanations (Child, Beginner, Expert)
- Prompt shaping by user major or profession
- Structured LLM output parsing (JSON extraction and validation)
- Jargon tooltips generated from model-provided glossary terms
- YouTube video search + embeddability filtering + fallback watch link
- File attachments:
	- TXT
	- PDF (via PDF.js)
	- DOCX (via Mammoth)
- Image attachments for vision-capable model requests
- Firebase Google Auth sign-in/out flow
- Firestore-backed per-user chat history
- Sidebar history and restore of saved chats
- Dark mode toggle
- Undo/redo for rendered conversation state (Cmd/Ctrl+Z, Shift+Cmd/Ctrl+Z, Cmd/Ctrl+Y)

## Runtime dependencies (CDN)

- Tailwind CSS (CDN)
- Firebase JS SDK v10 modules
- PDF.js
- Mammoth (DOCX text extraction)
- YouTube IFrame API (loaded on demand)

## Configuration and secrets

Secrets are loaded from a local file named config.js and exposed at runtime via window.ATHENA_CONFIG.

Required keys:

- FIREBASE_API_KEY
- FIREBASE_AUTH_DOMAIN
- FIREBASE_PROJECT_ID
- FIREBASE_STORAGE_BUCKET
- FIREBASE_MESSAGING_SENDER_ID
- FIREBASE_APP_ID
- GROQ_API_KEY
- YT_API_KEY

Optional key:

- FIREBASE_MEASUREMENT_ID

If required keys are missing, app startup fails early with a descriptive error.

### Local setup

1. Copy config.example.js to config.js.
2. Fill config.js with real values.
3. Optionally copy .env.example to .env for local reference.

Notes:

- .env and config.js are ignored by git.
- Do not commit real keys.

## Firebase setup checklist

1. Create a Firebase project.
2. Enable Authentication:
	 - Sign-in method: Google
3. Add authorized domains for your local/dev and deployed hosts.
4. Create Firestore database.
5. Ensure rules allow authenticated users to read/write their own chat data.

Firestore path used by the app:

- users/{uid}/chats/{chatId}

Stored chat fields:

- userPrompt
- promptLabel
- aiResponseJSON
- difficultyLevel
- createdAt

## Local development

Serve the folder over localhost (do not open index.html directly with file://).

Examples:

```bash
# Ruby (works on many macOS setups)
ruby -run -e httpd . -p 8080

# Python
python3 -m http.server 8080
```

Open:

- http://localhost:8080

## Production deployment

Because this is a static app, you can deploy to:

- GitHub Pages
- Netlify
- Vercel (static)
- Firebase Hosting
- Cloudflare Pages

Before deploying:

1. Add your production domain to Firebase authorized domains.
2. Provide production config.js securely in your deploy workflow.
3. Confirm Groq and YouTube keys are restricted where possible.

## Operational behavior and resiliency

- Groq request timeout: 90 seconds
- Retries on rate-limit responses (429): up to 3 attempts
- Fallback if strict JSON mode is unsupported by model
- YouTube fallback behavior:
	- Tries multiple candidate videos
	- Skips non-embeddable/private videos
	- Shows watch link even if embed playback fails

## Security notes

Important: This is a client-side app, so browser-delivered keys can be inspected by users.

For stronger production security, move sensitive API calls (especially Groq) to a backend/proxy and keep provider secrets server-side.

At minimum:

- Rotate any keys that were previously committed or shared
- Use provider-side key restrictions
- Monitor usage and quotas

## Troubleshooting

### Sign-in fails with unauthorized-domain

- Add your host to Firebase Authentication authorized domains.
- Local dev is expected on localhost (or explicitly allow 127.0.0.1).

### No explanation returned

- Check browser console for Groq API errors
- Verify GROQ_API_KEY in config.js
- Check quota/rate limits in Groq dashboard

### YouTube section stays empty

- Verify YT_API_KEY
- Ensure YouTube Data API is enabled
- Some videos cannot be embedded due to owner restrictions

### App fails on startup with config error

- Confirm all required keys exist in config.js
- Reload page after updating config.js

## Git and push safety

Ignored local-secret files:

- .env
- config.js

Tracked safe templates:

- .env.example
- config.example.js

## License

No license file is currently present in this repository. Add one before broader distribution.
