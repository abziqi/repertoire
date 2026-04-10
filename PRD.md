# Fuego Focus — Product Requirements Document

## What Is Fuego Focus?

Fuego Focus is a free, web-based AI study platform that transforms raw course materials (PDFs, PowerPoints, images of handwritten notes, pasted text) into active study tools — flashcards, quizzes, structured notes, and an AI tutor chat — all grounded in the user's own content. It adapts to how the student actually needs to study right now (cramming for tomorrow vs. building long-term retention) and uses spaced-repetition scheduling (Anki-style) to surface the right cards at the right time.

Think of it as: **"Upload your stuff → the app teaches it back to you, the way you need to learn it."**

---

## Core Value Proposition

1. **Material-grounded AI.** Every generated flashcard, quiz, and tutor response is derived from the user's uploaded content — not generic internet knowledge. Your test is based on your professor's slides, so your study tools should be too.
2. **Adaptive study modes.** The same material produces different study experiences depending on the user's goal: a cram session the night before an exam looks nothing like a semester-long retention plan.
3. **Spaced repetition built in.** Flashcards and quiz questions follow an SM-2-based algorithm so the student reviews what they're weakest on, more often.

---

## What Fuego Focus IS

- A **study tool generator**: upload materials → get flashcards, quizzes, and summarized notes instantly.
- An **AI tutor chat** (grounded in the user's materials) that explains concepts, quizzes the student conversationally, and adapts its explanations to what the student doesn't understand.
- A **spaced-repetition system** for flashcards and quiz questions, with scheduling that surfaces weak items more frequently.
- A **multi-mode study platform** that lets the user pick how they want to study:
  - **Cram Mode** — high-volume, exam-focused, prioritizes breadth and flagged weak spots.
  - **Long-Term Mode** — spaced repetition over days/weeks, optimized for durable memory.
  - **Review Mode** — quick daily review of due cards only.
  - **Test Me Mode** — timed practice quiz simulating exam conditions.
- A **free, open side project** with no paywalls, subscriptions, or usage limits.

## What Fuego Focus IS NOT

- **Not a note-taking app.** Users don't write notes here. They upload existing materials and the AI generates structured study content from them.
- **Not a homework solver or essay writer.** The AI tutor helps the student *understand* — it does not produce answers to hand in. It should actively refuse to write essays, solve problem sets for submission, or do the student's work.
- **Not a social/community platform.** No study groups, shared decks, public profiles, or social feeds. Single-player only for MVP.
- **Not a lecture recorder or live transcription tool.** Users upload materials after the fact. No real-time audio capture.
- **Not a course management system.** No syllabi, calendars, assignment tracking, LMS integrations, or teacher dashboards.
- **Not a mobile app.** Web only (responsive, but not a native iOS/Android build).
- **Not a content library.** There are no pre-made decks or shared study sets. Everything is generated from the user's own uploads.

---

## Target User

College and university students (18–25) who have course materials (slides, PDFs, notes) and want to study more efficiently. They are already using some combination of Anki, Quizlet, ChatGPT, and re-reading their notes. Fuego Focus replaces that fragmented workflow with one tool.

---

## Feature Breakdown

### 1. Material Upload & Processing

**What it does:** Accepts the user's course materials and extracts text content for AI processing.

| Input Type | How It Works |
|---|---|
| PDF | Server-side text extraction (pdf-parse or similar) |
| PowerPoint (.pptx) | Server-side slide text extraction |
| Images (handwritten notes, photos of whiteboards) | OCR via Claude's vision capability |
| Pasted text / typed notes | Direct input, no processing needed |

**Requirements:**
- Max file size: 20MB per upload.
- A single upload creates a **Study Set** — the fundamental unit of organization.
- Multiple files can be added to the same Study Set over time.
- The extracted text is stored and used as context for all AI generation.
- The user can view the extracted text and correct OCR errors before generating study tools.

### 2. AI-Generated Study Tools

From a Study Set's extracted content, the user can generate:

**Flashcards**
- Front/back card pairs.
- AI identifies key concepts, definitions, relationships, and facts.
- User can edit, delete, or manually add cards after generation.
- Cards are tagged with difficulty (easy/medium/hard) on initial generation.
- Each card tracks: ease factor, interval, next review date, review count, last review result.

**Quizzes**
- Multiple choice (4 options), true/false, and short answer.
- AI generates questions at varying difficulty levels.
- Questions are tied to specific sections of the source material.
- User sees explanations for correct/incorrect answers (grounded in their material).
- Quiz results update the spaced-repetition schedule for related flashcards.

**Structured Notes**
- AI summarizes the uploaded material into organized, hierarchical notes.
- Headings, subheadings, key terms bolded, relationships called out.
- Not a replacement for the original material — a condensed study reference.

### 3. Study Modes

| Mode | Behavior | When to Use |
|---|---|---|
| **Cram** | All cards shown in priority order (weakest first). No spacing delays. High volume. Session ends when all cards reviewed or time limit hit. | Night before the exam |
| **Long-Term** | SM-2 spaced repetition. Only shows cards that are "due." Introduces new cards gradually (default: 20 new/day). | Ongoing semester study |
| **Review** | Only due cards. Quick session. No new cards introduced. | Daily 10-minute review |
| **Test Me** | Timed quiz from the Study Set's quiz bank. Simulates exam pressure. Scored at the end. | Practice testing |

### 4. Spaced Repetition Engine (Anki-Style)

**Algorithm:** SM-2 variant.

- Each flashcard has: `easeFactor` (default 2.5), `interval` (days), `repetitions`, `nextReviewDate`.
- After each review, user rates: **Again** (0), **Hard** (1), **Good** (2), **Easy** (3).
- Rating updates the scheduling:
  - **Again** → interval resets to 1 day, ease factor decreases.
  - **Hard** → interval × 1.2, ease factor decreases slightly.
  - **Good** → interval × ease factor.
  - **Easy** → interval × ease factor × 1.3, ease factor increases.
- Minimum ease factor: 1.3.
- Cards with `nextReviewDate ≤ today` appear in Review and Long-Term modes.

### 5. AI Tutor Chat

**What it does:** A conversational AI tutor that answers questions about the user's study set content.

**Requirements:**
- The tutor receives the Study Set's extracted text as context with every message.
- Responses are grounded in the uploaded material. If the student asks something not covered, the tutor says so.
- The tutor should **teach, not tell.** When a student asks "what is X?", the tutor explains it, then asks a follow-up to check understanding. It does not just dump the answer.
- The tutor adapts its explanation style: if the student says "I don't get it," it tries a different angle (analogy, simpler language, worked example).
- The tutor refuses to write essays, solve homework for submission, or produce content the student would hand in as their own work.
- Chat history persists per Study Set.

### 6. Dashboard & Progress

- List of all Study Sets with last-studied date and card counts.
- Per Study Set: cards due today, cards learned, cards remaining.
- Simple streak tracker (days studied in a row).
- No gamification beyond streaks. No XP, levels, or achievements for MVP.

---

## Pages / Routes

| Route | Page | Purpose |
|---|---|---|
| `/` | Landing / Dashboard | Shows all Study Sets, due cards summary, streak |
| `/upload` | Upload | Create a new Study Set from uploaded materials |
| `/set/[id]` | Study Set Detail | View generated content, choose study mode, see progress |
| `/set/[id]/flashcards` | Flashcard Review | Active flashcard session (mode-dependent) |
| `/set/[id]/quiz` | Quiz Session | Take a generated quiz |
| `/set/[id]/notes` | Generated Notes | View/read AI-generated notes |
| `/set/[id]/chat` | AI Tutor Chat | Chat with tutor about this Study Set |
| `/settings` | Settings | Study preferences, daily new card limit, theme |
| `/auth/login` | Login | Supabase auth (email + OAuth) |
| `/auth/signup` | Signup | Account creation |

---

## Non-Functional Requirements

- **Performance:** Flashcard transitions < 100ms. AI generation should stream (show content as it's produced, not wait for full completion).
- **Responsiveness:** Usable on mobile browsers (responsive layout), but optimized for desktop.
- **Accessibility:** Keyboard navigation for flashcard review. Sufficient color contrast. Screen reader labels on interactive elements.
- **Data ownership:** Users can export their flashcards as JSON or CSV at any time.
- **No analytics/tracking beyond basic Supabase auth.** This is a side project, not a business.

---

## Out of Scope (Do Not Build)

These are explicitly excluded to prevent scope creep:

- Mobile native apps (iOS/Android)
- Voice-to-voice AI tutor
- Live lecture recording / transcription
- Social features (sharing, groups, public decks)
- LMS integrations (Canvas, Blackboard, D2L)
- Payment/subscription infrastructure
- Admin dashboards or teacher views
- Browser extensions
- Multi-language UI (English only; content in any language is fine)
- Offline support / PWA
- AI-generated videos or audio explanations
