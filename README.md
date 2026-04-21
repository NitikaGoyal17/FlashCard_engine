# FlashMind — AI Flashcard Engine

Built for Cuemath. Turns any PDF into a smart, spaced-repetition flashcard deck.

## Project Structure

```
flashmind/
├── api/
│   ├── auth.js          # Register, login, /me
│   ├── decks.js         # CRUD decks, study queue, SM-2 reviews
│   ├── generate.js      # PDF upload → Claude AI → flashcards
│   └── stats.js         # Dashboard stats
├── lib/
│   ├── models.js        # MongoDB schemas (User, Deck, Card)
│   ├── db.js            # MongoDB connection
│   ├── auth.js          # JWT middleware
│   └── sm2.js           # Spaced repetition algorithm
├── public/
│   └── index.html       # Frontend (deploy separately or serve statically)
├── server.js            # Express entry point
├── vercel.json          # Vercel deployment config
├── .env.example         # Environment variables template
└── package.json
```

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /api/auth/register | No | Create account |
| POST | /api/auth/login | No | Sign in |
| GET | /api/auth/me | Yes | Get current user |
| GET | /api/decks | Yes | List all decks |
| GET | /api/decks/:id | Yes | Get deck + cards |
| POST | /api/decks | Yes | Create deck manually |
| DELETE | /api/decks/:id | Yes | Delete deck |
| GET | /api/decks/:id/study | Yes | Get study queue |
| POST | /api/decks/:id/review | Yes | Submit card rating (SM-2) |
| POST | /api/generate | Yes | Upload PDF → generate deck |
| GET | /api/stats | Yes | Dashboard stats |
| GET | /api/health | No | Health check |

---

## Deployment Guide

### Step 1 — MongoDB Atlas (free database)

1. Go to [mongodb.com/atlas](https://mongodb.com/atlas) → Create free account
2. Create a **free M0 cluster** (choose any region)
3. Under **Database Access** → Add user with password
4. Under **Network Access** → Add `0.0.0.0/0` (allow all IPs for Vercel)
5. Click **Connect** → **Connect your application** → copy the connection string
6. Replace `<password>` in the string with your actual password

### Step 2 — Deploy Backend to Vercel

1. Install Vercel CLI:
   ```bash
   npm install -g vercel
   ```

2. In the `flashmind/` folder, run:
   ```bash
   vercel
   ```
   - Set up: `Y`
   - Which scope: your account
   - Link to existing project: `N`
   - Project name: `flashmind-api`
   - In which directory: `.` (current)
   - Override settings: `N`

3. Add environment variables in Vercel dashboard → your project → **Settings → Environment Variables**:

   | Key | Value |
   |-----|-------|
   | `ANTHROPIC_API_KEY` | `sk-ant-...` (from console.anthropic.com) |
   | `MONGODB_URI` | `mongodb+srv://...` (from Step 1) |
   | `JWT_SECRET` | Any long random string (e.g. `openssl rand -base64 32`) |
   | `FRONTEND_URL` | Your frontend URL (add after Step 3) |

4. Redeploy after adding env vars:
   ```bash
   vercel --prod
   ```

5. Note your backend URL, e.g.: `https://flashmind-api.vercel.app`

### Step 3 — Deploy Frontend to Vercel

1. Open `public/index.html`
2. Find this line near the top of the `<script>`:
   ```javascript
   const API_BASE = 'http://localhost:3001';
   ```
3. Change it to your backend URL:
   ```javascript
   const API_BASE = 'https://flashmind-api.vercel.app';
   ```
4. Create a new folder called `flashmind-frontend/`
5. Copy `public/index.html` into it as `index.html`
6. Deploy:
   ```bash
   cd flashmind-frontend
   vercel
   ```
   - Project name: `flashmind`
7. Your app is live at e.g. `https://flashmind.vercel.app` 🎉

8. Go back to your **backend** Vercel project → Settings → Environment Variables
   → Update `FRONTEND_URL` to `https://flashmind.vercel.app`
   → Redeploy backend

---

## Local Development

```bash
# Install dependencies
npm install

# Copy env file and fill in values
cp .env.example .env

# Run backend
npm run dev
# → Running at http://localhost:3001

# Open public/index.html in browser
# (or use Live Server extension in VS Code)
```

---

## Tech Stack

- **Backend**: Node.js + Express
- **Database**: MongoDB + Mongoose
- **Auth**: JWT (30-day tokens) + bcrypt password hashing
- **AI**: Anthropic Claude claude-sonnet-4-20250514 (server-side, key never exposed)
- **Algorithm**: SM-2 spaced repetition
- **Hosting**: Vercel (serverless)
- **Frontend**: Vanilla HTML/CSS/JS (zero dependencies)
