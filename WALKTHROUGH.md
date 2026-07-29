# Build Walkthrough

_Last updated: July 9, 2026_

This is the actual worklog. The [README](README.md) says what the project is and the [build plan](BUILD_PLAN.md) lays out the steps. This file is where I record what I actually did, in order, with screenshots, so the whole thing is reproducible from scratch.

I am building in a free ServiceNow Personal Developer Instance (PDI). Each step below is the real work as it happened, including the parts that did not go smoothly.

## Step 0. Set up the developer instance

Before any IT Service Management (ITSM) work, I needed a working instance to build in.

### Create a ServiceNow ID

I went to developer.servicenow.com and created a free ServiceNow ID. The sign-up sends a six digit code to your email to confirm the account.

![ServiceNow ID sign-up](images/setup-servicenow-id.png)

### Get to the right place

One thing worth flagging for anyone following along. After signing in, I first landed on the corporate MyNow site, which is not where you build. It is a product portal with a promo video and nothing to do with a developer instance. The Personal Developer Instance lives on developer.servicenow.com, so if you end up on MyNow, go back to the developer site.

![MyNow is not the developer portal](images/setup-mynow-detour.png)

### Finish the developer onboarding

The developer portal runs a short two question setup. I said yes to coding since I want the developer tools, and picked IT Admin as the closest role. Neither answer changes the instance, they just tune the portal.

![Developer onboarding, do you code](images/setup-onboarding-code.png)

![Developer onboarding, role](images/setup-onboarding-role.png)

### Request the instance

From the developer home page I clicked Request Instance.

![Developer home](images/setup-developer-home.png)

It asked which release to build on. Australia was the latest release and the only one with instances available, so I chose it. The other two releases showed no instances available.

![Choose a release](images/setup-request-instance.png)

The request kicked off and the instance went into provisioning.

![Provisioning the instance](images/setup-provisioning.png)

### Reality check: the free instance pool was full

The provisioning did not finish. The first request failed, the second request failed, and on the third try the only option offered was to join a waitlist. This is the free tier behaving as designed when demand is high. The pool of available instances runs dry and new requests queue until one frees up.

I joined the waitlist. ServiceNow emails you when an instance is allocated. Nothing here is broken or misconfigured on my end, it is a capacity limit on the free program, so the move is to wait for the allocation and pick the build back up the moment it lands.

**Status: waiting on instance allocation as of June 11, 2026.** Everything below is staged and ready to build the moment the instance is live.

**Update, July 9, 2026: the instance came off the waitlist.** Logged in as admin on the Australia release with no issues. Picking the build back up at Step 1.

## Step 1. Incident management

Goal: log an incident with a caller, category, and priority from impact and urgency, work it through its states with work notes, resolve it, and build a filtered list view of active incidents.

### Finding the incident form

I typed "Incident" into the All menu and landed on an Incident list filtered to Self Service, with one row already in it: INC0008111, "ATF: Test1". That's ServiceNow's own out-of-box demo data, not mine.

![Incident list, Self Service view](images/incident-list-self-service-view.png)

Clicking New from here gave me a stripped-down form: Number, Caller, Watch list, Urgency, State, Short description, Additional comments. Nothing for Category, Impact, Priority, or Assignment group.

![New incident, Self Service form](images/incident-new-self-service-form.png)

That's the Self Service view, the simplified layout meant for an end user submitting their own ticket from the portal. It's not what an agent works from. I right-clicked the list header and found a View submenu with the actual list of views the instance has configured: Default view, Indicators Panel, Major incidents, Mobile, Portal, Self Service (the one I was on), Service Operations Workspace, and a few report-specific views.

![Switching the list view](images/incident-list-view-switch-menu.png)

Switching to Default view reloaded the list with the columns an agent actually needs: Priority, State, Category, Assignment group, Assigned to.

![Incident list, Default view](images/incident-list-default-view.png)

### Logging the incident

From Default view, New opens the full form: Caller, Category, Subcategory, Service, Assignment group, Impact, Urgency, Priority, State, plus Notes and Resolution Information tabs further down.

![Blank full incident form](images/incident-new-full-form-blank.png)

I logged this one against a real problem I'd already written up as a knowledge base article, so the incident and the KB article tie together later in Step 4. Caller stayed System Administrator. Category: Network. Short description: "Linux workstation cannot reach internal hosts behind Tailscale subnet router."

