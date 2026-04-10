# Fuego Focus — AI Rules & Engineering Standards

This document is the law. If you're an engineer (human or AI agent) working on this codebase, you follow these rules. No exceptions, no "I'll fix it later," no "it works on my machine." Read this before you write a single line of code.

---

## Non-Negotiables

These are hard rules. Violating any of these is a blocking issue that must be fixed before merge.

### Stack & Language
- **TypeScript only.** No `.js` files in `src/`. Strict mode enabled in `tsconfig.json`. No `any` types unless you leave a `// TODO: type this properly` comment explaining why and what the real type should be.
- **Next.js App Router only.** No Pages Router. No `getServerSideProps`. No `getStaticProps`. Use server components, server actions, and route handlers.
- **Tailwind CSS + shadcn/ui for all styling.** No CSS modules. No styled-components. No inline style objects. If shadcn/ui has a component for it, use it instead of building your own.
- **Supabase for all backend services.** Auth, database, file storage. Do not introduce Firebase, Prisma, Drizzle, or any other database layer. Use the Supabase JS client directly. If you need complex queries, write raw SQL in Supabase migrations — do not add an ORM.
- **Anthropic Claude API for all AI.** No OpenAI, no Gemini, no local models. Use `@anthropic-ai/sdk`. Model: `claude-sonnet-4-20250514` for all generation and chat calls.

### Security
- **API keys in environment variables only.** `ANTHROPIC_API_KEY` and `SUPABASE_SERVICE_ROLE_KEY` live in `.env.local` and Vercel environment settings. Never hardcode. Never log. Never send to the client.
- **All AI calls go through API routes.** The Anthropic SDK is imported only in files under `src/app/api/`. Client components never import it. Client components never see the API key.
- **Supabase RLS is mandatory.** Every table must have Row Level Security enabled with policies that restrict access to `auth.uid() = user_id`. No table is publicly readable. Test RLS policies after every migration.
- **Use `createServerClient` for server-side Supabase calls.** Use `createBrowserClient` for client-side only. Never use the service role key on the client.
- **Validate all inputs on the server.** API routes must validate request bodies before processing. Use Zod schemas. Don't trust anything from the client.

### File & Folder Conventions
- **One component per file.** The filename matches the component name: `FlashcardViewer.tsx` exports `FlashcardViewer`.
- **Colocate related files.** If a component has a hook that only it uses, put the hook in the same directory, not in a global `hooks/` folder. Global hooks go in `src/hooks/` only if multiple components use them.
- **Migrations are sequential and numbered.** `001_create_profiles.sql`, `002_create_study_sets.sql`, etc. Never modify a migration that has been applied. Create a new one.
- **No barrel exports.** No `index.ts` files that re-export from multiple modules. Import directly from the file.

---

## Quality Standards

### Code Style
- **Functional components only.** No class components.
- **Named exports preferred** over default exports (exception: page/layout files required by Next.js).
- **Destructure props.** `function Button({ label, onClick }: ButtonProps)` not `function Button(props: ButtonProps)`.
- **Early returns for guard clauses.** Check error conditions first, return early, then handle the happy path.
- **No commented-out code.** If you're not using it, delete it. Git has history.
- **No `console.log` in production code.** Use it for debugging, remove before committing. If you need server-side logging, use a structured approach (even `console.error` with context is better than bare `console.log`).

### Error Handling
- **Every API route has a try/catch.** Return proper HTTP status codes: 400 for bad input, 401 for unauthenticated, 403 for unauthorized, 500 for server errors. Include a human-readable `error` field in the JSON response.
- **AI calls can fail.** Handle rate limits (429), context too long (400), and network errors. Show the user a clear error message, not a blank screen or cryptic stack trace.
- **Supabase calls can fail.** Always check `.error` on Supabase responses. Don't assume success.
- **File uploads can fail.** Validate file type and size on the client AND server. Reject anything over 20MB. Reject file types other than PDF, PPTX, PNG, JPG, JPEG.

