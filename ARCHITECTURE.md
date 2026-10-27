# TRACEVAULT — Architecture

## Technology Stack
**Frontend:**
- React 18
- Vite
- TypeScript
- Tailwind CSS
- React Flow (Graph Visualization)
- Zustand (State Management)
- Axios (HTTP Client)
- React Router (Routing - PLANNED)

**Backend:**
- Node.js (v24+)
- Express
- TypeScript
- `@google/generative-ai` (Gemini API Integration)

**Shared / Monorepo:**
- npm workspaces (`shared`, `server`, `client`)
- Shared TypeScript types

## Frontend Architecture
The frontend is a React Single Page Application (SPA). It uses a modular component design separated into domains (`layout`, `lookup`, `overview`, `journey`, `ui`, `analysis` [PLANNED]). State is managed globally via Zustand to handle active traces, selected stages, and UI tabs. Data fetching is isolated in a dedicated `api.ts` utility using Axios.

## Backend Architecture
The backend is a stateless Express REST API. It delegates logic to a service layer (`traceService.ts`, `geminiService.ts`). To simplify the hackathon scope and guarantee a flawless offline demo, there is no database. Data is served directly from structured JSON fixtures containing simulated supply chain records. 

## API Structure
- `GET /api/products` — List all demo products
- `GET /api/products/:batchId` — Fetch full trace data for a batch
- `GET /api/products/:batchId/passport` — Fetch Evidence Passport summary
- `GET /api/products/:batchId/analysis` — Fetch deterministic missing links and contradictions
- `POST /api/ai/analyze` — Request AI natural-language analysis via Gemini

## State Management
Global state is managed by `useTraceStore` (Zustand). It holds:
- `currentTrace`: The loaded TraceResponse.
- `selectedStageId` / `selectedTransitionId`: Drives the Stage Detail slide-over panel.
- `activeTab`: Controls view switching between the Graph, Analysis, Passport, and AI Analyst.
- Loading and error states.

## Component Structure
- **AppShell:** Main layout, includes the mandatory Demo Banner.
- **BatchLookup:** Entry point for searching IDs and triggering the simulated QR workflow.
- **ProductOverview:** Renders product metadata and high-level status counts.
- **JourneyGraph:** React Flow canvas displaying the supply chain.
- **StageDetailPanel:** Slide-over displaying evidence, gaps, and conflicts for selected nodes/edges.
- **Pages (LandingPage, ProductPage):** [PLANNED / NOT YET IMPLEMENTED]
- **Analysis Components (MissingLinkDetector, EvidencePassport, etc.):** [PLANNED / NOT YET IMPLEMENTED]

## Evidence Graph Architecture
Built with `React Flow`. 
- **Nodes (`StageNode`):** Custom nodes rendering stage information, evidence counts, and status indicators.
- **Edges (`TransitionEdge`):** Custom edges rendering transition paths. A dashed edge indicates a gap (`UNKNOWN` or `PARTIALLY_DOCUMENTED`). Thickened red edges indicate `CONFLICTING` transitions. 

## Missing Link Detector Architecture
Logic is fully deterministic and server-side (`traceService.ts`). The system iterates over transitions, loads an `expectedEvidence.json` configuration for that specific transition type (e.g., `COLLECTION__PROCESSING`), and checks if the required `MissingField` schemas exist in the actual evidence array. 
*UI representation is [PLANNED / NOT YET IMPLEMENTED]*

## Contradiction Detector Architecture
Deterministically parsed on the server. Conflicts are defined in the JSON data model (e.g., conflicting carrier IDs, mismatched quantities). The backend aggregates these conflicts and serves them with severity levels and the mandatory neutral label ("Record conflict — verification required.").
*UI representation is [PLANNED / NOT YET IMPLEMENTED]*

## Evidence Passport Architecture
The Passport calculates a transparent coverage score on the server:
`coverage = (verifiedCount + 0.5 * partialCount) / totalStages * 100`
This avoids black-box "trust scores."
*Dedicated UI view is [PLANNED / NOT YET IMPLEMENTED]* (Summary exists in `ProductOverview`).

## AI Analyst Architecture
Located in `geminiService.ts`. Receives structured JSON data of the trace and returns a structured JSON payload identifying gaps, conflicts, and recommendations. 
**Safety:** Prompt is heavily constrained. The AI cannot mutate the deterministic statuses.
*UI representation is [PLANNED / NOT YET IMPLEMENTED]*

## Demo-mode Architecture
Driven by JSON fixtures in `server/src/data/fixtures/`. 
All API responses append `isDemo: true` and a `demoLabel: "SIMULATED DEMO DATA"`. The frontend `AppShell` permanently renders a banner warning users that data is synthetic. 

## Error/Fallback Handling
- **Missing API Key:** If `GEMINI_API_KEY` is absent, the backend safely catches this and returns a deterministically generated fallback response matching the AI schema, allowing the UI to render gracefully.
- **API Failures:** Axios interceptors map errors cleanly. UI components like `ErrorState` and `EmptyState` render without crashing the React tree.

## Data Flow
1. User enters Batch ID in `BatchLookup`.
2. Router navigates to `/product/:id`.
3. Frontend calls `GET /api/products/:id`.
4. Express loads JSON fixture, runs `traceService.ts` to compute statuses, analysis, and passport.
5. Response is stored in Zustand (`useTraceStore`).
6. `JourneyGraph` and `ProductOverview` reactively render the data.
7. User selects a node -> Zustand state updates -> `StageDetailPanel` opens.

## Development/Run Commands
- Install: `npm.cmd install`
- Dev Server: `npm.cmd run dev` (Concurrently runs Vite on 5173 and Nodemon/Express on 3001)
- Build: `npm.cmd run build`
