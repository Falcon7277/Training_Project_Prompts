Project Initializer

Act as an Expert DevOps and Enterprise Java Architect. Generate a clean, optimized root Parent `pom.xml` for a Maven multi-module microservices project named 'smart-hospital-system'. 

Strict Requirements:
1. Java Version: 21
2. Spring Boot Version: 3.3.x (or latest stable 3.x release compatible with Java 21)
3. Spring Cloud Version: 2023.0.x (latest compatible with Boot 3.3)
4. Packaging: pom
5. Define the following sub-modules in the <modules> section:
   - discovery-server
   - api-gateway
   - patient-service
   - doctor-service
   - appointment-service
   - billing-service
   - web-portal
6. Include standard global properties for dependencies so child modules inherit consistent versions:
   - Lombok, Spring Cloud, MapStruct, and PostgreSQL Driver.
7. Set up <dependencyManagement> for 'spring-boot-dependencies' and 'spring-cloud-dependencies' to prevent version mismatches across services.
8. Add common dependencies inside the main <dependencies> block that EVERY microservice will need:
   - Lombok
   - Spring Boot Starter Test (JUnit 5 / Mockito)
   - PostgreSQL Driver (configured globally as a runtime dependency or managed via dependencyManagement)

Provide only the clean, compilation-ready xml block. Do not include introductory or explanatory text.

Patient Service Module

Act as a Principal Enterprise Java Architect. Generate a clean, production-ready Spring Boot 3.x Core Architecture layer for our 'Patient Service' module using Java 21, Spring Data JPA, and Lombok. This module will run inside a standalone PostgreSQL database environment and must be ready to support application-level security filters.

Please generate the following dependencies for our module's `pom.xml` and the core Java files under `com.smarthospital.patientservice`:

1. REQUIRED STARTERS (Include inside the output pom dependencies snippet)
   - `spring-boot-starter-data-jpa`
   - `spring-boot-starter-web`
   - `spring-boot-starter-security` (Required for downstream self-defending authentication filters)
   - `spring-boot-starter-validation` (JSR-303 annotations)
   - `org.postgresql:postgresql` (Runtime scope)

2. ENUMS (package: entity)
   - `PatientStatus`: ACTIVE, INACTIVE, DISCHARGED

3. ENTITIES (package: entity)
   - `Patient` (Annotate as @Entity, table: "patients"): fields: id (Long, IDENTITY), firstName, lastName, email (unique, nullable=false), phoneNumber, dateOfBirth (LocalDate), status (ENUM String, default ACTIVE), registrationDate (LocalDateTime), emergencyContact (Embedded). Include a 1:1 bidirectional mapping to `MedicalProfile`.
   - `EmergencyContact` (Annotate as @Embeddable): fields: contactName, relationship, contactPhone.
   - `MedicalProfile` (Annotate as @Entity, table: "medical_profiles"): fields: id (Long, IDENTITY), patient (OneToOne, LAZY, JoinColumn "patient_id"), bloodGroup, allergies, chronicConditions, currentMedications.

4. DATA TRANSFER OBJECTS (package: dto)
   - `PatientRegistrationDTO`: Captures onboarding metrics including nested emergency contact inputs.
   - `PatientUpdateDTO`: For demographic changes.
   - `MedicalProfileUpdateDTO`: For updating clinical records (allergies, medications).
   - `PatientDTO`: Complete profile matrix view.
   - `PatientSummaryDTO`: Lightweight version (id, firstName, lastName, phone, status) for rapid inter-service lookups.

5. REPOSITORIES (package: repository)
   - `PatientRepository`: Extends JpaRepository. Include a query method to find patients by a dynamic text search string (matching first name, last name, or email) and status.
   - `MedicalProfileRepository`: Extends JpaRepository.

6. CUSTOM EXCEPTIONS (package: exception)
   - `ResourceNotFoundException` (Runtime)
   - `DuplicateResourceException` (Runtime)

7. BUSINESS LAYER (package: service & service.impl)
   - `PatientService` (Interface) and `PatientServiceImpl` (Class): Implement standard CRUD operations, profile filtering, separate medical profile updating, and a bulk summary retrieval mapping. Enforce strict database validation:
     * Check email uniqueness in the PostgreSQL database before executing registrations.
     * Soft delete patients by shifting the status string to INACTIVE.

Generate only compilation-ready, clean code structured by clear file indicators. Optimize for PostgreSQL dialect paradigms. Do not include chat introductory text.


Prompt 2:

Act as a Principal Java Security Architect. Generate a clean, highly structured Spring Boot 3.x implementation for the Exposure and Security Layer of our 'Patient Service' using Java 21, Spring Web, Spring Security, and Lombok. All data operations must target a standalone PostgreSQL configuration.

Please generate the following files across the appropriate packages under `com.smarthospital.patientservice`:

1. SECURITY & APPLICATION ISOLATION (package: security)
   - `GatewaySecurityFilter` (Extend OncePerRequestFilter): 
     * Intercept all incoming HTTP requests before they reach the main controllers.
     * Check for a custom security header named `X-Gateway-Token`.
     * If the header is missing, or does not match a configured environment property (`smart.hospital.gateway.secret`), instantly halt execution and write an HTTP 403 Forbidden response with a clean JSON body: {"error": "Forbidden", "message": "Direct access to backend microservices is strictly prohibited."}.
     * If valid, let the request pass seamlessly into the Spring Security authentication filter chain.
   - `SecurityConfig` (Class, @Configuration, @EnableWebSecurity):
     * Configure a `SecurityFilterChain` bean. 
     * Disable CSRF (stateless REST design).
     * Add `GatewaySecurityFilter` into the filter chain before the OAuth2 / JWT resource server validation filters.
     * Enforce `.anyRequest().authenticated()` using standard Spring Boot OAuth2 Resource Server JWT validation rules.

