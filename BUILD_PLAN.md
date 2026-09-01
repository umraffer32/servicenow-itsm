# ServiceNow Build Plan

_Last updated: September 1, 2026_

The working plan for this project. The README is the public-facing summary and WALKTHROUGH.md is the full worklog. This file is how I run the build and track where I am.

## Goal

Stand up a complete IT Service Management (ITSM) slice in a free ServiceNow Personal Developer Instance (PDI) and document each piece with screenshots, the full worklog in WALKTHROUGH.md and a summary per area in the README. ServiceNow is the system of record for IT support at most large organizations, and I hadn't used it hands-on before.

The project does not have to be large. It has to be real, complete, and reproducible.

## Scope

Five areas, each a standard piece of help desk work:

1. Incident and request management.
2. A service catalog item with a Flow Designer workflow.
3. Two knowledge base articles.
4. Role-Based Access Control (RBAC): users, groups, roles.
5. A small Configuration Management Database (CMDB) example.

Plus a dashboard to read the incident and request queue.

## Build order

I build in the order a real ticket flows, so each piece has something to connect to. Each step is done in the browser inside the PDI, then written up in WALKTHROUGH.md and summarized in the matching README section with a screenshot before moving on.

### Step 0. Provision the instance
- Sign up at developer.servicenow.com and request a Personal Developer Instance.
- Note the release version for the README. Do not commit the instance URL, username, or password.
- Confirm I can log in as admin.

### Step 1. Incident management
- Log an incident as if a user reported it. Set caller, category, short description, priority, and assignment group.
- Work it through its states (New, In Progress, Resolved, Closed) and add work notes.
- Build a filtered list view of active incidents.
- Document the fields, the priority logic (impact and urgency), and the lifecycle.

### Step 2. Request management
- Submit a basic service request and show how it differs from an incident.
- Show the request, the requested item, and the catalog task relationship.
- This sets up Step 3.

### Step 3. Service catalog item with a Flow Designer workflow
- Build a catalog item the user submits from the portal, for example a laptop or software access request, with a few variables on the form.
- Build a Flow Designer flow triggered by the request: an approval step routed to a manager or group, then a fulfillment task created on approval.
- Test it end to end as a requester, then as the approver, then as the fulfiller.
- Document the form, the flow diagram, and the run.

### Step 4. Knowledge base articles
- Publish the two articles already drafted: one common fix and one standard procedure.
- Categorize and publish them in a knowledge base.
- Link one article to an incident to show how knowledge cuts repeat tickets.

**Log a second incident first.** INC0010003 from Step 1 is Closed, and closing an incident locks most of its fields. Configuration item is one of the locked ones, so Step 6 can't use it either. Log a fresh incident at the start of Step 4 and work it only as far as In Progress. Step 4 links the article to it and Step 6 attaches the configuration item to it. Do not resolve or close it until both steps are done, then close it last as the finale.

### Step 5. Role-Based Access Control
- Create a few users. Use existing groups rather than creating new ones. Assign roles.
- Show that access follows the role: a fulfiller can work tickets, a requester sees only their own.
- Reuse the groups from Step 3's approval routing so this is not a throwaway example.

### Step 6. CMDB example
- Add a handful of configuration items (CIs), for example a couple of laptops, a server, and a business service.
- Create one or two relationships between them.
- Attach a CI to the still-open incident logged in Step 4 so the ticket shows the affected asset, which supports root cause analysis. Not INC0010003, which is closed and locked.

**No new CIs were added.** The instance ships with roughly 2,800 seeded configuration items, so inventing a handful of new ones would have meant inventing data. I reused `PS LinuxApp01`, an existing Linux server CI whose name already matched the incident's own description, along with the infrastructure relationships that came attached to it.

### Step 7. Dashboard and reporting
- Build a simple dashboard with the active incident and request queue and a breakdown by priority or category.
- This is the reporting side of reading a service desk queue.

### Step 8. Final pass
- Consistency pass across the README, this plan, and WALKTHROUGH.md. The repository already lives at github.com/umraffer32/servicenow-itsm with the full history pushed.
- Confirm every section has steps and a screenshot.
- The README status badge now reads Complete.

## How each piece gets documented

For every step:
1. Do the work in the PDI.
2. Take a clean screenshot and save it to `images/` with a descriptive name.
3. Write the full worklog section in WALKTHROUGH.md: what I did, why, the wrong turns, and the screenshots.
4. Summarize it in the matching README section with one screenshot.
5. Update the Current Status table in the README and the Progress table here.

## Progress

| Step | Status | Date |
|---|---|---|
| 0. Provision instance | Done | Jul 9 |
| 1. Incident management | Done | Jul 9 |
| 2. Request management | Done | Jul 29 |
| 3. Catalog item and flow | Done | Jul 29 |
| 4. Knowledge base articles | Done | Aug 19 |
| 5. RBAC | Done | Aug 31 |
| 6. CMDB | Done | Aug 31 |
| 7. Dashboard and reporting | Done | Sep 1 |
| 8. Final pass | Done | Sep 1 |
