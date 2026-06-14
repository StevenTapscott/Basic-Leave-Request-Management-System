# Leave Request Management System

## Project Overview

The Leave Request Management System is a Power Platform solution developed using Power Apps, Dataverse, Power Automate, and Power Fx. The application streamlines employee leave requests, manager approvals, leave balance management, and workflow automation through an integrated low-code platform.

The system provides employees with a self-service leave request portal while enabling managers to review, approve, and reject requests through a dedicated approval interface. Business rules automate annual leave deductions and maintain accurate leave balances.

---

## Business Problem

Many organisations rely on spreadsheets, emails, or paper-based processes to manage employee leave requests. These methods often lead to:

- Manual administrative effort
- Delayed approvals
- Poor visibility of leave balances
- Inconsistent data management
- Lack of auditability

This solution centralises leave management into a single platform while automating key business processes.

---

## Solution Architecture

### Power Apps

Canvas application containing:

- Employee Leave Request Screen
- Manager Approval Screen
- Leave Request Form
- Approval and Rejection Actions
- Leave Balance Forecasting

### Dataverse

#### Employees Table

| Column |
|----------|
| Employee ID |
| Employee Name |
| Leave Balance |

#### Leave Requests Table

| Column |
|----------|
| Request Name |
| Employee |
| Leave Type |
| Start Date |
| End Date |
| Number of Days |
| Reason |
| Status |
| Approved |
| Rejected |
| Submitted Date |
| Approved Date |
| Rejected Date |

### Relationship Model

```text
Employees (1)
    │
    └───< Leave Requests (Many)
```

This relationship allows leave requests to retrieve employee information directly through Dataverse lookup columns.

---

## Features

### Employee Functionality

- Submit leave requests
- Select leave type
- Enter leave dates
- Provide leave reasons
- Automatically calculate leave duration
- Track request status

### Manager Functionality

- View pending leave requests
- Approve requests
- Reject requests
- Review employee leave balances
- Forecast leave balance after approval

### Business Rules

#### Annual Leave

Approved annual leave requests automatically deduct leave days from the employee's leave balance.

Example:

```text
Current Balance = 25
Requested Days = 5

New Balance = 20
```

#### Other Leave Types

The following leave types do not impact annual leave balances:

- Sick Leave
- Emergency Leave
- Study Leave

---

# Power Apps Implementation

## Employee Leave Request Screen

### Purpose

- Capture employee leave requests
- Validate required information
- Submit records to Dataverse

### Components

- Edit Forms
- Date Pickers
- Dropdown Controls
- Labels
- Buttons

---

## Manager Approval Screen

### Purpose

- Review submitted requests
- Process approvals and rejections
- Display employee balances

### Components

- Galleries
- Labels
- Approval Buttons
- Rejection Buttons

---

# Power Fx Implementation

## Leave Request Submission

### Submit Leave Request

```powerfx
SubmitForm(frmLeaveRequest)
```

### Purpose

- Validates user input
- Creates a Dataverse record
- Submits leave request data

---

## Manager Approval Logic

### Approve Request

```powerfx
If(
    Text(ThisItem.'Leave Type') = "Annual Leave",
    Patch(
        Employees,
        ThisItem.Employee,
        {
            'Leave Balance':
                ThisItem.Employee.'Leave Balance' - ThisItem.'Number of Days'
        }
    )
);

Patch(
    'Leave Requests',
    ThisItem,
    {
        Approved: true,
        Rejected: false
    }
);

Refresh(Employees);
Refresh('Leave Requests');

Notify(
    "Request approved and leave balance updated",
    NotificationType.Success
)
```

### Purpose

- Deducts annual leave balance
- Updates approval status
- Refreshes Dataverse records
- Displays confirmation notification

### Business Rule

```text
Annual Leave = Deduct Leave Balance
Other Leave Types = No Deduction
```

---

### Reject Request

```powerfx
Patch(
    'Leave Requests',
    ThisItem,
    {
        Approved: false,
        Rejected: true
    }
);

Refresh('Leave Requests');

Notify(
    "Request rejected",
    NotificationType.Warning
)
```

### Purpose

- Updates rejection status
- Refreshes request records
- Displays warning notification

---

## Manager Approval Queue

