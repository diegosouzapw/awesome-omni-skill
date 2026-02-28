---
name: Admin and Seed Data
description: Manage database seeding, reset operations, and the admin interface.
globs:
  - "RestReactAspire.Server/Endpoints/AdminEndpoints.cs"
  - "RestReactAspire.Server/Stores/SeedDataGenerator.cs"
  - "RestReactAspire.Server/Telemetry/AdminTelemetry.cs"
  - "RestReactAspire.Server/Models/AdminDto.cs"
  - "frontend/src/pages/AdminPage.tsx"
---

# Admin and Seed Data

## Admin API Endpoints
Located in `RestReactAspire.Server/Endpoints/AdminEndpoints.cs`, registered under `/api/admin`.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/seed` | POST | Populates database with sample data (additive) |
| `/reset` | POST | Deletes all data from all collections |
| `/stats` | GET | Returns current count of patients, doctors, exams |

### Response DTOs (`AdminDto.cs`)
- `SeedResponse(int PatientsCreated, int DoctorsCreated, int ExamsCreated, Links)`
- `ResetResponse(int PatientsDeleted, int DoctorsDeleted, int ExamsDeleted, Links)`
- `StatsResponse(int PatientCount, int DoctorCount, int ExamCount, Links)`

## Seed Data Generator
Located in `RestReactAspire.Server/Stores/SeedDataGenerator.cs`.

### Current Data Volumes
- **100 patients** — random Italian names, varied dates of birth, email, phone
- **30 doctors** — random Italian names, assigned from 15 medical specialties
- **200 exams** — distributed across patients and doctors with realistic types, durations, dates, statuses, results, and notes

### Data Characteristics
- Patient names drawn from pools of 50 first names and 50 last names.
- Doctor specialties: Cardiology, Neurology, Orthopedics, Dermatology, Gastroenterology, Ophthalmology, Pulmonology, Endocrinology, Urology, Oncology, Rheumatology, Nephrology, Hematology, Infectious Disease, General Surgery.
- Exam types: Blood Test, MRI Brain, X-Ray Chest, ECG, Skin Biopsy, Colonoscopy, Eye Exam, Spirometry, Thyroid Panel, Ultrasound, Urinalysis, Mammography, CT Scan, Bone Density Scan, Stress Test.
- Each exam type has min/max duration ranges for realistic `DurationMinutes`.
- Completed exams have type-specific result strings and notes.
- Scheduled dates span a 12-month range for meaningful time-series charts.

### Modifying Seed Data
- Adjust counts in `GeneratePatients()`, `GenerateDoctors()`, `GenerateExams()`.
- Add new name pools, specialties, or exam types to the static arrays.
- Ensure cross-references remain valid (exams reference valid patient and doctor IDs).

## Frontend Admin Page
- Displays current database stats (patient, doctor, exam counts).
- Provides "Seed Database" and "Reset Database" buttons.
- Shows operation results after seeding or resetting.
