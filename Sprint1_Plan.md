# Sprint 1 Plan: Lead Ingestion, Enrichment & Trial Initiation

## Sprint Overview
| Item | Detail |
|------|--------|
| **Duration** | 2 Weeks (10 business days) |
| **Sprint Goal** | Enable automated lead ingestion from all channels, enrich with Hospital Network data, route by territory, and initiate trial equipment requests with ERP handoff. |
| **Target Phases** | Phase 1 (Lead Ingestion & Enrichment) + Phase 2 (Qualification & Device Trial Request) |
| **Team** | 1 Salesforce Admin/Dev, 1 Integration Specialist, 1 QA, 1 Product Owner |

---

## Story Backlog

### Story 1: Lead Capture & Ingestion — 8 pts
**As a** Marketing Manager, **I want** leads from websites, events, and purchased lists automatically created in Salesforce, **so that** no lead is lost.

**Acceptance Criteria:**
- Web-to-Lead form configured with fields: First Name, Last Name, Email, Phone, Company, Title, Hospital Name, Hospital Address, State, Country, Lead Source, Marketing Event.
- Bulk lead import tool/process for purchased lists.
- API endpoint available for event management system integration.
- Duplicate prevention rules (email + hospital name matching).

**Tasks:**
- [ ] Create Lead record type: `Medical_Device_Lead`  Completed
- [ ] Configure Lead fields: `Hospital_Name__c`, `Hospital_Address__c`, `Hospital_Network__c` (lookup), `NPI_Number__c`, `Lead_Source_Detail__c`, `Marketing_Event__c` completed
- [ ] Set up Web-to-Lead with reCAPTCHA working
- [ ] Configure Duplicate Rules and Matching Rules
- [ ] Create list import template and validation rules
- [ ] Build API-connected app for event system (if API available)

---

### Story 2: Hospital Network Enrichment & Matching — 13 pts
**As a** Sales Rep, **I want** leads automatically matched to existing Hospital Networks, **so that** I can see the parent organization and account history.

**Acceptance Criteria:**
- Flow runs on Lead create/update to check for existing Hospital Network.
- Matching logic uses Hospital Name + Address (fuzzy) OR NPI Number (exact).
- If match found: Lead linked to Hospital Network Account via lookup.
- If no match: Lead flagged for manual review or auto-create Hospital Network (configurable).
- Hospital Network data model supports parent-child IDN relationships.

**Tasks:**
- [ ] Create `Hospital_Network__c` custom object
  - Fields: Name, NPI__c, Parent_Network__c (self-lookup), Address, City, State, ZIP, Country, Tier__c, GPO_Affiliation__c
- [ ] Create `Account` record type: `Hospital`
  - Fields: Hospital_Network__c (lookup), Bed_Count__c, Department__c
- [ ] Build `Lead_Enrichment_Flow` (Record-Triggered, Fast Field Updates)
  - Decision: Is existing Hospital Network found?
  - Action: Update Lead.Hospital_Network__c lookup
- [ ] Build `Hospital_Network_Matching_Apex` (if fuzzy matching needed beyond Flow capabilities)
- [ ] Create matching logic matrix document
- [ ] Seed Hospital Network reference data (top 100 IDNs)

---

### Story 3: Territory Routing — 5 pts
**As a** Sales Rep, **I want** leads routed to the correct rep based on territory, **so that** I receive leads in my geographic area.

**Acceptance Criteria:**
- Territory assignment based on State/Region or Hospital Network assignment.
- Round-robin or named rep assignment based on territory hierarchy.
- Email notification sent to assigned rep within 5 minutes.
- Escalation if unassigned after 1 hour.

**Tasks:**
- [ ] Configure Territory Management or custom Territory__c object
- [ ] Build `Lead_Routing_Flow` (Scheduled + Record-Triggered)
  - Input: Lead with Hospital Network or Address
  - Logic: Match Territory → Assign Owner
  - Exception: Queue for manual assignment if no match
- [ ] Create email templates for lead assignment notification
- [ ] Set up escalation rules/time-based workflows
- [ ] Build "Unassigned Leads" dashboard component

---

