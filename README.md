# Leave Request & Approval

A Microsoft Power Platform MVP for submitting employee leave requests, processing sequential **Manager → HR** decisions, tracking leave balances, and synchronizing approved leave with Outlook Calendar.

Built as an interview project using Power Apps, Microsoft Dataverse, Power Automate, Office 365 Outlook, and Power BI.

## Demo and deliverables

**Demo video:** https://drive.google.com/file/d/1B8nhygG616fjBg6FVfOP56JwwJ3BUC9b/view?usp=drivesdk

The recorded demonstration follows one request through submission, manager approval, HR approval, balance updates, and calendar synchronization. Screenshots document the application and workflow results.

| Deliverable | Description |
|---|---|
| Solution ZIP | Unmanaged Power Platform solution containing the app, Dataverse components, flows, and included security configuration |
| Power BI `.pbix` | Report with team/month leave analysis, balances, and configured row-level security |
| Screenshots | Captured project screens and demonstration results |
| Demo video | Screen recording linked above |
| This README | Architecture, setup, business rules, and MVP limitations |

Upload the solution ZIP, PBIX, and screenshots alongside this README, or add download links here if the files are hosted separately. The video is hosted externally.

## Workflow

1. An employee enters a leave type, dates, and reason in the Canvas app.
2. The app creates a draft leave request and a queued Submit action.
3. The scheduled **Leave Action Processor** validates the actor, request details, overlap, and available balance.
4. An accepted submission becomes **Pending Manager**, and its days are reserved.
5. Manager approval advances it to **Pending HR**.
6. HR approval changes it to **Approved**, releases the pending reservation, and adds the days to used leave.
7. The separate **Leave Calendar Sync** flow creates the Outlook event and records synchronization information.

Approval buttons create action records rather than directly setting a request to Approved. The processor checks the expected stage and request version before applying a decision.

## Architecture

| Component | Responsibility |
|---|---|
| Power Apps Canvas app | Request entry, personal requests, approval decisions, balances, and approved-leave calendar |
| Dataverse | Employee mappings, leave balances, requests, and action records |
| Leave Action Processor | Scheduled validation, sequential decisions, request updates, and balance processing |
| Leave Calendar Sync | Scheduled Outlook event creation/deletion and synchronization results |
| Power BI | Daily leave analysis, team/month heatmap, balance reporting, and UPN-based RLS configuration |

The Canvas app contains Home, New Request, My Requests, Approvals, and Calendar screens. Bulk decisions use separate action records for each selected request.

This project uses custom application approvals backed by Dataverse actions. It does not use a separate “Start and wait for an approval” request as the decision source.

## Data model

| Table | Main contents |
|---|---|
| Leave Employees | Name, work email, sign-in UPN, Dataverse user mapping, team, manager, and HR approver |
| Leave Balances | Employee, leave type, leave year, entitlement, used days, pending days, remaining days, and available days |
| Leave Requests | Employee, leave type, dates, requested days, reason, request status, approval stage, version, last applied action, and calendar fields |
| Leave Actions | Request, actor, action type, operation key, expected stage/version, comments, processing result, and notification tracking |

Unique keys are configured for the employee user mapping, employee/type/year balance combination, and action Operation Key.

### Balance definitions

- **Remaining Days = Entitled Days − Used Days**
- **Available Days = Entitled Days − Used Days − Pending Days**

Pending reservations reduce available leave before final approval. Moving a reservation to used leave does not deduct the same days from availability a second time.

## MVP business rules

- Leave types: **Annual, Sick, Others**.
- Working days are Monday–Friday, inclusive of the selected dates; public holidays are not excluded.
- Each request must stay within one calendar year.
- End Date must not precede Start Date, and the request must contain at least one working day.
- Employee overlaps are checked against pending and approved requests.
- Team overlap warnings concern other employees’ approved leave.
- Manager approval must precede HR approval.
- Cancellation is intended for pending or approved leave before its start date.
- Rejection/cancellation processing and bulk decision paths are included in the design; their test coverage is distinguished below from the recorded approval demonstration.

## Setup in another environment

### Prerequisites

- A Power Platform environment with Dataverse and permissions to import solutions.
- Appropriate access to run the Canvas app and Dataverse flows.
- An Office 365 Outlook connection with access to the intended calendar/mailbox.
- Power BI Desktop for the report file.

