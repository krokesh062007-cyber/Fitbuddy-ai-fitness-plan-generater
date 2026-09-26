# DOCUMENTATION.md — FitBuddy AI Fitness Platform
## 1. Executive Overview
**FitBuddy AI** is an intelligent, full-stack fitness generation and tracking platform designed to build personalized, goal-driven workout plans, provide real-time AI coaching, and track exercise progress. By combining dynamic user profiling, automated plan generation via Large Language Models (LLMs), and interactive calendar management, FitBuddy bridges the gap between static workout logs and expensive personal trainers.
## 2. System Architecture
```
+-----------------------------------------------------------------------+
|                             CLIENT (React / Vite)                      |
|   +-------------------+    +--------------------+    +------------+   |
|   |  Onboarding Form  |    | Interactive Cal.   |    |  AI Chat   |   |
|   +---------+---------+    +---------+----------+    +-----+------+   |
+-------------|------------------------|---------------------|----------+
              |                        |                     |
              v                        v                     v
+-----------------------------------------------------------------------+
|                          BACKEND & AI LAYER                           |
|   +---------------------------------------------------------------+   |
|   | API Service (Express / Node.js)                               |   |
|   | - Auth Middleware (JWT)                                       |   |
|   | - Context-Aware Prompt Engine                                 |   |
|   +-------------------------------+-------------------------------+   |
|                                   |                                   |
|                                   v                                   |
|   +---------------------------------------------------------------+   |
|   | Google Gemini / LLM Generation Engine                         |   |
|   | - Structured JSON Generation                                  |   |
|   | - Plan Adaptation & Chat Action Execution                     |   |
|   +---------------------------------------------------------------+   |
+-----------------------------------|-+---------------------------------+
                                    |
                                    v
+-----------------------------------------------------------------------+
|                           PERSISTENCE LAYER                           |
|   +---------------------------------------------------------------+   |
|   | Supabase / PostgreSQL Database                                |   |
|   | - User Profiles & Preferences                                 |   |
|   | - Generated Workouts & Action History                         |   |
|   +---------------------------------------------------------------+   |
+-----------------------------------------------------------------------+

```
## 3. Data Schema & Contracts
### 3.1 Database Schema (PostgreSQL / Supabase)
```sql
-- User Profile & Application State Table
CREATE TABLE IF NOT EXISTS public.user_profiles (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Fitness Intake & Generated Preferences
CREATE TABLE IF NOT EXISTS public.user_fitness_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.user_profiles(user_id) ON DELETE CASCADE,
    fitness_level VARCHAR(50) CHECK (fitness_level IN ('beginner', 'intermediate', 'advanced')),
    primary_goal VARCHAR(50) CHECK (primary_goal IN ('weight_loss', 'muscle_gain', 'endurance', 'general_fitness')),
    available_equipment JSONB NOT NULL DEFAULT '[]'::jsonb,
    workout_days_per_week INT DEFAULT 4,
    health_restrictions JSONB DEFAULT '[]'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Generated Fitness Plan
CREATE TABLE IF NOT EXISTS public.workout_plans (
    plan_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.user_profiles(user_id) ON DELETE CASCADE,
    title VARCHAR(150) NOT NULL,
    duration_days INT DEFAULT 30,
    plan_payload JSONB NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Row Level Security (RLS)
ALTER TABLE public.user_fitness_data ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.workout_plans ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can manage their own fitness data" 
ON public.user_fitness_data 
FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "Users can manage their own workout plans" 
ON public.workout_plans 
FOR ALL USING (auth.uid() = user_id);

```
### 3.2 AI Request & Response Payload Contract
#### Request Payload (POST /api/ai/generate-plan)
```json
{
  "userId": "usr_982347102938",
  "profile": {
    "age": 28,
    "gender": "non-binary",
    "fitnessLevel": "intermediate",
    "goal": "muscle_gain",
    "targetDaysPerWeek": 4,
    "equipment": ["dumbbells", "pull_up_bar", "resistance_bands"],
    "healthRestrictions": ["knee pain on deep flexion"]
  }
}

```
#### Expected AI Generation Output (JSON Schema)
```json
{
  "planTitle": "4-Day Intermediate Hypertrophy & Joint-Friendly Plan",
  "summary": "Focuses on upper-body hypertrophy and knee-friendly lower-body movements.",
  "schedule": [
    {
      "day": 1,
      "focus": "Upper Body Push/Pull",
      "type": "Strength",
      "estimatedDurationMinutes": 45,
      "exercises": [
        {
          "name": "Dumbbell Bench Press",
          "sets": 4,
          "reps": "8-12",
          "restSeconds": 90,
          "notes": "Keep shoulder blades retracted."
        },
        {
          "name": "Bodyweight Pull-Ups",
          "sets": 3,
          "reps": "6-10",
          "restSeconds": 90,
          "notes": "Use resistance band if needed for assist."
        }
      ]
    },
    {
      "day": 2,
      "focus": "Active Rest & Mobility",
      "type": "Rest",
      "estimatedDurationMinutes": 20,
      "exercises": []
    }
  ]
}

```
## 4. Prompt Engineering Blueprint
To ensure stable structured outputs from LLMs (e.g., Google Gemini or OpenAI GPT models), use system instructions with rigid JSON parsing constraints.
```text
SYSTEM INSTRUCTION:
You are FitBuddy AI, an elite strength coach and sports nutritionist.
Your task is to convert raw user health data into an actionable, safe, highly structured workout plan.

RULES:
1. ALWAYS return strictly valid JSON matching the provided schema. Do not output conversational text or markdown wrappers outside the JSON structure.
2. ADAPT TO RESTRICTIONS: If the user lists injuries or equipment constraints, eliminate high-risk movements (e.g., if "knee pain", replace deep barbell squats with hip hinges or wall sits).
3. PROGRESSIVE OVERLOAD: Ensure sets and rep ranges align with the primary goal (Strength: 3-5 reps, Hypertrophy: 8-12 reps, Endurance: 15+ reps).

USER INTAKE CONTEXT:
- Fitness Level: {{fitnessLevel}}
- Goal: {{goal}}
- Equipment: {{equipment}}
- Constraints: {{healthRestrictions}}
- Target Days/Week: {{targetDaysPerWeek}}

```
## 5. Environment & Local Setup
### Prerequisites
 * **Node.js**: v18.x or higher
 * **Database**: PostgreSQL or Supabase instance
 * **AI Access**: Gemini API Key or OpenAI API Key
