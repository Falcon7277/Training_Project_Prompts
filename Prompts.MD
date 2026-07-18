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

# Create the physical sub-folders for every microservice
mkdir discovery-server api-gateway patient-service doctor-service appointment-service billing-service web-portal

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Point directly back up to your root parent project config -->
    <parent>
        <groupId>com.smarthospital</groupId>
        <artifactId>smart-hospital-system</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <!-- Change this name for each module (e.g., patient-service, doctor-service) -->
    <artifactId>patient-service</artifactId>

    <dependencies>
        <!-- Standard web starter inherited cleanly from parent management -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

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

# ============================================
# Security - Gateway isolation secret
# Used by GatewaySecurityFilter (X-Gateway-Token validation)
# ============================================
smart.hospital.gateway.secret=smart-hospital-secure-vault-key

# ============================================
# OAuth2 Resource Server JWT (Temporary Development Override)
# Disable JWT validation check during early standalone service testing
# ============================================
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration

Prompt 3

Act as a Principal QA and Test Automation Engineer. Generate a comprehensive, production-grade JUnit 5 and MockMvc testing suite for the exposure, validation, and business logic layers of our 'Patient Service' module using Spring Boot 3.x, Java 21, Mockito, and AssertJ. All tests must execute quickly and cleanly in isolation without requiring an active PostgreSQL instance.

Please generate the following test files across the appropriate packages under `src/test/java/com/smarthospital/patientservice`:

1. BASE TEST SPECIFICATION (package: controller)
   - `BaseControllerTest` (Abstract Class): 
     * Initialize the core MockMvc context framework using `@WebMvcTest(PatientController.class)`.
     * Define the security tracking property: `protected static final String GATEWAY_SECRET = "smart-hospital-secure-vault-key";`
     * Implement a reusable MockMvc request builder helper (or RequestPostProcessor) that automatically injects the custom HTTP header `X-Gateway-Token` with the value of `GATEWAY_SECRET` into test executions to satisfy our application-level Java security isolation rules.

2. CONTROLLER EXPOSURE & EDGE-CASE VERIFICATION (package: controller)
   - `PatientControllerTest` (Class, extends BaseControllerTest): 
     * Mock the core `PatientService` interface bean using `@MockBean` (or `@MockitoBean` depending on Boot version).
     * Test Case 1 (`registerPatient_Success`): Perform a POST to `/api/v1/patients` with a fully populated, valid `PatientRegistrationDTO` payload (including valid email and nested `EmergencyContact`). Assert HTTP 201 Created and verify the returned JSON fields match the `PatientDTO` layout.
     * Test Case 2 (`registerPatient_ValidationFailed`): Perform a POST to `/api/v1/patients` with a payload violating JSR-303 attributes (e.g., blank firstName, invalid email format like 'plain-text'). Assert HTTP 400 Bad Request, verifying that our `GlobalExceptionHandler` intercepts the execution and lists the specific validation errors.
     * Test Case 3 (`getPatientById_NotFound`): Mock `patientService.getPatientById(99L)` to throw a `ResourceNotFoundException`. Perform a GET request. Assert HTTP 404 Not Found and verify the JSON matches our standard `ErrorResponse` layout.
     * Test Case 4 (`searchPatients_Paginated_Success`): Mock the `patientService.searchPatients(anyString(), any(), any())` endpoint to return a populated Spring Data `Page<PatientDTO>` wrapper. Perform a GET request with query params (`?searchTerm=Tanishq&page=0&size=5`). Assert HTTP 200 OK and ensure the paginated JSON structure renders cleanly.
     * Test Case 5 (`getPatientSummariesByIds_Success`): Test the cross-service aggregation route. Perform a POST to `/api/v1/patients/summary` passing a `List<Long>` array of primitive IDs `[1L, 2L]`. Assert HTTP 200 OK and check that it accurately returns a `List<PatientSummaryDTO>`.
     * Test Case 6 (`accessDenied_MissingGatewayToken`): Perform a GET request to `/api/v1/patients/1` completely omitting the `X-Gateway-Token` header. Assert that our custom Java filter instantly halts execution, short-circuits the pipeline, and returns HTTP 403 Forbidden with the error message: "Direct access to backend microservices is strictly prohibited."

3. CORE BUSINESS SERVICE LOGIC VERIFICATION (package: service)
   - `PatientServiceTest` (Class):
     * Construct unit tests isolating the `PatientServiceImpl` class by mocking its underlying `PatientRepository` and `MedicalProfileRepository` dependencies using standard `@Mock`.
     * Test Case 1 (`registerPatient_ThrowsDuplicateException`): Mock `patientRepository.existsByEmail("duplicate@hospital.com")` to return true. Verify that calling `patientService.registerPatient(...)` cleanly throws a `DuplicateResourceException` and guarantees that `patientRepository.save()` is never triggered.
     * Test Case 2 (`softDeletePatient_Success`): Mock finding a valid active `Patient` entity. Assert that calling the deletion business method smoothly transforms their status state to `PatientStatus.INACTIVE`, saves it, and verify that the modified object is flushed to the persistence layer.
     * Test Case 3 (`softDelete_AlreadyInactive_EdgeCase`): Mock finding a `Patient` entity that is already marked as `PatientStatus.INACTIVE`. Verify that the business logic handles it gracefully (either short-circuiting efficiently without running an unnecessary repository update, or throwing an explicit domain business error).