### Story 4: Lead Qualification & Conversion — 8 pts
**As a** Sales Rep, **I want** to qualify leads and convert them to Accounts, Contacts, and Opportunities, **so that** I can track the sales pipeline.

**Acceptance Criteria:**
- Lead Status values: Open, Contacted, Qualified, Unqualified, Nurture.
- Qualification checklist fields: Budget, Authority, Need, Timeline (BANT).
- Convert button creates Account (Hospital), Contact, Opportunity.
- Opportunity auto-linked to Hospital Network.
- Post-conversion, Lead history retained.

**Tasks:**
- [ ] Configure Lead Status path and validation rules
- [ ] Add qualification fields to Lead: `Budget_Confirmed__c`, `Decision_Maker__c`, `Clinical_Need__c`, `Timeline__c`, `Qualification_Score__c`
- [ ] Customize Lead Conversion layout and mapping
- [ ] Build `Post_Conversion_Flow` to auto-populate Opportunity fields from Lead
- [ ] Create qualification guide (Lightning App Page with rich text/component)
- [ ] Train users on conversion process

---

### Story 5: Trial Equipment Request Initiation — 13 pts
**As a** Sales Rep, **I want** to initiate a trial equipment request from an Opportunity, **so that** the ERP can dispatch a unit to the hospital.

**Acceptance Criteria:**
- "Initiate Trial" button on Opportunity (or auto-flow from stage change).
- Creates `Device_Evaluation__c` record with status "Requested".
- Sends equipment request payload to ERP via Platform Event or Outbound Integration.
- ERP receives: Hospital Address, Contact Info, Device Model, Requested Date, Opportunity ID.
- Opportunity Stage updated to "Trial Requested".

**Tasks:**
- [ ] Create `Device_Evaluation__c` custom object
  - Fields: Opportunity__c (lookup), Account__c (lookup), Contact__c (lookup), Device_Model__c, Status__c (Requested, Approved, Shipped, In Trial, Returned, Converted), Request_Date__c, Ship_Date__c, Tracking_Number__c, ERP_Request_ID__c, Notes__c
- [ ] Create `Device_Model__c` custom object (reference data)
- [ ] Build `Initiate_Trial_Flow` (Screen Flow or Autolaunched)
  - Trigger: Opportunity Stage = "Qualification" + User clicks "Request Trial"
  - Screen: Select Device Model, Confirm Shipping Address, Add Notes
  - Action: Create Device_Evaluation__c, Update Opportunity Stage
- [ ] Build `ERP_Equipment_Request_Integration`
  - Platform Event: `Equipment_Request_Event__e`
  - Apex Trigger/Flow: Publish event with payload
  - (If ERP has API): Build Apex callout class with retry logic
- [ ] Create "My Trial Requests" list view for reps

---

### Story 6: ERP Dispatch Acknowledgment — 5 pts
**As a** System, **I want** to receive ERP dispatch confirmations, **so that** Device Evaluation records are updated with tracking data.

**Acceptance Criteria:**
- ERP sends tracking number and serial number via API/Platform Event.
- Device Evaluation record updated: Status = "Shipped", Tracking_Number__c populated.
- Sales Rep receives email notification.
- Opportunity Stage advances to "Trial Active" (optional, can be manual).

**Tasks:**
- [ ] Build `ERP_Inbound_API` (Apex REST or Platform Event subscriber)
- [ ] Create Platform Event: `ERP_Shipment_Confirmation__e`
- [ ] Build `Process_Shipment_Confirmation_Flow`
  - Match ERP_Request_ID__c to Device_Evaluation__c
  - Update status and tracking fields
- [ ] Configure email alert to rep
- [ ] Build error handling for unmatched ERP requests

---

### Story 7: Sprint 1 Reporting & Dashboard — 5 pts
**As a** Sales Manager, **I want** visibility into lead volume, routing accuracy, and trial requests, **so that** I can measure Sprint 1 success.

**Acceptance Criteria:**
- Dashboard with: Leads by Source, Leads by Territory, Conversion Rate, Trial Requests by Week, Pending Trial Approvals.
- Reports auto-refresh daily.
- Mobile-friendly layout.

