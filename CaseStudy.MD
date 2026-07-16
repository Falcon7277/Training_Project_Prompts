Architecture Planning Report: Smart Hospital Management System (SHMS) MVP
1. Core Features (Minimum Viable Product)
The MVP focus is to deliver a functional, end-to-end digital lifecycle for a patient visiting a hospital, managing everything from onboarding to clinical scheduling and ultimate financial settlement.

Patient Lifecycle & Profile Management: Digital onboarding of patients, maintaining critical baseline medical profiles (allergies, chronic conditions), and tracking admission statuses.

Clinical Staff Directory & Schedule Management: Organization of medical professionals by clinical departments alongside their standard daily availability constraints.

End-to-End Appointment Orchestration: Booking, validation, status tracking, and chronological coordination between patients and active doctors.

Automated Financial Ledger & Settlement: Generation of itemized invoices based on physical consultations, recording transaction flows, and handling basic payment states.

2. Microservices Architecture Matrix
The system is divided into highly decoupled, standalone services that run concurrently to maintain strict domain boundaries:

Discovery Server (Eureka): Acts as the central service registry allowing all microservices to dynamically locate each other without hardcoded network ports.

API Gateway (Spring Cloud Gateway): Serves as the single entry point for all client requests, intercepting traffic to execute unified JWT security validation and reactive request routing.

Patient Service: Operates as the isolated boundary for managing all administrative, demographic, and baseline clinical profiles of the hospital's patients.

Doctor Service: Manages internal clinical staff profiles, department structures, and structural availability rules.

Appointment Service: Serves as the operational engine, binding patient timelines with doctor schedules without relying on direct database joins.

Billing Service: Functions as the financial ledger, automatically converting completed clinical milestones into itemized, payable invoices.

Web Portal (Thymeleaf Frontend Server): A stateful Spring MVC server that renders HTML/Bootstrap views for the end-user while making server-side REST client calls to the API Gateway.

3. Main Entities Managed Per Service
To guarantee database isolation, each microservice maintains its own physical database instance inside PostgreSQL:

Patient Service (hospital_patient_db)

Patient: Core identity, contact data, and active statuses (ACTIVE, INACTIVE, DISCHARGED).

MedicalProfile: Baseline clinical metrics (bloodGroup, allergies, chronicConditions) in a 1:1 mapping.

EmergencyContact: Embedded component containing crucial relative notification fields.

Doctor Service (hospital_doctor_db)

Doctor: Medical practitioner profiles and operational statuses.

Department: Clinical divisions (Cardiology, Pediatrics) structuring the staff grid.

AvailabilitySlot: Weekly recurring time configurations defining standard working windows.

Appointment Service (hospital_appointment_db)

Appointment: Chronological binding record tracking the relationship between patientId, doctorId, and targeted scheduling windows.

Billing Service (hospital_billing_db)

Invoice: The financial parent ledger managing totals, payable balances, and transaction states.

InvoiceItem: Itemized child line items representing individual fees (e.g., consultation fee, lab costs).

Payment: Tracks separate transactional receipts mapped against an open invoice.

4. Overall Request Flow
The typical execution path follows a patient from arrival to discharge:

[Patient Arrival] ──► Onboarded via Thymeleaf UI ──► Registered in Patient Service
                                                            │
[Booking Slot]    ◄── Selects Dept & Doctor ◄───────────────┘
     │
     ▼
[Appointment Service] ──(Validates Doctor Availability)──► Confirms & Schedules Slot
     │
     ▼ (Status sets to COMPLETED)
[Billing Service]     ──(Generates Invoice + Line Items)──► Patient Clears Balance via UI
Registration: A receptionist inputs patient data into the web-portal UI. The portal forwards a validated JSON request through the API Gateway to the Patient Service, writing a new record to hospital_patient_db.

Scheduling: The user browses available clinical staff. The web-portal queries the Doctor Service to list departments and matching doctor availability slots. A selection triggers an orchestration call to the Appointment Service.

Validation & Confirmation: The Appointment Service cross-references the scheduling window using an inter-service client callback to ensure the doctor isn't double-booked, saving the confirmation to hospital_appointment_db.

Completion & Invoicing: Once the medical checkup is completed, the appointment status transitions to COMPLETED. This triggers the generation of a corresponding invoice record inside the Billing Service, appending basic consultation costs.

Settlement: The patient reviews their bill on the frontend dashboard. Processing a payment alters the billing status to PAID, completing the workflow.

5. Service-to-Service Interaction Model
Communication between modules uses synchronous HTTP REST endpoints backed by Spring Cloud OpenFeign, organized through the central API Gateway:

Web Portal to API Gateway: The Thymeleaf UI framework securely transmits all data mutation requests via the Gateway (port 8080) using saved JWT tokens.

Gateway to Downstream Services: The Gateway queries the Discovery Server (Eureka) to route /api/v1/patients/**, /api/v1/doctors/**, etc., directly to the matching active service port.

Appointment Service to Doctor Service: The Appointment engine dynamically pulls specific availability profiles via Feign client endpoints to prevent cross-booking anomalies.

UI Composition Aggregation: When building data grids (like the Appointment Dashboard), the Web Portal queries the core Appointment records, reads the isolated IDs, and executes bulk summary calls (/summary) to the Patient and Doctor services to resolve names efficiently for the UI view.

6. Future Architectural Evolution (AI Integration Scope)
Once this stable MVP foundation is established, the decoupled microservices design allows seamlessly plugging in an isolated AI-Service via the Gateway to execute advanced processing:

Predictive Scheduling Models: Incorporating classification algorithms to evaluate historical cancellation behaviors, flags high-probability patient "no-shows," and suggest optimal appointment reminders.

Clinical Natural Language Processing (NLP): Embedding Large Language Models (LLMs) to scan raw, unstructured clinical text entries typed by doctors during a visit, automatically parsing it into structured text arrays to update the MedicalProfile entity.

Financial Anomaly Detection: Scanning InvoiceItems prior to insurance submission to catch billing entry errors or project insurance claim rejection risks based on historical transaction profiles.
