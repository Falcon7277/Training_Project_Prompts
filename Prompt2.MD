Spring AI Module

Prompt 1:

Act as a Principal Spring AI & Enterprise Java Architect. Generate a 100% complete, compilation-ready NLP Document Parsing Engine for our 'Web Portal' module (`web-portal`) under the package `com.smarthospital.webportal.ai`. This service uses Spring AI 1.0.0-M6, the ChatClient Structured Output API, and the PDF Document Reader to extract clinical parameters from unstructured PDF reports and return a structured Java Record. Do not use code omissions, placeholders, or ellipses (...).

Please generate the following complete Java files, explicitly indicating their target path destinations:

1. TARGET PATH: src/main/java/com/smarthospital/webportal/ai/dto/ExtractedMedicalProfileRecord.java
   - Full Java `record` definition mapping clinical entity fields.
   - Include strictly the 4 medical profile parameters:
     * `String bloodGroup`
     * `String allergies`
     * `String chronicConditions`
     * `String currentMedications`

2. TARGET PATH: src/main/java/com/smarthospital/webportal/ai/service/ClinicalDocumentParserService.java
   - Full Java service class annotated with `@Service`.
   - Inject `org.springframework.ai.chat.client.ChatClient.Builder` via constructor dependency injection and build a default `ChatClient` instance.
   - Define a public method `public ExtractedMedicalProfileRecord parseClinicalPdf(org.springframework.web.multipart.MultipartFile file)`:
     * Write or convert the incoming `MultipartFile` byte contents into a temporary file/resource context.
     * Use `org.springframework.ai.reader.pdf.PagePdfDocumentReader` to load and extract text content from the PDF pages.
     * Aggregate the document text into a single String context.
     * Construct a system and user prompt instructing the AI model to act as a certified Medical Data Extractor: "Analyze the provided unstructured clinical document text and extract the patient's medical profile parameters: blood group, allergies, chronic conditions, and current medications. If any field is not explicitly mentioned in the text, set its value to 'None' or 'Not Specified'."
     * Invoke the Spring AI `ChatClient` fluent pipeline passing the extracted document text:
       `chatClient.prompt().user(promptText).call().entity(ExtractedMedicalProfileRecord.class)`
     * Clean up any temporary file resources created during PDF ingestion.
     * Handle exceptions gracefully by returning a fallback `ExtractedMedicalProfileRecord` with default "Not Specified" values if parsing fails.

Generate only raw, compilation-ready, 100% complete Java code strings grouped cleanly by their explicit path indicators. Do not skip any imports, constructor parameters, or try-catch cleanup blocks, and do not include conversational introductions.

Prompt 2

Act as a Principal Enterprise Java Architect. Generate a 100% complete, compilation-ready Spring MVC Web Controller for the 'Web Portal' module (`web-portal`) under the package `com.smarthospital.webportal.ai.controller`. This controller processes clinical document PDF uploads, triggers the Spring AI document parsing pipeline, maps extracted clinical parameters to the patient's medical profile, and updates the backend 'patient-service' via OpenFeign. Do not use code omissions, placeholders, or ellipses (...).

Please generate the complete source code for the following Java file, explicitly indicating its target path destination:

TARGET PATH: src/main/java/com/smarthospital/webportal/ai/controller/ClinicalDocumentUploadController.java
- Full Java controller class annotated with `@Controller` and `@RequestMapping("/portal/patients")`.
- Inject the following components via constructor dependency injection:
  * `com.smarthospital.webportal.ai.service.ClinicalDocumentParserService`
  * `com.smarthospital.webportal.client.PatientFeignClient`
- Map a POST request handler to `/{patientId}/upload-clinical-doc`:
  * Method parameters: `@PathVariable("patientId") Long patientId`, `@RequestParam("file") org.springframework.web.multipart.MultipartFile file`, and `org.springframework.web.servlet.mvc.support.RedirectAttributes redirectAttributes`.
  * Validate that the uploaded file is not empty and has a `.pdf` extension/content type. If invalid, add a flash error attribute ("error", "Please upload a valid PDF document.") and redirect back to `redirect:/portal/patients/my-profile/` + patientId (or `redirect:/portal/patients/view/` + patientId depending on caller).
  * Invoke `clinicalDocumentParserService.parseClinicalPdf(file)` to receive the `ExtractedMedicalProfileRecord`.
  * Map the extracted fields (`bloodGroup`, `allergies`, `chronicConditions`, `currentMedications`) into the medical profile update transfer object expected by `PatientFeignClient`.
  * Call `patientFeignClient` to update the patient's medical profile for the given `patientId`.
  * Add a success flash attribute ("message", "Medical profile successfully updated from uploaded document.") and redirect cleanly to `redirect:/portal/patients/my-profile/` + patientId.
  * Wrap in a robust try-catch block: catch any exception during extraction or Feign communication, set a descriptive flash error attribute ("error", "Failed to parse document or update medical profile: " + e.getMessage()), and redirect safely back to the patient profile page.

Generate only raw, compilation-ready, 100% complete Java code. Do not skip any imports, class annotations, or error handling blocks, and do not include conversational introductions.
