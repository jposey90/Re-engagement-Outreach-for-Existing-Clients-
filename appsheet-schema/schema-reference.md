# AppSheet Schema Reference

App ID: 617a0b67-5eda-4abe-b2a8-2076a3b56462

## Clients Table (extend existing)

Add these fields if missing:
- ClientID (Auto, PK)
- ClientName (Text)
- WebsiteURL (URL)
- CareersURL (URL)
- LinkedInCompanyURL (URL)
- PrimaryContactName (Text)
- PrimaryContactEmail (Email)
- AccountManagerEmail (Email)
- MonitoringEnabled (Y/N, default TRUE)
- PreferredSenderEmail (Email, default jacob@fcplacements.com)
- Status (Enum: Active / Dormant / Lost)

## DetectedRoles Table (new)

- DetectionID (Auto, PK)
- ClientID (Ref to Clients)
- ClientName (Text)
- RoleTitle (Text)
- Location (Text)
- Source (Enum: Careers Page / ATS / LinkedIn / Board)
- SourceURL (URL)
- JobID (Text)
- NormalizedKey (Text, format: clientid|title|location)
- FirstSeen (Date, default TODAY)
- LastSeen (Date, default TODAY)
- ReviewStatus (Enum: Pending / Approved / Rejected)
- IsNewRole (Y/N, default TRUE)
- RoleStatus (Enum: Open / Filled / On Hold)

## Campaigns Table (new)

- CampaignID (Auto, PK)
- DetectionID (Ref to DetectedRoles)
- ClientID (Ref to Clients)
- ContactEmail (Email)
- SenderEmail (Email, default jacob@fcplacements.com)
- CurrentStage (Enum: Day0 / Day30)
- Day0SentAt (DateTime)
- Day30SentAt (DateTime)
- NextSendDate (Date)
- CampaignStatus (Enum: Active / Complete / Stopped)
- StopReason (Text)
- LastEmailSubject (Text)

## PlacedCandidates Table (new)

- PlacementID (Auto, PK)
- ClientID (Ref to Clients)
- ClientName (Text)
- CandidateName (Text)
- RoleTitle (Text)
- PlacementDate (Date)
- Day30Flag (Y/N, default FALSE)
- Day60Flag (Y/N, default FALSE)
- Day90Flag (Y/N, default FALSE)
- Day180Flag (Y/N, default FALSE)
- Day365Flag (Y/N, default FALSE)
- PlacementStatus (Enum: Active / Terminated / Completed)
- Notes (Text)

## API Patterns

Base URL: https://api.appsheet.com/api/v2/apps/617a0b67-5eda-4abe-b2a8-2076a3b56462

Read: POST /tables/TableName/Action with Action=Find
Add: POST /tables/TableName/Action with Action=Add
Edit: POST /tables/TableName/Action with Action=Edit

All requests need ApplicationAccessKey header.

## Microsoft Graph

Send mail: POST https://graph.microsoft.com/v1.0/users/jacob@fcplacements.com/sendMail
Scope: Mail.Send
Auth: OAuth2

## Deduplication

Key format: ClientID|normalized_title|normalized_location
Example: 42|controller|st-louis-mo
Suppression: 30 days from FirstSeen