![Category and short description set](images/incident-category-short-description.png)

For Impact and Urgency, only one workstation was affected, not a site or a team, so Impact went to 3 - Low. It was blocking the user's work but nothing was down, so Urgency went to 2 - Medium. Priority is a read-only field ServiceNow derives from the Impact x Urgency matrix, and it calculated to 4 - Low.

![Impact, Urgency, and calculated Priority](images/incident-impact-urgency-priority.png)

Assignment group went to Network, the group that would actually own a routing problem like this.

![Assignment group set to Network](images/incident-assignment-group.png)

I added a description with the actual symptom before submitting: the user could reach the office LAN over Tailscale but not hosts on the remote subnet routed through it, and direct tailnet peers responded fine while only the subnet-routed hosts failed.

![Description added, ready to submit](images/incident-description-ready-to-submit.png)

Submitting created INC0010003 with State New, Priority 4 - Low, Category Network, and Assignment group Network, sitting in the list next to the untouched demo record.

![INC0010003 submitted](images/incident-submitted-list.png)

### Working the incident through its states

Typing "System Administrator" into Assigned to came back invalid. The picker offered five names instead, Bow Ruggeri, David Dan, David Loo, Fred Luddy, and ITIL User, all out-of-box demo accounts that ship with the instance.

My first read on that was wrong, and I'm leaving the correction in rather than quietly fixing the sentence. I assumed the field was filtering to users who hold an agent role. It isn't. It filters to members of the assignment group, which I'd set to Network a moment earlier. Those five names are the Network group's membership.

![Network group members](images/incident-network-group-members.png)