Generate only compilation-ready, high-coverage clean Java code clearly grouped by file header boundaries. Do not include chat introduction prose or conversational summaries.


Prompt 4:

Act as a Principal Full-Stack Engineer and Automation Architect. Generate a zero-manual-intervention, compilation-ready implementation of the pure HTML/CSS Thymeleaf Web Portal layer for our 'Patient Management System' within the `web-portal` module using Java 21, Spring Cloud OpenFeign, Spring Web, and HTML5/Thymeleaf templates. Strictly prohibit any JavaScript scripts, external JS files, or client-side scripting frameworks. Rely completely on server-side rendering, standard HTTP POST/GET lifecycles, and raw Bootstrap 5 utility classes for layout and visual transitions.

Please generate the following complete code files, explicitly indicating their target file path destinations:

1. TARGET PATH: src/main/java/com/smarthospital/webportal/config/FeignGatewaySecurityInterceptor.java
   - Full Java class implementing `feign.RequestInterceptor`.
   - Read the property `smart.hospital.gateway.secret` using `@Value`.
   - Implement `apply(RequestTemplate template)` to inject the exact HTTP Header: `template.header("X-Gateway-Token", gatewaySecret);` to satisfy downstream microservice security filters.

2. TARGET PATH: src/main/java/com/smarthospital/webportal/client/PatientFeignClient.java
   - Full client interface annotated with `@FeignClient(name = "patient-service", url = "${backend.patient-service.url}")`.
   - Include complete mappings for:
     * GET `/api/v1/patients` matching search params (`String searchTerm`, `Pageable pageable`).
     * GET `/api/v1/patients/{id}` fetching details.
     * POST `/api/v1/patients` sending `PatientRegistrationDTO`.
     * PUT `/api/v1/patients/{id}/medical-profile` sending `MedicalProfileUpdateDTO`.

3. TARGET PATH: src/main/java/com/smarthospital/webportal/controller/PatientWebController.java
   - Full `@Controller` handling requests under `@RequestMapping("/portal/patients")`.
   - Inject `PatientFeignClient`.
   - Map GET `/` to invoke the client, inject paginated data arrays to the Spring UI `Model`, and return `"patients/list"`.
   - Map GET `/register` to supply an empty registration DTO wrapper and return `"patients/register"`.
   - Map POST `/register` to process the form, forward data to the Feign client, and redirect using RedirectAttributes to append a server-side `successMessage` flash attribute back to `/portal/patients`.

4. TARGET PATH: src/main/resources/templates/patients/list.html
   - Provide the complete, production-ready HTML5 template with Bootstrap 5 and Bootstrap Icons integration. Zero `<script>` tags allowed.
   - Implement a functional search form using standard HTML form inputs linking back to `/portal/patients` via GET.
   - Render a data table displaying patient data using `th:each`, with status badge highlights using native Bootstrap alert/badge classes (ACTIVE: success-subtle, INACTIVE: danger-subtle, DISCHARGED: secondary-subtle).
   - Add native HTML/CSS notification alert banners matching `${successMessage}` that display based entirely on server-side compilation (`th:if`).
   - Add an empty database handler checking `th:if="${#lists.isEmpty(patients)}"`.
   - Implement server-side pagination footer navigation using native HTML `<a>` links that pass explicit `page` and `size` parameters back to the URL.

5. TARGET PATH: src/main/resources/templates/patients/register.html
   - Provide the complete HTML5 registration form template using Bootstrap 5 layout grids. Zero Javascript code permitted for form handling or validation submission.
   - Map a form component with `th:action="@{/portal/patients/register}"` and `th:object="${patientRegistrationDTO}"` using method="POST".
   - Include complete structural text inputs for `firstName`, `lastName`, `email`, `phoneNumber`, `dateOfBirth` (using native browser `type="date"` controls), and nested fields for `emergencyContact.contactName`, `emergencyContact.relationship`, and `emergencyContact.contactPhone`.
   - Include a cancel hyperlink (`<a>` tag) returning safely back to `/portal/patients`.

Generate only raw, production-grade, un-truncated file outputs structured by their explicit path directives. Do not skip any line of code or HTML tags, do not use placeholder ellipses (...), and do not include conversational introductions. 

Doctor Service Module

Prompt 1:

Act as a Principal Enterprise Java Architect. Generate a clean, 100% complete, production-ready Spring Boot 3.x Core Architecture layer for our 'Doctor Service' module using Java 21, Spring Data JPA, and Lombok. This module will target a standalone PostgreSQL database environment and must contain full dependencies to support downstream application-level security filters. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following dependencies for our module's `pom.xml` snippet and the complete Java files under `com.smarthospital.doctorservice`:

1. REQUIRED STARTERS (Include inside the output pom dependencies snippet)
   - `spring-boot-starter-data-jpa`
   - `spring-boot-starter-web`
   - `spring-boot-starter-security` (Required for downstream self-defending gateway token filter implementation)
   - `spring-boot-starter-validation` (JSR-303 annotations)
   - `org.postgresql:postgresql` (Runtime scope)

2. ENUMS (package: entity)
   - `DoctorStatus`: ACTIVE, ON_LEAVE, SUSPENDED, RETIRED

3. ENTITIES (package: entity)
   - `Doctor` (Annotate as @Entity, table: "doctors"): fields: id (Long, IDENTITY), firstName (String), lastName (String), email (String, unique=true, nullable=false), phoneNumber (String), specialization (String), licenseNumber (String, unique=true, nullable=false), status (ENUM String, default ACTIVE), joiningDate (LocalDateTime). Include a 1:1 bidirectional mapping to `DoctorAvailability`.
   - `DoctorAvailability` (Annotate as @Entity, table: "doctor_availabilities"): fields: id (Long, IDENTITY), doctor (OneToOne, LAZY, JoinColumn "doctor_id"), shiftStart (String, e.g., "09:00"), shiftEnd (String, e.g., "17:00"), maxPatientsPerDay (int).

4. DATA TRANSFER OBJECTS (package: dto)
   - `DoctorRegistrationDTO`: Fields: firstName, lastName, email, phoneNumber, specialization, licenseNumber, shiftStart, shiftEnd, maxPatientsPerDay.
   - `DoctorUpdateDTO`: Fields: firstName, lastName, phoneNumber, specialization, status.
   - `AvailabilityUpdateDTO`: Fields: shiftStart, shiftEnd, maxPatientsPerDay.
   - `DoctorDTO`: Complete profile matrix view including full ID, availability details, and joining timestamp metrics.
   - `DoctorSummaryDTO`: Lightweight version containing only fields: id (Long), firstName (String), lastName (String), specialization (String), status (DoctorStatus) for rapid inter-service cross-referencing lookups.

5. REPOSITORIES (package: repository)
   - `DoctorRepository`: Extends JpaRepository. Include a query method named `findBySearchCriteria` using @Query to find doctors by a dynamic text search string (matching firstName, lastName, or specialization case-insensitively) and status.
   - `DoctorAvailabilityRepository`: Extends JpaRepository.

6. CUSTOM EXCEPTIONS (package: exception)
   - `ResourceNotFoundException` (Runtime exception taking a String message argument).
   - `DuplicateResourceException` (Runtime exception taking a String message argument).

7. BUSINESS LAYER (package: service & service.impl)
   - `DoctorService` (Interface): Declare methods for registerDoctor, getDoctorById, updateDoctor, updateAvailability, searchDoctors, softDeleteDoctor, and getDoctorSummariesByIds.
   - `DoctorServiceImpl` (Class implementing DoctorService): Implement full business logic. Enforce strict database rule validation:
     * Check email and licenseNumber uniqueness in the PostgreSQL database before executing registrations, throwing DuplicateResourceException on collisions.
     * Soft delete doctors by shifting the status string to SUSPENDED or RETIRED instead of hard database row removal.
     * Implement the bulk summary retrieval mapping method taking a List<Long> of IDs and returning a List<DoctorSummaryDTO>.

Generate only compilation-ready, 100% complete Java code files clearly grouped by explicit target path indicators. Do not skip any line of logic, do not use placeholders, and do not include conversational introduction summaries.

Prompt 2:

Act as a Principal Enterprise Java Security Architect. Generate a 100% complete, production-ready REST Exposure and Security Isolation Layer for our 'Doctor Service' module using Spring Boot 3.x, Spring Security, Java 21, and Lombok. This layer must enforce strict application-level perimeter defense by validating gateway security tokens. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following complete Java files under `com.smarthospital.doctorservice`, explicitly indicating their target file path destinations:

1. TARGET PATH: src/main/java/com/smarthospital/doctorservice/security/GatewaySecurityFilter.java
   - Full Java class extending `OncePerRequestFilter`.
   - Read an internal environment property named `smart.hospital.gateway.secret` using the `@Value` annotation.
   - Override the `doFilterInternal` method to intercept all incoming HTTP requests.
   - Extract the HTTP request header: `X-Gateway-Token`.
   - If the header is missing or does not exactly match the value of the internal gateway secret string, immediately abort execution, short-circuit the request chain, set the HTTP response status to 403 Forbidden, set the Content-Type to "application/json", and write the exact JSON error payload:
     `{"error": "Forbidden", "message": "Direct access to backend microservices is strictly prohibited."}`
   - If the token matches perfectly, execute `filterChain.doFilter(request, response)` to allow the request to proceed smoothly.