### Pending Requests Filter

```powerfx
Filter(
    'Leave Requests',
    Approved = false &&
    Rejected = false
)
```

### Purpose

- Displays only pending leave requests
- Excludes completed approvals and rejections

---

## Dataverse Relationship Usage

### Employee Name

```powerfx
ThisItem.Employee.'Employee Name'
```

### Purpose

- Retrieves employee information through Dataverse lookup relationships

---

### Current Leave Balance

```powerfx
ThisItem.Employee.'Leave Balance'
```

### Purpose

- Displays employee leave entitlement

---

### Balance After Approval

```powerfx
If(
    Text(ThisItem.'Leave Type') = "Annual Leave",
    ThisItem.Employee.'Leave Balance' - ThisItem.'Number of Days',
    ThisItem.Employee.'Leave Balance'
)
```

### Purpose

- Forecasts employee balance after approval
- Applies only to annual leave requests

---

## Status Tracking

### Display Status

```powerfx
ThisItem.'Status (crf83_status)'
```

### Purpose

- Displays Dataverse Choice field value
- Tracks workflow progression

---

## Notifications

### Approval Notification

```powerfx
Notify(
    "Request approved and leave balance updated",
    NotificationType.Success
)
```

### Rejection Notification

```powerfx
Notify(
    "Request rejected",
    NotificationType.Warning
)
```

### Testing Notification

```powerfx
Notify(
    "Button clicked",
    NotificationType.Success
)
```

Used during application testing and troubleshooting.

---

# Power Automate Implementation

## Workflow 1 – Leave Request Approval Workflow

### Trigger

```text
When a row is added
```

### Action

```text
Status = Submitted
```

### Purpose

- Automatically updates new requests to Submitted status

---

## Workflow 2 – Approved Timestamp

### Trigger

```text
When a row is modified
```

### Condition

```text
Approved = Yes
```

### Action

```text
Approved Date = utcNow()
```

### Purpose

- Records approval timestamps
- Creates audit trail

---

## Workflow 3 – Rejected Timestamp

### Trigger

```text
When a row is modified
```

### Condition

```text
Rejected = Yes
```

### Action

```text
Rejected Date = utcNow()
```

### Purpose

- Records rejection timestamps
- Maintains audit history

---

# Testing

## Test Dataset

The application was tested using multiple leave request scenarios including:

- Annual Leave
- Sick Leave
- Emergency Leave
- Study Leave

Testing covered:

- Leave submission
- Manager approvals
- Manager rejections
- Leave balance deductions
- Dataverse relationship retrieval
- Workflow automation
- Audit timestamp generation

---

## Example Annual Leave Test

| Employee | Leave Type | Days |
|----------|------------|------|
| Jimmy Test | Annual Leave | 5 |

### Result

```text
Balance Before = 25
Balance After = 20
```

---

## Example Rejection Test

| Employee | Leave Type |
|----------|------------|
| Jen Test | Emergency Leave |

### Result

```text
Approved = No
Rejected = Yes
```

---

# Skills Demonstrated

## Power Apps

- Canvas Apps
- Forms
- Galleries
- Navigation
- User Interface Design
- Power Fx

## Dataverse

- Table Design
- One-to-Many Relationships
- Lookups
- Choice Fields
- Data Modelling

## Power Automate

- Automated Cloud Flows
- Dataverse Triggers
- Conditional Logic
- Record Updates
- Workflow Automation

## Business Analysis

- Requirements Gathering
- Process Design
- Workflow Modelling
- Business Rules Implementation

---

# Technologies Used

- Microsoft Power Apps
- Microsoft Dataverse
- Microsoft Power Automate
- Power Fx

---

# Future Enhancements

- Employee Leave History Screen
- Power BI Dashboard Integration
- Automated Email Notifications
- Department-Level Approvals
- Leave Utilisation Reporting
- Mobile Optimisation
- Approval Comments and Notes
- Advanced Leave Analytics

---

# Project Outcome

Successfully developed an end-to-end Leave Request Management System using the Microsoft Power Platform. The solution demonstrates low-code application development, workflow automation, business rule implementation, Dataverse data modelling, and Power Fx programming while providing a scalable foundation for future enterprise enhancements.
