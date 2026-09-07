<div align="center">

<img src="https://img.shields.io/badge/ServiceNow-Scoped%20Application-00c853?style=for-the-badge&logo=servicenow&logoColor=white"/>
<img src="https://img.shields.io/badge/CAD-Certified%20Developer-0066cc?style=for-the-badge&logo=servicenow&logoColor=white"/>
<img src="https://img.shields.io/badge/Source%20Control-Git-f05032?style=for-the-badge&logo=git&logoColor=white"/>

# ServiceNow Loaner Management System

*A fully scoped ServiceNow application built to replace manual, email-based loaner equipment management with a fully automated platform solution — handling the complete request lifecycle from submission through fulfilment to return, across laptops, phones, and projectors.*

</div>

---

## Overview

Loaner equipment requests were previously managed through email and spreadsheets — untracked, unreliable, and expensive to manage. This application replaces that process entirely with a structured, automated ServiceNow solution built on a custom scoped application architecture.

The system manages the full lifecycle of a loaner request: a user submits a request through the Service Catalog or directly via the application, the request is assigned and tracked through states (Requested → Reserved → Ready for Pickup → Checked Out → Returned), automated reminders fire at key points, and fulfillers are notified throughout.

---

## Features

| Capability | Details |
|---|---|
| **Scoped application** | Custom namespace `x_cdltd_loaner_r_0`, isolated roles, portable across instances |
| **Data model** | Two custom tables extending Task, with reference fields to User, CMDB, and Groups |
| **Role-based access** | Three personas (Employee, Requester, Fulfiller) with ACLs and differentiated form views |
| **Flow Designer** | State-transition automation and child Loaner Task creation |
| **Scheduled scripts** | Daily pickup, return, and overdue reminder checks |
| **Script Include** | `LoanerUtils` — reusable utility class centralising all reminder logic |
| **Event system** | Platform events decoupling reminder detection from email delivery |
| **REST integration** | Outbound REST via `RESTMessageV2` retrieving credentials from a password vault |
| **Email notifications** | Dynamic content with Notification Scripts for per-equipment-type logic |
| **Import Sets** | Bulk data migration from spreadsheet with Transform Map |
| **Service Catalog** | Self-service request submission for end users |
| **App Properties** | Lead times and reminder intervals configurable without code changes |
| **ATF** | Automated Test Framework test suite validating core behaviour |
| **Source control** | Full Git commit history maintained throughout development |

---

## Architecture

### Scoped application model

The entire application lives within an isolated namespace, ensuring no interference with other platform applications and making the app fully portable across instances.

### LoanerUtils Script Include

All reminder logic (pickup, return, overdue) is centralised in a single reusable utility class rather than duplicated across multiple scheduled scripts. Each scheduled script calls the utility in three lines — the complexity lives in one maintainable place.

```javascript
var utils = new LoanerUtils();
var ids = utils.getNullPickupReminders();
for (var i = 0; i < ids.length; i++) {
    utils.sendPickupReminder(ids[i]);
}
```

### Event-driven email architecture

Reminders are fired as platform events rather than sending emails directly from scripts. This decouples reminder detection logic from email delivery — email templates can be updated independently of business logic.

### Persona-driven UI

Three distinct form views surface different fields to different user types. Modules in the Application Navigator are filtered per persona using filter conditions.

| Module | Link Type | Filter |
|---|---|---|
| Create New | New Record | — |
| All | List of Records | — |
| Open | List of Records | Active = true |
| Open - Unassigned | List of Records | Active = true AND Assigned to is empty |
| Closed | List of Records | Active = false |

---

## Key Design Decisions

**Extending Task rather than creating a standalone table**
The Loaner Request table extends the platform Task table rather than starting from scratch. This was a deliberate choice. Task already carries fields like number, state, assigned to, short description, and priority, and inheriting them means the application participates in platform-wide behaviours like assignment rules and approval workflows without any additional configuration. Building standalone would have meant recreating that foundation manually.

**Centralising reminder logic in a Script Include**
Pickup reminders, return reminders, and overdue notifications all follow the same pattern: query for qualifying records, check a condition, fire an event. Rather than duplicating that logic across three separate Scheduled Scripts, it lives in a single `LoanerUtils` class. Each scheduled script is three lines. If the reminder interval or query logic ever changes, there is one place to change it.

**Using platform events instead of sending emails directly**
Reminders fire platform events rather than calling email APIs directly from scripts. This decouples the detection logic from the delivery mechanism. The scheduled script does not need to know anything about email templates, recipients, or formatting. Those concerns live in the notification configuration. Swapping or updating an email template requires no script changes.

**Application Properties for configurable behaviour**
Lead times and reminder intervals are stored as Application Properties rather than hardcoded values. An administrator can adjust how far in advance pickup reminders fire without touching a single script. This is the right separation between configuration and code.

**Role-based views over a single form**
Rather than building one form that tries to serve everyone, the application has distinct views per persona. Fulfillers see the full record. Requesters see only what is relevant to them. This reduces cognitive load for end users and prevents accidental edits to fields they should not be touching.

---

## Technical Stack

| Layer | Implementation |
|---|---|
| Platform | ServiceNow (Scoped Application) |
| IDE | ServiceNow Studio |
| Data | Custom tables extending Task, GlideRecord API |
| Automation | Flow Designer, Scheduled Script Executions |
| Scripting | Business Rules, Client Scripts, UI Policies, Script Includes |
| Security | ACLs, Application Access settings, scoped roles |
| Integration | Outbound REST (RESTMessageV2), Import Sets, Transform Maps |
| Testing | Automated Test Framework (ATF) with Client Test Runner |
| Source Control | Git |

---

## Setup

To deploy to a ServiceNow Personal Developer Instance (PDI):

1. Provision a PDI at [developer.servicenow.com](https://developer.servicenow.com)
2. Open ServiceNow Studio
3. Select **Import from Source Control**
4. Enter this repository URL and authenticate with a GitHub Personal Access Token
5. Open the imported Loaner Request application
6. Recreate the **Loaner Request Users** group via User Administration → Groups and assign the `x_cdltd_loaner_r_0.user` role
7. Configure Application Properties with appropriate values for your instance

> **Note:** Hardcoded credentials and instance URLs have been removed from scripts. Update the relevant Business Rule and Email Script with your instance credentials before testing the REST integration.

---

## Built By

**Samuel Ogwu**
Built as part of ServiceNow Certified Application Developer (CAD) certification preparation.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077b5?style=flat&logo=linkedin)](https://www.linkedin.com/in/samuel-ogwu-361655334/)
[![ServiceNow CSA](https://img.shields.io/badge/ServiceNow-CSA%20Certified-00c853?style=flat)](https://www.servicenow.com/now/nav/ui/classic/params/target/cert_record)
[![ServiceNow CAD](https://img.shields.io/badge/ServiceNow-CAD%20Certified-00c853?style=flat)](https://www.servicenow.com/now/nav/ui/classic/params/target/cert_record)
