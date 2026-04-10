# How to Use These Documents with Claude Code

## The Documents

You have 5 files:

| File | What It Does | When Claude Code Needs It |
|---|---|---|
| `PRD.md` | Defines what the app is and is not | Always loaded — prevents scope creep |
| `ARCHITECTURE.md` | Folder structure, database schema, data flows, tech stack | Always loaded — tells the agent where things go |
| `AI_RULES.md` | Non-negotiable engineering rules, coding standards | Always loaded — quality control |
| `DESIGN_SYSTEM.md` | Colors, typography, aura effect, layout principles | Loaded when building any UI component |
| `PLAN.md` | Step-by-step roadmap | You reference this yourself to know what to ask for next |

---

## Setup: Load All Foundational Docs into CLAUDE.md

Claude Code uses a file called `CLAUDE.md` in your project root as persistent context. This is where your rules live.

**Step 1: Create your project and initialize Claude Code.**

```bash
mkdir fuego-focus && cd fuego-focus
```

**Step 2: Create a `CLAUDE.md` file in the project root.**

Copy the FULL contents of these three files into `CLAUDE.md`, in this order:
1. `PRD.md`
2. `ARCHITECTURE.md`
3. `AI_RULES.md`

These three are your "always-on" context. Claude Code reads `CLAUDE.md` automatically on every prompt, so the agent will always know what the app is, how it's structured, and what rules to follow.

**Step 3: Keep `DESIGN_SYSTEM.md` as a separate file in the project root.**

Don't put it in CLAUDE.md (it's long and you don't need it for backend-only steps). Instead, reference it when needed:

```
> Look at DESIGN_SYSTEM.md and build the Navbar component following the design system's color palette, typography, and layout principles. Use the blue aura effect as described.
```

**Step 4: Keep `PLAN.md` for yourself.**

PLAN.md is YOUR roadmap. Don't dump it into Claude Code's context. Read it, pick the current step, and prompt for that step only.

---

## How to Prompt for Each Step

### General Prompting Rules

1. **One step at a time.** Don't say "build the whole app." Say "Complete Step 0 from the plan: scaffold the Next.js project with all tooling configured."

2. **Reference the docs, don't re-explain.** Since PRD, ARCHITECTURE, and AI_RULES are in CLAUDE.md, you can say things like:
   - "Follow the folder structure in ARCHITECTURE.md"
   - "Use the database schema from ARCHITECTURE.md for the study_sets table"
   - "Remember: all AI calls go through API routes as specified in AI_RULES.md"

3. **For UI work, always reference the design system.** Any time you're asking for a component with visual output:
   - "Read DESIGN_SYSTEM.md first, then build the dashboard page"
   - "Use the aura glow effect from DESIGN_SYSTEM.md on this page"
   - "Follow the flashcard component spec in DESIGN_SYSTEM.md"

4. **Be specific about what "done" means.** End your prompt with acceptance criteria:
   - "This is done when: the user can upload a PDF, see extracted text, and a study_sets row exists in the database."

5. **Ask Claude Code to check its own work.** After each step:
   - "Run the dev server and verify there are no TypeScript errors."
   - "Check that the RLS policy works by querying as a different user."

---

## Step-by-Step Prompts

Here's roughly what to say for each step. Adapt based on what Claude Code produces.

### Step 0: Scaffolding

```
Scaffold a new Next.js project called fuego-focus following the ARCHITECTURE.md 
folder structure. Use TypeScript strict mode, Tailwind CSS, and the App Router. 
Install these dependencies: @supabase/supabase-js, @supabase/ssr, 
@anthropic-ai/sdk, zod, lucide-react. Initialize shadcn/ui and install: Button, 
Card, Input, Dialog, Textarea, Badge, Tabs, Progress. Create .env.local and 
.env.example with placeholder keys. Create the full folder structure with empty 
placeholder files. Set up Supabase client factories in src/lib/supabase/. Add auth 
middleware that protects all routes except /auth/*.

Done when: npm run dev starts with no errors and the folder structure matches 
ARCHITECTURE.md.
```

### Step 1: Database & Auth

```
Complete Step 1: Database and Auth. Write all 6 SQL migrations from 
ARCHITECTURE.md. Build the login and signup pages at /auth/login and /auth/signup 
with email/password and Google OAuth. Build the OAuth callback handler. Read 
DESIGN_SYSTEM.md and style the auth pages using the blue/orange color palette and 
the aura glow background effect.

Done when: a user can sign up, log in, see a profile row in the database, and 
get redirected to the dashboard.
```

### Step 2: Upload & Text Extraction

```
Complete Step 2: Upload and Text Extraction. Build the /upload page with a 
FileDropzone (PDF, PPTX, PNG, JPG, max 20MB) and a TextInput for pasting text. 
Build the /api/upload API route that stores files in Supabase Storage, extracts 
text (pdf-parse for PDFs, Claude Vision for images), and creates a study_sets row. 
Show the ExtractedTextPreview so users can verify. Read DESIGN_SYSTEM.md for the 
dropzone styling (dashed blue border, orange CTA button pattern).

Done when: I can upload a PDF and see extracted text, upload a photo of notes and 
see OCR text, and paste text directly. Study set appears in the database.
```

### Steps 3–9: Same Pattern

For each subsequent step:
1. Open PLAN.md, read the step
2. Tell Claude Code: "Complete Step N: [title]. [Paste the tasks from PLAN.md]. Reference DESIGN_SYSTEM.md for all UI components."
3. Include the "Done when" criteria from PLAN.md
4. Test before moving to the next step

### When fixing issues between steps:

```
There's a bug: [describe it]. Check AI_RULES.md for the relevant standard and 
fix this. Don't change anything else.
```

---

## Common Mistakes to Avoid

| Mistake | Why It's Bad | What to Do Instead |
|---|---|---|
| Dumping all 5 docs + "build the app" | Agent gets overwhelmed, skips details, produces mediocre everything | One step at a time with focused context |
| Not putting PRD/ARCH/RULES in CLAUDE.md | Agent forgets rules mid-session, drifts from architecture | Always keep the three foundational docs in CLAUDE.md |
| Skipping design system reference for UI work | Generic-looking components, no aura, wrong colors | Always say "read DESIGN_SYSTEM.md" before UI tasks |
| Moving to Step N+1 before Step N is verified | Broken foundations compound. Step 4 bugs are actually Step 2 bugs. | Test each step against its "done when" criteria |
| Asking Claude Code to "make it look good" | Vague. It'll use defaults. | Reference specific design system elements: "use the aura glow," "orange CTA button," "asymmetric 40/60 layout" |
| Letting the agent add libraries not in the stack | Dependency creep | AI_RULES.md says: ask before adding dependencies. Enforce this. |

---

## File Organization Summary

```
fuego-focus/
├── CLAUDE.md           ← Contains: PRD + ARCHITECTURE + AI_RULES (auto-loaded)
├── DESIGN_SYSTEM.md    ← Referenced manually for UI steps
├── PLAN.md             ← Your personal roadmap (don't load into agent)
├── .env.local
├── .env.example
├── src/
│   └── ... (the actual app)
└── supabase/
    └── migrations/
```
