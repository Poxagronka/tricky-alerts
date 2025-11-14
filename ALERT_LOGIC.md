# Alert Logic Documentation

## Overview
This document describes the enhanced alert classification system that uses volume-based filtering and impact scoring to prioritize campaign performance anomalies.

## Key Improvements

### 1. **Volume-Based Campaign Tiers**
Campaigns are classified into tiers based on their daily volume:

| Tier | Min Installs | Min Spend | Description |
|------|--------------|-----------|-------------|
| **CRITICAL** | 50 | $500 | Large campaigns with significant budget |
| **HIGH** | 25 | $200 | Medium-sized campaigns |
| **MEDIUM** | 10 | $50 | Small campaigns |
| **LOW** | 5 | $20 | Micro campaigns (reduced sensitivity) |
| **IGNORE** | <5 | <$20 | Too small to monitor |

### 2. **Dynamic Deviation Thresholds**

Different thresholds apply based on campaign tier:

#### For CRITICAL Tier Campaigns (most sensitive):
- **15%** deviation triggers alert (MINOR threshold)
- Reason: Large campaigns need immediate attention for small changes

#### For HIGH/MEDIUM Tier Campaigns (standard):
- **20%** deviation triggers alert (MODERATE threshold)
- Reason: Balanced approach for regular campaigns

#### For LOW Tier Campaigns (less sensitive):
- **30%** deviation triggers alert (SEVERE threshold)
- Reason: Small campaigns have more natural variance

### 3. **Impact Score System**

Each anomaly receives an impact score (0-300+) based on:

#### Volume Component (10-100 points):
- CRITICAL tier: +100 points
- HIGH tier: +60 points
- MEDIUM tier: +30 points
- LOW tier: +10 points

#### Deviation Severity (5-50 points per metric):
- ≥50% deviation: +50 points
- ≥40% deviation: +35 points
- ≥30% deviation: +25 points
- ≥20% deviation: +15 points
- <20% deviation: +5 points

#### Multiple Metrics Bonus (15-30 points):
- 3+ affected metrics: +30 points (systemic issues)
- 2 affected metrics: +15 points

#### Spend Impact Bonus (10-20 points):
- Daily spend >$1000: +20 points
- Daily spend >$500: +10 points

### 4. **Alert Severity Levels**

Final severity is determined by total impact score:

| Severity | Impact Score | Description | Action Required |
|----------|--------------|-------------|-----------------|
| 🔴 **CRITICAL** | ≥150 | High-impact issues on large campaigns | Immediate attention |
| 🟠 **HIGH** | 100-149 | Significant issues requiring action | Investigate within hours |
| 🟡 **MEDIUM** | 60-99 | Notable issues to monitor | Check within day |
| 🔵 **LOW** | 30-59 | Minor issues, FYI only | Optional review |
| **INFO** | <30 | Not sent (logged only) | No action needed |

## Alert Examples

### Example 1: CRITICAL Alert
```
Campaign: Large brand campaign
Daily Stats: 80 installs, $850 spend
Anomalies:
- CPI +35% ($10.50 vs avg $7.80)
- CTR -28% (1.2% vs avg 1.7%)

Calculation:
- Volume tier: CRITICAL (+100)
- CPI deviation 35%: +25
- CTR deviation 28%: +25
- 2 metrics bonus: +15
- Spend >$500: +10
Total Impact Score: 175 → CRITICAL
```

### Example 2: MEDIUM Alert
```
Campaign: Small test campaign
Daily Stats: 12 installs, $75 spend
Anomalies:
- IPM -25% (8.5 vs avg 11.3)

Calculation:
- Volume tier: MEDIUM (+30)
- IPM deviation 25%: +25
- 1 metric (no bonus): +0
Total Impact Score: 55 → LOW (not HIGH because small volume)
```

### Example 3: Filtered Out (Too Small)
```
Campaign: Micro campaign
Daily Stats: 3 installs, $15 spend
Result: IGNORE tier - no alert sent
```

## Metrics Monitored

### CPI (Cost Per Install)
- **Direction**: Increase is bad
- **Why**: Higher costs reduce profitability
- **Threshold**: 15-30% depending on tier

### CTR (Click-Through Rate)
- **Direction**: Decrease is bad
- **Why**: Lower engagement indicates creative or targeting issues
- **Threshold**: 15-30% depending on tier

### IPM (Installs Per Mille)
- **Direction**: Decrease is bad
- **Why**: Lower conversion rate indicates quality issues
- **Threshold**: 15-30% depending on tier

## Slack Message Format

### Critical Alerts (Top 3 shown in detail)
```
• Campaign [ID] | Date
  💰 Spend: $X | 📦 Installs: X | Impact: XXX
  Anomaly details...
```

### High Alerts (Top 4 shown)
```
• Campaign [ID] | Date | $X
  Anomaly details...
```

### Medium/Low Alerts (Condensed)
```
• Campaign [ID] | Date: Brief details
```

## Configuration Parameters

Located in `Analyze vs Average and Prepare Slack Alert` node:

```javascript
// Adjust these values to tune sensitivity
const VOLUME_TIERS = {
  CRITICAL: { min_installs: 50, min_spend: 500 },
  HIGH: { min_installs: 25, min_spend: 200 },
  MEDIUM: { min_installs: 10, min_spend: 50 },
  LOW: { min_installs: 5, min_spend: 20 }
};

const DEVIATION_THRESHOLDS = {
  CRITICAL: { cpi: 40, ctr: 40, ipm: 40 },
  SEVERE: { cpi: 30, ctr: 30, ipm: 30 },
  MODERATE: { cpi: 20, ctr: 20, ipm: 20 },
  MINOR: { cpi: 15, ctr: 15, ipm: 15 }
};
```

## Benefits of New System

### 1. **Reduced Alert Fatigue**
- Small campaigns with natural variance don't trigger false alarms
- Only significant issues on important campaigns get CRITICAL status

### 2. **Better Prioritization**
- Impact score helps teams focus on what matters most
- Alerts sorted by business impact, not just metric change

### 3. **Context-Aware Thresholds**
- Large campaigns get early warnings
- Small campaigns only alert on severe issues

### 4. **Transparent Scoring**
- Impact score visible in alerts
- Easy to understand why something is CRITICAL vs MEDIUM

### 5. **Flexible Configuration**
- Thresholds easily adjustable based on business needs
- Tier definitions can be tuned over time

## Monitoring Best Practices

### For CRITICAL Alerts:
- Check immediately (within 1 hour)
- Review campaign settings and recent changes
- Check if issue is systemic or campaign-specific

### For HIGH Alerts:
- Review within same business day
- Investigate root cause
- Consider pausing if issue persists

### For MEDIUM Alerts:
- Review daily during routine check-ins
- Monitor for trends

### For LOW Alerts:
- Optional review
- Good for weekly trend analysis
