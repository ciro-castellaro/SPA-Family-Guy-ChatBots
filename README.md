# Family Guy Chat

SPA (Single Page Application) that allows users to chat with Family Guy characters (Peter, Stewie, Brian, and Lois Griffin) powered by **Google Gemini AI**, built with HTML, CSS, and Vanilla JavaScript (ES Modules), with a Vercel Serverless Function that protects the API Key.

> Academic project for non-commercial purposes. Family Guy and its characters are owned by Fox/20th Television — this app is an adaptation created for educational purposes to strengthen frontend, serverless functions, AI, and related skills.

## Links

* **GitHub Repository:** https://github.com/ciro-castellaro/ProyectoM3_Ciro-Castellaro
* **Deployed App:** https://chat-fg.vercel.app
* **AI Usage:** https://drive.google.com/drive/folders/1_wFPQQfGfvCgE96-Eka9Gfo6I_8VIF3C?usp=sharing

---

## Overview

The application allows users to choose one of four Family Guy characters and chat with them in real time. Each character has their own personality, speaking style, and knowledge boundaries, defined through a specific System Prompt that is sent to Gemini for each conversation.

It is a real SPA: navigation between Home, Chat, and About is handled using the browser's History API (`pushState`/`popstate`), without reloading the page.

## Main Features

* Selection between 4 Family Guy characters, each with their own personality and System Prompt.
* Real-time chat with Google Gemini, with conversation history persisted throughout the session (per character).
* SPA navigation using the History API (without page reloads).
* Responsive, Mobile First design (Flexbox + Grid).
* Visual states: "typing..." indicator, disabled buttons while waiting for a response, and error handling.
* Gemini API Key is protected at all times: it is never exposed on the frontend and is only accessible by the Serverless Function.
* Unit test suite with Vitest covering the project's pure logic.

## Technologies Used

