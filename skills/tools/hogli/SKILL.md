---
id: hogli
name: Hogli
description: Build and refine Hogli observability views for log and metric correlation during troubleshooting. Use when investigating production behavior through Hogli queries, filters, and dashboards.
category: Tools
requires: []
examples:
  - Use Hogli to correlate request latency spikes with error logs for one service in production.
  - Create a focused Hogli view for deployment-window anomalies across multiple components.
---

# Hogli

Use Hogli effectively by applying scoped filters first, then correlating metrics and logs to verify hypotheses.

## When to Use

- You need to investigate incidents using Hogli as the primary observability tool.
- You want to isolate noisy logs to service-, env-, and timeframe-specific evidence.
- You need shareable views that support incident handoff.

## Investigation Workflow

### Structured Query Building

Build queries incrementally. Start with the narrowest known dimension (e.g., service name, pod ID, request ID), then add filters. Use AND logic to intersect constraints rather than OR. For time-bounded incidents, always anchor to a known event timestamp and expand symmetrically (e.g., ┬▒15 minutes). Include deployment markers or config-change timestamps as explicit boundaries in your query so you can compare before/after behavior.

### Metric-Log Correlation Patterns

Correlate in this order: (1) identify metric anomaly (latency, error rate, throughput); (2) align log query to the same time window; (3) filter logs by severity or error-type first; (4) look for log bursts that precede or coincide with metric inflection points. Use trace IDs or correlation IDs when available to link a single request across logs and metrics. For distributed issues, correlate across services by shared identifiers (trace_id, request_id) rather than time alone.

### Time-Window Comparison Techniques

Compare baseline vs incident windows. Define a "normal" window (e.g., same hour yesterday, same day last week) and an "incident" window. Run identical queries for both and diff the results. Look for new error signatures, changed volume ratios, or new log sources in the incident window. Use deployment timestamps to split "pre-deploy" vs "post-deploy" windows when investigating release-related issues.

## Workflow Steps

1. Define incident scope: service, environment, timeframe, deployment marker.
2. Apply high-signal filters first (service, severity, tenant/region where relevant).
3. Correlate key metrics with log signatures in the same time window.
4. Pivot to representative traces/events to validate root-cause hypotheses.
5. Save the view/query set and annotate findings for other responders.
6. Capture one "next action" query for follow-up verification after mitigation.

## Investigation Heuristics

- Start with "what changed" around deploy/config windows.
- Look for error bursts that precede saturation metrics.
- Validate whether symptoms are isolated or cross-service.
- Prefer one hypothesis at a time to avoid query thrash.
- Use deployment markers as anchors; missing them leads to ambiguous timelines.
- When correlating across services, verify causality (A before B) rather than mere coincidence.
- Narrow by tenant, region, or shard when symptoms are partial to reduce noise.

## Common Pitfalls

- **Broad unfiltered queries**: Scanning all logs or metrics without filters wastes time and obscures signal. Always start with at least service + time + severity.
- **Hypothesis confirmation bias**: Designing queries to confirm a favored theory instead of testing alternatives. Run disconfirming queries explicitly.
- **Missing deploy markers**: Failing to include deployment or config-change timestamps makes before/after comparison impossible.
- **Not saving queries for handoff**: Leaving queries in chat or local notes prevents the next responder from reproducing findings. Save and annotate every diagnostic view.

## Output Format

Return:

- **Scoped Hogli query/view definition**: Exact filter strings, time range, and any saved view name or link.
- **Correlated evidence**: Metrics (name, value, timestamp) + matching log excerpts (message, severity, timestamp) with trace/request IDs where applicable.
- **Root-cause hypothesis confidence level**: High / Medium / Low, with one-sentence rationale.
- **Next diagnostic or mitigation action**: One concrete query or check to run after applying a fix.
- **Baseline vs incident comparison summary**: If applicable, key differences between normal and incident windows.
- **Saved view annotation**: What was saved, where, and what it demonstrates for handoff.

## Constraints

- Keep filters explicit; avoid broad wildcard scans by default.
- Separate observed facts from hypotheses.
- Preserve exact time ranges and query terms for reproducibility.