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

Act as a Principal Full-Stack Java Developer. Generate the Thymeleaf Web Portal exposure layer for our 'Patient Management Dashboard' inside the `web-portal` module using Java 21, Spring Web, Spring Cloud OpenFeign, and Thymeleaf templates. This layer must seamlessly communicate with our self-defending backend 'Patient Service'.

Please generate the following files across the appropriate packages under `com.smarthospital.webportal`:

1. SECURITY ORCHESTRATION & FEIGN INTERCEPTOR (package: config)
   - `FeignGatewaySecurityInterceptor` (Class, implements feign.RequestInterceptor):
     * Read a configured environment property: `smart.hospital.gateway.secret`.
     * Override the `apply(RequestTemplate template)` method to automatically inject the HTTP Header `X-Gateway-Token` with the secret value into every outbound Feign client call. This ensures our frontend requests safely pass the backend service's Java firewall checks without modifying client interfaces.

2. FEIGN CLIENT INTERFACE (package: client)
   - `PatientFeignClient` (Interface, annotated with @FeignClient(name = "patient-service", url = "${backend.patient-service.url}")):
     * Map the required backend endpoints:
       - GET `/api/v1/patients` (with search and pagination parameters)
       - GET `/api/v1/patients/{id}`
       - POST `/api/v1/patients` (takes PatientRegistrationDTO)
       - PUT `/api/v1/patients/{id}/medical-profile` (takes MedicalProfileUpdateDTO)

3. WEB FRONTEND CONTROLLER (package: controller)
   - `PatientWebController` (Class, annotated with @Controller, @RequestMapping("/portal/patients")): Inject `PatientFeignClient`.
     * **GET `/`**: Call the Feign client to fetch a paginated list of patients based on an optional search query. Add the results to the Spring UI `Model` and return the view name `"patients/list"`.
     * **GET `/register`**: Add an empty `PatientRegistrationDTO` to the model and return view `"patients/register"`.
     * **POST `/register`**: Accept form submission, trigger the Feign client `POST`, and redirect to `/portal/patients` with a success flash attribute.
     * **GET `/{id}/medical-profile`**: Fetch details for a specific patient, map them to the model, and return view `"patients/medical-profile"`.

4. THYMELEAF TEMPLATE (Path: src/main/resources/templates/patients/list.html)
   - Generate a clean, responsive HTML5 layout using Bootstrap 5 styling. Include a functional search bar targeting the search endpoint, a dynamic text data table displaying patient records via `th:each`, status badge styles matching our ENUMs, and pagination control links ("Next" / "Previous").

Generate only compilation-ready, high-quality code structured cleanly by file boundaries. Do not include chat introduction prose or conversational summaries.

Also add the following in the application.properties file:
smart.hospital.gateway.secret=smart-hospital-secure-vault-key
backend.patient-service.url=http://localhost:8081
and verify the application.properties file of patient service to see if they both have thesame hardcoded key 

Doctor Service Module

Prompt 1:

