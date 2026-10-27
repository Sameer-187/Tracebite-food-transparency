# TRACEVAULT — Project Specification

## Project Name
**TRACEVAULT**

## Tagline
*Don't just trace the food. Trace the proof.*

## Original Hackathon Problem
Consumers rarely know where their food originated, how it was processed, or how many stages it passed through before reaching the marketplace.

## Solution Concept
Create a traceability system that reconstructs a product's supply-chain journey while explicitly flagging missing, incomplete, or unverifiable information. The core question is: *"Which parts of this food journey can we actually prove?"*

## Core Differentiation
Do NOT build a generic food supply-chain timeline. TRACEVAULT must trace BOTH:
1. The journey of the food.
2. The evidence supporting each stage and transition.

Transitions between stages are modeled as first-class entities with their own evidence requirements, meaning a missing transition record is distinguishable from a missing stage record.

## Judging Criteria
- Traceability depth: 30%
- Gap & uncertainty flagging: 25%
- Innovation & originality: 20%
- Demo & presentation: 25%

## Target Users
Consumers seeking transparency about the food products they purchase, and auditors/supply-chain analysts verifying product claims.

## User Journey
1. Landing page.
2. Enter Batch ID or Scan QR Code (Demo-only workflow for MVP).
3. View Product Overview (basic product metadata).
4. Interact with the Evidence-backed Supply-Chain Journey (Graph).
5. Select a Stage or Transition to inspect attached evidence.
6. View Missing Links and Contradictions.
7. View "What Don't We Know?" analysis.
8. View "What Would Complete the Story?" recommendations.
9. Review the overall Evidence Passport.

## Functional Requirements
- **Product Lookup:** Retrieve product traces by ID.
- **Demo QR Scanner:** Simulate scanning to quickly load demo fixtures.
- **Evidence Graph:** Visually represent stages (nodes) and transitions (edges) using React Flow.
- **Evidence Inspection:** Detailed slide-over panels to view evidence records and gaps.
- **Deterministic Status Calculation:** Compute stage/transition statuses using hardcoded rules.
- **AI Analysis:** Generate natural-language summaries and gap explanations using the Gemini API.

## Non-functional Requirements
- Modern responsive web application (React/Vite/Tailwind).
- Simple, reliable architecture (Express/Node.js).
- Environment variables for secrets (no hardcoded API keys).
- Robust error handling and loading states.
- Fully functional offline/demo mode that does not require external APIs to demonstrate the core value.

## Status Definitions
The system evaluates the data to assign one of five statuses:
- **VERIFIED:** Evidence exists and supports the recorded event.
- **PARTIALLY_DOCUMENTED:** Some relevant information exists, but important information is missing.
- **UNKNOWN:** There is not enough information to establish what happened.
- **UNVERIFIABLE:** A claim exists, but available evidence is insufficient to verify it.
- **CONFLICTING:** Available records disagree and require verification.

## Evidence Principles
- **UNKNOWN DOES NOT MEAN FALSE.** The system must never turn missing evidence into a negative claim. 
- A missing record simply means the event cannot currently be independently established.

## Missing-Link Principles
- Every transition (e.g., Collection → Processing) has an expected set of evidence (e.g., shipment event, dispatch timestamp, arrival event, carrier).
- Gaps are calculated deterministically by comparing actual evidence against expected evidence.

## Contradiction Principles
- Structured inconsistencies (event ordering, quantities, batch IDs, facilities, carriers) are flagged.
- **Language Rule:** Never label these as fraud. Use strictly neutral language: *"Record conflict — verification required."*

## AI Safety/Accuracy Rules
- **Rule 1:** The AI must only analyze the structured data provided.
- **Rule 2:** The AI must NEVER invent evidence or fabricate supply-chain events.
- **Rule 3:** The AI must not claim a stage is verified without evidence.
- **Rule 4:** The AI must not treat UNKNOWN as FALSE.
- **Rule 5:** Output must be structured (JSON).

## Demo-Data Rules
- The MVP relies on a synthetic demo dataset.
- The UI and API must explicitly identify demo data using the label: `"SIMULATED DEMO DATA"`.
- Simulated data must never be presented as real-world verified data.

## MVP Scope
1. Product lookup 
2. Product overview
3. Interactive supply-chain journey
4. Evidence attached to each stage
5. Evidence Graph
6. Missing Link Detector [PLANNED / NOT YET IMPLEMENTED IN UI]
7. Contradiction Detector [PLANNED / NOT YET IMPLEMENTED IN UI]
8. "What Don't We Know?" analysis [PLANNED / NOT YET IMPLEMENTED IN UI]
9. "What Would Complete the Story?" recommendations [PLANNED / NOT YET IMPLEMENTED IN UI]
10. Evidence Passport [PLANNED / NOT YET IMPLEMENTED AS DEDICATED COMPONENT, summary implemented]
11. AI Traceability Analyst [PLANNED / NOT YET IMPLEMENTED IN UI]
12. Demo/fallback mode

## Future Scope
- Integration with a real database (PostgreSQL/Prisma).
- Live web-cam QR code scanning.
- User accounts and dashboard for supply chain operators to upload evidence.
