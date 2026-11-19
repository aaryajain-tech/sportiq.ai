# SportIQ AI

SportIQ AI is a sports training and movement-analysis app. It uses browser-based pose estimation to analyze strength exercises, records workout telemetry, and provides performance summaries and coaching through Google Gemini.

> **Fitness disclaimer:** SportIQ AI is for general fitness education and training feedback only. It is not a medical device and does not diagnose injuries or replace advice from a qualified healthcare professional.

## Features

- **Live pose analysis:** Use a webcam and MediaPipe Pose Landmarker to track body landmarks in the browser.
- **Demo simulation:** Try the movement analyzer without a webcam.
- **Exercise tracking:** Analyze squats, push-ups, and lunges. Rep counting follows movement phases and records completed and incomplete repetitions, angles, alignment, confidence, and performance scores.
- **Training dashboard:** Review weekly activity, recent sessions, personal bests, streaks, and achievements.
- **Progress analytics and history:** Explore exercise volume and performance trends, and review or delete saved sessions.
- **AI coaching:** Ask questions about training or request insights and next-session suggestions grounded in your profile and workout history.
- **Profiles and reminders:** Set training preferences and weekly targets, and configure a daily workout reminder.
- **Demo and signed-in modes:** Explore the app with browser-local demo data or sign in with Google and save user data to Firebase.

## Technology

- React 19 and TypeScript
- Vite 8
- Express server with `tsx` for development
- MediaPipe Tasks Vision for browser-side pose estimation
- Google Gemini through the server-side `@google/genai` SDK
- Firebase Authentication and Cloud Firestore
- Tailwind CSS 4 and Recharts

## Requirements

- Node.js **20.19+** or **22.12+** (Vite 8 requirement)
- npm
- A modern browser for camera access
- Optional: a Google Gemini API key for live AI coaching
- Optional: a Firebase project for Google sign-in and cloud persistence

Run commands from the directory containing `package.json`.

## Getting started

Install the dependencies:

```bash
npm install
```

Start the application in development mode:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). The Express server serves the Vite development middleware and the `/api` endpoints on the same port.

To check TypeScript and create a production frontend build:

```bash
npm run lint
npm run build
```

The frontend build is written to `dist/`.

## Configuration

### Gemini AI

The server reads `GEMINI_API_KEY` from its environment. For local development, create a `.env` file next to `package.json`:

```dotenv
GEMINI_API_KEY=your_gemini_api_key
PORT=3000
```

`PORT` is optional and defaults to `3000`. Keep `.env` private; it is ignored by Git. Configure `GEMINI_API_KEY` as a secret in your hosting provider for deployed environments. **Do not add this key to frontend code or a `VITE_` variable.**

If the key is not configured, the app still runs, but live Gemini chat and AI insights are unavailable.

### Firebase

Firebase settings are loaded by the frontend from `firebase-applet-config.json`. The checked-in file is configured for a particular Firebase project; replace it with your own project's web-app configuration when using a different project.

For signed-in mode:

1. Create or select a Firebase project and register a web app.
2. Enable Google as an Authentication provider.
3. Enable Cloud Firestore and configure security rules so users can access only their own records.
4. Update `firebase-applet-config.json` with the project configuration.

The app stores profiles, workout sessions, and coach-message history under each authenticated user's Firestore records. The `firebase-blueprint.json` file describes the expected data shape; it is not a substitute for deploying Firestore security rules.

### Demo mode and camera

You can use the dashboard demo and demo simulation without signing in or configuring Firebase. Demo profiles, workouts, and messages are stored in the current browser's `localStorage`; clearing browser storage removes them.

Live webcam analysis requires browser camera permission and a secure context. `localhost` is supported for local development; deployed sites should use HTTPS. If camera access or the MediaPipe model is unavailable, choose **Demo Simulation** in the live analysis view.

## API

The Express server exposes these endpoints:

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Reports server health and whether Gemini is configured. |
| `POST` | `/api/coach/chat` | Generates a coaching response using the request's athlete profile and workout context. |
| `POST` | `/api/coach/insights` | Returns structured coaching insights for a session or recent workout history. |

Gemini requests are made on the server so the API key does not need to be sent to the browser. The app sends the relevant profile and workout context to the server when requesting AI coaching; those requests are then sent to Google Gemini.

## Production and deployment

The production frontend is generated by `npm run build`. The Express server in `server.ts` serves `dist/` and the API when `NODE_ENV=production` and the server is started with the configured `PORT` (or port `3000` by default).

Deploy this as a Node.js application that runs the Express server, provides the `GEMINI_API_KEY` secret if AI features are needed, and supports the configured Node.js version. For Vercel or another serverless/static platform, configure deployment so that both the SPA and `/api/*` routes are served; a static-only deployment will not run the Express API. Verify the deployed `/api/health` endpoint as well as the frontend.

The repository's `start` script is `node server.ts`. Use a Node.js runtime that supports direct TypeScript execution for that command, or run the server with a TypeScript-aware runtime such as `tsx`.

## Project layout

```text
.
├── server.ts                     # Express API and SPA server
├── vite.config.ts                # Vite, React, Tailwind, and build configuration
├── firebase-applet-config.json   # Firebase web-app configuration
├── firebase-blueprint.json       # Firestore data-shape blueprint
└── src/
    ├── App.tsx                   # Authentication, app state, and page routing
    ├── components/               # Landing page and app views
    ├── lib/
    │   ├── demoData.ts           # Demo profiles, sessions, and derived statistics
    │   ├── exerciseAnalyzer.ts   # Pose telemetry and rep-counting logic
    │   ├── firebase.ts           # Firebase initialization and error handling
    │   └── store.ts              # Demo localStorage and Firestore persistence
    └── types/                    # Shared TypeScript types
```

## Available scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Starts the Express server with Vite development middleware. |
| `npm run build` | Builds the frontend into `dist/`. |
| `npm start` | Starts `server.ts` using Node.js. |
| `npm run preview` | Starts Vite's preview server for the built frontend. |
| `npm run lint` | Runs `tsc --noEmit` to check TypeScript. |
| `npm run clean` | Removes generated build output and `server.js` (uses a Unix-style `rm` command). |