* HTML5
* CSS3 (Flexbox, Grid, Mobile First, Media Queries)
* Vanilla JavaScript (ES Modules)
* Fetch API
* History API
* [Google Gemini API](https://ai.google.dev/) through the official [`@google/genai`](https://www.npmjs.com/package/@google/genai) SDK
* Vercel Serverless Functions
* [Vitest](https://vitest.dev/) for testing
* Git / GitHub
* Vercel (hosting and deployment)

## Project Structure

```text
project-root/
├── api/
│   └── functions.js          # Serverless Function: proxy to Gemini, protects the API Key
├── src/
│   ├── index.html             # Structure of the 3 views (Home, Chat, About)
│   ├── styles.css             # Mobile First styles (Flexbox + Grid)
│   ├── app.js                 # Entry point: SPA routing (History API)
│   ├── chat.js                # Chat state and Serverless Function integration
│   ├── characters.js          # Character data: name, tagline, avatar, and System Prompt
│   └── utils.js               # Reusable pure functions (parsing, validation, fetch, routing)
├── tests/
│   ├── utils.test.js          # Tests for the pure functions in utils.js
│   └── app.test.js            # Tests for characters.js and the pure functions in api/functions.js
├── capturas de pantalla M3/   # App screenshots used in this README
├── .env                        # Actual environment variables (NEVER committed — included in .gitignore)
├── .env.example                # Environment variable template
├── .gitignore
├── vercel.json                 # Vercel configuration: outputDirectory + rewrite for SPA routing
├── package.json
└── README.md
```

## Prerequisites

* [Node.js](https://nodejs.org/) 18 or higher
* npm (included with Node.js)
* A [Google AI Studio](https://aistudio.google.com/) account to generate a Gemini API Key
* Git
* A [Vercel](https://vercel.com/) account (required to run Serverless Functions locally and for deployment)

## Installation

```bash
# Clone the repository
git clone https://github.com/ciro-castellaro/ProyectoM3_Ciro-Castellaro.git

# Enter the project folder
cd ProyectoM3_Ciro-Castellaro

# Install dependencies
npm install
```

## `.env` Configuration

The project requires a Google Gemini API Key to work. It is never committed to the repository (it is included in `.gitignore`) — anyone running the project must create their own local `.env` file.

1. Copy `.env.example` to a new `.env` file in the project root:

```bash
cp .env.example .env
```

2. Add your actual API Key (generated through [Google AI Studio](https://aistudio.google.com/)):

```env
GEMINI_API_KEY=your_google_gemini_api_key_here
```

## Running the Project Locally

⚠️ **Important:** this project uses a Serverless Function (`api/functions.js`), so it **cannot be tested with Live Server** or any static file server — these tools do not execute serverless functions, and requests to `/api/functions` will fail. The project must be run using the Vercel CLI:

```bash
npx vercel dev --local --listen 3000
```

* `--local` prevents the need to log in or link the project to a Vercel account for local development.
* The app will be available at `http://localhost:3000`.

If you have already linked the project to your Vercel account (`vercel link`), you can also simply run:

```bash
vercel dev
```

## Running the Tests

The project uses [Vitest](https://vitest.dev/):

```bash
npm test
```

This runs `vitest run` against `tests/utils.test.js` and `tests/app.test.js`, covering the project's pure functions: chat header text parsing, message validation and sanitization, route resolution, retrieving characters by ID, and building the payload sent to Gemini (role mapping and conversation history construction).

## Vercel Deployment

1. Push the repository to GitHub (already done for this project).
2. Import the repository into [Vercel](https://vercel.com/).
3. **Keep "Root Directory" pointing to the repository root** (not `src`) — the `api/` folder must remain visible so Vercel can detect the Serverless Function. The configuration for the static files (`src/`) and the SPA routing rewrite is already defined in `vercel.json`, so no additional configuration is required.
4. Under **Settings → Environment Variables**, add `GEMINI_API_KEY` with your actual API Key (enable at least the Production environment).
5. Deploy. If you added the environment variable after an existing deployment, a **redeploy** is required for the new value to take effect.

The deployed app for this project is available at: **https://chat-fg.vercel.app**

## Serverless Function: Why It Protects the API Key

The browser is a public environment: anyone can open DevTools and inspect the JavaScript code sent to the browser, including variables. If the frontend called the Gemini API directly, the API Key would have to be included in that code, allowing any visitor to copy it and use it at the project's owner's expense.

That's why `api/functions.js` exists: it is a Serverless Function that runs on Vercel's server rather than in the user's browser. The flow is:

```text
Browser (fetch) → /api/functions (Serverless Function, with the key) → Gemini API
Browser (receives response) ← /api/functions (forwards response) ← Gemini API
```

The frontend never knows the API Key: it only sends the selected character's System Prompt, conversation history, and latest message to `/api/functions`. The function reads `GEMINI_API_KEY` from a server-side environment variable (never from the code) and builds the actual Gemini request using the official [`@google/genai`](https://www.npmjs.com/package/@google/genai) SDK. The function also handles invalid methods, missing configuration, incomplete request bodies, and errors returned by Gemini, always returning a consistent JSON response to the frontend.

## Characters and System Prompts

The app allows users to choose between **4 Family Guy characters**, each defined in `src/characters.js` with: `id`, `name`, `tagline` (a short phrase displayed on the selection card), `avatar` (an emoji, avoiding reliance on copyrighted images from the original show), and `systemPrompt` (the complete set of instructions sent to Gemini).

Each System Prompt follows the same structure:

* **Personality:** the character's core personality traits.
* **How they speak:** tone, catchphrases, and speaking style.
* **What they know:** the knowledge and narrative context they can discuss.
* **What they don't know:** explicit boundaries (real-world technology, current politics/news), along with instructions not to break character or reveal that they are an AI model.
* **Mandatory limits:** content guardrails — never violent, sexual, discriminatory, or hateful; no real medical/legal/financial advice; absurd humor, but never cruel.

The four available characters are:

| Character             | Description                                                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 🍺 **Peter Griffin**  | Impulsive and distracted family man, lover of Pawtucket beer and TV, loyal to his family and friends, with absurd and tangential humor. |
| 👶 **Stewie Griffin** | Genius baby with sophisticated vocabulary and always-absurd, harmless world-domination plans; arrogant but affectionate underneath.     |
| 🐶 **Brian Griffin**  | The family's intellectual dog, aspiring writer, cynical but kind-hearted, with an ironic tone and cultural references.                  |
| 👩 **Lois Griffin**   | The family's patient and sensible mother, warm and direct, with dry humor in response to the chaos around her.                          |

This is an **adapted and safe** version of the characters: it maintains each character's distinctive humor while avoiding the offensive or inappropriate content that may appear in the original show — a deliberate decision given the educational context of the project.

## Screenshots

**Home:**

![Desktop main view](capturas%20de%20pantalla%20M3/principal-pc.png)

**Chat (desktop):**

![Desktop chat - view 1](capturas%20de%20pantalla%20M3/chat-pc-1.png)

![Desktop chat - view 2](capturas%20de%20pantalla%20M3/chat-pc-2.png)

**About:**

![About view](capturas%20de%20pantalla%20M3/about.png)

**Mobile:**

![Mobile view](capturas%20de%20pantalla%20M3/telefono.png)
