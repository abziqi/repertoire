# Fuego Focus — Build Plan

## How to Use This Plan

Work through the steps **in order**. Do not skip ahead. Do not start Step 4 while Step 3 is half-done. Each step has a clear definition of "done" — meet that before moving on. If you're using AI coding agents, give them one step at a time with the relevant context from PRD.md, ARCHITECTURE.md, and AI_RULES.md.

When assigning a step to an agent, copy the step's section below along with the relevant architecture/rules sections. Don't dump the entire codebase context — give it what it needs for that step and nothing more.

---

## Step 0: Project Scaffolding

**Goal:** Empty Next.js project with all tooling configured. No features yet.

**Tasks:**
- `npx create-next-app@latest fuego-focus --typescript --tailwind --app --src-dir`
- Enable strict mode in `tsconfig.json`
- Install dependencies: `@supabase/supabase-js`, `@supabase/ssr`, `@anthropic-ai/sdk`, `zod`, `lucide-react`
- Set up shadcn/ui: `npx shadcn@latest init`, install Button, Card, Input, Dialog, Textarea, Badge, Tabs, Progress, Avatar, DropdownMenu
- Create `.env.local` with placeholder keys: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `ANTHROPIC_API_KEY`
- Create `.env.example` with the same keys (no values)
- Create the full folder structure from ARCHITECTURE.md (empty files are fine)
- Set up `src/lib/supabase/client.ts` and `src/lib/supabase/server.ts` with Supabase client factories
- Add auth middleware in `src/middleware.ts` to protect all routes except `/auth/*`

