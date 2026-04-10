# Fuego Focus — Design System

## Design Inspiration

The visual direction is inspired by the FENSEA* aesthetic: clean editorial layout, generous white space, bold typography, soft radial aura/glow effects as background accents, and a mix of photography with organic color washes. The overall feeling is premium, modern, and calm — not cluttered or gamified.

**Key differences from the reference:**
- Replace all greens with a blue palette (light sky blue → deep navy).
- Add orange as a sharp accent color (CTAs, active states, streaks, highlights).
- The "aura" glow effect (the soft radial gradient in the background) should use blues instead of green.

---

## Color Palette

### CSS Variables (Tailwind + Custom Properties)

```css
:root {
  /* ── Blue Aura (Primary) ── */
  --blue-50:  #EFF6FF;   /* Lightest — page backgrounds, card surfaces */
  --blue-100: #DBEAFE;   /* Hover backgrounds, subtle fills */
  --blue-200: #BFDBFE;   /* Borders, dividers */
  --blue-300: #93C5FD;   /* Aura glow — light ring */
  --blue-400: #60A5FA;   /* Aura glow — mid ring */
  --blue-500: #3B82F6;   /* Primary interactive — links, selected states */
  --blue-600: #2563EB;   /* Primary buttons, active nav items */
  --blue-700: #1D4ED8;   /* Button hover */
  --blue-800: #1E3A5F;   /* Dark text on light backgrounds */
  --blue-900: #0F1D2E;   /* Headings, high-contrast text */
  --blue-950: #0A1628;   /* Dark mode background */

  /* ── Orange Accent ── */
  --orange-400: #FB923C;  /* Accent highlights, badges, streak flame */
  --orange-500: #F97316;  /* CTA buttons, primary actions */
  --orange-600: #EA580C;  /* CTA hover */
  --orange-700: #C2410C;  /* CTA active/pressed */

  /* ── Neutrals ── */
  --gray-50:  #F8FAFC;
  --gray-100: #F1F5F9;
  --gray-200: #E2E8F0;
  --gray-300: #CBD5E1;
  --gray-400: #94A3B8;
  --gray-500: #64748B;
  --gray-600: #475569;
  --gray-700: #334155;
  --gray-800: #1E293B;
  --gray-900: #0F172A;

  /* ── Semantic ── */
  --success:  #22C55E;
  --error:    #EF4444;
  --warning:  #FACC15;
}
```

### How Colors Are Used

| Element | Color | Notes |
|---|---|---|
| Page background | `--gray-50` or `--blue-50` | Clean, almost-white base |
| Background aura glow | `--blue-300` → `--blue-400` radial gradient at ~20% opacity | Soft, diffused — not a solid block. Positioned off-center like the FENSEA reference. |
| Card surfaces | `white` with subtle `--blue-100` border | Flat, no drop shadows unless on hover |
| Primary text | `--blue-900` | Near-black with blue undertone |
| Secondary text | `--gray-500` | Descriptions, timestamps, labels |
| Primary buttons (CTAs) | `--orange-500` background, white text | Sign up, Generate Flashcards, Start Session |
| Secondary buttons | `--blue-600` background, white text | Less prominent actions |
| Ghost/outline buttons | `--blue-200` border, `--blue-700` text | Tertiary actions |
| Links | `--blue-500` | Underline on hover |
| Flashcard front | White card, `--blue-900` text | Clean, no distractions |
| Flashcard back | `--blue-50` card, `--blue-800` text | Subtle color shift to signal "flipped" |
| Rating button: Again | `--error` | Red — you missed it |
| Rating button: Hard | `--orange-400` | Orange — struggled |
| Rating button: Good | `--blue-500` | Blue — got it |
| Rating button: Easy | `--success` | Green — nailed it |
| Streak flame icon | `--orange-500` | Fuego! |
| Nav active state | `--orange-500` underline or dot | Minimal but clear |
| "NEW" badge | Black circle, white text (like FENSEA reference) | Sparingly used |
| Quiz correct answer | `--success` background tint | |
| Quiz wrong answer | `--error` background tint | |
| AI tutor chat bubbles (user) | `--blue-600` background, white text | Right-aligned |
| AI tutor chat bubbles (tutor) | `--gray-100` background, `--gray-800` text | Left-aligned |

---

## The Aura Effect

The signature visual element. A soft, radial gradient glow that sits behind content, giving the page a luminous, ethereal feel.