I only confirmed that in Step 2, where the same field came back completely empty on a catalog task and forced me to go look at the group record. Same qualifier, same rule, and the group was empty. Full account is in [why the Assigned to field kept coming back empty](#why-the-assigned-to-field-kept-coming-back-empty).

So System Administrator wasn't rejected for lacking a role. It just isn't in the Network group.

I assigned the incident to ITIL User, the clearest of the five for a portfolio piece since it reads as an obvious demo account rather than a real name. Once RBAC is built in Step 5 with actual named users, I can come back and reassign this ticket to one of those instead.

With Assigned to set, I moved State to In Progress and logged a work note with the real diagnosis:

> tailscale status showed the client connected to the tailnet, but pings to the advertised subnet timed out. Confirmed with the user that this is a Linux client, not Windows. Cause: Tailscale does not accept advertised subnet routes on Linux by default (Windows/macOS/iOS/Android do). Ran `sudo tailscale set --accept-routes` on the client. Verified with `ip route` that the subnet now routes through tailscale0. User confirmed the remote hosts are reachable.

![Incident In Progress with work notes](images/incident-in-progress-work-notes.png)

The list view now shows INC0010003 as In Progress, Priority 4 - Low, assigned to the Network group and ITIL User.

![Incident list showing In Progress state](images/incident-list-in-progress.png)

### Resolving and closing the incident

On the Resolution Information tab, the Resolution code defaulted to "Resolved by request," which didn't fit, nothing was requested here, a technical cause was found and fixed. The dropdown offers Duplicate, Known error, No resolution provided, Resolved by caller, Resolved by change, Resolved by problem, Resolved by request, Solution provided, Workaround provided, and User error.

![Resolution code options](images/incident-resolution-code-options.png)

I picked Solution provided, since the assigned agent diagnosed the cause and applied a permanent fix rather than a workaround. Resolution notes:

> Enabled route acceptance on the Linux client with `tailscale set --accept-routes`. Confirmed the advertised subnet routes through tailscale0 and the user can reach the remote hosts.

Resolved by and Resolved are read-only until you act. They stamp themselves the moment you click Resolve.

![Resolution notes set to Solution provided](images/incident-resolved-solution-provided.png)

Clicking Resolve confirmed with a banner and dropped the incident to State Resolved in the list.

![Incident resolved](images/incident-resolved-confirmation.png)

From there I set State to Closed and used the Close Incident action, not a plain Update, the same way Resolve was its own action rather than a manual field save. That stamped it as permanently closed and it fell out of the Active = true filtered list entirely, since Closed incidents aren't active.

![Incident closed and gone from active filter](images/incident-closed-confirmation.png)

### Building a filtered list of active incidents

Dropping the Caller condition on the default list exposed the full set of ServiceNow's built-in demo incidents, well over a dozen, on top of my own. I set a single condition, Active is true, and confirmed INC0010003 (closed) correctly dropped out of the results while INC0008111 (still New) stayed in. I added a sort on Priority, ascending, so 1 - Critical rows sort to the top instead of sitting wherever they land in the raw data, the order a real service desk queue would use.

![Active filter with Priority sort configured](images/incident-active-filter-builder.png)

I saved it as a named list view, "Active Incidents," visible to me, so it is a reusable list rather than a one-off query.

![Active Incidents, sorted by Priority](images/incident-active-filtered-list.png)

That closes out Step 1: an incident logged with a real caller, category, and description; worked through New, In Progress, Resolved, and Closed with work notes and resolution notes at each step; and a saved, prioritized view of active incidents.

## Step 2. Request management

Goal: submit a service request and show how it differs from an incident, through the request, the requested item, and the catalog task.

An incident is something broken. A request is something wanted. ServiceNow keeps them on separate tables with different fields and a different lifecycle, and the point of this step is to walk one request end to end so the difference is visible rather than asserted.

### Ordering from the service catalog

The Service Catalog is the end user's front door. It's organized by category (Hardware, Software, Desktops, Mobiles, Peripherals, Office, Services) with a Top Requests panel on the right, and it reads like a storefront, not a ticket form.

![Service Catalog home](images/request-service-catalog-home.png)

I ordered a Standard Laptop, a Lenovo Carbon x1 at $1,100 with a five day delivery estimate. The item has its own variables rather than the generic fields an incident has. I checked Adobe Acrobat under Optional Software and filled in the free text box with "VPN client and standard office suite per department image."

![Standard Laptop order form](images/request-standard-laptop-form.png)

Order Now submitted it and returned an order status page with REQ0010001, an estimated delivery date of 2026-08-03, and a stage tracker showing eight steps with the first one lit. Nothing about this screen resembles an incident. There's no priority, no category, no assignment group, just a price, a delivery date, and a progress bar.

![Order confirmation for REQ0010001](images/request-order-confirmation.png)

### Request, Requested Item, and Catalog Task

One catalog order creates three linked records, and understanding why is most of this step.

**REQ0010001** is the Request, the shopping cart. It tracks Requested for, Price, Due date, Approval, and Request state, which came in as Pending Approval. It has no Category, Impact, Urgency, Priority, or Assignment group. Its Requested Items related list holds the actual thing ordered.

![Request record REQ0010001](images/request-req-record.png)

**RITM0010001** is the Requested Item, the line item. This is where the laptop itself lives, along with the variables I filled in on the catalog form and a read-only Stage field that showed Waiting for Approval. Worth noting from the Activities log at the bottom, this record does carry Impact 3 - Low and Priority 4 - Low under the hood. Requested Items extend the same base Task table an Incident does, so the plumbing is shared. The form just surfaces fulfillment fields instead of triage fields, because that's what the record is for.

![Requested Item RITM0010001](images/request-ritm-record.png)

Approval turned out to be two layers deep. Approving at the Request level moved Request state to Approved but the item's Stage advanced only as far as Dept. Head Approval, which is read-only and derived from the workflow, the same way Priority was calculated rather than typed on the incident. The Requested Item carried its own Approvers list with two entries, Natasha Ingram already approved and Bow Ruggeri still Requested.

![Two approvers on the Requested Item](images/request-ritm-approvers.png)

Approving the second one released the workflow, and a Catalog Task appeared.

**SCTASK0010001** is the Catalog Task, the actual work order. Short description "Please fulfill this order," with instructions to pull from stock or order from the vendor and flag it Backordered with an estimated delivery date if it has to be ordered. This is the record a fulfiller works, and it has the fields you'd expect for that, Assigned to, State, Priority, and Work notes.

![Catalog Task SCTASK0010001](images/request-catalog-task.png)

So the chain runs Request (what was ordered and approved), Requested Item (which item, with the options chosen), and Catalog Task (who does the work). An incident collapses all three into one record because there's nothing to approve and nothing to fulfill, there's just something broken that needs fixing.

### Why the Assigned to field kept coming back empty

This is the part that took the longest, and the reason is worth writing down.

Trying to assign SCTASK0010001, I typed a name into Assigned to and got a red invalid field. Opening the picker instead gave me an empty list. Not a short list, an empty one, "No records to display" on a table that plainly has users in it.

The field is a reference to the user table with a qualifier on it, and on a catalog task that qualifier limits the choices to members of the task's assignment group. This task had no assignment group. The form didn't even have the field on it, which is why there was nowhere obvious to set one.

I added it through the form layout editor, reached from the hamburger menu next to the record title, then Configure and Form Layout. Assignment group moved from Available to Selected, positioned just above Assigned to since the group gets filled in first.

Worth knowing before you do this. Editing the Default view changes the form for every catalog task in the instance, not just the one you're looking at. That's usually what you want, but it isn't a local change.

With Assignment group set to Hardware, the picker offered ITIL User and the assignment went through. Work notes recorded the fulfillment.

![Catalog task assigned to Hardware and ITIL User](images/request-catalog-task-assigned.png)

Close Task set State to Closed Complete, cleared the Active flag, and stamped the activity log.

![First catalog task closed](images/request-catalog-task-closed.png)

### A second task, and the same problem for a different reason

Closing the first task immediately created a second one. SCTASK0010002, "Please deploy item to the user," assigned to the Field Services group, opened at 09:35:22, the same second the first one closed. The workflow splits fulfillment into procure and deploy, so the Requested Item wasn't going anywhere until both were done.

This one already had an assignment group, so by my own explanation the picker should have worked. It came back empty again.

Rather than guess a second time, I opened the group itself. Field Services had two roles and zero members.

![Field Services group with no members](images/request-field-services-empty.png)

That's the real rule. The qualifier filters to group members, so an empty group produces an empty list whether or not a group is set. Hardware worked because ITIL User happened to belong to it. My first explanation was right about the mechanism and wrong about which condition had failed.

I added ITIL User to Field Services through the Edit picker on Group Members. The save queues a background job to propagate the group's roles to the user, so the related list still read empty until I reloaded the record.

![ITIL User added to Field Services](images/request-field-services-member-added.png)

Back on SCTASK0010002 the picker worked with no form changes needed, since the layout edit had already applied instance-wide.

### The chain closing itself

Closing the deploy task finished the sequence. Both catalog tasks show an Actual end, both assigned to ITIL User, and the Requested Item flipped to Closed Complete at 09:50:56, the moment the last task closed.

![Both catalog tasks complete](images/request-ritm-tasks-complete.png)

RITM0010001 Stage reads Completed and State reads Closed Complete.

![Requested Item completed](images/request-ritm-completed.png)

REQ0010001 followed with Approval Approved and Request state Closed Complete, and the stage tracker on the requested item filled in green.

![Request closed complete](images/request-req-closed.png)

Nothing in that last sequence was a manual state change. I closed one task and the item and the request closed themselves, because the workflow owns the parent states and the tasks own the work. That's the structural difference from an incident, where I set every state by hand. A request is a workflow with records attached. An incident is a record with a state field.

## Step 3. Service catalog item with a Flow Designer workflow

_Pending the instance. Goal: build a catalog item the user submits from the portal, then wire it to a Flow Designer flow that routes an approval and creates a fulfillment task automatically._

## Step 4. Knowledge base articles

_Pending the instance to publish. Goal: write two or three knowledge articles for a common fix and a standard procedure, categorize and publish them, and link one to a resolved incident._

Drafted and ready to paste into the knowledge base:

1. [Linux client cannot reach hosts behind a Tailscale subnet router](knowledge-base/kb01-linux-tailscale-subnet-routes.md) (troubleshooting, common fix)
2. [Configure SSH for key-based authentication and disable password login](knowledge-base/kb02-ssh-key-based-authentication.md) (how-to, hardening procedure)

## Step 5. Role-Based Access Control

_Pending the instance. Goal: create users, put them in groups, assign roles, and show that access follows the role._

## Step 6. CMDB example

_Pending the instance. Goal: add a few configuration items, give them a relationship or two, and attach one to an incident so the ticket shows the affected asset._

## Step 7. Dashboard and reporting

_Pending the instance. Goal: build a dashboard with the active incident and request queue and a breakdown by priority or category._
