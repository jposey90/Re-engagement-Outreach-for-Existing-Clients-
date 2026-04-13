# Re-engagement Outreach for Existing Clients

## Overview
Automated system to monitor existing client job postings and trigger re-engagement email campaigns. Includes placement anniversary check-ins sent to jacob@fcplacements.com.

## System Architecture

### Stack
- **AppSheet** - Source of truth for clients, placements, detected roles, and campaigns
- **n8n** - Workflow orchestration (monitoring, deduplication, email triggers)
- **Outlook 365 / Microsoft Graph** - Email sending from jacob@fcplacements.com

### Core Workflows

| Workflow | Purpose | Schedule |
|---|---|---|
| Client Monitor | Scrape careers pages and detect new roles | Daily 7:00 AM |
| Campaign Creator | Create campaign when new role detected | On new detection |
| Follow-up Runner | Send Day 30 follow-up emails | Daily 8:00 AM |
| Anniversary Check-in | Send placement anniversary updates to Jacob | Daily 9:00 AM |

## MVP Scope

### Phase 1 (Current)
- Monitor client careers pages and ATS pages
- Detect new roles via domain matching
- Day 0 + Day 30 email sequence
- Placement anniversary alerts (30 / 60 / 90 / 180 / 365 day)
- Manual review queue for low-confidence detections

### Phase 2
- LinkedIn company jobs cross-check
- Day 60 + Day 90 emails
- Confidence scoring
- Reporting dashboard in AppSheet

### Phase 3
- Job board monitoring
- Multi-sender routing by account manager
- Auto-assignment logic

## AppSheet Data Model

### Clients Table
| Field | Type | Notes |
|---|---|---|
| ClientID | Auto | Primary key |
| ClientName | Text | Company name |
| WebsiteURL | URL | Client website |
| CareersURL | URL | Careers/jobs page |
| PrimaryContactEmail | Email | Main hiring contact |
| PrimaryContactName | Text | Contact name |
| AccountManagerEmail | Email | Assigned AM |
| MonitoringEnabled | Y/N | Include in monitoring |
| PreferredSenderEmail | Email | Default: jacob@fcplacements.com |
| Status | Enum | Active / Dormant / Lost |

### DetectedRoles Table
| Field | Type | Notes |
|---|---|---|
| DetectionID | Auto | Primary key |
| ClientID | Ref | Link to Clients |
| RoleTitle | Text | Detected job title |
| Location | Text | Job location |
| Source | Enum | Careers Page / ATS / LinkedIn / Board |
| SourceURL | URL | Direct job link |
| JobID | Text | Requisition ID if available |
| NormalizedKey | Text | clientid-title-location hash |
| FirstSeen | Date | First detection date |
| LastSeen | Date | Most recent detection |
| ReviewStatus | Enum | Pending / Approved / Rejected |
| IsNewRole | Y/N | Net-new flag |
| RoleStatus | Enum | Open / Filled / On Hold |

### Campaigns Table
| Field | Type | Notes |
|---|---|---|
| CampaignID | Auto | Primary key |
| DetectionID | Ref | Link to DetectedRoles |
| ClientID | Ref | Link to Clients |
| ContactEmail | Email | Recipient |
| SenderEmail | Email | Sending mailbox |
| CurrentStage | Enum | Day0 / Day30 |
| Day0SentAt | DateTime | Timestamp |
| Day30SentAt | DateTime | Timestamp |
| NextSendDate | Date | Next scheduled send |
| CampaignStatus | Enum | Active / Complete / Stopped |
| StopReason | Text | Reply / Filled / On Hold / Opt-out |

### PlacedCandidates Table
| Field | Type | Notes |
|---|---|---|
| PlacementID | Auto | Primary key |
| ClientID | Ref | Link to Clients |
| ClientName | Text | Company name |
| CandidateName | Text | Placed candidate |
| RoleTitle | Text | Role they were placed in |
| PlacementDate | Date | Start date |
| Day30Flag | Y/N | 30-day check-in sent |
| Day60Flag | Y/N | 60-day check-in sent |
| Day90Flag | Y/N | 90-day check-in sent |
| Day180Flag | Y/N | 180-day check-in sent |
| Day365Flag | Y/N | 1-year anniversary sent |
| PlacementStatus | Enum | Active / Terminated / Completed |
| Notes | Text | Any notes |

## Placement Anniversary System

Sends internal update emails to jacob@fcplacements.com at key milestones after a candidate is placed.

### Milestones
| Day | Purpose |
|---|---|
| 30 | Early check-in - is the placement going well? |
| 60 | Mid-probation check - any concerns? |
| 90 | End of probation - confirm retention |
| 180 | 6-month mark - relationship strengthening |
| 365 | 1-year anniversary - celebrate and re-engage |

### How It Works
1. n8n runs daily at 9:00 AM
2. Reads PlacedCandidates from AppSheet
3. Calculates days since PlacementDate
4. If a milestone is hit and the flag is FALSE, sends email to jacob@fcplacements.com
5. Updates the flag to TRUE in AppSheet

## Email Templates

See `/email-templates/` directory:
- `day0-new-role.html` - Initial outreach when new role detected
- `day30-followup.html` - 30-day follow-up
- `anniversary-30.html` - 30-day placement check-in
- `anniversary-60.html` - 60-day placement check-in
- `anniversary-90.html` - 90-day placement check-in
- `anniversary-180.html` - 180-day placement check-in
- `anniversary-365.html` - 1-year anniversary

## n8n Workflows

See `/n8n-workflows/` directory:
- `client-monitor.json` - Daily careers page monitoring
- `campaign-creator.json` - New role to campaign trigger
- `followup-runner.json` - Scheduled follow-up sender
- `anniversary-checker.json` - Placement anniversary monitoring

## Setup

### Prerequisites
- AppSheet app with tables above
- n8n instance
- Microsoft 365 account (jacob@fcplacements.com)
- Microsoft Graph API credentials (Mail.Send scope)
- AppSheet API key

### Configuration
1. Import AppSheet tables from `/appsheet-schema/`
2. Import n8n workflows from `/n8n-workflows/`
3. Set credentials in n8n:
   - AppSheet API key
   - Microsoft Graph OAuth2 (Mail.Send)
4. Update App ID in workflow nodes: `617a0b67-5eda-4abe-b2a8-2076a3b56462`
5. Enable monitoring on target clients in AppSheet
6. Activate workflows in n8n

## Matching Rules (MVP)
- Primary: exact domain match
- Secondary: normalized company name
- Dedupe key: `clientid|normalized_title|normalized_location`
- Suppression window: 30 days
- Low-confidence matches go to review queue

## Stop Conditions
- Contact replies
- Role status changes to Filled / On Hold
- Client status changes to Lost
- Manual stop via AppSheet
- Placement terminated (for anniversary sequence)