```css
/* Apply to a page-level pseudo-element or a dedicated div */
.aura-glow {
  position: fixed;
  top: -20%;
  left: -10%;
  width: 60vw;
  height: 60vh;
  background: radial-gradient(
    ellipse at center,
    rgba(147, 197, 253, 0.35) 0%,    /* --blue-300 */
    rgba(96, 165, 250, 0.15) 40%,     /* --blue-400 */
    rgba(59, 130, 246, 0.05) 70%,     /* --blue-500 */
    transparent 100%
  );
  filter: blur(60px);
  pointer-events: none;
  z-index: 0;
}

/* Optional: second smaller aura with orange for accent pages */
.aura-glow-accent {
  position: fixed;
  bottom: -15%;
  right: -5%;
  width: 30vw;
  height: 30vh;
  background: radial-gradient(
    ellipse at center,
    rgba(249, 115, 22, 0.15) 0%,      /* --orange-500 */
    rgba(251, 146, 60, 0.05) 50%,     /* --orange-400 */
    transparent 100%
  );
  filter: blur(80px);
  pointer-events: none;
  z-index: 0;
}
```

**Rules for the aura:**
- Always behind content (z-index: 0, pointer-events: none).
- Never centered — position it off to one side like the FENSEA reference.
- Opacity should be subtle (15–35%). If you can draw a hard edge, it's too strong.
- On the dashboard, the aura is blue only. On study sessions, add the faint orange accent aura.
- In dark mode, reduce opacity by half and shift colors slightly brighter.

---

## Typography

| Role | Font | Weight | Size | Tracking |
|---|---|---|---|---|
| Brand name / "FUEGO FOCUS" | `Space Grotesk` or `Syne` | 700 | 20px | 0.15em (wide) |
| Hero headings (like "SALT LEVEL ACTIVATED") | `Space Grotesk` or `Syne` | 700 | 48–64px | 0.02em |
| Section headings | `Space Grotesk` | 600 | 24–32px | 0 |
| Body text | `DM Sans` | 400 | 16px | 0 |
| Small labels, metadata | `DM Sans` | 500 | 12–14px | 0.05em (slightly wide) |
| Code/monospace (if needed) | `JetBrains Mono` | 400 | 14px | 0 |
| Flashcard front text | `DM Sans` | 600 | 20px | 0 |
| Flashcard back text | `DM Sans` | 400 | 18px | 0 |

**Typography rules:**
- Headings are uppercase and tracked wide (like the FENSEA reference: "SALT LEVEL ACTIVATED").
- Body text is sentence case, never uppercase.
- Do not use more than 2 font families on any single page.
- Line height: 1.2 for headings, 1.6 for body.

---

## Layout Principles

1. **Generous whitespace.** The FENSEA reference uses massive margins and breathing room. Do the same. Don't cram elements together. Content width maxes out at 1200px centered.

2. **Asymmetric compositions.** The FENSEA hero splits roughly 40/60 (text left, image right). Study set pages, the dashboard, and the upload page should use similar asymmetric layouts — not boring centered columns.

3. **Cards are flat.** No heavy drop shadows. Use subtle borders (`--blue-200` or `--gray-200`) and slight background tints. Shadow only on hover, and even then keep it soft.

4. **The aura provides depth.** Instead of shadows and gradients on individual elements, the page-level aura glow creates the sense of depth. Individual elements stay flat and clean.

5. **Minimal chrome.** Thin borders, no rounded-everything. Border radius: 8px for cards, 6px for buttons, 4px for inputs. Not 16px+ — that looks bubbly and unserious.

---

## Dark Mode

- Background: `--blue-950`
- Card surfaces: `--gray-800`
- Text: `--gray-100`
- Aura glow: same blue gradient but at 15% opacity (half of light mode)
- Orange accent: unchanged (orange pops on dark backgrounds)
- Borders: `--gray-700`

---

## Component-Level Notes

**Navbar:** Logo left, nav links center, action (Cart equivalent → user avatar / settings) right. Thin bottom border, not a shadow. Fixed/sticky on scroll.

**Flashcard:** Single centered card, max-width 600px. Front shows question in large text. Tap/click to flip (3D CSS transform on Y-axis, 0.4s ease). Back shows answer. Below the card: 4 rating buttons in a horizontal row.

**Chat:** Full-height panel (calc(100vh - navbar)). Messages scroll. Input fixed at bottom. No fancy bubbles — clean rectangles with rounded corners. Tutor messages have a small "FF" avatar icon.

**Upload dropzone:** Dashed border (`--blue-300`), large area, icon + "Drop your files here" text centered. On dragover: border becomes solid `--blue-500`, background tints to `--blue-50`.

**Email/CTA input (like the FENSEA catalogue signup):** We use this pattern on the landing page or Study Set title input: wide white input with an orange "Sign Up" / "Create" button flush to its right edge. Clean, high-contrast.
