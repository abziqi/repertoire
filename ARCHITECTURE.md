# Fuego Focus — Architecture Document

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Framework** | Next.js 14+ (App Router) | Server components, API routes, file-based routing, streaming support |
| **Language** | TypeScript (strict mode) | Catch bugs at compile time, self-documenting code |
| **Styling** | Tailwind CSS + shadcn/ui | Utility-first, consistent design system, accessible components out of the box |
| **Database** | Supabase (PostgreSQL) | Auth, database, file storage, row-level security — one service for everything |
| **Auth** | Supabase Auth | Email/password + Google OAuth. JWT-based, integrates with RLS |
| **File Storage** | Supabase Storage | PDF, PPTX, and image uploads stored in buckets with per-user access policies |
| **AI** | Anthropic Claude API (claude-sonnet-4-20250514) | Content generation (flashcards, quizzes, notes), tutor chat, OCR via vision |
| **State Management** | React Server Components + `useState`/`useContext` for client state | No Redux. Keep it simple. Server components handle data fetching. |
| **Deployment** | Vercel | Zero-config for Next.js, edge functions, environment variable management |

---

## Folder Structure

```
fuego-focus/
├── .env.local                          # Local environment variables (never committed)
├── .env.example                        # Template showing required env vars
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── package.json
│
├── public/
│   └── icons/                          # Favicon, OG images
│
├── src/
│   ├── app/                            # Next.js App Router
│   │   ├── layout.tsx                  # Root layout (providers, global styles)
│   │   ├── page.tsx                    # Dashboard (landing page when logged in)
│   │   ├── globals.css
│   │   │
│   │   ├── auth/
│   │   │   ├── login/page.tsx
│   │   │   ├── signup/page.tsx
│   │   │   └── callback/route.ts       # Supabase OAuth callback handler
│   │   │
│   │   ├── upload/
│   │   │   └── page.tsx                # Upload materials → create Study Set
│   │   │
│   │   ├── set/
│   │   │   └── [id]/
│   │   │       ├── page.tsx            # Study Set detail / overview
│   │   │       ├── flashcards/page.tsx # Flashcard review session
│   │   │       ├── quiz/page.tsx       # Quiz session
│   │   │       ├── notes/page.tsx      # AI-generated notes viewer
│   │   │       └── chat/page.tsx       # AI tutor chat
│   │   │
│   │   ├── settings/
│   │   │   └── page.tsx
│   │   │
│   │   └── api/                        # API Routes (server-side only)
│   │       ├── generate/
│   │       │   ├── flashcards/route.ts # POST: generate flashcards from extracted text
│   │       │   ├── quiz/route.ts       # POST: generate quiz questions
│   │       │   └── notes/route.ts      # POST: generate structured notes
│   │       ├── chat/
│   │       │   └── route.ts            # POST: AI tutor chat (streaming)
│   │       ├── upload/
│   │       │   └── route.ts            # POST: handle file upload + text extraction
│   │       └── review/
│   │           └── route.ts            # POST: submit flashcard review → update SRS schedule
│   │
│   ├── components/
│   │   ├── ui/                         # shadcn/ui primitives (button, card, dialog, etc.)
│   │   ├── layout/
│   │   │   ├── Navbar.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── PageWrapper.tsx
│   │   ├── dashboard/
│   │   │   ├── StudySetCard.tsx         # Card showing a study set on the dashboard
│   │   │   ├── DueCardsSummary.tsx      # "You have X cards due today"
│   │   │   └── StreakDisplay.tsx
│   │   ├── upload/
│   │   │   ├── FileDropzone.tsx         # Drag-and-drop upload area
│   │   │   ├── TextInput.tsx            # Paste/type text directly
│   │   │   └── ExtractedTextPreview.tsx # Show extracted text for user review
│   │   ├── flashcards/
│   │   │   ├── FlashcardViewer.tsx      # The card flip UI
│   │   │   ├── RatingButtons.tsx        # Again / Hard / Good / Easy
│   │   │   ├── SessionProgress.tsx      # Progress bar for current session
│   │   │   └── ModeSelector.tsx         # Cram / Long-Term / Review picker
│   │   ├── quiz/
│   │   │   ├── QuizRunner.tsx           # Steps through quiz questions
│   │   │   ├── QuestionCard.tsx         # Single question display
│   │   │   └── QuizResults.tsx          # Score summary + explanations
│   │   ├── chat/
│   │   │   ├── ChatWindow.tsx           # Message list + input
│   │   │   ├── ChatMessage.tsx          # Single message bubble
│   │   │   └── ChatInput.tsx            # Text input + send
│   │   └── notes/
│   │       └── NotesViewer.tsx          # Rendered markdown notes
│   │
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts               # Browser Supabase client (createBrowserClient)
│   │   │   ├── server.ts               # Server Supabase client (createServerClient)
│   │   │   └── middleware.ts            # Auth middleware for protected routes
│   │   ├── ai/
│   │   │   ├── anthropic.ts            # Anthropic SDK client initialization
│   │   │   ├── prompts.ts              # All prompt templates (flashcard gen, quiz gen, notes, tutor)
│   │   │   └── parsers.ts              # Parse Claude's responses into typed objects
│   │   ├── srs/
│   │   │   └── scheduler.ts            # SM-2 algorithm: calculateNextReview(card, rating)
│   │   ├── extract/
│   │   │   ├── pdf.ts                  # PDF text extraction
│   │   │   ├── pptx.ts                 # PowerPoint text extraction
│   │   │   └── ocr.ts                  # Image → text via Claude Vision
│   │   └── utils.ts                    # General helpers (date formatting, etc.)
│   │
│   ├── types/
│   │   ├── database.ts                 # Generated Supabase types (supabase gen types)
│   │   ├── flashcard.ts                # Flashcard, ReviewRating, SRSData
│   │   ├── quiz.ts                     # QuizQuestion, QuizResult
│   │   ├── study-set.ts                # StudySet, StudyMode
│   │   └── chat.ts                     # ChatMessage, ChatSession
│   │
│   └── hooks/
│       ├── useStudySet.ts              # Fetch and manage a single study set
│       ├── useFlashcardSession.ts      # Manages active review session state
│       ├── useChat.ts                  # Manages chat messages + streaming
│       └── useDueCards.ts              # Fetch cards due today across all sets
│
├── supabase/
│   ├── migrations/                     # SQL migration files (sequential, numbered)
│   │   ├── 001_create_profiles.sql
│   │   ├── 002_create_study_sets.sql
│   │   ├── 003_create_flashcards.sql
│   │   ├── 004_create_quiz_questions.sql
│   │   ├── 005_create_chat_messages.sql
│   │   └── 006_create_review_logs.sql
│   └── seed.sql                        # Optional dev seed data
│
└── scripts/
    └── generate-types.sh               # Runs supabase gen types typescript
```

