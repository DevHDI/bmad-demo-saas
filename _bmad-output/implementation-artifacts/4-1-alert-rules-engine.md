---
status: in-progress
startedAt: "2026-02-11"
epic: "Epic 4: Alerting & Incident Management"
storyPoints: 13
---

# Story 4-1: Alert Rules Engine

## Description

Build a flexible alert rules engine that evaluates metric conditions in real-time and triggers notifications when thresholds are breached or anomalies detected. The engine supports threshold-based rules (value above/below threshold, percentage change), statistical anomaly detection (standard deviation, IQR outliers), and composite rules combining multiple conditions with AND/OR logic for sophisticated alerting scenarios.

The rules engine runs as a scheduled service evaluating all active rules every 1 minute against the latest metrics from continuous aggregates. Each rule tracks its firing state (not firing, firing, resolved) and implements hysteresis to prevent alert flapping when values oscillate around thresholds. The engine supports customizable evaluation windows (last 5min, 15min, 1hr) and recovery windows (how long below threshold before resolving).

Rule testing is critical for confidence before deployment. The system provides a backtesting interface where users can run rules against historical data to see when alerts would have fired, helping tune thresholds and reduce false positives. This enables data-driven alert configuration rather than guessing appropriate thresholds.

## Acceptance Criteria

- AC-1: Threshold rules support operators: above, below, equal, change % increase/decrease
- AC-2: Anomaly detection using Z-score (standard deviations from mean) and IQR (interquartile range) methods
- AC-3: Composite rules combine multiple conditions with AND/OR logic (e.g., "error_rate > 5% AND response_time > 500ms")
- AC-4: Alert evaluation scheduler runs every 1 minute processing all active rules
- AC-5: Rule state tracking: not_firing → firing → resolved with transition timestamps
- AC-6: Hysteresis prevents flapping: threshold + 10% margin for recovery
- AC-7: Configurable evaluation window (last 5min, 15min, 1hr) and recovery window
- AC-8: Rule testing interface runs rule against historical data showing when it would fire
- AC-9: Alert suppression during maintenance windows (scheduled downtime)
- AC-10: Rule validation prevents invalid configurations (e.g., threshold > 100% for percentages)

## Technical Notes

**Alert Rule Schema**:
```typescript
interface AlertRule {
  id: string;
  tenant_id: string;
  name: string;
  description: string;
  metric: string;
  condition: ThresholdCondition | AnomalyCondition | CompositeCondition;
  evaluation_window: '5m' | '15m' | '1h';
  recovery_window: '5m' | '15m' | '1h';
  state: 'not_firing' | 'firing' | 'resolved';
  last_evaluated_at: string;
  last_fired_at?: string;
  enabled: boolean;
}

interface ThresholdCondition {
  type: 'threshold';
  operator: 'above' | 'below' | 'equal';
  value: number;
  hysteresis_percent: number;  // Default 10%
}

interface AnomalyCondition {
  type: 'anomaly';
  method: 'z_score' | 'iqr';
  sensitivity: 'low' | 'medium' | 'high';  // Maps to sigma levels
  baseline_window: '1h' | '24h' | '7d';  // Historical baseline
}

interface CompositeCondition {
  type: 'composite';
  operator: 'AND' | 'OR';
  conditions: Array<ThresholdCondition | AnomalyCondition>;
}
```

**Evaluation Scheduler**:
- Cron job runs every 1 minute
- Batch fetch all enabled rules from database
- For each rule, query current metric value from continuous aggregates
- Evaluate condition and determine if threshold breached
- Update rule state and fire alerts for new firing states
- Skip rules in maintenance windows

**Anomaly Detection**:
```typescript
// Z-score method: Flag if value > mean + (sensitivity * std_dev)
function evaluateZScore(value: number, historical: number[], sensitivity: 'low' | 'medium' | 'high'): boolean {
  const mean = calculateMean(historical);
  const stdDev = calculateStdDev(historical, mean);
  const sigmaLevels = { low: 3, medium: 2, high: 1.5 };
  return value > mean + (sigmaLevels[sensitivity] * stdDev);
}

// IQR method: Flag if value outside Q1-1.5*IQR to Q3+1.5*IQR
function evaluateIQR(value: number, historical: number[]): boolean {
  const sorted = [...historical].sort((a, b) => a - b);
  const q1 = sorted[Math.floor(sorted.length * 0.25)];
  const q3 = sorted[Math.floor(sorted.length * 0.75)];
  const iqr = q3 - q1;
  return value < q1 - 1.5 * iqr || value > q3 + 1.5 * iqr;
}
```

**Hysteresis Implementation**:
- Firing threshold: 100 (trigger when value > 100)
- Recovery threshold: 90 (resolve when value < 90)
- Prevents flapping when value oscillates around 100

**Edge Cases**:
- Missing metric data during evaluation window (skip evaluation, log warning)
- Rule modified while in firing state (re-evaluate with new condition)
- Scheduler falls behind schedule (process backlog in next cycle, alert ops team)
- Anomaly detection with insufficient historical data (require minimum 100 data points)
- Composite rule with mixed firing states (AND: all must fire, OR: any can fire)

## Tasks

- [x] Design alert rule database schema with state tracking
- [x] Build threshold evaluation engine with operator support
- [x] Implement hysteresis logic for threshold recovery
- [ ] Create Z-score anomaly detection algorithm
- [ ] Implement IQR outlier detection method
- [ ] Build composite rule evaluator with AND/OR logic
- [ ] Create alert scheduler service with cron execution
- [ ] Implement rule state machine (not_firing → firing → resolved)
- [ ] Build maintenance window suppression logic
- [ ] Create rule testing interface with historical backtest
- [ ] Add rule validation for invalid configurations
- [ ] Implement metrics for rule evaluation performance
- [ ] Write integration tests for all condition types
- [ ] Load test scheduler with 1000+ active rules
- [ ] Document alerting best practices and threshold tuning guide

## Dependencies

- Story 2-3: Data Storage Layer (metrics to evaluate)
- Story 1-3: Multi-Tenancy (tenant-aware rules)

## Estimation

**Story Points**: 13

**Breakdown**:
- Threshold evaluation: 2 points
- Anomaly detection algorithms: 4 points
- Composite rules: 2 points
- Scheduler and state management: 3 points
- Testing interface: 2 points
