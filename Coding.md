Here is a complete, production-ready CODING.md file tailored for a full-stack **FitBuddy** project (an AI-powered fitness and workout generation application).
Save the content below directly into a CODING.md file in your repository.
# CODING.md — FitBuddy AI Guidelines & Standards
This document establishes development conventions, architectural patterns, coding practices, and AI-prompting contracts for the **FitBuddy** AI fitness project. All developers and automated AI coders (Copilot, Cursor, Gemini, Claude) must adhere to these guidelines.
## 1. Tech Stack Overview
 * **Frontend:** React 18 / Next.js (TypeScript), Tailwind CSS, Lucide Icons, Framer Motion.
 * **Backend / DB:** Supabase / PostgreSQL, Vercel Serverless Functions / Express.js, Node.js.
 * **AI Integration:** Google Gemini API / OpenAI API with structured JSON output.
 * **State & Validation:** Zustand / React Context, Zod / Pydantic.
## 2. Directory Structure
```text
fitbuddy/
├── src/
│   ├── components/         # Reusable UI components
│   │   ├── ui/             # Primitive UI components (Button, Input, Card)
│   │   └── fitness/        # Domain-specific UI (WorkoutCard, PlanViewer)
│   ├── features/           # Feature modules
│   │   ├── ai-generator/   # AI prompt handlers & parser tools
│   │   ├── workout/        # Workout state & tracking logic
│   │   └── profile/        # User biometric & goal management
│   ├── lib/                # Third-party configs (supabase, gemini client)
│   ├── services/           # API handlers and server interactions
│   ├── types/              # Global TypeScript interfaces
│   └── utils/              # Pure helper utilities
├── supabase/               # SQL migrations and schema definitions
├── CODING.md               # Code conventions (This document)
└── package.json

```
## 3. Core Coding Rules
### TypeScript Standards
 1. **No any Types:** Always define specific interfaces or types. Use unknown with runtime validation (Zod) when handling AI or third-party responses.
 2. **Explicit Return Types:** Specify explicit return types for export functions and custom hooks.
 3. **Strict Null Checks:** Always handle null or undefined defensively when working with dynamic AI outputs.
### React & Component Patterns
 1. **Functional Components Only:** Use named functions for components:
   ```tsx
   export function WorkoutCard({ workout }: WorkoutCardProps): JSX.Element { ... }
   
   ```
 2. **Custom Hooks for State Logic:** Separate UI from AI fetch logic. Put API calls, AI generators, and Supabase listeners into custom hooks (e.g., useGenerateWorkout).
 3. **Immutability:** Never mutate state directly. Use updater functions or immutability helpers.
### CSS & Styling
 1. **Tailwind-First:** Use utility classes for styling. Avoid inline styles unless computing dynamic values (e.g., progress bar percentages).
 2. **Responsive Design:** Default to mobile-first responsive design (sm:, md:, lg:).
## 4. AI Generation & Prompt Engineering Rules
### Schema Strictness
All AI outputs **must** strictly conform to JSON schemas using structured prompts or JSON mode. Never accept free-form plain text responses for workout routines.
```typescript
// types/workout.ts
export interface Exercise {
  name: string;
  sets: number;
  reps: number | string;
  restPeriodSeconds: number;
  notes?: string;
}

export interface DailyWorkout {
  dayTitle: string;
  focusArea: string;
  warmupMinutes: number;
  exercises: Exercise[];
  cooldownMinutes: number;
}

export interface WorkoutPlanResponse {
  planTitle: string;
  targetGoal: string;
  weeklyPlan: DailyWorkout[];
}

```
### AI Call Handler Template
```typescript
// features/ai-generator/services/workoutGenerator.ts
import { GoogleGenerativeAI } from "@google/generative-ai";
import { WorkoutPlanResponse } from "@/types/workout";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);

export async function generateFitBuddyPlan(userProfile: Record<string, unknown>): Promise<WorkoutPlanResponse> {
  const model = genAI.getGenerativeModel({
    model: "gemini-1.5-flash",
    generationConfig: { responseMimeType: "application/json" }
  });

  const prompt = `
    You are FitBuddy, an expert AI fitness coach.
    Based on the following user details: ${JSON.stringify(userProfile)}
    
    Generate a structured workout plan formatted as valid JSON adhering to this JSON schema:
    {
      "planTitle": "String",
      "targetGoal": "String",
      "weeklyPlan": [
        {
          "dayTitle": "String",
          "focusArea": "String",
          "warmupMinutes": 5,
          "exercises": [
            { "name": "String", "sets": 3, "reps": "10-12", "restPeriodSeconds": 60, "notes": "Optional advice" }
          ],
          "cooldownMinutes": 5
        }
      ]
    }
  `;

  const result = await model.generateContent(prompt);
  const responseText = result.response.text();
  
  try {
    return JSON.parse(responseText) as WorkoutPlanResponse;
  } catch (err) {
    throw new Error("Failed to parse AI workout output. Format mismatch.");
  }
}

```
## 5. Security & Environment Variable Policy
 1. **API Keys Security:** Never expose GEMINI_API_KEY or admin keys on the client-side bundle.
 2. **Serverless Isolation:** Proxy all AI processing calls through backend API routes (/api/generate-plan).
 3. **Frontend Keys:** Only expose public, non-sensitive parameters with NEXT_PUBLIC_ or VITE_ prefixes.
```bash
# .env.local (Server-side secrets)
GEMINI_API_KEY=your_gemini_api_key_here
SUPABASE_SERVICE_ROLE_KEY=your_supabase_role_key

# Public (Client-side)
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key

```
## 6. Git Workflow & Commit Conventions
Follow the **Conventional Commits** specification:
 * feat: add AI workout modification capability
 * fix: resolve JSON parsing error in Gemini response
 * docs: update setup steps in README
 * style: apply Tailwind updates to workout cards
 * refactor: optimize Supabase user profile query