2. TARGET PATH: src/main/java/com/smarthospital/doctorservice/security/SecurityConfig.java
   - Full configuration class annotated with `@Configuration` and `@EnableWebSecurity`.
   - Inject the custom `GatewaySecurityFilter` bean.
   - Define a `SecurityFilterChain` bean that disables CSRF, configures session management to `SessionCreationPolicy.STATELESS`, and permits all HTTP traffic to flow through Spring Security's filter chain so our custom `GatewaySecurityFilter` can handle the explicit validation logic. Register `GatewaySecurityFilter` to run before `UsernamePasswordAuthenticationFilter`.

3. TARGET PATH: src/main/java/com/smarthospital/doctorservice/controller/DoctorController.java
   - Full `@RestController` handling requests under `@RequestMapping("/api/v1/doctors")`.
   - Inject `DoctorService`.
   - Implement the following zero-placeholder endpoints with proper JSR-303 `@Valid` annotations:
     * POST `/`: Onboard a new doctor. Takes `DoctorRegistrationDTO`. Returns HTTP 201 Created and the compiled `DoctorDTO`.
     * GET `/{id}`: Fetch a doctor profile. Returns HTTP 200 OK and the matching `DoctorDTO`.
     * PUT `/{id}`: Update demographics. Takes `DoctorUpdateDTO`. Returns HTTP 200 OK and the updated `DoctorDTO`.
     * PUT `/{id}/availability`: Update scheduling limits. Takes `AvailabilityUpdateDTO`. Returns HTTP 200 OK.
     * GET `/`: Paginated text query search engine. Accepts query parameters: `searchTerm`, `status`, and `Pageable`. Returns HTTP 200 OK wrapping a Spring Data `Page<DoctorDTO>`.
     * POST `/summary`: Cross-service aggregate lookup route. Accepts a `List<Long>` body containing doctor IDs. Returns HTTP 200 OK containing a `List<DoctorSummaryDTO>`.
     * DELETE `/{id}`: Soft delete trigger. Triggers the service layer to toggle status state. Returns HTTP 204 No Content.

4. TARGET PATH: src/main/java/com/smarthospital/doctorservice/exception/GlobalExceptionHandler.java
   - Full `@RestControllerAdvice` class containing explicit `@ExceptionHandler` methods.
   - Map `ResourceNotFoundException` to return HTTP 404 Not Found with a standardized JSON payload structure: `{"status": 404, "error": "Not Found", "message": "..."}`.
   - Map `DuplicateResourceException` to return HTTP 409 Conflict with a standardized JSON payload structure: `{"status": 409, "error": "Conflict", "message": "..."}`.
   - Map `MethodArgumentNotValidException` to handle JSR-303 field errors. Loop through all errors and return HTTP 400 Bad Request with a clear validation map details field list.

Generate only raw, compilation-ready, 100% complete Java code strings grouped cleanly by their explicit path indicators. Do not skip any line of code or structural brackets, and do not include conversational responses.

Prompt 3:

Act as a Principal QA and Test Automation Engineer. Generate a comprehensive, production-grade JUnit 5 and MockMvc testing suite for the exposure, validation, and business logic layers of our 'Doctor Service' module using Spring Boot 3.x, Java 21, Mockito, and AssertJ. All tests must execute quickly and cleanly in isolation without requiring an active PostgreSQL instance. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following test files across the appropriate packages under `src/test/java/com/smarthospital/doctorservice`:

1. BASE TEST SPECIFICATION (package: controller)
   - `BaseControllerTest` (Abstract Class): 
     * Initialize the core MockMvc context framework using `@WebMvcTest(DoctorController.class)`.
     * Define the security tracking property: `protected static final String GATEWAY_SECRET = "smart-hospital-secure-vault-key";`
     * Implement a reusable MockMvc request builder helper (or RequestPostProcessor) that automatically injects the custom HTTP header `X-Gateway-Token` with the value of `GATEWAY_SECRET` into test executions to satisfy our application-level Java security isolation rules.

