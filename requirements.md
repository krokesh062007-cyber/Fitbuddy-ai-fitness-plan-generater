# REQUIREMENTS.md — FitBuddy AI Fitness Platform
## 1. System Requirements & Functional Specifications
### 1.1 Core Functional Requirements (FR)
 * **FR-1: Biometric Intake & User Profiling**
   * The system MUST collect user demographics, primary fitness goals (e.g., muscle gain, fat loss, endurance), workout frequency, available equipment, and physical restrictions or injuries.
   * The system MUST validate intake fields before issuing generation requests to the AI engine.
 * **FR-2: AI Workout Plan Generation**
   * The system MUST generate structured, periodized workout plans matching the user's intake profile using LLMs (e.g., Google Gemini).
   * The AI output MUST conform strictly to a predefined JSON schema (exercise name, sets, rep ranges, rest times, target muscle groups, safety notes).
 * **FR-3: Dynamic Plan Modification & Swap Engine**
   * Users MUST be able to swap individual exercises or modify full workout days using natural language prompts (e.g., *"Replace pull-ups due to elbow strain"*).
   * The AI engine MUST retain target muscle group stimulus and overall volume when substituting movements.
 * **FR-4: Progress & Completion Tracking**
   * The system MUST log completed workouts, exercises, weights, reps, and subjective ratings (RPE / fatigue).
   * The system MUST auto-calculate total volume lifted and workout adherence metrics over time.
## 2. Non-Functional Requirements (NFR)
 * **NFR-1: Performance & Latency**
   * Initial workout plan generation via AI MUST return a complete payload within **< 5.0 seconds**.
   * Dynamic exercise substitution via natural language MUST process within **< 2.5 seconds**.
 * **NFR-2: Reliability & Fallbacks**
   * If the primary AI API service fails or times out, the system MUST fallback to a rule-based algorithmic builder using static exercise templates to maintain availability.
 * **NFR-3: Security & Data Privacy**
   * All communications MUST be encrypted in transit using **TLS 1.3**.
   * User health data and workout history MUST be isolated using Database Row Level Security (RLS) policies.
   * API keys for AI services MUST be securely stored server-side and never exposed to the client.
 * **NFR-4: Scalability**
   * The backend API MUST support horizontally scaled stateless instances handling up to **1,000 concurrent active workout sessions**.
## 3. Technology Stack Requirements
| Component | Technology | Version / Specification |
|---|---|---|
| **Frontend Framework** | React / React Native | React 18+ |
| **Build Tool & Styling** | Vite + Tailwind CSS | Vite 5+, Tailwind CSS 3.4+ |
| **Backend Runtime** | Node.js / Express | Node.js v18 LTS or higher |
| **Language** | TypeScript | v5.0+ |
| **Database** | PostgreSQL / Supabase | Supabase Pg 15+ |
| **AI Platform Integration** | Google Gemini API / Vercel AI SDK | Gemini 1.5 Flash / Pro models |
| **Schema Validation** | Zod | v3.22+ |
## 4. Software Dependencies (package.json)
```json
{
  "name": "fitbuddy-ai-engine",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "server": "node dist/server.js",
    "lint": "eslint ."
  },
  "dependencies": {
    "@google/generative-ai": "^0.14.0",
    "@supabase/supabase-js": "^2.43.0",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2",
    "lucide-react": "^0.380.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "zod": "^3.23.8",
    "zustand": "^4.5.2"
  },
  "devDependencies": {
    "@types/cors": "^2.8.17",
    "@types/express": "^4.17.21",
    "@types/jsonwebtoken": "^9.0.6",
    "@types/node": "^20.12.12",
    "@types/react": "^18.3.2",
    "@vitejs/plugin-react": "^4.2.1",
    "autoprefixer": "^10.4.19",
    "postcss": "^8.4.38",
    "tailwindcss": "^3.4.3",
    "typescript": "^5.4.5",
    "vite": "^5.2.11"
  }
}

```
## 5. API Interface Contracts
### 5.1 Intake & Plan Generation Endpoint
#### POST /api/v1/generate
Generates a full personalized workout plan based on intake parameters.
 * **Headers:**
   ```http
   Content-Type: application/json
   Authorization: Bearer <jwt_token>
   
   ```
 * **Request Body Schema:**
   ```json
   {
     "userId": "string (uuid)",
     "profile": {
       "age": "number (min: 13, max: 100)",
       "fitnessLevel": "string (beginner | intermediate | advanced)",
       "goal": "string (muscle_gain | weight_loss | strength | endurance)",
       "workoutDaysPerWeek": "number (min: 1, max: 7)",
       "equipment": ["string (dumbbells | barbell | bodyweight | cables | bands)"],
       "healthRestrictions": ["string"]
     }
   }
   
   ```
 * **Success Response (200 OK):**
   ```json
   {
     "status": "success",
     "data": {
       "planId": "f789d0a2-1234-4bc1-9012-3456789abcde",
       "createdAt": "2026-09-26T12:00:00Z",
       "plan": {
         "title": "Custom 4-Day Intermediate Muscle Gain",
         "description": "Tailored for home gym with dumbbells and resistance bands.",
         "weeklySchedule": [
           {
             "dayNumber": 1,
             "title": "Upper Body Push Focus",
             "type": "Workout",
             "exercises": [
               {
                 "exerciseId": "ex_01",
                 "name": "Dumbbell Incline Chest Press",
                 "sets": 4,
                 "reps": "8-12",
                 "restSeconds": 90,
                 "targetMuscle": "Upper Pectorals",
                 "safetyTip": "Keep wrists stacked directly over elbows."
               }
             ]
           }
         ]
       }
     }
   }
   
   ```
 * **Error Response (400 Bad Request):**
   ```json
   {
     "status": "error",
     "code": "INVALID_INPUT",
     "message": "Field 'workoutDaysPerWeek' must be between 1 and 7."
   }
   
   ```
## 6. Verification & Acceptance Criteria
```
+------------------------------------+---------------------------------------------------+
| Feature                            | Acceptance Test Criteria                          |
+------------------------------------+---------------------------------------------------+
| Biometric Input Validation         | Rejects invalid age (<13 or >100) and days (>7). |
| AI Output Format Integrity          | Must strictly validate against Zod JSON schema;   |
|                                    | failure triggers auto-retry or static fallback.   |
| Injury Safety Exclusion Filter     | If "knee pain" is listed, system MUST NOT         |
|                                    | include heavy axial/knee flexion exercises.       |
| Exercise Swap Engine               | Swapping an exercise maintains target muscle group|
|                                    | without changing total daily session volume.      |
+------------------------------------+---------------------------------------------------+

```
requirements 