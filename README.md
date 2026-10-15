# Automated Car Rental Management System in ServiceNow

An enterprise vehicle reservation and fleet management solution built natively on the ServiceNow platform. The application streamlines car requests, manager approvals, transport fulfillment, notifications, exception handling, and fleet-demand reporting.

## Project Overview

| Item | Details |
|---|---|
| Developer | Segabandi Sasidhar |
| Platform | ServiceNow Personal Developer Instance |
| Instance | `dev428737` |
| Application scope | `x_1894596_car_re_0` |
| Custom table | `x_1894596_car_re_0_car_rental_request` |
| Auto-number prefix | `CRR` |
| Workflow | Workflow Studio / Flow Designer — `Car Rental Req` |
| Deployment artifact | `Car Rental Management.xml` |
| Update Set state | Complete |

> **Demo video:** [Watch the Project Demo Video on Google Drive](https://drive.google.com/file/d/1QrXKw4F466TesOmQvm0rlMnQKAzjmPIy/view?usp=sharing)

## Key Features

### 1. Self-Service Portal Intake
- Users submit a **Car Rental Request** catalog item through the Service Portal (`/sp`).
- The form captures six required fields:
  - Requested date
  - Pickup location
  - Drop location
  - Duration (1–10 days)
  - Car type (Sedan, SUV, or Mini)
  - Business reason

### 2. Client-Side Validation
- An `onChange` Catalog Client Script validates the requested date.
- Past departure dates are rejected and the field is cleared after an error message is displayed.
- Booking duration is intended to be restricted to valid numeric values from 1 to 10 days.

### 3. Approval and Fulfillment Automation
- The Flow Designer workflow, **Car Rental Req**, starts from the catalog request process (`sc_req_item`).
- Approval is routed to the requester's line manager through the ServiceNow approval mechanism (`sysapproval_approver`).
- After approval, a Catalog Task is created and assigned to the Transport Fulfillment team.
- A system email notification is sent with the subject **“Your Car Request is approved”**.

### 4. Exception Handling
- An **After Business Rule** monitors state changes on the custom car rental request table.
- When a request enters state `3` (documented by this project as rejected/unavailable), an Incident record is automatically created.
- The generated incident includes the car request number and requested date in its description.

### 5. Fleet Analytics Dashboard
- **Donut chart:** Shows request distribution across operational states, including Submitted, Approved, Vehicle Assigned, and Rejected.
- **Bar chart:** Compares demand across Sedan, SUV, and Mini vehicle categories.

## Architecture at a Glance

1. The user opens the Service Portal and submits the Car Rental Request catalog item.
2. Client-side validation checks the requested date and booking inputs.
3. The request enters the Flow Designer workflow.
4. The requester's line manager reviews the approval.
5. On approval, the workflow creates a Catalog Task for Transport Fulfillment and sends an email notification.
6. If a request is marked rejected or unavailable (`state == 3`), the Business Rule creates an Incident.
7. Request and vehicle-category data are presented in the fleet analytics dashboard.

## Functional and UAT Test Summary

All eight primary test scenarios were reported as **Pass** during system verification and User Acceptance Testing.

| Test ID | Test scenario | Expected result | Status |
|---|---|---|---|
| TC01 | Service Portal form intake | Form loads with all six mandatory variables | Pass |
| TC02 | Departure date validation | Client script blocks past dates and shows an error message | Pass |
| TC03 | Order Now submission | Generates a Request (REQ) and Requested Item (RITM) | Pass |
| TC04 | Line manager approval | Approval advances the flow; fulfillment waits for sign-off | Pass |
| TC05 | Catalog Task dispatch | Flow creates a task with trip parameters | Pass |
| TC06 | Outbound email dispatch | Email logged with subject “Your Car Request is approved” | Pass |
| TC07 | Stockout exception handling | Incident created when state is `3` | Pass |
| TC08 | Analytics visualization | Dashboard displays live Donut and Bar charts | Pass |

Example record numbers supplied in the project notes include `RITM0010001`, `SCTASK0010001`, and incidents `INC0010011`–`INC0010014`. These are instance-specific examples and may differ after redeployment or retesting.

## Technical Code Artifacts

### 1. Business Rule — Create an Incident on Rejection/Unavailability

**Table:** `x_1894596_car_re_0_car_rental_request`  
**When:** After  
**Purpose:** Create an Incident when the current record's state is `3`.

```javascript
(function executeRule(current, previous /*null when async*/) {
    if (current.state == 3) { // State 3 represents Rejected / Unavailable
        var inc = new GlideRecord('incident');
        inc.initialize();
        inc.short_description = 'Car unavailable for request: ' + current.number;
        inc.description = 'Car not available for reservation on ' + current.requested_date;
        inc.category = 'Inquiry / Help';
        inc.priority = 4; // Low priority
        inc.insert();
    }
})(current, previous);
```

**Implementation note:** As written, this rule may create another Incident on every qualifying update while the record remains in state `3`. Consider adding a condition that checks whether the state changed to `3`, or a duplicate-prevention mechanism, before using it in a production environment. Also verify that the Incident category choice value and the custom table's state values match the target instance.

### 2. Catalog Client Script — Validate Requested Date

**Type:** `onChange`  
**Variable:** `requested_date`  
**Purpose:** Prevent a past date from being selected.

```javascript
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue == '') {
        return;
    }

    var selectedDate = new Date(newValue);
    var today = new Date();
    today.setHours(0, 0, 0, 0);

    if (selectedDate < today) {
        g_form.showFieldMsg(
            'requested_date',
            'Departure date must be today or a future date.',
            'error'
        );
        g_form.setValue('requested_date', '');
    }
}
```

**Implementation note:** Date parsing can vary by input format and time zone. Test this script using the actual ServiceNow variable type and date format. Server-side validation is also recommended for rules that must not be bypassed.

## Repository Structure

The following is the project directory structure as supplied:

```text
Car-Rental-Management-System-in-ServiceNow/
├── 01_Ideation Phase/
│   ├── 01_Brainstorm_Idea_Generation_Car_Rental_Management.docx
│   ├── 02_Define_Problem_Statements_Car_Rental_Management.docx
│   └── 03_Empathy_Map_Car_Rental_Management.docx
│
├── 02_Requirement Analysis Phase/
│   ├── 01_Customer_Journey_and_User_Stories.docx
│   └── 02_Technology_Stack_Car_Rental.docx
│
├── 03_Project Design Phase/
│   ├── 01_Problem_Solution_Fit.docx
│   ├── 02_Proposed_Solution_Template.docx
│   ├── 03_Solution_Requirements.docx
│   └── 04_Solution_Architecture.docx
│
├── 04_Project Planning Phase/
│   ├── 01_Project_Logic.docx
│   └── Project Planning_Car_Rental_Management.docx
│
└── 05_Project Development Phase/
    ├── Demo video/
    │   ├── Project_Demo_Video_Walkthrough.mp4
    │   └── Project_Demo_Video_Link.txt
    ├── Documentation/
    │   └── Automated_Car_Rental_Management_Project_Report.docx
    ├── Testing_and_Evidence/
    │   ├── 01_Model_Performance_Test_Car_Rental.docx
    │   └── _UAT_Execution_Report_Car_Rental.docx
    └── Update_Set/
        └── Car Rental Management.xml
```

## Deployment

The project includes the scoped update set `Car Rental Management.xml`, reported as **Complete**.

1. Back up or export the target instance configuration before importing changes.
2. Import the update set into the target ServiceNow instance using the platform's update-set workflow.
3. Preview the update set and resolve any conflicts or missing dependencies.
4. Commit the update set after reviewing the preview results.
5. Verify the application scope, custom table, catalog item, client script, Business Rule, Flow Designer flow, notifications, and dashboard.
6. Run the UAT scenarios listed above in the target instance.

**Important:** A “Complete” state in the source project does not by itself guarantee successful deployment to another instance. Validate all dependencies and permissions in the destination environment.

## Future Enhancements

- Add server-side validation for requested dates, duration, and required values.
- Prevent duplicate Incident creation when a rejected/unavailable request is updated repeatedly.
- Add vehicle inventory checks and conflict detection for overlapping reservations.
- Track vehicle assignment, pickup, return, and maintenance status.
- Add role-based access controls and audit logging.
- Expand dashboard reporting with date filters, utilization, cancellation rates, and fulfillment times.
- Add automated tests for invalid inputs, rejected approvals, and unavailable vehicles.

## Project Outcome

The solution demonstrates how ServiceNow catalog intake, approvals, task creation, notifications, business rules, and platform analytics can be combined to streamline vehicle reservation and transport fulfillment. The project notes report successful completion of all eight primary functional and UAT scenarios.

## Author

**Segabandi Sasidhar**

---