2. CONTROLLER EXPOSURE & EDGE-CASE VERIFICATION (package: controller)
   - `DoctorControllerTest` (Class, extends BaseControllerTest): 
     * Mock the core `DoctorService` interface bean using `@MockBean` (or `@MockitoBean` depending on Boot version).
     * Test Case 1 (`registerDoctor_Success`): Perform a POST to `/api/v1/doctors` with a fully populated, valid `DoctorRegistrationDTO` payload (including valid email, specialization, and nested availability metrics). Assert HTTP 201 Created and verify the returned JSON fields match the `DoctorDTO` layout.
     * Test Case 2 (`registerDoctor_ValidationFailed`): Perform a POST to `/api/v1/doctors` with a payload violating JSR-303 attributes (e.g., blank firstName, invalid email format, or empty licenseNumber). Assert HTTP 400 Bad Request, verifying that our `GlobalExceptionHandler` intercepts the execution and lists the specific validation errors.
     * Test Case 3 (`getDoctorById_NotFound`): Mock `doctorService.getDoctorById(99L)` to throw a `ResourceNotFoundException`. Perform a GET request. Assert HTTP 404 Not Found and verify the JSON matches our standard `ErrorResponse` layout.
     * Test Case 4 (`searchDoctors_Paginated_Success`): Mock the `doctorService.searchDoctors(anyString(), any(), any())` endpoint to return a populated Spring Data `Page<DoctorDTO>` wrapper. Perform a GET request with query params (`?searchTerm=General&status=ACTIVE&page=0&size=5`). Assert HTTP 200 OK and ensure the paginated JSON structure renders cleanly.
     * Test Case 5 (`getDoctorSummariesByIds_Success`): Test the cross-service aggregation route. Perform a POST to `/api/v1/doctors/summary` passing a `List<Long>` array of primitive IDs `[1L, 2L]`. Assert HTTP 200 OK and check that it accurately returns a `List<DoctorSummaryDTO>`.
     * Test Case 6 (`accessDenied_MissingGatewayToken`): Perform a GET request to `/api/v1/doctors/1` completely omitting the `X-Gateway-Token` header. Assert that our custom Java filter instantly halts execution, short-circuits the pipeline, and returns HTTP 403 Forbidden with the error message: "Direct access to backend microservices is strictly prohibited."

3. CORE BUSINESS SERVICE LOGIC VERIFICATION (package: service)
   - `DoctorServiceTest` (Class):
     * Construct unit tests isolating the `DoctorServiceImpl` class by mocking its underlying `DoctorRepository` and `DoctorAvailabilityRepository` dependencies using standard `@Mock`.
     * Test Case 1 (`registerDoctor_ThrowsDuplicateException`): Mock `doctorRepository.existsByEmail("duplicate@hospital.com")` to return true. Verify that calling `doctorService.registerDoctor(...)` cleanly throws a `DuplicateResourceException` and guarantees that `doctorRepository.save()` is never triggered.
     * Test Case 2 (`softDeleteDoctor_Success`): Mock finding a valid active `Doctor` entity. Assert that calling the deletion business method smoothly transforms their status state to `DoctorStatus.SUSPENDED`, saves it, and verify that the modified object is flushed to the persistence layer.
     * Test Case 3 (`softDelete_AlreadySuspended_EdgeCase`): Mock finding a `Doctor` entity that is already marked as `DoctorStatus.SUSPENDED`. Verify that the business logic handles it gracefully (either short-circuiting efficiently without running an unnecessary repository update, or throwing an explicit domain business error).

Generate only raw, compilation-ready, high-coverage clean Java code clearly grouped by file header boundaries. Do not skip any line of code, do not use placeholder ellipses (...), and do not include conversational introductions.

Prompt 4:


Act as a Principal Full-Stack Engineer and Automation Architect. Generate a zero-manual-intervention, compilation-ready implementation of the pure HTML/CSS Thymeleaf Web Portal layer for our 'Doctor Management System' within the `web-portal` module using Java 21, Spring Cloud OpenFeign, Spring Web, and HTML5/Thymeleaf templates. Strictly prohibit any JavaScript scripts, external JS files, or client-side scripting frameworks. Rely completely on server-side rendering, standard HTTP POST/GET lifecycles, and raw Bootstrap 5 utility classes for layout and visual transitions.

Please generate the following complete code files, explicitly indicating their target file path destinations:

1. TARGET PATH: src/main/java/com/smarthospital/webportal/client/DoctorFeignClient.java
   - Full client interface annotated with `@FeignClient(name = "doctor-service", url = "${backend.doctor-service.url}")`.
   - The global `FeignGatewaySecurityInterceptor` already handles header injection, so provide straight client mappings for:
     * GET `/api/v1/doctors` matching search params (`String searchTerm`, `Pageable pageable`).
     * GET `/api/v1/doctors/{id}` fetching details.
     * POST `/api/v1/doctors` sending `DoctorRegistrationDTO`.
     * PUT `/api/v1/doctors/{id}` sending `DoctorUpdateDTO`.
     * DELETE `/api/v1/doctors/{id}` passing soft delete requests.

2. TARGET PATH: src/main/java/com/smarthospital/webportal/controller/DoctorWebController.java
   - Full `@Controller` handling web requests under `@RequestMapping("/portal/doctors")`.
   - Inject `DoctorFeignClient`.
   - Map GET `/` to invoke the client, inject paginated doctor data arrays to the Spring UI `Model`, and return the template view name `"doctors/list"`.
   - Map GET `/register` to supply an empty registration DTO wrapper to the model and return view `"doctors/register"`.
   - Map POST `/register` to process the form submission, forward data to the Feign client, and redirect using `RedirectAttributes` to append a server-side `successMessage` flash attribute back to `/portal/doctors`.
   - Map POST `/{id}/delete` to trigger the backend soft-delete lifecycle via Feign and redirect back to the main list with a success message banner.

3. TARGET PATH: src/main/resources/templates/doctors/list.html
   - Provide the complete, production-ready HTML5 template with Bootstrap 5 and Bootstrap Icons integration. Zero `<script>` tags allowed.
   - Core Navigation Panel: Link the 'Patients' button to the working `/portal/patients` route. For the rest of the layout navigation (Appointments, Billing, etc.), explicitly implement safe dead links using `th:href="@{#}"` annotated as `disabled` to prevent compilation failures while keeping the frontend structure completely un-truncated.
   - Implement a functional search form using standard HTML form inputs linking back to `/portal/doctors` via GET.
   - Render a data table displaying medical provider profiles using `th:each`, with status badge highlights using native Bootstrap alert/badge classes (ACTIVE: success-subtle, ON_LEAVE: warning-subtle, SUSPENDED: danger-subtle, RETIRED: secondary-subtle).
   - Add native HTML/CSS notification alert banners matching `${successMessage}` that display based entirely on server-side compilation (`th:if`).
   - Add an empty database handler checking `th:if="${#lists.isEmpty(doctors)}"`.
   - Implement server-side pagination footer navigation using native HTML `<a>` links that pass explicit `page` and `size` parameters back to the URL.

4. TARGET PATH: src/main/resources/templates/doctors/register.html
   - Provide the complete HTML5 registration form template using Bootstrap 5 layout grids. Zero Javascript code permitted for form handling or validation submission.
   - Map a form component with `th:action="@{/portal/doctors/register}"` and `th:object="${doctorRegistrationDTO}"` using method="POST".
   - Include complete structural text inputs for `firstName`, `lastName`, `email`, `phoneNumber`, `specialization`, `licenseNumber`, `shiftStart` (type="time"), `shiftEnd` (type="time"), and `maxPatientsPerDay` (type="number").
   - Include a cancel hyperlink (`<a>` tag) returning safely back to `/portal/doctors`.

Generate only raw, production-grade, un-truncated file outputs structured by their explicit path directives. Do not skip any line of code or HTML tags, do not use placeholder ellipses (...), and do not include conversational introductions.

Appointment Service Module

Prompt 1:

Act as a Principal Enterprise Java Architect. Generate a clean, 100% complete, production-ready Spring Boot 3.x Core Architecture and Configuration layer for our 'Appointment Service' module using Java 21, Spring Data JPA, and Lombok. This module will target a standalone PostgreSQL database environment and must contain full configurations to support downstream application-level security filters. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following dependencies for our module's `pom.xml` snippet, the target properties file, and the complete Java files under `com.smarthospital.appointmentservice`:

1. REQUIRED STARTERS (Include inside the output pom dependencies snippet)
   - `spring-boot-starter-data-jpa`
   - `spring-boot-starter-web`
   - `spring-boot-starter-security` (Required for downstream self-defending gateway token filter implementation)
   - `spring-boot-starter-validation` (JSR-303 annotations)
   - `org.postgresql:postgresql` (Runtime scope)

2. CONFIGURATION MIRRORING (TARGET PATH: src/main/resources/application.properties)
   - Generate the complete application properties file for this service. 
   - Mirror the exact structure used in previous services: include the `spring.autoconfigure.exclude` string for `OAuth2ResourceServerAutoConfiguration` to allow standalone testing, and include the exact `smart.hospital.gateway.secret=smart-hospital-secure-vault-key` property mapping.
   - Configure the server port to `8083`.
   - Set up the Spring Data JPA connection string targeting PostgreSQL, explicitly changing only the database name segment to `smart_hospital_appointments`.

3. ENUMS (package: entity)
   - `AppointmentStatus`: SCHEDULED, FULFILLED, CANCELLED, NO_SHOW

4. ENTITIES (package: entity)
   - `Appointment` (Annotate as @Entity, table: "appointments"): fields: id (Long, IDENTITY), patientId (Long, for cross-service tracking), doctorId (Long, for cross-service tracking), appointmentDateTime (LocalDateTime), reasonForVisit (String), status (ENUM String, default SCHEDULED), notes (String), createdAt (LocalDateTime).

5. DATA TRANSFER OBJECTS (package: dto)
   - `AppointmentSchedulingDTO`: Fields: patientId, doctorId, appointmentDateTime, reasonForVisit, notes.
   - `AppointmentStatusUpdateDTO`: Fields: status, notes.
   - `AppointmentDTO`: Complete execution matrix view including full ID, patientId, doctorId, datetime, reason, status, notes, and creation timestamp metrics.

6. REPOSITORIES (package: repository)
   - `AppointmentRepository`: Extends JpaRepository. Include query methods:
     * `findByPatientId`: Find all records for a specific patient.
     * `findByDoctorIdAndAppointmentDateTimeBetween`: Find all appointments for a doctor within a specific date range window.

7. CUSTOM EXCEPTIONS (package: exception)
   - `ResourceNotFoundException` (Runtime exception taking a String message argument).
   - `AppointmentConflictException` (Runtime exception taking a String message argument for double-booking validations).

8. BUSINESS LAYER (package: service & service.impl)
   - `AppointmentService` (Interface): Declare methods for scheduleAppointment, getAppointmentById, updateAppointmentStatus, getAppointmentsByPatient, getAppointmentsByDoctorAndDate.
   - `AppointmentServiceImpl` (Class implementing AppointmentService): Implement full business logic. Enforce strict database rule validation:
     * Check if the doctor already has an existing appointment matching the exact same `appointmentDateTime` timestamp, throwing an `AppointmentConflictException` if a scheduling overlap occurs.
     * Implement soft cancellation by shifting the status string to CANCELLED via the status update method instead of hard database row removal.

Generate only compilation-ready, 100% complete Java and properties file outputs clearly grouped by explicit target path indicators. Do not skip any line of logic, do not use placeholders, and do not include conversational introduction summaries.

Prompt 2:

Act as a Principal Enterprise Java Security Architect. Generate a 100% complete, production-ready REST Exposure and Security Isolation Layer for our 'Appointment Service' module using Spring Boot 3.x, Spring Security, Java 21, and Lombok. This layer must enforce strict application-level perimeter defense by validating gateway security tokens. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following complete Java files under `com.smarthospital.appointmentservice`, explicitly indicating their target file path destinations:

1. TARGET PATH: src/main/java/com/smarthospital/appointmentservice/security/GatewaySecurityFilter.java
   - Full Java class extending `OncePerRequestFilter`.
   - Read an internal environment property named `smart.hospital.gateway.secret` using the `@Value` annotation.
   - Override the `doFilterInternal` method to intercept all incoming HTTP requests.
   - Extract the HTTP request header: `X-Gateway-Token`.
   - If the header is missing or does not exactly match the value of the internal gateway secret string, immediately abort execution, short-circuit the request chain, set the HTTP response status to 403 Forbidden, set the Content-Type to "application/json", and write the exact JSON error payload:
     `{"error": "Forbidden", "message": "Direct access to backend microservices is strictly prohibited."}`
   - If the token matches perfectly, execute `filterChain.doFilter(request, response)` to allow the request to proceed smoothly.

2. TARGET PATH: src/main/java/com/smarthospital/appointmentservice/security/SecurityConfig.java
   - Full configuration class annotated with `@Configuration` and `@EnableWebSecurity`.
   - Inject the custom `GatewaySecurityFilter` bean.
   - Define a `SecurityFilterChain` bean that disables CSRF, configures session management to `SessionCreationPolicy.STATELESS`, and permits all HTTP traffic to flow through Spring Security's filter chain so our custom `GatewaySecurityFilter` can handle the explicit validation logic. Register `GatewaySecurityFilter` to run before `UsernamePasswordAuthenticationFilter`.

3. TARGET PATH: src/main/java/com/smarthospital/appointmentservice/controller/AppointmentController.java
   - Full `@RestController` handling requests under `@RequestMapping("/api/v1/appointments")`.
   - Inject `AppointmentService`.
   - Implement the following zero-placeholder endpoints with proper JSR-303 `@Valid` annotations:
     * POST `/`: Schedule a new appointment slot. Takes `AppointmentSchedulingDTO`. Returns HTTP 201 Created and the compiled `AppointmentDTO`.
     * GET `/{id}`: Fetch individual appointment data records. Returns HTTP 200 OK and the matching `AppointmentDTO`.
     * PUT `/{id}/status`: Alter appointment lifecycle state (FULFILLED, CANCELLED, NO_SHOW). Takes `AppointmentStatusUpdateDTO`. Returns HTTP 200 OK and the updated `AppointmentDTO`.
     * GET `/patient/{patientId}`: Retrieve all historical appointment records for a specific patient. Returns HTTP 200 OK containing a `List<AppointmentDTO>`.
     * GET `/doctor/{doctorId}`: Retrieve schedule metrics for a provider within a specific window. Accepts query params: `start` (String) and `end` (String). Returns HTTP 200 OK containing a `List<AppointmentDTO>`.

4. TARGET PATH: src/main/java/com/smarthospital/appointmentservice/exception/GlobalExceptionHandler.java
   - Full `@RestControllerAdvice` class containing explicit `@ExceptionHandler` methods.
   - Map `ResourceNotFoundException` to return HTTP 404 Not Found with a standardized JSON payload structure: `{"status": 404, "error": "Not Found", "message": "..."}`.
   - Map `AppointmentConflictException` to return HTTP 409 Conflict with a standardized JSON payload structure: `{"status": 409, "error": "Conflict", "message": "..."}`.
   - Map `MethodArgumentNotValidException` to handle JSR-303 field errors. Loop through all validation errors and return HTTP 400 Bad Request with a clear validation map details field list.

Generate only raw, compilation-ready, 100% complete Java code strings grouped cleanly by their explicit path indicators. Do not skip any line of code or structural brackets, and do not include conversational responses.

Prompt 3:

Act as a Principal QA and Test Automation Engineer. Generate a comprehensive, production-grade JUnit 5 and MockMvc testing suite for the exposure, validation, and business logic layers of our 'Appointment Service' module using Spring Boot 3.x, Java 21, Mockito, and AssertJ. All tests must execute quickly and cleanly in isolation without requiring an active PostgreSQL instance. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following test files across the appropriate packages under `src/test/java/com/smarthospital/appointmentservice`:

1. BASE TEST SPECIFICATION (package: controller)
   - `BaseControllerTest` (Abstract Class): 
     * Initialize the core MockMvc context framework using `@WebMvcTest(AppointmentController.class)`.
     * Define the security tracking property: `protected static final String GATEWAY_SECRET = "smart-hospital-secure-vault-key";`
     * Implement a reusable MockMvc request builder helper (or RequestPostProcessor) that automatically injects the custom HTTP header `X-Gateway-Token` with the value of `GATEWAY_SECRET` into test executions to satisfy our application-level Java security isolation rules.

2. CONTROLLER EXPOSURE & EDGE-CASE VERIFICATION (package: controller)
   - `AppointmentControllerTest` (Class, extends BaseControllerTest): 
     * Mock the core `AppointmentService` interface bean using `@MockBean` (or `@MockitoBean` depending on Boot version).
     * Test Case 1 (`scheduleAppointment_Success`): Perform a POST to `/api/v1/appointments` with a fully populated, valid `AppointmentSchedulingDTO` payload. Assert HTTP 201 Created and verify the returned JSON fields match the `AppointmentDTO` layout.
     * Test Case 2 (`scheduleAppointment_ValidationFailed`): Perform a POST to `/api/v1/appointments` with a payload violating JSR-303 attributes (e.g., null patientId, null doctorId, or missing appointmentDateTime). Assert HTTP 400 Bad Request, verifying that our `GlobalExceptionHandler` intercepts the execution and lists the specific validation errors.
     * Test Case 3 (`getAppointmentById_NotFound`): Mock `appointmentService.getAppointmentById(99L)` to throw a `ResourceNotFoundException`. Perform a GET request. Assert HTTP 404 Not Found and verify the JSON matches our standard `ErrorResponse` layout.
     * Test Case 4 (`getAppointmentsByPatient_Success`): Mock the `appointmentService.getAppointmentsByPatient(1L)` endpoint to return a populated array of historical records. Perform a GET request to `/api/v1/appointments/patient/1`. Assert HTTP 200 OK and check that the array elements map correctly.
     * Test Case 5 (`updateStatus_Success`): Perform a PUT to `/api/v1/appointments/1/status` with a valid `AppointmentStatusUpdateDTO` shifting state to CANCELLED. Assert HTTP 200 OK.
     * Test Case 6 (`accessDenied_MissingGatewayToken`): Perform a GET request to `/api/v1/appointments/1` completely omitting the `X-Gateway-Token` header. Assert that our custom Java filter instantly halts execution, short-circuits the pipeline, and returns HTTP 403 Forbidden with the error message: "Direct access to backend microservices is strictly prohibited."

3. CORE BUSINESS SERVICE LOGIC VERIFICATION (package: service)
   - `AppointmentServiceTest` (Class):
     * Construct unit tests isolating the `AppointmentServiceImpl` class by mocking its underlying `AppointmentRepository` dependency using standard `@Mock`.
     * Test Case 1 (`scheduleAppointment_ThrowsConflictException`): Mock the repository finding a conflicting slot via `findByDoctorIdAndAppointmentDateTimeBetween` to return an existing appointment. Verify that calling `appointmentService.scheduleAppointment(...)` cleanly throws an `AppointmentConflictException` and guarantees that `appointmentRepository.save()` is never triggered.
     * Test Case 2 (`updateStatus_SoftCancellation_Success`): Mock finding a valid active `Appointment` entity. Assert that calling the modification service method smoothly transforms their status state to `AppointmentStatus.CANCELLED`, appends the execution notes, saves it, and verify that the modified object is flushed to the persistence layer.
     * Test Case 3 (`getAppointmentById_ThrowsNotFound`): Mock finding a non-existent ID to return Optional.empty(). Verify that calling the service layer throws `ResourceNotFoundException`.

Generate only raw, compilation-ready, high-coverage clean Java code clearly grouped by file header boundaries. Do not skip any line of code, do not use placeholder ellipses (...), and do not include conversational introductions.




