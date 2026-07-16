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

Prompt 2:

Please generate the following files across the appropriate packages under `com.smarthospital.patientservice`:

1. EXCEPTION HANDLING (package: exception)
   - `ErrorResponse` (Class or Record): fields: timestamp (LocalDateTime), status (int), error (String), message (String).
   - `GlobalExceptionHandler` (Annotate as @RestControllerAdvice): 
     * Handle `ResourceNotFoundException` -> return HTTP 404 with ErrorResponse.
     * Handle `DuplicateResourceException` -> return HTTP 400 with ErrorResponse.
     * Handle standard JSR-303 validation exceptions (`MethodArgumentNotValidException`) -> gather field errors into a readable message and return HTTP 400.
     * Handle catch-all generic `Exception` -> return HTTP 500 with a clean "Internal Server Error" message.

2. EXPOSURE LAYER (package: controller)
   - `PatientController` (Annotate as @RestController, @RequestMapping("/api/v1/patients")): Inject `PatientService` using constructor injection. Implement the following endpoints with proper HTTP status codes:
     * **POST `/`**: Accept `@Valid @RequestBody PatientRegistrationDTO`. Returns the created `PatientDTO` with HTTP 201 Created.
     * **GET `/{id}`**: Accept `@PathVariable Long id`. Returns `PatientDTO` with HTTP 200 OK.
     * **GET `/`**: Accept optional `@RequestParam(required = false) String searchTerm`, `@RequestParam(required = false) PatientStatus status`, and standard Spring Data `@PageableDefault(size = 10, sort = "lastName") Pageable pageable`. Returns `Page<PatientDTO>` with HTTP 200 OK.
     * **POST `/summary`**: Accept `@RequestBody List<Long> ids`. **Inter-service API** to fetch lightweight summaries. Returns `List<PatientSummaryDTO>` with HTTP 200 OK.
     * **PUT `/{id}`**: Accept `@PathVariable Long id` and `@Valid @RequestBody PatientUpdateDTO`. Returns the updated `PatientDTO` with HTTP 200 OK.
     * **PUT `/{id}/medical-profile`**: Accept `@PathVariable Long id` and `@Valid @RequestBody MedicalProfileUpdateDTO`. Returns the updated `MedicalProfileDTO` with HTTP 200 OK.
     * **DELETE `/{id}`**: Accept `@PathVariable Long id`. Executes soft delete and returns HTTP 204 No Content.

Generate only clean, compilation-ready code structured by clear file headers. Do not include chat explanations or introductory prose.

Prompt 3:

Act as a Principal QA and Java Automation Engineer. Generate comprehensive unit and slice tests for our 'Patient Service' using JUnit 5, Mockito, and Spring Boot Test tools. 

Please generate the following two test classes under `src/test/java/com/smarthospital/patientservice`:

1. SERVICE LAYER TESTS (File: `PatientServiceTest.java`)
   - Use `@ExtendWith(MockitoExtension.class)` to test `PatientServiceImpl`.
   - Mock the `PatientRepository`.
   - Implement the following test cases using Assertions (`assertEquals`, `assertThrows`, `assertNotNull`):
     * `registerPatient_Success`: Verify that a patient is saved successfully when the email is unique.
     * `registerPatient_ThrowsDuplicateResourceException`: Verify that a DuplicateResourceException is thrown if the email already exists.
     * `getPatientById_Success`: Verify the correct DTO is returned for a valid existing ID.
     * `getPatientById_ThrowsResourceNotFoundException`: Verify it throws an exception when the ID doesn't exist.
     * `deletePatient_Success`: Verify that soft delete modifies the patient status to INACTIVE and saves it.

2. CONTROLLER LAYER TESTS (File: `PatientControllerTest.java`)
   - Use `@WebMvcTest(PatientController.class)` to perform slice testing on the API layer.
   - Inject `MockMvc` and mock the `PatientService` using `@MockBean` (or `@MockitoBean` for Spring Boot 3.4+).
   - Implement the following test cases using MockMvc Fluent APIs (`perform`, `status().is*`, `jsonPath`):
     * `createPatient_Returns201AndJson`: Validates POST `/api/v1/patients` triggers a 201 Created and maps fields smoothly.
     * `createPatient_Returns400BadRequest`: Validates JSR-303 input validations fail gracefully when passing an invalid email or missing fields.
     * `getPatientById_Returns404NotFound`: Mock the service to throw ResourceNotFoundException and verify MockMvc accurately handles the GlobalExceptionHandler mapping to return an HTTP 404 status.
     * `searchPatients_ReturnsPaginatedList`: Validates GET `/api/v1/patients` processes parameters and returns a Page matrix cleanly.

Generate only clean, compilation-ready test code with clear class boundaries. Do not include chat explanations or introductory prose.
