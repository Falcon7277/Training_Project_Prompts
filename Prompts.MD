Project Initializer

Act as an Expert DevOps and Enterprise Java Architect. Generate a clean, optimized root Parent `pom.xml` for a Maven multi-module microservices project named 'smart-hospital-system'. 

Strict Requirements:
1. Java Version: 21
2. Spring Boot Version: 3.3.x (or latest stable 3.x release compatible with Java 21)
3. Spring Cloud Version: 2023.0.x (Leyton release train or latest compatible with Boot 3.3)
4. Packaging: pom
5. Define the following sub-modules in the <modules> section:
   - api-gateway
   - patient-service
   - doctor-service
   - appointment-service
   - billing-service
   - web-portal
6. Include standard global properties for dependencies so child modules inherit consistent versions:
   - Lombok, Spring Cloud, MapStruct, and H2 Database.
7. Set up <dependencyManagement> for 'spring-boot-dependencies' and 'spring-cloud-dependencies' to prevent version mismatches across services.
8. Add common dependencies inside the main <dependencies> block that EVERY microservice will need (Lombok, Spring Boot Starter Test).

Provide only the clean, compilation-ready xml block. Do not include introductory or explanatory text.

Patient Service Module

Prompt 1: 

Act as a Principal Java Architect. Generate a clean, highly structured Spring Boot 3.x implementation for the Core Architecture Layer of our 'Patient Service' using Java 21, Spring Data JPA, and Lombok. All data models must support an in-memory H2 Database workflow.

Please generate the following files across the appropriate packages under `com.smarthospital.patientservice`:

1. ENUMS (package: entity)
   - `Gender`: MALE, FEMALE, OTHER
   - `PatientStatus`: ACTIVE, INACTIVE, DISCHARGED

2. ENTITIES (package: entity)
   - `EmergencyContact` (Annotate as @Embeddable): fields: contactName, relationship, contactPhone.
   - `MedicalProfile` (Annotate as @Entity, table: medical_profiles): fields: id (Long, IDENTITY), patient (OneToOne, LAZY, JoinColumn patient_id), bloodGroup, chronicConditions (TEXT), allergies (TEXT), currentMedications (TEXT).
   - `Patient` (Annotate as @Entity, table: patients): fields: id (Long, IDENTITY), firstName, lastName, dateOfBirth (LocalDate), gender (ENUM String), email (String, unique), phoneNumber, address, status (ENUM String, default ACTIVE), medicalProfile (OneToOne, mappedBy, cascade ALL, LAZY), emergencyContact (@Embedded). Include a @PrePersist method to enforce default status.

3. DATA TRANSFER OBJECTS (package: dto)
   - `PatientRegistrationDTO`: Core info + embedded fields needed to onboard a patient and clear their initial medical baseline.
   - `PatientUpdateDTO`: For updating demographics and emergency contact data only.
   - `MedicalProfileUpdateDTO`: For updating clinical data lists.
   - `PatientDTO`: Comprehensive data view mapping the full profile.
   - `PatientSummaryDTO`: Lightweight version containing only id, firstName, lastName, and phoneNumber for rapid cross-service lookups.

4. REPOSITORY (package: repository)
   - `PatientRepository`: Extends JpaRepository. Implement a paginated search query using standard query methods or JPQL to filter patients dynamically where:
     (firstName OR lastName contains searchTerm case-insensitive) OR (phoneNumber equals searchTerm), filtering optionally by PatientStatus. It must accept a Pageable object.

5. BUSINESS LOGIC (package: service & service.impl)
   - Create custom runtime exceptions: `ResourceNotFoundException` and `DuplicateResourceException`.
   - `PatientService` (Interface): Define full CRUD methods, paginated search retrieval, and bulk summary lookups using lists of IDs.
   - `PatientServiceImpl` (Class): Implement the interface. Add explicit business rules:
     * Throw DuplicateResourceException if an email already exists during registration.
     * Throw ResourceNotFoundException if any target ID does not exist.
     * Handle soft deletes by setting status to INACTIVE.
     * Manage mapping data bi-directionally between DTOs and Entities explicitly (or via simple assignment methods).

Generate only clean, compilation-ready code structured by file headers. Do not include chat explanations or introductory prose.