### 1. Import the solution

1. Download the unmanaged solution ZIP from this repository or its linked location.
2. Open Power Apps and select the intended environment.
3. Open **Solutions → Import solution**, select the ZIP, and follow the import prompts.
4. Map the Microsoft Dataverse and Office 365 Outlook connection references to connections in the destination environment.
5. Publish the imported customizations. Keep the flows off until configuration and sample records are ready.

The solution contains component definitions, not demo employee, balance, request, or action rows. Existing credentials are not supplied; configure your own connections.

### 2. Configure users and sample data

1. Make the intended app users available in the environment.
2. Create Leave Employees records and link each to its actual Dataverse User account.
3. Enter the correct sign-in UPN and work email, then assign manager and HR approver mappings.
4. Create one Leave Balances record per employee, leave type, and year.
5. For a fresh demonstration, use a sample entitlement such as Annual 20, Sick 10, and Others 5, with zero used and pending days and no pre-existing leave history.
6. Assign the included **Leave App User** and **Leave Automation** roles to the appropriate identities, and configure the required record sharing and column-security access. Verify access with non-administrator accounts before relying on it for deployment.

The recorded demo assigns the same account to the employee, manager, and HR roles. This simplifies presentation while preserving the two sequential decisions; it does not prove isolation between different users.

### 3. Configure the app and flows

1. Confirm the app and flows use the intended Dataverse environment.
2. Check the Outlook calendar selection and timezone in Leave Calendar Sync. Re-select a destination calendar if the imported identifier belongs to the original environment.
3. Confirm the action processor’s recurrence concurrency is set to one and no duplicate processor is running.
4. Enable Leave Action Processor and Leave Calendar Sync.
5. Save, publish, and share the Canvas app with the intended users.

The flows are scheduled. Submission, approval transitions, and calendar synchronization may take more than one scheduled interval to appear in the app. Refresh the screens after processing.

### 4. Configure the Power BI report

1. Open the PBIX in Power BI Desktop.
2. Update the Dataverse source to the destination environment and sign in with an authorized account.
3. Update the ReportAccess mapping with the intended ViewerUPN and EmployeeUPN values.
4. Refresh the report after sample data exists.
5. Verify the active Employee relationships filter Requests, LeaveDays, and Balances as intended.
6. Test the LeaveReportAccess role using **View as**. For service deployment, publish to an appropriate workspace, assign role membership, and test with restricted Viewer accounts.

The LeaveDays query expands approved requests into individual weekdays so cross-month leave is allocated to the correct month. Power BI RLS and Dataverse permissions are separate controls.

## Demonstration and validation status

| Area | Status for this submission |
|---|---|
| Leave submission | Demonstrated in the recorded run |
| Manager → HR sequencing | Demonstrated in the recorded run |
| Final approval and balance update | Demonstrated in the recorded run |
| Approved leave and Outlook calendar creation | Demonstrated in the recorded run |
| Power BI visuals and RLS | Configured; separate-user enforcement is not claimed as verified |
| Bulk decisions, rejection, cancellation, and calendar deletion | Not covered by the final recorded approval walkthrough |
| Separate-user Dataverse security | Not validated by the single-account demonstration |
| Failure recovery and concurrency stress tests | Not claimed as comprehensively tested |

## Limitations and future improvements

- This is an interview MVP, not a production-certified leave management system.
- Dataverse writes and Outlook operations are not one atomic transaction. A successful flow run can also mean there was no queued work; request/action results must be checked to confirm the business outcome.
- Operation keys and stage/version checks reduce duplicate or stale processing, but do not provide an end-to-end exactly-once guarantee.
- Incomplete processing may require manual review. Do not reset failed actions or balances without checking whether earlier updates were already applied.
- Stronger production processing would use transactional Dataverse operations, durable recovery, monitoring, and broader automated testing.
- Holiday calendars, half-day leave, cross-year requests, and AI Builder classification are outside this MVP.

## Submission notes

The video and screenshots provide the working demonstration. Reviewers can inspect the unmanaged solution and PBIX and follow the setup instructions to configure their own environment. No public live-app access or credentials are included.
