# Tricky Campaign Alerts - n8n Workflow

## Overview
Automated monitoring system for Tricky advertising campaigns that detects anomalies in key performance metrics (CPI, CTR, IPM) and sends alerts to Slack.

## Workflow Description

This n8n workflow monitors campaign performance every hour and alerts the team when negative anomalies are detected.

### Key Features
- **Automated Data Collection**: Fetches campaign metrics every hour from AppGrowth API
- **Smart Anomaly Detection**: Compares current performance against historical averages with volume-aware thresholds
- **Volume-Based Filtering**: Prioritizes alerts based on campaign size and business impact
- **Impact Scoring System**: Each anomaly receives an impact score (0-300+) for intelligent prioritization
- **Multi-Tier Alert System**: 4 severity levels (CRITICAL, HIGH, MEDIUM, LOW) based on impact
- **Alert Management**: Tracks which anomalies have been reported to prevent duplicate alerts
- **Slack Integration**: Sends formatted, prioritized alerts to `#chardonnay_tricky_alerts` channel

### Workflow Nodes

1. **Schedule Trigger - Every Hour**: Initiates the workflow hourly
2. **Fetch Campaign Metrics from API**: Gets last 30 days of campaign data
3. **Parse CSV to JSON**: Converts API response to JSON format
4. **Format Data and Detect Day-to-Day Anomalies**: Formats data and flags anomalies
5. **Save Formatted Data to Google Sheets**: Stores data in tracking spreadsheet
6. **Read All Historical Data from Google Sheets**: Retrieves all historical records
7. **Analyze vs Average and Prepare Slack Alert**: Detects anomalies vs campaign averages
8. **Route by Action Type**: Routes to appropriate action (Slack or Sheet update)
9. **Check if Slack Alert Needed**: Validates alert conditions
10. **Send Alert to Slack Channel**: Sends formatted alert message
11. **Mark Alert as Sent in Google Sheets**: Updates alert status in spreadsheet

### Metrics Monitored
- **CPI** (Cost Per Install): Alert on increase (15-30% threshold based on campaign size)
- **CTR** (Click Through Rate): Alert on decrease (15-30% threshold based on campaign size)
- **IPM** (Installs Per Mille): Alert on decrease (15-30% threshold based on campaign size)

### Volume-Based Campaign Tiers
**Calibrated for Tricky campaign volumes:**

| Tier | Min Installs | Min Spend | Sensitivity |
|------|--------------|-----------|-------------|
| **CRITICAL** | 150 | $80 | High (15% threshold) |
| **HIGH** | 60 | $40 | Standard (20% threshold) |
| **MEDIUM** | 20 | $15 | Standard (20% threshold) |
| **LOW** | 5 | $5 | Reduced (30% threshold) |
| **IGNORE** | <5 | <$5 | No alerts |

### Alert Severity Levels
- **🔴 CRITICAL** (Impact Score ≥150): High-impact issues on large campaigns requiring immediate attention
- **🟠 HIGH** (Impact Score 100-149): Significant issues requiring action within hours
- **🟡 MEDIUM** (Impact Score 60-99): Notable issues to monitor and check within day
- **🔵 LOW** (Impact Score 30-59): Minor issues, FYI only, optional review
- **INFO** (Impact Score <30): Not sent (logged only)

### Impact Score Calculation
Impact scores are calculated based on:
- **Campaign Volume**: Larger campaigns get higher base scores (10-100 points)
- **Deviation Severity**: Larger metric changes add more points (5-50 points per metric)
- **Multiple Metrics**: Bonus for 2+ affected metrics (15-30 points)
- **Absolute Spend**: Extra points for campaigns with >$500-1000 daily spend (10-20 points)

See [ALERT_LOGIC.md](./ALERT_LOGIC.md) for detailed documentation.

## Setup

### Prerequisites
- n8n instance (cloud or self-hosted)
- Google Sheets API credentials
- Slack API credentials
- AppGrowth API access

### Installation
1. Import the workflow JSON file into your n8n instance
2. Configure credentials:
   - HTTP Bearer Auth for AppGrowth API
   - Google Sheets OAuth2
   - Slack API token
3. Update the Google Sheets document ID if needed
4. Activate the workflow

## Configuration

### Google Sheets Structure
The workflow expects a sheet with these columns:
- unique_key
- campaign_id
- date
- impressions
- clicks
- installs
- spend
- cpi
- ctr
- ipm
- anomaly_flag
- anomaly_details
- alert_sent
- alert_sent_at

### Customization

#### Alert Thresholds
Edit the configuration in "Analyze vs Average and Prepare Slack Alert" node:

```javascript
// Volume tier thresholds (calibrated for Tricky campaigns)
const VOLUME_TIERS = {
  CRITICAL: { min_installs: 150, min_spend: 80 },  // Top campaigns
  HIGH: { min_installs: 60, min_spend: 40 },       // Large campaigns
  MEDIUM: { min_installs: 20, min_spend: 15 },     // Medium campaigns
  LOW: { min_installs: 5, min_spend: 5 }           // Small campaigns
};

// Deviation thresholds (%)
const DEVIATION_THRESHOLDS = {
  MINOR: { cpi: 15, ctr: 15, ipm: 15 },      // For CRITICAL tier
  MODERATE: { cpi: 20, ctr: 20, ipm: 20 },   // For HIGH/MEDIUM tier
  SEVERE: { cpi: 30, ctr: 30, ipm: 30 }      // For LOW tier
};
```

#### Other Settings
- Change schedule frequency in "Schedule Trigger - Every Hour" node
- Modify Slack channel in "Send Alert to Slack Channel" node
- Adjust impact score weights in `calculateImpactScore()` function

## Monitoring

View the full tracking data: [Google Sheets Report](https://docs.google.com/spreadsheets/d/1RFC-qsaJiJPIv0lIgAW9ndUXThto3Qf0iiovMMqNl1w)

## License
Internal use only
