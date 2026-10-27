# TRACEVAULT — Data Model

All models are defined as TypeScript interfaces in the `@tracevault/shared` package.

## Product
Represents the tracked food item at a high level.
- `id` (string): The Batch ID or Lot Number (e.g., "CPR-001").
- `name` (string): Product name.
- `category` (string): Food category (e.g., Grain, Oil).
- `origin` (string): High-level origin string.
- `description` (string): Summary of the product.
- `isDemo` (boolean): Flag indicating synthetic data (always true for MVP).
- `demoLabel` (string): "SIMULATED DEMO DATA".
- `metadata` (Record): Extensible key-value store for certifications, harvest year, etc.
- `stages` (Stage[]): Array of stage records.
- `transitions` (Transition[]): Array of transition records linking the stages.

## Batch
Batch details are decentralized to track variations across stages (e.g., weight loss, split lots).
- Tracked via `batchInfo` inside the `Stage` entity.
- Fields: `batchId`, `quantity`, `unit`, `lotNumber`.

## Supply-Chain Stage (Stage)
A major node in the supply chain.
- `id` (string): Unique stage identifier.
- `order` (number): Execution sequence.
- `type` (StageType): Enum (FARM_ORIGIN, HARVEST, COLLECTION, PROCESSING, etc.).
- `label` (string): Display name.
- `status` (TraceStatus): The deterministically evaluated status.
- `timestamp` (string): ISO 8601 date.
- `location` (Location object): Name, region, country, coords.
- `organization` (Organization object): Entity responsible for this stage.
- `batchInfo` (BatchInfo object): Quantity and ID at this specific stage.
- `evidence` (EvidenceRecord[]): Documentation attached to this stage.
- `missingFields` (MissingField[]): Gaps evaluated at the stage level.
- `notes` (string): Contextual notes.

## Event
Events are implicit within the `Stage` and `EvidenceRecord` models. They are represented by the `timestamp` field denoting when the stage occurred or when the evidence was verified.

## Evidence (EvidenceRecord)
Documentation proving a claim.
- `id` (string): Unique ID.
- `type` (EvidenceType): Enum (CERTIFICATE, SHIPMENT_RECORD, SENSOR_LOG, etc.).
- `source` (string): Issuer of the evidence.
- `description` (string): Human-readable summary.
- `documentRef` (string): URL or ID of the underlying document.
- `timestamp` (string): When the evidence was recorded.
- `verifiedBy` (string): Signatory or verifying body.
- `confidence` (HIGH, MEDIUM, LOW): Weight of the evidence type.
- `isDemo` (boolean): Always true for MVP.

## Transition
A first-class entity representing the movement or handover between two stages.
- `id` (string): Unique identifier.
- `fromStageId` (string): Origin node ID.
- `toStageId` (string): Destination node ID.
- `status` (TraceStatus): Evaluated status of the transition.
- `expectedEvidence` (ExpectedEvidenceSpec[]): What *should* exist for this link.
- `evidence` (EvidenceRecord[]): What *actually* exists.
- `missingEvidence` (MissingField[]): The calculated delta (gaps).
- `conflicts` (Conflict[]): Identified contradictions during this transition.
- `explanation` (string): Human-readable summary of the transition state.

## Gap (MissingField)
Represents a missing piece of required evidence.
- `field` (string): The missing attribute.
- `description` (string): Human-readable definition.
- `importance` (CRITICAL, IMPORTANT, SUPPLEMENTARY): Severity of the gap.
- `resolutionHint` (string): Actionable advice on what evidence would resolve the gap.

## Conflict (Conflict)
Represents conflicting records that cast doubt on the trace.
- `id` (string): Unique ID.
- `type` (ConflictType): Enum (EVENT_ORDERING, QUANTITY_DISCREPANCY, CARRIER_CONFLICT, etc.).
- `affectedStageIds` (string[]): Stages involved in the conflict.
- `record1` / `record2`: Objects detailing the conflicting sources and values.
- `description` (string): Human-readable explanation.
- `severity` (HIGH, MEDIUM, LOW).
- `label` (string): Mandatory string: "Record conflict — verification required."

## Status (TraceStatus Enum)
- `VERIFIED`, `PARTIALLY_DOCUMENTED`, `UNKNOWN`, `UNVERIFIABLE`, `CONFLICTING`.

## AI Analysis Result
Returned by the Gemini API endpoint.
- `summary` (string): Natural language summary.
- `findings` (AIFinding[]): Array of GAP, CONFLICT, RECOMMENDATION, or OBSERVATION.
- `recommendations` (string[]): High-level action items.
- `limitations` (string): AI disclaimer.
- `isAIGenerated` (boolean): Flag to distinguish AI output from raw evidence.
- `fallback` (boolean): True if the backend bypassed the AI due to errors/missing keys.

---

## Simulated Demo-Product Scenarios

### 1. CPR-001: ClearPath Rice
**Scenario:** Mostly documented journey with minor gaps.
- **Trace Depth:** Farm to Retail (10 stages).
- **Status:** Mostly VERIFIED.
- **Key Gaps:** Storage to Transportation transition is missing a carrier record. Quality inspection is self-reported (PARTIALLY_DOCUMENTED).

### 2. MGF-002: MorningGold Wheat Flour
**Scenario:** Multiple incomplete records.
- **Trace Depth:** Farm to Retail (10 stages).
- **Status:** Heavy mix of UNKNOWN and PARTIALLY_DOCUMENTED.
- **Key Gaps:** Collection stage is entirely UNKNOWN (no records). Processing to Packaging transition is UNVERIFIABLE. Retail stage is self-reported. 

### 3. OLV-003: OliveValley Extra Virgin Olive Oil
**Scenario:** Conflicting carrier and quantity records.
- **Trace Depth:** Farm to Retail (10 stages).
- **Status:** Mix of VERIFIED and CONFLICTING.
- **Key Conflicts:** 
  - **Carrier Conflict:** Two different logistics providers are recorded for the Jodhpur to Mumbai leg.
  - **Quantity Discrepancy:** Packaging reports 500 kg, but the distribution intake logs 520 kg.
