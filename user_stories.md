# User Story Template

**Title:**
_As a [user role], I want [feature/goal], so that [reason]._

**Acceptance Criteria:**
1. [Criteria 1]
2. [Criteria 2]
3. [Criteria 3]

**Priority:** [High/Medium/Low]
**Story Points:** [Estimated Effort in Points]
**Notes:**
- [Additional information or edge cases]


# Doctor User Stories

## 1) View Today’s Appointments
Title: As a Doctor, I want to see my appointments for today, so that I can prepare and manage my schedule.

Acceptance Criteria:
- [Given I am authenticated as a Doctor] [When I open the “Today” view] [Then I see a list with time, patient name, reason, and status for today’s appointments]
- [Given filters available] [When I filter by status or time] [Then the list updates accordingly]
- [Given no appointments] [When I open the view] [Then I see an empty state message]

Priority: High  
Story Points: 3  
Notes: Pagination if >50 items.

---

## 2) Write & Update Visit Notes
Title: As a Doctor, I want to enter and update visit notes, so that patient records are accurate.

Acceptance Criteria:
- [Given an appointment] [When I open its details] [Then I can add structured notes (SOAP fields)]
- [Given saved notes] [When I update] [Then versioning is maintained]
- [Given empty mandatory fields] [When saving] [Then I get validation errors]

Priority: High  
Story Points: 5  
Notes: Audit trail required.

---

## 3) E-Prescribe Medication
Title: As a Doctor, I want to create an e-prescription, so that the patient can obtain medication.

Acceptance Criteria:
- [Given a patient record] [When I enter medication details] [Then an unsigned Rx is stored]
- [Given a pending Rx] [When I sign] [Then it becomes “Signed” and is transmitted]
- [Given drug-allergy conflict] [When signing] [Then I get a blocking alert]

Priority: High  
Story Points: 8  
Notes: Check allergies/interactions.

---

## 4) Approve/Decline Appointment Requests
Title: As a Doctor, I want to approve or decline appointment requests, so that my calendar stays manageable.

Acceptance Criteria:
- [Given pending requests] [When I view requests] [Then I see details]
- [Given a decision] [When I approve] [Then patient notified + calendar updated]
- [Given a decline] [When I add a reason] [Then the patient sees the reason]

Priority: Medium  
Story Points: 5  
Notes: No double-booking.

---

## 5) View Patient History (Read-Only)
Title: As a Doctor, I want to review patient history, so that I can make informed decisions.

Acceptance Criteria:
- [Given a valid patient record] [When I view history] [Then I see meds, results, diagnoses]
- [Given restricted access] [When emergency override] [Then reason required + logged]

Priority: High  
Story Points: 5  
Notes: Strict RBAC.

---

# Patient User Stories

## 1) Register & Verify Account
Title: As a Patient, I want to register and verify my account, so that I can access my records.

Acceptance Criteria:
- [Given registration form] [When valid submission] [Then verification sent]
- [Given verification link] [When confirmed] [Then account becomes active]
- [Given expired link] [When clicked] [Then request new verification]

Priority: High  
Story Points: 3  
Notes: Password policy.

---

## 2) Request an Appointment
Title: As a Patient, I want to request an appointment, so that I can visit the Doctor.

Acceptance Criteria:
- [Given I’m logged in] [When I choose slot] [Then request set to Pending]
- [Given invalid slot] [When submitting] [Then conflict message]
- [Given decision made] [When status changes] [Then notification delivered]

Priority: High  
Story Points: 5  
Notes: Time zone handling.

---

## 3) View Prescriptions
Title: As a Patient, I want to view my prescriptions, so that I can follow treatment.

Acceptance Criteria:
- [Given logged in] [When opening Prescriptions] [Then show active + past meds]
- [Given signed e-Rx] [When viewing details] [Then show pharmacy + status]
- [Given expired med] [When visible] [Then tagged as Expired]

Priority: Medium  
Story Points: 3  
Notes: Mask identifiers.

---

## 4) Secure Messaging
Title: As a Patient, I want to message clinic staff, so that I can ask non-urgent questions.

Acceptance Criteria:
- [Given messaging UI] [When sending] [Then message stored + routed]
- [Given replies] [When logged in] [Then unread notifications]
- [Given emergency keywords] [When detected] [Then show emergency warning]

Priority: Medium  
Story Points: 5  
Notes: Keyword detection required.

---

## 5) Update Profile & Insurance
Title: As a Patient, I want to update my profile, so that records remain accurate.

Acceptance Criteria:
- [Given profile form] [When valid update] [Then changes saved]
- [Given insurance upload] [When submitted] [Then status Pending Review]
- [Given invalid fields] [When submit] [Then validation blocks]

Priority: Low  
Story Points: 3  
Notes: Store document metadata.

---

# Admin User Stories

## 1) Manage User Roles
Title: As an Admin, I want to create and manage users, so that access is properly controlled.

Acceptance Criteria:
- [Given user form] [When created] [Then activation link sent]
- [Given role change] [When saved] [Then access updates instantly]
- [Given self-role change] [When removing Admin] [Then block action]

Priority: High  
Story Points: 5  
Notes: Full audit required.

---

## 2) Configure Clinic Hours & Slots
Title: As an Admin, I want to configure clinic hours, so that appointments follow rules.

Acceptance Criteria:
- [Given clinic settings] [When hours/slots set] [Then scheduler uses them]
- [Given conflicts] [When saving] [Then see errors]
- [Given publishing updates] [When confirmed] [Then future slots updated]

Priority: High  
Story Points: 5  
Notes: Handle weekends + holidays.

---

## 3) Audit Logs
Title: As an Admin, I want to view audit logs, so that compliance can be monitored.

Acceptance Criteria:
- [Given filters] [When applied] [Then see results by timestamp]
- [Given export triggered] [When download CSV] [Then only visible columns included]
- [Given emergency access] [When used] [Then log reason]

Priority: High  
Story Points: 8  
Notes: Tamper-evident logs.

---

## 4) Master Data Management
Title: As an Admin, I want to maintain medication and allergy lists, so that Doctors use valid data.

Acceptance Criteria:
- [Given catalog] [When edit items] [Then versioning maintained]
- [Given deprecated item] [When searching] [Then hidden from new usage]
- [Given bulk upload] [When importing] [Then error report shown]

Priority: Medium  
Story Points: 8  
Notes: Standard medical codes optional.

---

## 5) System Health Monitoring
Title: As an Admin, I want to monitor system health, so that I can respond to issues quickly.

Acceptance Criteria:
- [Given dashboard] [When opened] [Then show uptime and DB metrics]
- [Given thresholds] [When exceeded] [Then send alerts]
- [Given maintenance window] [When scheduled] [Then display user banner]

Priority: Medium  
Story Points: 5  
Notes: Integrate basic health checks first.

