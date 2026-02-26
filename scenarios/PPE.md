# PPE Compliance

## Business Use Case

Monitor whether workers are wearing high-visibility personal protective equipment (safety vests) in designated PPE zones on the factory floor. Like the Failure to Stop workflow, this is a **compliance monitoring** pattern — every person passing through a PPE zone generates an event, either "PPE Compliant" or "PPE Violation", enabling a compliance percentage KPI.

## Why This Pattern Is Interesting

This is the first workflow that uses all three layers of validation:
1. **Type I check** — is the person in a PPE zone?
2. **Type III check** — does the person have a PPE detection overlapping them?
3. **VLM fallback** — if no PPE detection is found by the CV model, run the image through a Vision Language Model for a second opinion

It also demonstrates:
- **Batch mode with both person and PPE tags** — both must be included in the trigger for interactions to be calculated between them
- **Early filtering with Switch nodes** — dropping non-relevant data before running checks (more efficient than letting the orchestrator handle it)
- **VLM as a check gate** (not just metadata) — the VLM boolean determines whether it's compliant or not when the CV model didn't detect PPE
- **Type III check for bbox overlap** — the classic interaction use case (person bbox overlapping PPE bbox = wearing it)

## Workflow Structure

```
Trigger (batch, person+ppe) → Filter to person tags → Type I (PPE zone?) → Filter to PPE zones
    → Type III (person + ppe overlap?)
        ├── No PPE Found → Get Image → VLM Check → Merge ──┐
        └── PPE Found → Get Image (both tracks) → Merge ───┘
            → Edit Fields (pass all) → Orchestrator → Event Manager
```

## Configuration Details

### Trigger
- **Mode**: Batch (60s interval — longer than default to accumulate more tracks)
- **Object Types**: `person` AND `ppe` — both must be included so the state machine calculates interactions between them
- **Data Sources**: 9 cameras covering PPE zones

### Switch 1 — Filter to Person Tags
Since the trigger ingests both person and PPE tags (required for interactions), the first step filters to **person tags only**. We only care about evaluating people — the PPE detections are only relevant as interactions on person tracks.

### Type I Check — PPE Zone
- **Zones**: 8 designated PPE zones
- **Threshold**: Intersection >= 10% only
- Determines whether the person was in an area that requires PPE

### Switch 2 — Filter to PPE Zones
If the person wasn't in a PPE zone (Type I failed), drop them — no need to evaluate PPE compliance outside PPE zones. This is an **early exit pattern** that's more efficient than running the full check chain and letting the orchestrator route to "No Action".

### Type III Check — Person + PPE Overlap
- **Primary tag**: `person`
- **Secondary tag**: `ppe`
- **Threshold**: Overlap >= 1% (any bounding box overlap counts)
- If the person track has an interaction with a PPE track that overlaps, it means the CV model detected PPE on or near this person

### Switch 3 — PPE Found or Not?
Based on `checks.type3.passed`:
- **PPE Found (true)**: The CV model detected PPE overlapping this person. Create a still image showing both the person and PPE tracks at the moment of first interaction. This path goes directly to the merge.
- **No PPE Found (false)**: The CV model didn't detect PPE. Before calling it a violation, run the image through a VLM for a second opinion.

### VLM Fallback — Second Opinion on No-PPE
When no PPE interaction is found:
1. **Create Still Image** — capture the person at their last seen timestamp
2. **VLM Chain** — pass the image to a fine-tuned Azure OpenAI model with a prompt specifically looking for high-visibility PPE (safety vests in neon yellow/green or construction orange)
3. **Structured Output**: `{ "ppe_boolean": true/false, "ppe_confidence": 0.0-1.0, "image_context": "..." }`
4. **Merge** the VLM output back with the original data

This is important because PPE detections from the CV model can be inconsistent — a person might be wearing a vest but the model didn't tag it. The VLM provides a second layer of validation to reduce false violations.

### Edit Fields — Pass All Valid Data
Similar to Failure to Stop, we override `checks.type3.passed = true` so the orchestrator always creates an event. The actual compliance determination comes from the VLM output or the original Type III result downstream.

### Event Manager — Dynamic Classification
The event subtype is determined by the VLM output:
```
={{ $json.output?.ppe_boolean === false ? "PPE Violation" : "PPE Compliant" }}
```

Both start and end time are set (batch mode), so the event is auto-closed.

Metadata includes:
- `image_context` — VLM description of what the person looks like
- Track ID, device info, position

## Key Patterns Demonstrated

1. **Type III check for bbox overlap** — the canonical interaction use case (person + PPE overlay)
2. **VLM as a fallback check** — second opinion when the CV model doesn't detect PPE, reducing false violations
3. **Dual-tag trigger for interactions** — must include both `person` and `ppe` tags so batch mode calculates interactions between them
4. **Early filtering with Switch nodes** — drop non-person tags and non-PPE-zone tracks before running expensive checks
5. **Compliance monitoring** — creates events for both compliant and non-compliant outcomes
6. **Type I + Type III composition** — zone presence combined with interaction detection
7. **Branching workflow** — different processing paths based on whether the CV model found PPE or not, converging before the orchestrator