**Done when:**
- `npm run dev` starts without errors
- The folder structure matches ARCHITECTURE.md
- Supabase clients are wired up (even if the DB isn't configured yet)
- Middleware redirects unauthenticated users to `/auth/login`

---

## Step 1: Database & Auth

**Goal:** Users can sign up, log in, and have a profile. Database tables exist with RLS.

**Tasks:**
- Create a Supabase project (or use an existing one for dev)
- Write and apply all 6 SQL migrations from ARCHITECTURE.md:
  - `001_create_profiles.sql` — profiles table + trigger to auto-create on auth.users insert
  - `002_create_study_sets.sql` — study_sets table
  - `003_create_flashcards.sql` — flashcards table
  - `004_create_quiz_questions.sql` — quiz_questions table
  - `005_create_chat_messages.sql` — chat_messages table
  - `006_create_review_logs.sql` — review_logs table
- Enable RLS on every table. Write policies: users can only CRUD their own rows.
- Create a Supabase Storage bucket: `study-materials` (private, authenticated access only)
- Build `/auth/login` page: email/password + Google OAuth button
- Build `/auth/signup` page: email/password registration
- Build `/auth/callback/route.ts`: handle OAuth redirect
- Run `supabase gen types typescript` to generate `src/types/database.ts`
- Test: sign up → profile created → log in → redirected to dashboard → log out → redirected to login

**Done when:**
- User can create an account (email or Google) and log in
- Profile row is auto-created in `profiles` table
- RLS blocks cross-user data access (test this manually in SQL editor)
- All tables exist and match ARCHITECTURE.md schema
- Generated TypeScript types are in `src/types/database.ts`

---

## Step 2: Upload & Text Extraction

**Goal:** User can upload a file (or paste text), and the app extracts text and creates a Study Set.

**Tasks:**
- Build `/upload` page with:
  - `FileDropzone` component (drag-and-drop, accepts PDF/PPTX/PNG/JPG)
  - `TextInput` component (textarea for pasting text directly)
  - Title input field for naming the Study Set
  - File type and size validation (max 20MB) on the client
- Build `/api/upload/route.ts`:
  - Accept multipart form data (file + title) or JSON (text + title)
  - Validate file type and size on the server
  - Upload file to Supabase Storage bucket `study-materials`
  - Extract text based on file type:
    - PDF → use `pdf-parse` npm package
    - PPTX → use `pptx-parser` or similar
    - Image → send to Claude Vision API with prompt "Extract all text from this image. Return only the text, preserving structure."
    - Text → passthrough
  - Create `study_sets` row with extracted_text
  - Return the new study set ID
- Build `ExtractedTextPreview` component: show the extracted text so the user can verify it looks right before proceeding
- After successful upload, redirect to `/set/[id]`

**Done when:**
- User can upload a PDF and see extracted text
- User can upload an image of handwritten notes and see OCR'd text
- User can paste text directly
- Study Set row exists in database with correct `extracted_text`
- File is stored in Supabase Storage
- Error states work: wrong file type, file too large, extraction failure

**Agent context:** Give the agent ARCHITECTURE.md (Upload Flow section), AI_RULES.md (Security + File upload rules), and the relevant type definitions.

---

## Step 3: Flashcard Generation

**Goal:** From a Study Set, the user can generate AI flashcards and view them as a simple list.

**Tasks:**
- Create the flashcard generation prompt in `src/lib/ai/prompts.ts`:
  - Input: extracted text
  - Output: JSON array of `{ front, back, difficulty }` objects
  - Prompt should instruct Claude to identify key concepts, definitions, relationships, and testable facts
  - Include a JSON schema example in the prompt
- Create Zod schema for parsing the response in `src/lib/ai/parsers.ts`
- Build `/api/generate/flashcards/route.ts`:
  - Accept `study_set_id`
  - Fetch `extracted_text` from study_sets
  - Call Claude API with generation prompt (temperature: 0.3, max_tokens: 4096)
  - Parse response with Zod
  - Bulk insert flashcards into database
  - Update `study_sets.flashcards_generated = true`
  - Return generated flashcards
- Build `/set/[id]` page (Study Set detail):
  - Show study set title, source type, date created
  - "Generate Flashcards" button (disabled if already generated)
  - List of generated flashcards (simple front/back list, editable)
  - User can manually add, edit, or delete cards
- Handle loading state during generation (show spinner or progress message)

**Done when:**
- User clicks "Generate Flashcards" → sees loading → flashcards appear
- Flashcards are relevant to the uploaded material (manual quality check)
- Cards are saved in the database with correct SRS defaults
- User can edit card front/back inline
- User can delete a card
- User can add a card manually
- Regenerating is blocked (button disabled after first generation)

**Agent context:** Give the agent AI_RULES.md (AI-Specific Rules), the flashcard type definition, and the prompts.ts file.

---

## Step 4: Flashcard Review & SRS Engine

**Goal:** User can review flashcards in a session with card flip UI and spaced repetition scheduling.

**Tasks:**
- Implement SM-2 algorithm in `src/lib/srs/scheduler.ts`:
  - Pure function: `calculateNextReview(card, rating) → updatedCard`
  - Handle all four ratings: Again (0), Hard (1), Good (2), Easy (3)
  - Enforce minimum ease factor of 1.3
  - Write unit tests for this function (every rating path + edge cases)
- Build `/api/review/route.ts`:
  - Accept `flashcard_id` + `rating`
  - Fetch current card SRS data
  - Run scheduler
  - Update flashcard row with new SRS values
  - Insert review_log row
  - Return updated card
- Build `ModeSelector` component: user picks Cram / Long-Term / Review / Test Me
- Build `FlashcardViewer` component:
  - Shows card front → user taps to flip → shows back
  - Flip animation (CSS transform, nothing fancy)
  - After flip, show rating buttons (Again / Hard / Good / Easy)
- Build `RatingButtons` component
- Build `SessionProgress` component (cards reviewed / total in session)
- Build `/set/[id]/flashcards` page:
  - Fetch cards based on selected mode:
    - Cram: all cards ordered by ease_factor ASC
    - Long-Term: due cards + new cards up to daily limit
    - Review: only due cards
  - Step through cards using FlashcardViewer
  - Session ends when all cards reviewed → show summary (cards reviewed, time spent)
- Wire up keyboard shortcuts: Space to flip, 1/2/3/4 for ratings

**Done when:**
- SM-2 unit tests pass for all rating paths
- User can complete a flashcard session in each mode
- Cards flip with animation
- Ratings update SRS data correctly (verify in database)
- Review logs are created for each review
- Keyboard shortcuts work
- Session summary shows at the end

**Agent context:** Give the agent the SRS section from AI_RULES.md, ARCHITECTURE.md (Review Flow), and the scheduler.ts file.

---

## Step 5: Quiz Generation & Runner

**Goal:** User can generate a quiz from a Study Set and take it.

**Tasks:**
- Create the quiz generation prompt in `src/lib/ai/prompts.ts`:
  - Output: JSON array of question objects with type, question, options, correct_answer, explanation, difficulty
  - Mix of multiple choice, true/false, and short answer
- Create Zod schema for quiz parsing
- Build `/api/generate/quiz/route.ts`:
  - Same pattern as flashcard generation
  - Bulk insert into quiz_questions
  - Update `study_sets.quiz_generated = true`
- Add "Generate Quiz" button to Study Set detail page
- Build `QuizRunner` component:
  - Steps through questions one at a time
  - For MC: radio buttons for options
  - For T/F: two buttons
  - For short answer: text input (AI evaluates closeness to correct answer — this is a stretch, start with exact match and iterate)
  - User submits answer → immediately shown correct/incorrect + explanation
  - Next button advances to next question
- Build `QuizResults` component:
  - Score: X/Y correct
  - List of questions with user's answer, correct answer, and explanation
  - Missed questions optionally feed back into flashcard SRS (mark related cards for re-review)
- Build `/set/[id]/quiz` page that ties it all together

**Done when:**
- User clicks "Generate Quiz" → quiz questions created
- User can take a quiz, answering all three question types
- Immediate feedback shown after each answer
- Final score summary displays at the end
- Quiz results are stored
- "Generate Quiz" button disabled after first generation

---

## Step 6: AI-Generated Notes

**Goal:** User can generate structured notes from their Study Set.

**Tasks:**
- Create notes generation prompt in `src/lib/ai/prompts.ts`:
  - Output: Markdown-formatted structured notes
  - Prompt instructs Claude to organize into headings, subheadings, bold key terms, and note relationships between concepts
- Build `/api/generate/notes/route.ts`:
  - Fetch extracted text, call Claude, save to generated_notes table
  - Update `study_sets.notes_generated = true`
- Build `NotesViewer` component:
  - Renders markdown to HTML (use `react-markdown` or similar)
  - Clean typography, readable layout
- Add "Generate Notes" button to Study Set detail page
- Build `/set/[id]/notes` page

**Done when:**
- User clicks "Generate Notes" → structured notes appear
- Notes are well-organized with headings and key terms highlighted
- Notes are persisted and load on revisit
- Markdown renders cleanly

---

## Step 7: AI Tutor Chat

**Goal:** User can chat with an AI tutor that answers questions grounded in their study set content.

**Tasks:**
- Create the tutor system prompt in `src/lib/ai/prompts.ts`:
  - Tutor is grounded in the provided material
  - Tutor teaches, doesn't just answer — asks follow-up questions
  - Tutor refuses to write essays or solve homework
  - Tutor says "I don't see that in your materials" for uncovered topics
  - Tutor adapts explanation style when student says they don't understand
- Build `/api/chat/route.ts`:
  - Accept study_set_id, message, and chat history
  - Fetch extracted_text as context
  - Call Claude API in streaming mode (temperature: 0.7, max_tokens: 2048)
  - Stream response back via ReadableStream / server-sent events
  - After stream completes, save both messages to chat_messages table
- Build `ChatWindow`, `ChatMessage`, and `ChatInput` components:
  - Scrolling message list
  - User messages on right, tutor on left
  - Streaming: tutor message builds token by token
  - Input with send button + Enter to send, Shift+Enter for newline
- Build `/set/[id]/chat` page:
  - Load existing chat history on mount
  - New messages append to the conversation
- Build `useChat` hook to manage message state and streaming

**Done when:**
- User can send a message and see the tutor respond in real-time (streaming)
- Tutor answers are grounded in the uploaded material
- Tutor asks follow-up questions to check understanding
- Chat history persists across page reloads
- Tutor refuses to write essays (test this)
- Tutor admits when something isn't in the materials (test this)

---

## Step 8: Dashboard & Progress

**Goal:** The home page shows all study sets, due cards, and streak tracking.

**Tasks:**
- Build the Dashboard page (`/` route):
  - List of all Study Sets as cards (title, source type, date, card counts)
  - Sorted by most recently studied
  - "Due today" badge showing cards due across all sets
  - Click a card → navigate to `/set/[id]`
- Build `DueCardsSummary` component:
  - Query flashcards where `next_review_date <= today` grouped by study_set
  - Show total due count prominently
- Build `StreakDisplay` component:
  - Check if user studied today (any review_log with today's date)
  - If yes and they studied yesterday, increment streak
  - Display current streak count with a flame icon (on brand)
- Update `profiles` table: update `streak_count` and `last_study_date` after each review session
- Add empty states: no study sets yet → "Upload your first materials" CTA

**Done when:**
- Dashboard shows all study sets with relevant metadata
- Due card count is accurate
- Streak updates correctly (test: study today → streak increments; skip a day → streak resets)
- Empty state looks good for new users
- Navigation between dashboard and study sets works smoothly

---

## Step 9: Settings & Polish

**Goal:** User preferences work. The app feels finished.

**Tasks:**
- Build `/settings` page:
  - Daily new cards limit (number input, default 20)
  - Default study mode preference
  - Theme toggle (light/dark) using Tailwind dark mode
  - Export flashcards as JSON button
- Implement theme with `next-themes` or Tailwind's `darkMode: 'class'`
- Add loading skeletons to all pages that fetch data
- Add proper error boundaries
- Responsive design pass: make sure upload, flashcard review, quiz, and chat work on mobile viewport
- Keyboard accessibility pass: tab navigation, focus rings, screen reader labels
- Add page titles and meta descriptions for each route
- Final cleanup: remove unused files, run linter, fix TypeScript errors

**Done when:**
- Settings save and persist
- Light/dark theme works across all pages
- No layout breakage on mobile
- All interactive elements are keyboard accessible
- No TypeScript errors, no linter warnings
- Export produces valid JSON with all flashcard data

---

## Step 10: Deploy

**Goal:** App is live on the internet.

**Tasks:**
- Create a Vercel project connected to the GitHub repo
- Set environment variables in Vercel: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `ANTHROPIC_API_KEY`
- Configure Supabase auth redirect URLs for production domain
- Test full flow on production: sign up → upload → generate → study → chat
- Set up Supabase Storage CORS for production domain
- Verify RLS works in production (not just local)

**Done when:**
- App is accessible at a public URL
- Full user flow works end-to-end in production
- No environment variable leaks
- Auth redirects work correctly on the production domain

---

## After MVP (Future Ideas — Do Not Build Yet)

These are ideas for later. Do not build any of these during the 10 steps above.

- Voice-to-voice AI tutor (like StudyFetch's Spark.E call feature)
- Lecture recording + live transcription
- AI-generated audio summaries / podcast-style recaps
- Import from Canvas, Google Docs, Quizlet
- Shared study sets / social features
- Spaced repetition analytics dashboard (review heatmap, retention curves)
- Mobile native apps
- Browser extension for capturing web content into study sets
- Multi-language UI
- Offline / PWA support