**Tasks:**
- [ ] Create reports: Lead Ingestion Report, Territory Routing Report, Trial Pipeline Report
- [ ] Build `Sprint 1 Command Center` dashboard
- [ ] Configure dashboard sharing and scheduling
- [ ] Create data validation report (unmatched hospital networks, unassigned leads)

---

## Data Model (Sprint 1 Scope)

```
Lead
 ├── Hospital_Network__c (lookup → Hospital_Network__c)
 ├── NPI_Number__c
 ├── Budget_Confirmed__c
 ├── Decision_Maker__c
 ├── Clinical_Need__c
 ├── Timeline__c
 └── Standard Lead fields

Hospital_Network__c
 ├── Name
 ├── NPI__c
 ├── Parent_Network__c (self-lookup)
 ├── Address, City, State, ZIP, Country
 ├── Tier__c
 └── GPO_Affiliation__c

Account (Record Type: Hospital)
 ├── Hospital_Network__c (lookup)
 ├── Bed_Count__c
 └── Standard + Custom fields

Opportunity
 ├── Hospital_Network__c (formula/lookup)
 ├── Device_Evaluation__c (related list)
 └── Trial_Status__c

Device_Evaluation__c
 ├── Opportunity__c (lookup)
 ├── Account__c (lookup)
 ├── Contact__c (lookup)
 ├── Device_Model__c (lookup)
 ├── Status__c (Requested, Approved, Shipped, In Trial, Returned, Converted)
 ├── Request_Date__c
 ├── Ship_Date__c
 ├── Tracking_Number__c
 ├── Serial_Number__c
 └── ERP_Request_ID__c

Device_Model__c
 ├── Name
 ├── Model_Number__c
 ├── Category__c
 └── Unit_Cost__c
```

---

## Technical Architecture (Sprint 1)

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Lead Enrichment | Salesforce Flow + Apex | Hospital Network matching |
| Territory Routing | Salesforce Flow + Custom Settings | Rep assignment |
| Trial Initiation | Screen Flow + Apex | User interface + logic |
| ERP Outbound | Platform Events + Apex Callout | Async request to ERP |
| ERP Inbound | Apex REST / Platform Event | Receive shipment data |
| Data Store | Custom Objects + Big Objects (if high volume) | Trial history |

---

## Testing Strategy

**Unit Testing:**
- Apex test classes for matching logic (minimum 90% coverage)
- Flow test debug logs
- Integration callout mock tests

**UAT Scenarios:**
1. Submit web lead → verify routing → verify Hospital Network match
2. Qualify lead → convert → verify Account/Contact/Opp creation
3. Initiate trial from Opportunity → verify Device Evaluation created → verify ERP payload
4. Simulate ERP response → verify tracking update → verify email alert

**Data Migration:**
- Load 500 test Hospital Network records
- Load 1,000 test Leads for volume testing

---

## Definition of Done

- [ ] Code reviewed and merged to UAT sandbox
- [ ] All test classes pass (≥90% coverage)
- [ ] UAT scenarios executed and signed off
- [ ] Documentation updated (Admin guide, User guide)
- [ ] Dashboard published to Sales Managers
- [ ] Rollback plan documented

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| ERP API not ready | High | Use Platform Events as buffer; build mock ERP endpoint |
| Hospital Network data incomplete | Medium | Start with top 100 IDNs; build manual matching UI |
| Territory model undefined | Medium | Use simple State-based assignment v1; iterate in Sprint 2 |
| Lead volume higher than expected | Low | Enable duplicate management; monitor flow limits |

---

## Sprint 1 Success Metrics

| Metric | Target |
|--------|--------|
| Lead routing time | 100% routed within 5 min of creation |
| Hospital Network match rate | ≥80% automated match |
| Trial request loss rate | 0% (all create Device Evaluation record) |
| ERP integration handshake | Successful in test environment |
| Code coverage | ≥90% |

---

## Sprint 2 Preview (Dependencies)

- Regulatory clearance verification (Phase 3 gate)
- GPO membership lookup and discounting
- Quote/CPQ configuration
- Advanced approval workflows
- Margin threshold logic

---

*Document Version: 1.0 | Created: 2026-06-29 | Sprint 1 Kickoff Ready*
