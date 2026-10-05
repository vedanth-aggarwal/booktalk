<div align="center">

# 📚 BookTalk

**Talk to your books.** Upload any PDF and have a real-time voice conversation with an AI that has actually read it.

[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-47A248?logo=mongodb&logoColor=white)](https://www.mongodb.com)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?logo=tailwindcss&logoColor=white)](https://tailwindcss.com)

### 🎬 [Watch the Demo](https://drive.google.com/file/d/11tf_Ro4C1MyZH5EtNCsZBBB6jGyRHRzf/view?usp=sharing)

</div>

---

## Overview

Bookify turns static PDFs into conversation partners. Users upload a book, pick a narrator voice, and start a hands-free voice session where they can ask questions, discuss ideas, or get summaries — and the AI answers using the book's actual content rather than generic knowledge.

Under the hood, each uploaded PDF is parsed in the browser, split into overlapping text segments, and indexed in MongoDB. During a call, the voice assistant (powered by Vapi + ElevenLabs) invokes a `searchBook` tool that retrieves the most relevant passages in real time, grounding every response in the text.

## ✨ Features

- **📄 PDF upload & processing** — Client-side parsing with `pdf.js`; the first page is automatically rendered as the book cover if none is provided.
- **🎙️ Real-time voice conversations** — Low-latency, natural turn-taking voice chat with live call states (connecting, listening, thinking, speaking).
- **🔍 Grounded answers (RAG)** — Book text is chunked into ~500-word segments with 50-word overlap and searched via MongoDB full-text search, with a keyword regex fallback.
- **🗣️ Selectable AI voices** — Choose from five curated ElevenLabs voices (male/female, British/American) tuned for conversational delivery.
- **📝 Live transcript** — Streaming transcript of both sides of the conversation, including partial user speech as it's recognized.
- **🔎 Library search** — Browse and search the book library by title or author.
- **🔐 Authentication** — Secure sign-in and route protection with Clerk.
- **💳 Subscription tiers** — Plan-based limits on books, monthly sessions, and session length, enforced server-side and in the live call timer.
- **☁️ Cloud file storage** — PDFs and cover images stored on Vercel Blob via secure, authenticated client uploads.

## 🧭 How It Works

```
┌──────────────┐   parse PDF    ┌──────────────┐   upload    ┌──────────────┐
│  Upload Form │ ─────────────► │  pdf.js      │ ──────────► │ Vercel Blob  │
└──────────────┘  (in browser)  │  + cover gen │             └──────────────┘
                                └──────┬───────┘
                                       │ text → overlapping segments
                                       ▼
                                ┌──────────────┐
                                │   MongoDB    │  Book + BookSegment (text index)
                                └──────▲───────┘
                                       │ searchBook(bookId, query)
┌──────────────┐  voice call    ┌──────┴───────┐
│     User     │ ◄────────────► │  Vapi agent  │  ElevenLabs voice + LLM
└──────────────┘                │  tool call → │  POST /api/vapi/search-book
                                └──────────────┘
```

1. **Upload** — The user submits a title, author, voice persona, PDF, and optional cover image.
2. **Parse** — The PDF is read in the browser; text is extracted page by page and the first page is rendered to an image for the cover.
3. **Store** — Files go to Vercel Blob; book metadata and text segments are saved to MongoDB.
4. **Converse** — Starting a session launches a Vapi voice call configured with the book's title, author, ID, and chosen voice.
5. **Retrieve** — When the user asks something, the assistant calls the `/api/vapi/search-book` webhook, which returns the top matching passages for the AI to answer from.
6. **Track** — Voice sessions are logged with duration and billing period to enforce plan limits.

## 💎 Subscription Plans

| Plan | Books | Sessions / month | Max session length | Session history |
|------|:-----:|:----------------:|:------------------:|:---------------:|
| **Free** | 3 | 5 | 5 min | — |
| **Standard** | 10 | 100 | 15 min | ✅ |
| **Pro** | 100 | Unlimited | 60 min | ✅ |

Plans are managed through Clerk Billing and checked in `lib/subscription.server.ts`.

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 (App Router, Server Actions), React 19, TypeScript |
| Styling / UI | Tailwind CSS 4, shadcn/ui, Radix UI, Lucide icons, Sonner toasts |
| Forms & validation | React Hook Form, Zod |
| Voice AI | Vapi (`@vapi-ai/web`), ElevenLabs voices |
| Database | MongoDB with Mongoose |
| File storage | Vercel Blob |
| Auth & billing | Clerk |
| PDF processing | pdf.js (`pdfjs-dist`) |

## 📁 Project Structure

```
bookify/
├── app/
│   ├── (root)/
│   │   ├── page.tsx              # Home — hero + searchable library
│   │   ├── books/new/page.tsx    # Upload a new book
│   │   └── subscriptions/page.tsx# Pricing & plans
│   ├── books/[slug]/page.tsx     # Book page with voice session
│   └── api/
│       ├── upload/route.ts       # Authenticated Vercel Blob uploads
│       └── vapi/search-book/     # Vapi tool webhook for passage retrieval
├── components/                   # UI: UploadForm, VapiControls, Transcript, VoiceSelector…
├── database/
│   ├── mongoose.ts               # Cached DB connection
│   └── models/                   # Book, BookSegment, VoiceSession schemas
├── hooks/
│   ├── useVapi.ts                # Voice call lifecycle, transcript, timers
│   └── useSubscription.ts        # Client-side plan limits
├── lib/
│   ├── actions/                  # Server actions for books & sessions
│   ├── utils.ts                  # PDF parsing, segmenting, slugs
│   ├── constants.ts              # Voices, file limits, Vapi config
│   └── subscription-constants.ts # Plan definitions
└── proxy.ts                      # Clerk middleware
```

## 🚀 Getting Started

### Prerequisites

- Node.js 20+
- A MongoDB database (e.g. MongoDB Atlas)
- Accounts for [Clerk](https://clerk.com), [Vapi](https://vapi.ai), and [Vercel Blob](https://vercel.com/docs/storage/vercel-blob)

### 1. Clone and install

```bash
git clone https://github.com/vedanth-aggarwal/bookify.git
cd bookify
npm install
```

### 2. Configure environment variables

Create a `.env.local` file in the project root:

```env
# MongoDB
MONGODB_URI=your_mongodb_connection_string

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key
CLERK_SECRET_KEY=your_clerk_secret_key

# Vapi
NEXT_PUBLIC_VAPI_API_KEY=your_vapi_public_key
NEXT_PUBLIC_ASSISTANT_ID=your_vapi_assistant_id

# Vercel Blob
bookified_READ_WRITE_TOKEN=your_vercel_blob_token
```

### 3. Set up the Vapi assistant

In the Vapi dashboard, create an assistant and add a tool named **`searchBook`** with parameters `bookId` and `query`, pointing its server URL to:

```
https://<your-domain>/api/vapi/search-book
```

The assistant receives `{{title}}`, `{{author}}`, and `{{bookId}}` as variables at call start. Recommended turn-taking and timing settings are documented in `VAPI_DASHBOARD_CONFIG` in `lib/constants.ts`.

### 4. Run the app

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## 📜 Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start the development server |
| `npm run build` | Create a production build |
| `npm run start` | Run the production server |
| `npm run lint` | Lint the codebase with ESLint |

## 🗺️ Roadmap

- [ ] Product analytics for uploads and sessions (PostHog)
- [ ] Session history view for Standard and Pro users
- [ ] Semantic (embedding-based) search for richer retrieval

---

<div align="center">

Built by [Vedanth Aggarwal](https://github.com/vedanth-aggarwal)

</div>
