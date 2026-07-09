# ServiceNow Build Plan

_Last updated: July 9, 2026_

The working plan for this project. The README is the public-facing record. This file is how I run the build and track where I am.

## Goal

Stand up a complete IT Service Management (ITSM) slice in a free ServiceNow Personal Developer Instance (PDI), document each piece in the README with screenshots, and publish the repository at github.com/umraffer32. This closes the one honest gap in my California state IT applications, since ServiceNow is weighted heavily in the help desk specialist duty statement (RPA 30622, JC-520758) and I had not used it hands-on before.

Tie-in deadline: JC-520758 final filing date is June 19, 2026. The project does not have to be large. It has to be real, complete, and reproducible.

## Scope

Five areas, each tied to a real duty from the role:

1. Incident and request management.
2. A service catalog item with a Flow Designer workflow.
3. Two or three knowledge base articles.
4. Role-Based Access Control (RBAC): users, groups, roles.
5. A small Configuration Management Database (CMDB) example.

Plus a dashboard to read the incident and request queue, since the duty statement names ServiceNow Dashboards and Reports.

## Build order

I build in the order a real ticket flows, so each piece has something to connect to. Each step is done in the browser inside the PDI, then written up in the matching README section with a screenshot before moving on.

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
- Write two or three articles in plain language: one common fix and one standard procedure (a third if it adds value).
- Categorize and publish them in a knowledge base.
- Link one article to a resolved incident from Step 1 to show how knowledge cuts repeat tickets.

### Step 5. Role-Based Access Control
- Create a few users. Create groups (for example Service Desk and an approver group). Assign roles.
- Show that access follows the role: a fulfiller can work tickets, a requester sees only their own.
- Reuse the groups from Step 3's approval routing so this is not a throwaway example.

### Step 6. CMDB example
- Add a handful of configuration items (CIs), for example a couple of laptops, a server, and a business service.
- Create one or two relationships between them.
- Attach a CI to the incident from Step 1 so the ticket shows the affected asset, which supports root cause analysis.

### Step 7. Dashboard and reporting
- Build a simple dashboard with the active incident and request queue and a breakdown by priority or category.
- This is the ServiceNow Dashboards and Reports duty.

### Step 8. Publish and tie back to the application
- Final pass on the README, confirm every section has steps and a screenshot.
- Create the public repository at github.com/umraffer32 and push.
- Re-tailor the JC-520758 resume and SOQ to reference the project (see the CalCareers progress log next steps), and add ServiceNow to the skills line.

## How each piece gets documented

For every step:
1. Do the work in the PDI.
2. Take a clean screenshot and save it to `images/` with a descriptive name.
3. Fill in the matching README section: what I did, why, the steps to reproduce, and the screenshot.
4. Update the Current Status table in the README from Not started to Done.

## Progress

| Step | Status | Date |
|---|---|---|
| 0. Provision instance | Done | Jul 9 |
| 1. Incident management | In progress | Jul 9 |
| 2. Request management | Not started | |
| 3. Catalog item and flow | Not started | |
| 4. Knowledge base articles | Not started | |
| 5. RBAC | Not started | |
| 6. CMDB | Not started | |
| 7. Dashboard and reporting | Not started | |
| 8. Publish and tie back | Not started | |
