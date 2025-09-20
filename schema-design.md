# Smart Clinic — Schema Design

This document captures a pragmatic hybrid design: **operational, relational data in MySQL** and **flexible, document-shaped data in MongoDB**.

## Real-World Assumptions
- A clinic registers patients & doctors, manages locations and working hours, schedules appointments, records payments.
- Free-form artifacts (prescription bodies, rich notes, attachments, chats/logs) vary a lot over time → better fit for NoSQL.

---

## MySQL Database Design

> Core, validated, relational data lives here. Use `InnoDB`, UTF-8, and `created_at/updated_at` audit columns. Prefer **soft deletes** (`is_active`) over hard deletes.

### Table: patients
- `id` INT PK AUTO_INCREMENT  
- `first_name` VARCHAR(100) NOT NULL  
- `last_name` VARCHAR(100) NOT NULL  
- `date_of_birth` DATE NOT NULL  
- `sex` ENUM('F','M','X') NULL  
- `email` VARCHAR(255) UNIQUE NULL  
- `phone` VARCHAR(20) UNIQUE NULL  
- `national_id` VARCHAR(20) UNIQUE NULL  <!-- optional, country-specific -->
- `is_active` TINYINT(1) NOT NULL DEFAULT 1  
- `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP  
- `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP  

**Notes:** Personally identifiable info is minimal; validate email/phone in code. Prefer **restrict** over cascade on deletes to preserve medical history.

---

### Table: clinic_locations
- `id` INT PK AUTO_INCREMENT  
- `name` VARCHAR(120) NOT NULL  
- `address_line1` VARCHAR(200) NOT NULL  
- `address_line2` VARCHAR(200) NULL  
- `city` VARCHAR(100) NOT NULL  
- `district` VARCHAR(100) NULL  
- `postal_code` VARCHAR(12) NULL  
- `phone` VARCHAR(20) NULL

---

### Table: doctors
- `id` INT PK AUTO_INCREMENT  
- `first_name` VARCHAR(100) NOT NULL  
- `last_name` VARCHAR(100) NOT NULL  
- `specialty` VARCHAR(100) NOT NULL  
- `license_no` VARCHAR(50) NOT NULL UNIQUE  
- `email` VARCHAR(255) UNIQUE NOT NULL  
- `phone` VARCHAR(20) NULL  
- `clinic_location_id` INT NOT NULL → FK `clinic_locations(id)` ON UPDATE CASCADE  
- `is_active` TINYINT(1) NOT NULL DEFAULT 1  
- `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP  
- `updated_at` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

---

### Table: doctor_working_hours
> Recurring weekly availability; exceptions (leave, holidays) can be handled later via an optional `doctor_time_off` table.

- `id` INT PK AUTO_INCREMENT  
- `doctor_id` INT NOT NULL → FK `doctors(id)` ON DELETE CASCADE  
- `day_of_week` TINYINT NOT NULL CHECK (`day_of_week` BETWEEN 0 AND 6)  <!-- 0=Sunday -->
- `start_time` TIME NOT NULL  
- `end_time` TIME NOT NULL  
- `location_id` INT NULL → FK `clinic_locations(id)`  
- **UNIQUE** (`doctor_id`, `day_of_week`, `start_time`, `end_time`)  
- **CHECK** (`end_time` > `start_time`)

---

### Table: admins
> Staff users who create appointments, take payments, etc.

- `id` INT PK AUTO_INCREMENT  
- `full_name` VARCHAR(120) NOT NULL  
- `email` VARCHAR(255) NOT NULL UNIQUE  
- `role` ENUM('admin','receptionist','nurse') NOT NULL DEFAULT 'admin'  
- `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP

---

### Table: appointments
- `id` INT PK AUTO_INCREMENT  
- `doctor_id` INT NOT NULL → FK `doctors(id)`  
- `patient_id` INT NOT NULL → FK `patients(id)`  
- `start_time` DATETIME NOT NULL  
- `end_time` DATETIME NOT NULL  
- `status` ENUM('scheduled','completed','cancelled','no_show','rescheduled') NOT NULL DEFAULT 'scheduled'  
- `reason` VARCHAR(255) NULL  
- `notes` TEXT NULL  <!-- brief operational note; rich notes go to MongoDB -->
- `created_by_admin_id` INT NULL → FK `admins(id)`  
- **INDEX** (`doctor_id`, `start_time`)  
- **INDEX** (`patient_id`, `start_time`)  
- **CHECK** (`end_time` > `start_time`)

**Overlap rule:** MySQL can’t natively enforce “no overlapping slots” for the same doctor via a single constraint; enforce in application/service layer (and/or via transactions). Consider a **deferrable lock** pattern around scheduling.

**Deletion policy:** Do **not** cascade delete appointments when a patient/doctor is deactivated—use `is_active=0` and retain history.

---

### Table: payments
- `id` INT PK AUTO_INCREMENT  
- `appointment_id` INT NOT NULL → FK `appointments(id)` ON DELETE RESTRICT  
- `patient_id` INT NOT NULL → FK `patients(id)`  
- `amount` DECIMAL(10,2) NOT NULL  
- `currency` CHAR(3) NOT NULL DEFAULT 'TRY'  
- `method` ENUM('cash','credit_card','debit_card','online') NOT NULL  
- `status` ENUM('pending','paid','refunded','failed') NOT NULL DEFAULT 'pending'  
- `paid_at` DATETIME NULL  
- `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP  
- **INDEX** (`patient_id`, `paid_at`)  
- **INDEX** (`appointment_id`)

---

### Why SQL vs Mongo here?
- **SQL**: identities, relationships, scheduling integrity, payments → strong constraints & joins.
- **MongoDB**: rich/variable structures (full prescription bodies, tags, attachments, long notes, chats, logs) that evolve.

---

## MongoDB Collection Design

> We’ll store **prescriptions** in MongoDB for flexibility (multi-drug, rich notes, pharmacy metadata, attachments). We keep **references** back to SQL by id.

### Collection: `prescriptions`
Example document:
```json
{
  "_id": { "$oid": "64abc1234567890abcdef123" },
  "appointmentId": 51,             // SQL appointments.id
  "patientId": 1203,               // SQL patients.id
  "doctorId": 42,                  // SQL doctors.id
  "createdAt": { "$date": "2025-09-20T11:00:00Z" },
  "status": "active",              // active | void | replaced
  "medications": [
    {
      "name": "Paracetamol",
      "form": "tablet",
      "dosageMg": 500,
      "route": "oral",
      "frequency": "1 tablet every 6 hours",
      "durationDays": 5,
      "instructions": "Take after meals."
    },
    {
      "name": "Ibuprofen",
      "form": "capsule",
      "dosageMg": 200,
      "route": "oral",
      "frequency": "1 capsule every 8 hours",
      "durationDays": 3,
      "instructions": "Avoid if gastric upset."
    }
  ],
  "doctorNotes": "Monitor temperature; hydrate.",
  "tags": ["analgesic", "OTC"],
  "pharmacy": {
    "name": "City Pharmacy",
    "location": { "text": "Izmir - Konak" },
    "contact": { "phone": "+90-232-000-0000" }
  },
  "attachments": [
    {
      "fileName": "prescription-51.pdf",
      "mimeType": "application/pdf",
      "sizeBytes": 18423,
      "url": "s3://smart-clinic/prescriptions/51.pdf"
    }
  ],
  "audit": {
    "createdBy": "admin:7",
    "updatedBy": "doctor:42",
    "updatedAt": { "$date": "2025-09-20T11:30:00Z" }
  }
}
