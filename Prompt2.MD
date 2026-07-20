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