### Step-by-Step Installation
 1. **Clone Repository & Install Dependencies**
   ```bash
   git clone https://github.com/your-org/fitbuddy-ai.git
   cd fitbuddy-ai
   npm install
   
   ```
 2. **Configure Environment Variables (.env)**
   ```env
   # App Config
   PORT=5000
   NODE_ENV=development
   
   # Database Configuration (Supabase/PostgreSQL)
   DATABASE_URL="postgresql://postgres:password@localhost:5432/fitbuddy"
   SUPABASE_URL="https://your-project.supabase.co"
   SUPABASE_ANON_KEY="your-anon-key"
   
   # AI Platform Integration
   GEMINI_API_KEY="your-gemini-api-key"
   
   # Security
   JWT_SECRET="super-secret-jwt-key"
   
   ```
 3. **Database Migration**
   ```bash
   npm run db:migrate
   
   ```
 4. **Run Server & Frontend**
   ```bash
   # Run local dev environment
   npm run dev
   
   ```
## 6. API Reference
### POST /api/v1/plans/generate
Generates a new AI fitness plan based on user context.
 * **Headers**: Authorization: Bearer <JWT_TOKEN>
 * **Status Codes**:
   * 200 OK: Plan successfully generated and saved.
   * 400 Bad Request: Missing mandatory profile fields.
   * 500 Internal Server Error: AI Service timeout or parsing error.
### PATCH /api/v1/plans/:planId/workout
Modifies an existing workout day using natural language AI instructions (e.g., *"Swap day 3 exercises for dumbbell-only moves"*).
 * **Headers**: Authorization: Bearer <JWT_TOKEN>
 * **Body**:
   ```json
   {
     "day": 3,
     "instruction": "I lost access to the pull up bar today, give me an alternative."
   }
   
   ```
documentation.md