2. EXCEPTION HANDLING (package: exception)
   - `ErrorResponse` (Class or Record): fields: timestamp (LocalDateTime), status (int), error (String), message (String).
   - `GlobalExceptionHandler` (Annotate as @RestControllerAdvice): 
     * Handle `ResourceNotFoundException` -> return HTTP 404 with ErrorResponse.
     * Handle `DuplicateResourceException` -> return HTTP 400 with ErrorResponse.
     * Handle JSR-303 input validations (`MethodArgumentNotValidException`) -> aggregate field errors cleanly and return HTTP 400.
     * Handle catch-all generic `Exception` -> return HTTP 500.

3. EXPOSURE LAYER (package: controller)
   - `PatientController` (Annotate as @RestController, @RequestMapping("/api/v1/patients")): Inject `PatientService` using constructor injection. Implement the following endpoints with strict response entities:
     * **POST `/`**: Accept `@Valid @RequestBody PatientRegistrationDTO`. Returns created `PatientDTO` with HTTP 201 Created.
     * **GET `/{id}`**: Accept `@PathVariable Long id`. Returns `PatientDTO` with HTTP 200 OK.
     * **GET `/`**: Accept optional `@RequestParam String searchTerm`, optional `@RequestParam PatientStatus status`, and standard Spring Data `@PageableDefault(size = 10, sort = "lastName") Pageable pageable`. Returns `Page<PatientDTO>` with HTTP 200 OK.
     * **POST `/summary`**: Inter-service aggregation route. Accept `@RequestBody List<Long> ids`. Returns `List<PatientSummaryDTO>` with HTTP 200 OK.
     * **PUT `/{id}`**: Accept `@PathVariable Long id` and `@Valid @RequestBody PatientUpdateDTO`. Returns updated `PatientDTO` with HTTP 200 OK.
     * **PUT `/{id}/medical-profile`**: Accept `@PathVariable Long id` and `@Valid @RequestBody MedicalProfileUpdateDTO`. Returns updated `MedicalProfileDTO` with HTTP 200 OK.
     * **DELETE `/{id}`**: Executes soft delete and returns HTTP 204 No Content.

Generate only clean, compilation-ready Java files structured explicitly by clear file headers. Do not include chat setup or introductory conversational text.


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

Prompt 4:


Act as a Senior Full-Stack Java Developer specializing in Spring Boot, Thymeleaf, and Bootstrap CSS. Generate the complete UI Integration layer for the Patient Management module inside our 'web-portal' service. 

Please generate the following files across the appropriate directories under `com.smarthospital.webportal` and `src/main/resources`:

1. CONFIGURATION & CLIENT (package: client)
   - Create a lightweight `PatientClient` interface using `@FeignClient(name = "api-gateway")` (or name = "patient-service" if routing directly). 
   - Mirror the core REST operations from the Patient backend:
     * GET `/api/v1/patients` with optional searchTerm, status, page, and size parameters.
     * GET `/api/v1/patients/{id}` to fetch a specific profile.
     * POST `/api/v1/patients` to register a patient.
     * DELETE `/api/v1/patients/{id}` for soft deletes.

2. VIEW CONTROLLER (package: controller)
   - `PatientWebController` (Annotate as @Controller, @RequestMapping("/portal/patients")): Inject `PatientClient`. Implement methods returning Thymeleaf template string paths:
     * **GET `/`**: Call client to fetch paginated/filtered patients. Add the resulting Page object, current `searchTerm`, and `status` to the Spring UI `Model`. Return `"patients/list"`.
     * **GET `/register`**: Add an empty `PatientRegistrationDTO` object to the `Model` to bind to the form template. Return `"patients/register"`.
     * **POST `/register`**: Accept `@ModelAttribute("patient") PatientRegistrationDTO dto`. Call `PatientClient` to save data. On success, return a redirect string: `"redirect:/portal/patients?success=true"`. Catch exceptions gracefully and pass error messages back to the view.
     * **POST `/delete/{id}`**: Accept `@PathVariable Long id`. Trigger client deletion and return `"redirect:/portal/patients?deleted=true"`.

3. THYMELEAF TEMPLATES (Directory: `src/main/resources/templates/patients/`)
   - **`list.html`**: A clean dashboard view styled with Bootstrap 5. 
     * Include a responsive navbar and a search header containing an inline form (input for `searchTerm`, dropdown selector for `status`, and a "Search" button).
     * A structured data table showing: ID, Full Name, DOB, Phone, Status (styled with dynamic badges: `bg-success` for ACTIVE, `bg-danger` for INACTIVE).
     * An action column containing a "View" button and a "Delete" button mapping to form POST triggers.
     * A clean pagination footer component (`Previous` / `Next` links handling page index state).
   - **`register.html`**: An elegant registration form card layout styled with Bootstrap 5.
     * Comprehensive form layout capturing First Name, Last Name, Email, Phone, DOB, Gender (Select dropdown), and basic Emergency Contact inputs (Name, Relationship, Phone).
     * Use native Thymeleaf binding selectors (`th:object`, `th:field`, `th:errors`) to handle form validation states cleanly.
     * Include "Save Profile" and "Cancel" buttons routing back to the main directory grid.

Generate only compilation-ready, clean Java and HTML source files clearly separated by path indicators. Do not include chat setup explanations or conversational prose.

Doctor Service Module

Prompt 1:

