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