---

## Database Schema

### `profiles`
Extends Supabase auth.users with app-specific data.

```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    display_name TEXT,
    daily_new_cards_limit INT DEFAULT 20,
    preferred_study_mode TEXT DEFAULT 'long_term',
    streak_count INT DEFAULT 0,
    last_study_date DATE,
    created_at TIMESTAMPTZ DEFAULT now()
);
```

### `study_sets`
One per upload session. The core organizational unit.

```sql
CREATE TABLE study_sets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    description TEXT,
    extracted_text TEXT NOT NULL,         -- The full extracted content used as AI context
    source_file_paths TEXT[],             -- Array of Supabase Storage paths
    source_type TEXT NOT NULL,            -- 'pdf', 'pptx', 'image', 'text'
    flashcards_generated BOOLEAN DEFAULT FALSE,
    quiz_generated BOOLEAN DEFAULT FALSE,
    notes_generated BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

### `flashcards`
Individual cards with SRS scheduling data baked in.

```sql
CREATE TABLE flashcards (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    study_set_id UUID NOT NULL REFERENCES study_sets(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    front TEXT NOT NULL,
    back TEXT NOT NULL,
    difficulty TEXT DEFAULT 'medium',      -- 'easy', 'medium', 'hard' (AI-assigned)
    -- SRS fields (SM-2)
    ease_factor FLOAT DEFAULT 2.5,
    interval_days INT DEFAULT 0,
    repetitions INT DEFAULT 0,
    next_review_date DATE DEFAULT CURRENT_DATE,
    -- Metadata
    is_user_created BOOLEAN DEFAULT FALSE, -- TRUE if manually added by user
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

### `quiz_questions`
Generated quiz questions tied to a study set.

```sql
CREATE TABLE quiz_questions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    study_set_id UUID NOT NULL REFERENCES study_sets(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    question_type TEXT NOT NULL,           -- 'multiple_choice', 'true_false', 'short_answer'
    question TEXT NOT NULL,
    options JSONB,                         -- For MC: ["optA", "optB", "optC", "optD"]
    correct_answer TEXT NOT NULL,
    explanation TEXT,                      -- Why this answer is correct (grounded in material)
    difficulty TEXT DEFAULT 'medium',
    times_answered INT DEFAULT 0,
    times_correct INT DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT now()
);
```

### `generated_notes`
AI-generated structured notes for a study set.

```sql
CREATE TABLE generated_notes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    study_set_id UUID NOT NULL REFERENCES study_sets(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    content TEXT NOT NULL,                 -- Markdown-formatted notes
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

### `chat_messages`
Persisted chat history per study set.

```sql
CREATE TABLE chat_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    study_set_id UUID NOT NULL REFERENCES study_sets(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    role TEXT NOT NULL,                    -- 'user' or 'assistant'
    content TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now()
);
```

### `review_logs`
Immutable log of every flashcard review (for analytics and debugging SRS).

```sql
CREATE TABLE review_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    flashcard_id UUID NOT NULL REFERENCES flashcards(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
    rating INT NOT NULL,                  -- 0=Again, 1=Hard, 2=Good, 3=Easy
    ease_factor_before FLOAT,
    ease_factor_after FLOAT,
    interval_before INT,
    interval_after INT,
    studied_at TIMESTAMPTZ DEFAULT now()
);
```

### Row-Level Security

Every table has RLS enabled. The policy is the same everywhere:

```sql
-- Users can only see and modify their own data
ALTER TABLE study_sets ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can CRUD their own study sets"
ON study_sets FOR ALL
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- Repeat for every table with a user_id column
```

---

## Data Flow

### Upload Flow
```
User uploads file
    → Client sends file to /api/upload
    → Server stores file in Supabase Storage (bucket: study-materials)
    → Server extracts text:
        PDF → pdf-parse
        PPTX → pptx text extraction
        Image → Claude Vision API (OCR)
        Text → passthrough
    → Server creates study_sets row with extracted_text
    → Client redirects to /set/[id]
```

### Generation Flow
```
User clicks "Generate Flashcards" on study set page
    → Client calls /api/generate/flashcards with study_set_id
    → Server fetches extracted_text from study_sets
    → Server calls Claude API with generation prompt + extracted_text
    → Claude returns structured JSON (array of {front, back, difficulty})
    → Server parses response, bulk inserts into flashcards table
    → Client receives flashcards and renders them
    (Same pattern for quiz and notes generation)
```

### Review Flow
```
User starts flashcard session (picks mode)
    → Client fetches cards based on mode:
        Cram: all cards, ordered by ease_factor ASC (weakest first)
        Long-Term: cards WHERE next_review_date <= today, plus new cards up to daily limit
        Review: cards WHERE next_review_date <= today only
    → User sees card front → flips → rates (Again/Hard/Good/Easy)
    → Client calls /api/review with flashcard_id + rating
    → Server runs SM-2 algorithm → updates flashcard SRS fields
    → Server inserts review_log row
    → Client advances to next card
```

### Chat Flow
```
User sends message in tutor chat
    → Client calls /api/chat with study_set_id + message + chat history
    → Server fetches extracted_text from study_sets
    → Server calls Claude API (streaming) with:
        System prompt: tutor behavior rules
        Context: extracted_text
        Messages: chat history + new message
    → Server streams response back to client
    → Client renders tokens as they arrive
    → On stream complete: save user message + assistant message to chat_messages
```

---

## Key Architectural Decisions

1. **API routes handle all AI calls.** Claude API key never touches the client. All AI interactions go through Next.js API routes.

2. **Extracted text is the source of truth.** The `extracted_text` column in `study_sets` is what every AI call uses as context. The original file is archived in Storage but not re-processed.

3. **SRS state lives on the flashcard row.** No separate scheduling table. The flashcard itself carries its `ease_factor`, `interval_days`, and `next_review_date`. The `review_logs` table is append-only for history.

4. **Streaming for chat, JSON for generation.** The tutor chat uses server-sent events (streaming) for real-time token delivery. Flashcard/quiz/notes generation returns complete JSON payloads because the user needs the full set at once.

5. **No client-side state management library.** Server components fetch data. Client components use `useState` and `useContext` for interactive state (current card index, chat messages in flight, etc.). If state gets complex later, consider Zustand — but not until it's actually needed.

6. **One Supabase project for everything.** Auth, database, and file storage in a single project. RLS handles authorization. No separate auth service.
