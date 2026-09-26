# TESTING.md — FitBuddy AI Fitness Platform Testing Strategy
## 1. Overview & Testing Philosophy
The **FitBuddy AI** testing strategy ensures data integrity, prompt accuracy, strict JSON schema validation, and reliable application state across all client-server and LLM interactions. Because FitBuddy relies on generative AI models (e.g., Google Gemini) to produce structured workout routines, unit and integration tests focus heavily on **schema enforcement**, **safety guardrails**, and **graceful fallback mechanisms**.
## 2. Testing Pyramid & Tooling
```
               / \
              /   \     E2E Tests
             / Play \   (Playwright)
            /  wright\
           /----------\
          /            \    Integration & AI Tests
         /   Supertest  \   (Vitest + Supertest + Zod Mocking)
        /                \
       /------------------\
      /                    \    Unit Tests
     /   Vitest / Jest      \   (Vitest + React Testing Library)
    /                        \
   /--------------------------\

```
| Testing Layer | Scope | Key Tools | Target Coverage |
|---|---|---|---|
| **Unit Testing** | Individual functions, hooks, state stores (Zustand), UI components. | Vitest, React Testing Library, jsdom | **> 80%** |
| **Integration Testing** | API endpoints, backend context engine, Zod schema validation, database RLS. | Vitest, Supertest, Supabase Local CLI | **> 75%** |
| **AI Prompt & Output Testing** | LLM response parsing, safety/injury guardrails, deterministic JSON format enforcement. | Vitest, Zod, Custom Mock Adapters | **100% of Prompts** |
| **End-to-End (E2E)** | Full user journeys: Onboarding → AI Plan Generation → Interactive Workout → Exercise Swap. | Playwright | Core User Flows |
## 3. Test Suites & Execution Scripts
### 3.1 Unit Testing (Frontend & Logic)
Testing isolated React components, custom hooks, and Zod validator functions.
```typescript
// tests/unit/validators.test.ts
import { describe, it, expect } from 'vitest';
import { userProfileSchema } from '../../src/validators/userProfile';

describe('UserProfile Intake Validation', () => {
  it('should pass with valid profile data', () => {
    const validProfile = {
      age: 28,
      fitnessLevel: 'intermediate',
      goal: 'muscle_gain',
      workoutDaysPerWeek: 4,
      equipment: ['dumbbells', 'pull_up_bar'],
      healthRestrictions: ['knee pain']
    };
    
    const result = userProfileSchema.safeParse(validProfile);
    expect(result.success).toBe(true);
  });

  it('should fail when age is under the legal minimum', () => {
    const invalidProfile = {
      age: 11,
      fitnessLevel: 'beginner',
      goal: 'general_fitness',
      workoutDaysPerWeek: 3,
      equipment: ['bodyweight']
    };

    const result = userProfileSchema.safeParse(invalidProfile);
    expect(result.success).toBe(false);
  });
});

```
### 3.2 Integration & AI Schema Testing
Verifying that the backend API properly validates LLM-generated payloads and handles fallback triggers on schema failure.
```typescript
// tests/integration/aiGeneration.test.ts
import { describe, it, expect, vi } from 'vitest';
import request from 'supertest';
import app from '../../src/app';

// Mock the external Gemini AI service
vi.mock('../../src/services/aiService', () => ({
  generateWorkoutPlanFromAI: vi.fn().mockResolvedValue({
    planTitle: "Mocked Hypertrophy Plan",
    summary: "Mocked plan for testing",
    schedule: [
      {
        day: 1,
        focus: "Upper Body",
        type: "Strength",
        estimatedDurationMinutes: 45,
        exercises: [
          {
            name: "Dumbbell Press",
            sets: 3,
            reps: "8-12",
            restSeconds: 90,
            notes: "Keep core tight"
          }
        ]
      }
    ]
  })
}));

describe('POST /api/v1/plans/generate', () => {
  it('should return 200 OK with valid JSON structured plan', async () => {
    const response = await request(app)
      .post('/api/v1/plans/generate')
      .send({
        userId: "usr_test123",
        profile: {
          age: 25,
          fitnessLevel: "intermediate",
          goal: "muscle_gain",
          targetDaysPerWeek: 4,
          equipment: ["dumbbells"],
          healthRestrictions: []
        }
      });

    expect(response.status).toBe(200);
    expect(response.body.status).toBe("success");
    expect(response.body.data.plan.planTitle).toBe("Mocked Hypertrophy Plan");
    expect(response.body.data.plan.schedule).toHaveLength(1);
  });
});

```
### 3.3 AI Safety & Constraint Verification Test Matrix
```
+------------------------------------+---------------------------------------------------+
| AI Constraint / Safety Rule        | Test Case Specification                           |
+------------------------------------+---------------------------------------------------+
| Injury Exclusion Guardrail         | Input: "Lower back pain"                          |
|                                    | Assert: Output schedule contains 0 exercises      |
|                                    | matching axial loading list (Deadlifts, Squats).  |
+------------------------------------+---------------------------------------------------+
| Equipment Boundary Enforcement     | Input: Equipment = ["bodyweight"]                 |
|                                    | Assert: Output contains NO dumbbell/barbell moves.|
+------------------------------------+---------------------------------------------------+
| JSON Format Resilience             | Input: AI engine returns malformed markdown/JSON. |
|                                    | Assert: Zod parser catches error, triggers retry  |
|                                    | or defaults to rule-based fallback generator.     |
+------------------------------------+---------------------------------------------------+

```
## 4. End-to-End (E2E) Test Suite (Playwright)
```typescript
// tests/e2e/onboardingFlow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('FitBuddy Onboarding & Generation Journey', () => {
  test('Completes intake wizard and generates workout calendar', async ({ page }) => {
    // 1. Visit onboarding page
    await page.goto('http://localhost:5173/onboarding');

    // 2. Complete steps
    await page.click('text=Intermediate');
    await page.click('text=Build Muscle');
    await page.click('button:has-text("4 Days")');
    await page.click('text=Dumbbells');
    
    // 3. Submit wizard
    await page.click('button[type="submit"]');

    // 4. Assert loading state and generated calendar
    await expect(page.locator('.loading-spinner')).toBeVisible();
    await expect(page.locator('h1')).toContainText('Your Personal Plan', { timeout: 8000 });
    await expect(page.locator('.exercise-card')).toHaveCount(4);
  });
});

```
## 5. Running the Tests locally
### Commands (package.json scripts)
Add these scripts to your existing project setup to run all automated test suites:
```bash
# Run all unit and integration tests
npm run test

# Run tests in watch mode during development
npm run test:watch

# Run test coverage report
npm run test:coverage

# Run Playwright E2E tests
npm run test:e2e

```
### Configuration (vitest.config.ts)
```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './tests/setup.ts',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'tests/'],
    },
  },
});

```
testing