# Tricky Campaign Alerts - n8n Workflow

## Overview
Automated monitoring system for Tricky advertising campaigns that detects anomalies in key performance metrics (CPI, CTR, IPM) and sends alerts to Slack.

## Workflow Description

This n8n workflow monitors campaign performance every hour and alerts the team when negative anomalies are detected.

### Key Features
- **Automated Data Collection**: Fetches campaign metrics every hour from AppGrowth API
- **Anomaly Detection**: Compares current performance against historical averages
- **Smart Alerting**: Only sends alerts for significant deviations (>20% from average)
- **Alert Management**: Tracks which anomalies have been reported to prevent duplicate alerts
- **Slack Integration**: Sends formatted alerts to `#chardonnay_tricky_alerts` channel

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
- **CPI** (Cost Per Install): Alert on increase >20% from average
- **CTR** (Click Through Rate): Alert on decrease >20% from average
- **IPM** (Installs Per Mille): Alert on decrease >20% from average

### Alert Severity
- **HIGH** (🔴): 2+ metrics showing anomalies
- **MEDIUM** (🟡): 1 metric showing anomaly

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
- Change schedule frequency in "Schedule Trigger - Every Hour" node
- Adjust anomaly threshold (default 20%) in analysis code
- Modify Slack channel in "Send Alert to Slack Channel" node
- Customize alert message format in analysis node

## Monitoring

View the full tracking data: [Google Sheets Report](https://docs.google.com/spreadsheets/d/1RFC-qsaJiJPIv0lIgAW9ndUXThto3Qf0iiovMMqNl1w)

## License
Internal use only
