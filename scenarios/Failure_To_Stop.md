# Failure to Stop

## Business Use Case

Detect when vehicles (forklifts, tuggers, AMRs) fail to come to a complete stop at designated stop sign zones on a factory floor. Unlike most workflows that only detect violations, this workflow captures **both compliant stops and failures** to calculate a compliance percentage as a KPI.

## Why This Pattern Is Interesting

This is a **compliance monitoring** workflow — every vehicle passing through a stop zone generates an event, either a "Successful Stop Event" or a "Failure to Stop Event". This allows you to measure the compliance rate (e.g., 85% of vehicles stopped correctly this week), not just count violations.

It also demonstrates:
- **Type I + Type II check composition** with conditional enforcement via the Check Aggregator
- **Code node for custom check logic** — overriding the Type I pass/fail to pass all valid data through
- **Check Aggregator** — different zones requiring different combinations of checks (directional vs non-directional stop signs)
- **Dynamic event subtype** — the same workflow creates different event types based on the check result

## Workflow Structure

```
Trigger (batch, vehicles) → Type I Check (velocity) → Code Node (pass all valid) → Type II Check (direction) → Check Aggregator → Orchestrator → Image → Event Manager
```

## Configuration Details

### Trigger
- **Mode**: Batch (only need the track summary after the vehicle passes through)
- **Object Types**: Vehicles only (forklift, lift, tugger, golf cart, amr, cmm, sludge king)
- **Data Sources**: 14 cameras covering stop sign areas

### Type I Check — Velocity in Stop Zone
- **Threshold**: Velocity only — each stop zone has a **different velocity threshold** based on its location and typical traffic speed
- **10 threshold groups** with velocity thresholds ranging from 25 to 120 px/s
- Zones with lower thresholds require vehicles to be moving more slowly (stricter stops)
- Zones with higher thresholds are in areas where vehicles naturally move faster
- If the vehicle's average velocity in the zone is below the threshold → Type I passes (they stopped)
- If above → Type I fails (they didn't stop)

### Code Node — "All valid data passes check"
This is the key innovation. Normally, the workflow would only continue if checks pass. But for compliance monitoring, we want to create events for **both** outcomes. The code node:
1. Saves the original Type I pass/fail result as `threshold_passed` (preserved for later use)
2. Overrides `type1.passed = true` for any track with sufficient detections (>= 3) in a checked zone
3. This ensures all valid tracks flow through to event creation regardless of whether they stopped or not

### Type II Check — Directional Enforcement
- **13 zone sequences** configured for directional stop signs
- Each sequence is a pair: [approach zone, stop zone]
- If a vehicle came from the approach zone before reaching the stop zone, the directional check passes
- Some stop signs are non-directional (enforced from any direction) — these don't need Type II

### Check Aggregator — Conditional Check Requirements
This is what ties it together. The aggregator defines two groups:

| Group | Zones | Required Checks | Meaning |
|-------|-------|-----------------|---------|
| **Directional** | 12 zones | Type I + Type II | Vehicle must have been in the stop zone AND came from the correct direction |
| **Non-Directional** | 4 zones | Type I only | Vehicle just needs to have been in the stop zone (any direction) |

The aggregator evaluates which group each zone belongs to and enforces only the required checks for that group.

### Event Manager — Dynamic Event Subtype
The event subtype is determined by the original Type I result (saved as `threshold_passed` by the code node):
```
={{ $json.checks.aggregator.source_checks.type1.threshold_passed === true
   ? "Successful Stop Event"
   : "Failure to Stop Event" }}
```

Key metadata includes:
- `avgVelocity` — the vehicle's actual average velocity in the stop zone
- `threshold` — the velocity threshold for that zone
- `complianceStatus` — "Successful Stop Event" or "Failure to Stop Event"
- `zoneName` — which stop sign zone was evaluated

## Key Patterns Demonstrated

1. **Compliance monitoring** (not just violation detection) — creates events for both pass and fail outcomes
2. **Code node to override check logic** — passes all valid data through regardless of threshold result, while preserving the original result for later classification
3. **Check Aggregator for conditional enforcement** — different zones require different check combinations (directional vs non-directional)
4. **Type I + Type II composition** — velocity check combined with directional zone sequence
5. **Threshold groups with varying values** — different stop zones have different speed requirements based on location
6. **Dynamic event classification** — same workflow produces different event subtypes based on check results
