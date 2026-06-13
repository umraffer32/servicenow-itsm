# Build Walkthrough

_Last updated: June 11, 2026_

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

## Step 1. Incident management

_Pending the instance. Goal: log an incident with a caller, category, and priority from impact and urgency, work it through its states with work notes, resolve it, and build a filtered list view of active incidents._

## Step 2. Request management

_Pending the instance. Goal: submit a service request and show how it differs from an incident, through the request, the requested item, and the catalog task._

## Step 3. Service catalog item with a Flow Designer workflow

_Pending the instance. Goal: build a catalog item the user submits from the portal, then wire it to a Flow Designer flow that routes an approval and creates a fulfillment task automatically._

## Step 4. Knowledge base articles

_Pending the instance to publish. Goal: write two or three knowledge articles for a common fix and a standard procedure, categorize and publish them, and link one to a resolved incident._

Drafted and ready to paste into the knowledge base:

1. [Linux client cannot reach hosts behind a Tailscale subnet router](knowledge-base/kb01-linux-tailscale-subnet-routes.md) (troubleshooting, common fix)

## Step 5. Role-Based Access Control

_Pending the instance. Goal: create users, put them in groups, assign roles, and show that access follows the role._

## Step 6. CMDB example

_Pending the instance. Goal: add a few configuration items, give them a relationship or two, and attach one to an incident so the ticket shows the affected asset._

## Step 7. Dashboard and reporting

_Pending the instance. Goal: build a dashboard with the active incident and request queue and a breakdown by priority or category._