### AI-Specific Rules
- **Every prompt is in `src/lib/ai/prompts.ts`.** No inline prompt strings in API routes. All prompts are named, exported, and documented with a comment explaining what they do.
- **Prompts include output format instructions.** Every generation prompt tells Claude to respond in structured JSON. Include a schema example in the prompt. Parse the response with a Zod schema in `parsers.ts`.
- **Tutor chat has a system prompt that enforces behavior.** The system prompt must:
  - Tell the tutor to ground all answers in the provided material.
  - Tell the tutor to teach, not just answer — ask follow-up questions.
  - Tell the tutor to refuse essay-writing and homework-solving requests.
  - Tell the tutor to say "I don't see that in your materials" when asked about uncovered topics.
- **Temperature settings:** 0.3 for flashcard/quiz/notes generation (deterministic, factual). 0.7 for tutor chat (more conversational).
- **Max tokens:** Set explicit `max_tokens` on every API call. Generation: 4096. Chat: 2048. Don't let calls run unbounded.
- **Stream chat responses.** Use the Anthropic SDK's streaming mode for the tutor chat. Deliver tokens to the client via server-sent events. The user should see words appearing, not a loading spinner followed by a wall of text.

### Spaced Repetition Rules
- **SM-2 algorithm implementation lives in `src/lib/srs/scheduler.ts`.** It's a pure function: `calculateNextReview(card: SRSData, rating: ReviewRating) → SRSData`. No side effects. No database calls inside it.
- **Minimum ease factor is 1.3.** Never let it drop below this.
- **New cards default to interval 0, ease 2.5, repetitions 0.** First successful review sets interval to 1 day. Second successful review sets interval to 6 days. After that, interval = previous interval × ease factor.
- **"Again" always resets.** Interval goes back to 1 day. Repetitions go back to 0. Ease factor decreases by 0.2 (but never below 1.3).
- **Review logs are append-only.** Never update or delete a review_log row. They are an immutable audit trail.

### Testing & Validation
- **Test the SRS scheduler with unit tests.** This is math. It must be correct. Write tests for every rating path (Again, Hard, Good, Easy) and edge cases (minimum ease factor, first review, card at maximum interval).
- **Manually test RLS policies.** After writing a migration, test that a user cannot read/write another user's data. Use Supabase's SQL editor to simulate queries with different `auth.uid()` values.
- **Test AI parsing with malformed responses.** Claude occasionally returns slightly off-format JSON. The parser should handle missing fields with sensible defaults, not crash.

### Performance
- **Use React Server Components for data fetching.** Don't `useEffect` + `fetch` on the client for initial page loads. Server components fetch data before the page reaches the browser.
- **Lazy-load heavy pages.** The quiz runner, flashcard viewer, and chat window can use `dynamic` imports with `ssr: false` since they're fully interactive.
- **Don't refetch what you already have.** If the study set detail page already loaded flashcards, pass them to the flashcard viewer as props. Don't re-query on route change.
- **Paginate dashboard queries.** If a user has 50 study sets, don't load all of them at once. Load 10, add a "Load more" button.

---

## Git & Workflow

- **One feature per branch.** Branch from `main`, name it `feat/upload-flow` or `fix/srs-calculation`. No working directly on `main`.
- **Small, focused commits.** Each commit should do one thing. "Add flashcard review UI" is good. "Update a bunch of stuff" is not.
- **Commit messages:** Start with a verb. `Add flashcard generation API route`, `Fix ease factor calculation for Again rating`, `Remove unused imports`. No periods at the end.
- **No force pushes to `main`.** Ever.

---

## When In Doubt

1. Keep it simple. The simplest solution that works correctly is the right one.
2. Check the PRD. If a feature isn't in the PRD, don't build it.
3. Check ARCHITECTURE.md. If you're about to create a new folder or introduce a new library, check if the architecture doc already specifies where things go.
4. Ask before adding dependencies. Every new `npm install` should be justified. This is a side project — the dependency count should stay small.
