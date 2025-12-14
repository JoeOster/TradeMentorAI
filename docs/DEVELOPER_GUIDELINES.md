# TradeMentorAI: Developer Guidelines & Coding Standards

## 1. Core Philosophy
* **DRY (Don't Repeat Yourself):** Never duplicate logic. If a function is used in two places (e.g., formatting currency or dates), move it to `src/lib/utils.ts`.
* **Single Source of Truth:** The Database (Postgres) is the truth. Do not rely on derived state in the frontend if you can fetch it fresh.
* **Type Safety:** We use TypeScript strictly. `any` is forbidden. All AI responses must be validated with Zod schemas.

## 2. Tech Stack & Patterns
* **Framework:** Next.js 15 (App Router). Use Server Components (`page.tsx`) for data fetching and Client Components (`components/...`) for interactivity.
* **Database:** PostgreSQL with **Drizzle ORM**.
    * Always export inferred types from `schema.ts`.
    * Use `drizzle-zod` for frontend form validation if possible.
* **Styling:** Tailwind CSS + Shadcn UI.
    * Do not write custom CSS in `globals.css` unless absolutely necessary.
    * Use `cn()` for class merging.

## 3. Architecture Rules

### A. The AI Integration ("The Brain")
* **Rule:** Never call `generateText` directly in UI components.
* **Rule:** Always use the centralized `src/lib/ai-engine.ts`.
* **Rule:** All AI interactions **must** use `generateObject` to ensure structured, type-safe JSON responses. We do not parse raw strings.

### B. Server Actions vs. API Routes
* **Preference:** Use **Server Actions** (`src/app/actions.ts` or `src/actions/...`) for all mutations and data fetching where possible.
* **Pattern:** Server Actions should return a standardized object:
    ```typescript
    type ActionResponse<T> = {
      success: boolean;
      data?: T;
      error?: string;
    }
    ```

### C. Directory Structure
* `src/app/`: Routes and Pages only. Minimal logic.
* `src/components/`: Reusable UI.
    * `ui/`: Shadcn primitives (Button, Card). Do not modify these logic-wise.
    * `features/`: Complex, domain-specific components (e.g., `MarketScanner`, `JournalEntryForm`).
* `src/lib/`: Utilities, Type Definitions, and Service Clients (AI, TradeStation).
* `src/db/`: Drizzle schema and connection logic.

## 4. Specific "Anti-Patterns" to Avoid
1.  **The "Raw SQL" Trap:** Do not write raw SQL queries string concatenation. Use Drizzle's query builder API.
2.  **The "Inline Fetch":** Do not write `fetch('tradestation...')` inside a Component. Create a service function in `src/lib/tradestation.ts` and call that.
3.  **The "Prop Drilling" Mess:** If a component needs 5+ props passed down 3 levels, use a Context or verify if the child component should fetch its own data.

## 5. TradeStation & External APIs
* **Token Safety:** Never expose Access Tokens to the client. All TradeStation calls happen server-side.
* **Rate Limiting:** All external calls must handle `429` errors gracefully and return a user-friendly message, not crash the app.

## 6. Testing (Future Proofing)
* Write logic in `src/lib/` so it can be tested independently of React components.
