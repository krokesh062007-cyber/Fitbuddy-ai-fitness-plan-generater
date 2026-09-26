Here is a ready-to-use DESIGN.md specification file tailored for FitBuddy, an AI-powered fitness and workout generation app.  
​Place this file in the root of your project directory as DESIGN.md (or .cursorrules / .claudecode/design.md) so AI coding agents (Cursor, Claude Code, Windsurf, etc.) read and execute consistent UI patterns.
DESIGN.md - FitBuddy AI Fitness Platform

---
theme: dark-energy
app_name: FitBuddy AI
target_audience: Active adults, gymgoers, athletic enthusiasts, personal training clients
primary_aesthetic: High-contrast athletic dark mode with vibrant energetic accents
---

## 1. Visual Identity & Color System

### Primary Brand Colors
- **Energy Orange (Primary Call to Action):** `#FF5722` (High urgency, start workout, submit)
- **Electric Lime (Accent/Success):** `#A3E635` (Active states, completed sets, streaks)
- **Deep Slate (Background Dark Base):** `#0F172A` (Main background)
- **Card Dark Surface:** `#1E293B` (Elevated component cards and sidebars)
- **Muted Border Grey:** `#334155` (Subtle dividers and outlines)

### Functional Colors
- **Rest/Cool Down:** `#38BDF8` (Sky blue for rest timers & lower intensity)
- **Warning/Heavy Weight:** `#F59E0B` (Amber for max effort or missing inputs)
- **Destructive/Cancel:** `#EF4444` (Red for end workout / clear data)
- **Text Primary:** `#F8FAFC` (Near white for maximum legibility)
- **Text Muted:** `#94A3B8` (Subtitles, metadata, timestamps)

---

## 2. Typography
### Font Families
- **Primary / Body:** `Inter`, `-apple-system`, `BlinkMacSystemFont`, `sans-serif`
- **Headings / Display Numbers:** `Plus Jakarta Sans` or `Oswald` (Athletic impact)
- **Code / Metrics / Timers:** `JetBrains Mono` or `ui-monospace`

### Scale & Hierarchy
- **Display 1 (Big Stats / Timers):** `3rem` (48px) | Bold (`700`) | Line Height: `1.1`
- **H1 (Screen Titles):** `2rem` (32px) | ExtraBold (`800`) | Line Height: `1.2`
- **H2 (Card Headers / Sections):** `1.5rem` (24px) | SemiBold (`600`)
- **H3 (Workout Names / Exercises):** `1.125rem` (18px) | SemiBold (`600`)
- **Body Large:** `1rem` (16px) | Regular (`400`)
- **Caption / Meta:** `0.875rem` (14px) | Medium (`500`) | Text Muted

---

## 3. Spacing & Layout Constraints

- **Base Unit:** `8px` scale (`8px`, `16px`, `24px`, `32px`, `48px`, `64px`)
- **Card Padding:** `16px` on mobile, `24px` on desktop
- **Border Radius Rules:**
  - Standard Cards: `12px` (`rounded-xl`)
  - Buttons / Badges: `8px` (`rounded-lg`) or Pill `9999px` (`rounded-full`)
  - Input Fields: `8px` (`rounded-lg`)
- **Max Container Width:**
  - Dashboard: `1200px`
  - Active Workout Tracker / Generator View: `768px` (Focused single-column)

---

## 4. Key Component Patterns

### Workout Generator Form (AI Input)
- **Layout:** Step-by-step or grouped card interface with clear toggle chips.
- **Inputs:**
  - Muscle group selector: Interactive pills with subtle hover/active states.
  - Duration & Equipment sliders: High-contrast track (`#334155`) with Energy Orange thumb (`#FF5722`).
- **AI Trigger Button:** Full-width primary CTA with prominent gradient fill (`linear-gradient(135deg, #FF5722 0%, #F97316 100%)`) and an inline spark icon.

### Exercise Card (Generated Result)
- **Header:** Exercise title in H3, category pill badge top-right (e.g., "Strength", "Hypertrophy").
- **Grid Layout for Sets/Reps:**
  - 3-column pill layout: `Sets` | `Target Reps` | `Rest Time`
- **Interactive Checkbox:** Large tap-target button on mobile (`44x44px` min) turning Electric Lime (`#A3E635`) when completed.

### Stats & Progress Dashboard
- **Metric Cards:** Numeric value in Display 1 font, label below in Text Muted.
- **Charts:** Minimalist clean line charts or bar charts using Electric Lime and Energy Orange strokes without cluttering gridlines.

---

## 5. Micro-Interactions & Motion Rules

- **Animations:** Fast & reactive (`150ms` - `200ms` `ease-out`).
- **AI Loading State:** Pulsing glowing ring (`#FF5722` aura) or animated skeleton loader inside cards. Avoid blank spinners.
- **Set Completion Feedback:** Subtle scale bounce (`transform: scale(1.03)`) on exercise card when all sets are checked.

---

## 6. Code Generation Instructions for AI Agents

When building React/Vue/Svelte or Tailwind components for FitBuddy:

1. **Always default to Dark Mode.** Do not generate light background classes (`bg-white`) unless explicitly asked.
2. **Use Semantic HTML:** `<main>`, `<section>`, `<article>`, and proper ARIA labels for exercise checkboxes.
3. **Mobile First:** Ensure all touch targets for workouts, sets, and inputpills are at least `44px` tall for easy tapping during active exercise.
4. **Icons:** Use `Lucide-React` (or equivalent SVG library):
   - `Dumbbell` for exercises
   - `Timer` / `Flame` for intensity/duration
   - `Sparkles` or `Zap` for AI generation actions
   - `CheckCircle2` for set completion