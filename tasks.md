# Implementation Plan: Maatri Maternal Health Assistant

## Overview

This implementation plan breaks down the Maatri system into discrete, incremental coding tasks using Python as the implementation language. The system will be built using AWS services (Lambda, OpenSearch, Bedrock, DynamoDB, S3) with a focus on modularity, testability, and safety.

The implementation follows a bottom-up approach: core data models → individual components → integration → testing. Each task builds on previous work, with checkpoints to ensure stability before proceeding.

## Technology Stack

- **Language**: Python 3.11+
- **AWS SDK**: boto3
- **Testing**: pytest, hypothesis (property-based testing)
- **API Framework**: FastAPI (for local development/testing)
- **Vector Search**: opensearch-py
- **Type Checking**: mypy with strict mode
- **Code Quality**: black, ruff, pylint

## Tasks

- [ ] 1. Set up project structure and core infrastructure
  - Create Python project with poetry/pip for dependency management
  - Set up directory structure: `src/`, `tests/`, `infrastructure/`, `docs/`
  - Configure pytest with hypothesis for property-based testing
  - Create base configuration for AWS services (boto3 clients)
  - Set up type checking with mypy
  - Create `.env.example` for environment variables
  - _Requirements: All (foundational)_

- [ ] 2. Implement core data models
  - [ ] 2.1 Create Pydantic models for all data structures
    - Define `QueryRequest`, `QueryResponse`, `Source`, `SafetyAlert`
    - Define `UserProfile`, `Passage`, `RankedPassage`
    - Define `FAQ`, `Document`, `DocumentChunk`
    - Add validation rules and type hints
    - _Requirements: 1.1, 2.6, 7.1, 8.1_
  
  - [ ]* 2.2 Write property test for data model validation
    - **Property: Data Model Round-Trip**
    - Test that all models can be serialized to JSON and deserialized back to equivalent objects
    - **Validates: Requirements 7.2, 7.3** (profile storage)
  
  - [ ]* 2.3 Write unit tests for data model edge cases
    - Test empty strings, None values, invalid enums
    - Test boundary conditions (e.g., risk score 0.0 and 1.0)
    - _Requirements: 1.1, 2.6, 5.1_

- [ ] 3. Implement Language Router component
  - [ ] 3.1 Create language detection module
    - Implement character set analysis (Devanagari vs Latin script)
    - Integrate with boto3 for Bedrock language classification
    - Create `detect_language(text: str) -> str` function
    - Handle mixed-language input (default to English)
    - _Requirements: 1.3, 1.5_
  
  - [ ]* 3.2 Write property test for language detection
    - **Property 1: Language Consistency**
    - Test that Hindi text is detected as 'hindi' and English text as 'english'
    - **Validates: Requirements 1.3**
  
  - [ ] 3.3 Create language routing logic
    - Implement `route_query(request: QueryRequest) -> str` function
    - Return 'hindi_rag' or 'english_rag' based on detection
    - Handle explicit language specification in request
    - _Requirements: 1.3, 1.4_
  
  - [ ]* 3.4 Write property test for language routing
    - **Property 2: Language Switching Detection**
    - Test that conversation sequences with language switches route correctly
    - **Validates: Requirements 1.5**

- [ ] 4. Implement User Profile management
  - [ ] 4.1 Create DynamoDB user profile repository
    - Implement `UserProfileRepository` class with boto3 DynamoDB client
    - Create methods: `create_profile()`, `get_profile()`, `update_profile()`, `delete_profile()`
    - Handle profile creation for new users
    - Implement caching with 5-minute TTL
    - _Requirements: 7.1, 7.2, 7.3, 7.4_
  
  - [ ]* 4.2 Write property test for profile round-trip
    - **Property 16: User Profile Round-Trip**
    - Test that any profile data stored can be retrieved with identical values
    - **Validates: Requirements 7.2, 7.3**
  
  - [ ]* 4.3 Write property test for profile creation
    - **Property 15: User Profile Creation**
    - Test that new userIds automatically get profiles created
    - **Validates: Requirements 7.1**
  
  - [ ]* 4.4 Write property test for profile retrieval
    - **Property 17: Profile Retrieval for Returning Users**
    - Test that existing users have profiles loaded on subsequent interactions
    - **Validates: Requirements 7.4**
  
  - [ ]* 4.5 Write unit tests for profile edge cases
    - Test profile creation with missing optional fields
    - Test concurrent profile updates
    - Test profile deletion
    - _Requirements: 7.1, 7.2, 7.3, 12.4_

- [ ] 5. Checkpoint - Core models and profile management
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 6. Implement Retrieval Engine
  - [ ] 6.1 Create OpenSearch client wrapper
    - Implement `OpenSearchClient` class with opensearch-py
    - Configure connection to OpenSearch domain
    - Implement health check and connection retry logic
    - _Requirements: 2.2, 2.3_
  
  - [ ] 6.2 Implement semantic vector search
    - Create `retrieve_passages(query: str, language: str, top_k: int) -> List[Passage]` function
    - Generate query embeddings using Bedrock or SageMaker bi-encoder
    - Perform kNN search against language-specific index
    - Apply trimester filters if present in user profile
    - Return top-k passages with metadata
    - _Requirements: 2.2, 2.3, 3.3_
  
  - [ ]* 6.3 Write property test for retrieval completeness
    - **Property 3: Retrieval Completeness**
    - Test that retrieval returns exactly k passages (or fewer if index has fewer)
    - **Validates: Requirements 2.2, 2.3**
  
  - [ ]* 6.4 Write unit tests for retrieval edge cases
    - Test empty query handling
    - Test retrieval with no results (empty index)
    - Test trimester filtering
    - _Requirements: 2.2, 2.3, 3.3, 3.4_

- [ ] 7. Implement Re-Ranker component
  - [ ] 7.1 Create SageMaker cross-encoder client
    - Implement `CrossEncoderReranker` class with boto3 SageMaker runtime
    - Create `rerank_passages(query: str, passages: List[Passage], top_n: int) -> List[RankedPassage]` function
    - Batch query-passage pairs for efficient inference
    - Sort by cross-encoder score descending
    - _Requirements: 2.4_
  
  - [ ]* 7.2 Write property test for re-ranking scoring
    - **Property 4: Re-Ranking Scoring**
    - Test that all passages receive scores and are ordered by descending score
    - **Validates: Requirements 2.4**
  
  - [ ]* 7.3 Write unit tests for re-ranking edge cases
    - Test re-ranking with single passage
    - Test re-ranking with empty passage list
    - Test top_n greater than passage count
    - _Requirements: 2.4_

- [ ] 8. Implement Answer Generator
  - [ ] 8.1 Create Bedrock LLM client
    - Implement `BedrockAnswerGenerator` class with boto3 Bedrock runtime
    - Configure Claude 3 Sonnet/Haiku model
    - Implement prompt template with safety instructions
    - Handle language-specific prompts (Hindi vs English)
    - _Requirements: 3.1, 3.2, 3.3, 3.5_
  
  - [ ] 8.2 Implement answer generation logic
    - Create `generate_answer(query: str, passages: List[RankedPassage], language: str, user_profile: UserProfile, mode: str) -> GenerationResponse` function
    - Build prompt with retrieved passages and sources
    - Include trimester context if available in profile
    - Adjust verbosity based on mode (standard vs short)
    - Extract sources used from generated answer
    - _Requirements: 3.1, 3.3, 3.5, 7.5_
  
  - [ ]* 8.3 Write property test for source citation presence
    - **Property 5: Source Citation Presence**
    - Test that all generated responses include at least one source citation
    - **Validates: Requirements 2.6**
  
  - [ ]* 8.4 Write property test for context grounding
    - **Property 6: Context Grounding**
    - Test that generated answers don't hallucinate beyond provided passages
    - **Validates: Requirements 2.5**
  
  - [ ]* 8.5 Write property test for trimester-aware responses
    - **Property 7: Trimester-Aware Responses**
    - Test that responses reference the user's trimester when available
    - **Validates: Requirements 3.3**
  
  - [ ]* 8.6 Write property test for short answer mode
    - **Property 8: Short Answer Mode**
    - Test that short mode produces shorter responses than standard mode
    - **Validates: Requirements 3.5**
  
  - [ ]* 8.7 Write unit tests for answer generation edge cases
    - Test generation with no passages (should return "no information" message)
    - Test generation with low-literacy profile
    - Test generation with very long passages
    - _Requirements: 3.1, 3.2, 3.4_

- [ ] 9. Checkpoint - Retrieval and generation pipeline
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Implement Safety Classifier
  - [ ] 10.1 Create high-risk symptom detection module
    - Define high-risk keyword lists for Hindi and English
    - Implement `detect_high_risk_keywords(text: str, language: str) -> bool` function
    - Create keyword matching with fuzzy matching for variations
    - _Requirements: 5.1, 5.2_
  
  - [ ] 10.2 Implement ML-based safety classification
    - Create `SafetyClassifier` class with Bedrock classification
    - Implement `classify_safety(query: str, answer: str, language: str) -> SafetyClassificationResponse` function
    - Combine rule-based and ML-based detection
    - Generate SafetyAlert for high-risk queries (severity >= 0.7)
    - _Requirements: 5.1, 5.2_
  
  - [ ] 10.3 Implement medical disclaimer injection
    - Create `add_medical_disclaimer(answer: str, language: str) -> str` function
    - Append disclaimer to all medical guidance responses
    - Use language-specific disclaimer text
    - _Requirements: 5.3_
  
  - [ ]* 10.4 Write property test for safety classification
    - **Property 10: Safety Classification**
    - Test that all symptom queries receive risk scores in range [0, 1]
    - **Validates: Requirements 5.1**
  
  - [ ]* 10.5 Write property test for high-risk alert generation
    - **Property 11: High-Risk Alert Generation**
    - Test that queries with high-risk keywords generate SafetyAlert
    - **Validates: Requirements 5.2**
  
  - [ ]* 10.6 Write property test for medical disclaimer presence
    - **Property 12: Medical Disclaimer Presence**
    - Test that all medical responses include disclaimer
    - **Validates: Requirements 5.3**
  
  - [ ]* 10.7 Write unit tests for safety edge cases
    - Test borderline risk scores (0.69, 0.70, 0.71)
    - Test mixed-language queries with risk keywords
    - Test false positive scenarios
    - _Requirements: 5.1, 5.2, 5.3_

- [ ] 11. Implement Voice Interface components
  - [ ] 11.1 Create Amazon Lex integration
    - Implement `LexVoiceInterface` class with boto3 Lex runtime
    - Create `transcribe_voice(audio_data: bytes, language: str) -> str` function
    - Handle low confidence transcriptions
    - _Requirements: 1.2, 4.1, 4.4_
  
  - [ ] 11.2 Create Amazon Polly integration
    - Implement `PollyTextToSpeech` class with boto3 Polly
    - Create `synthesize_speech(text: str, language: str) -> str` function (returns S3 URL)
    - Select appropriate voice based on language (Aditi for Hindi, Raveena for English)
    - Generate low-bandwidth MP3 (24kbps)
    - _Requirements: 4.2, 4.3_
  
  - [ ]* 11.3 Write property test for voice language selection
    - **Property 9: Voice Language Selection**
    - Test that Hindi text uses Hindi voice and English text uses English voice
    - **Validates: Requirements 4.3**
  
  - [ ]* 11.4 Write unit tests for voice interface
    - Test Lex integration with mock audio data
    - Test Polly integration with sample text
    - Test voice quality error handling
    - _Requirements: 1.2, 4.1, 4.2, 4.4_

- [ ] 12. Implement FAQ Browser
  - [ ] 12.1 Create FAQ repository
    - Implement `FAQRepository` class with DynamoDB client
    - Create methods: `get_all_faqs(language: str)`, `get_faq_by_id(faq_id: str, language: str)`, `get_faqs_by_category(category: str, language: str)`
    - Implement FAQ categorization and filtering
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5_
  
  - [ ]* 12.2 Write property test for FAQ categorization
    - **Property 19: FAQ Categorization**
    - Test that all FAQs have category and trimester fields
    - **Validates: Requirements 8.2, 8.3**
  
  - [ ]* 12.3 Write property test for FAQ bilingual availability
    - **Property 20: FAQ Bilingual Availability**
    - Test that every FAQ in one language has equivalent in other language
    - **Validates: Requirements 8.5**
  
  - [ ]* 12.4 Write unit tests for FAQ browser
    - Test FAQ retrieval by category
    - Test FAQ retrieval by trimester
    - Test FAQ with missing translations
    - _Requirements: 8.1, 8.2, 8.3, 8.4_

- [ ] 13. Implement Government Scheme retrieval
  - [ ] 13.1 Add government scheme filtering to retrieval
    - Extend retrieval engine to filter by topic="government_scheme"
    - Boost government scheme documents for relevant queries
    - _Requirements: 6.2_
  
  - [ ] 13.2 Implement government scheme response validation
    - Create `validate_government_scheme_response(answer: str, sources: List[Source]) -> bool` function
    - Check for eligibility and application keywords
    - Verify sources are from government domains
    - _Requirements: 6.3, 6.4_
  
  - [ ]* 13.3 Write property test for government scheme retrieval
    - **Property 13: Government Scheme Retrieval**
    - Test that government program queries retrieve government_scheme tagged passages
    - **Validates: Requirements 6.2**
  
  - [ ]* 13.4 Write property test for government scheme response completeness
    - **Property 14: Government Scheme Response Completeness**
    - Test that government scheme responses include eligibility, application, and official sources
    - **Validates: Requirements 6.3, 6.4**

- [ ] 14. Checkpoint - Safety, voice, and specialized features
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 15. Implement low bandwidth optimization
  - [ ] 15.1 Create bandwidth-aware response handler
    - Implement `optimize_for_bandwidth(response: QueryResponse, user_profile: UserProfile) -> QueryResponse` function
    - Disable voice output when low bandwidth flag is set
    - Reduce response verbosity for low bandwidth users
    - _Requirements: 9.1, 9.3_
  
  - [ ]* 15.2 Write property test for low bandwidth mode prioritization
    - **Property 21: Low Bandwidth Mode Prioritization**
    - Test that low bandwidth mode disables voice and returns text-only
    - **Validates: Requirements 9.1**
  
  - [ ]* 15.3 Write property test for low bandwidth response reduction
    - **Property 22: Low Bandwidth Response Reduction**
    - Test that low bandwidth responses are shorter than standard responses
    - **Validates: Requirements 9.3**
  
  - [ ] 15.4 Implement response caching
    - Create `ResponseCache` class with in-memory cache (TTL: 5 minutes)
    - Implement `cache_response()` and `get_cached_response()` methods
    - Handle cache invalidation
    - _Requirements: 9.5_
  
  - [ ]* 15.5 Write property test for response caching
    - **Property 23: Response Caching**
    - Test that recent queries return cached responses on network errors
    - **Validates: Requirements 9.5**
  
  - [ ]* 15.6 Write unit tests for bandwidth optimization
    - Test compression header presence
    - Test cache hit/miss scenarios
    - Test cache expiration
    - _Requirements: 9.2, 9.5_

- [ ] 16. Implement orchestration layer (Lambda handler)
  - [ ] 16.1 Create main query orchestrator
    - Implement `QueryOrchestrator` class that coordinates all components
    - Create `process_query(request: QueryRequest) -> QueryResponse` method
    - Wire together: language router → profile loading → retrieval → re-ranking → generation → safety → voice (optional)
    - Handle component failures with graceful degradation
    - _Requirements: 1.1, 1.3, 1.4, 2.2, 2.4, 3.1, 5.1, 7.4, 7.5_
  
  - [ ] 16.2 Create Lambda handler function
    - Implement `lambda_handler(event, context)` function
    - Parse API Gateway event to QueryRequest
    - Call QueryOrchestrator
    - Format response for API Gateway
    - Add error handling and logging
    - _Requirements: All_
  
  - [ ]* 16.3 Write property test for profile-based personalization
    - **Property 18: Profile-Based Personalization**
    - Test that responses reflect user profile preferences (language, trimester, bandwidth)
    - **Validates: Requirements 7.5**
  
  - [ ]* 16.4 Write property test for graceful degradation
    - **Property 24: Graceful Degradation**
    - Test that component failures return error responses rather than crashing
    - **Validates: Requirements 10.5**
  
  - [ ]* 16.5 Write integration tests for end-to-end flow
    - Test complete query flow from request to response
    - Test multi-language query sequences
    - Test voice mode end-to-end
    - Test FAQ browsing flow
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 8.1_

- [ ] 17. Implement API Gateway endpoints
  - [ ] 17.1 Create FastAPI application for local testing
    - Implement FastAPI app with endpoints: `/query`, `/voice-query`, `/faq`, `/faq/{id}`, `/profile`, `/profile/{userId}`
    - Add request validation with Pydantic models
    - Add CORS middleware
    - Add rate limiting middleware
    - _Requirements: All_
  
  - [ ] 17.2 Create API Gateway configuration (Terraform/CloudFormation)
    - Define REST API with Lambda integration
    - Configure authentication (API keys, optional OAuth)
    - Set up rate limiting and throttling
    - Configure CORS
    - _Requirements: All_
  
  - [ ]* 17.3 Write integration tests for API endpoints
    - Test all endpoints with valid requests
    - Test authentication and authorization
    - Test rate limiting
    - Test error responses (400, 401, 403, 500)
    - _Requirements: All_

- [ ] 18. Implement security and privacy features
  - [ ] 18.1 Implement data anonymization
    - Create `anonymize_query_log(log: QueryLog) -> QueryLog` function
    - Remove diagnostic terms from stored queries
    - Implement PII detection and removal
    - _Requirements: 12.3_
  
  - [ ] 18.2 Implement user data deletion
    - Create `delete_user_data(user_id: str)` function
    - Delete user profile from DynamoDB
    - Delete query logs for user
    - _Requirements: 12.4_
  
  - [ ] 18.3 Implement administrative access controls
    - Create admin endpoints with IAM authentication
    - Implement role-based access control
    - Add audit logging for admin actions
    - _Requirements: 12.5_
  
  - [ ]* 18.4 Write property test for no diagnostic storage
    - **Property 25: No Diagnostic Storage**
    - Test that stored logs don't contain diagnostic terms
    - **Validates: Requirements 12.3**
  
  - [ ]* 18.5 Write property test for user data deletion
    - **Property 26: User Data Deletion**
    - Test that deletion requests remove all user data
    - **Validates: Requirements 12.4**
  
  - [ ]* 18.6 Write property test for administrative access control
    - **Property 27: Administrative Access Control**
    - Test that admin endpoints reject unauthorized requests
    - **Validates: Requirements 12.5**
  
  - [ ]* 18.7 Write unit tests for security features
    - Test encryption configuration (DynamoDB, S3)
    - Test TLS configuration
    - Test PII detection accuracy
    - _Requirements: 12.1, 12.2, 12.3_

- [ ] 19. Checkpoint - Complete system integration
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 20. Implement logging and monitoring
  - [ ] 20.1 Create structured logging
    - Implement `StructuredLogger` class with CloudWatch Logs integration
    - Log all queries with metadata (language, latency, components used)
    - Log safety alerts separately for analysis
    - Add correlation IDs for request tracing
    - _Requirements: All_
  
  - [ ] 20.2 Create CloudWatch metrics
    - Implement custom metrics: query_volume, response_latency, error_rate, safety_alert_count
    - Add metrics by language and component
    - Create metric filters for alerting
    - _Requirements: All_
  
  - [ ] 20.3 Implement AWS X-Ray tracing
    - Add X-Ray SDK instrumentation to all components
    - Trace end-to-end request flow
    - Add custom subsegments for retrieval, re-ranking, generation
    - _Requirements: All_
  
  - [ ]* 20.4 Write unit tests for logging and monitoring
    - Test log format and structure
    - Test metric emission
    - Test trace propagation
    - _Requirements: All_

- [ ] 21. Create document ingestion pipeline
  - [ ] 21.1 Implement document processor
    - Create `DocumentProcessor` class for text extraction from PDF/HTML/DOCX
    - Implement chunking logic (512 tokens with 50-token overlap)
    - Add metadata extraction (source, topic, trimester)
    - _Requirements: 2.1_
  
  - [ ] 21.2 Implement embedding generation
    - Create `EmbeddingGenerator` class with Bedrock or SageMaker bi-encoder
    - Generate embeddings for document chunks
    - Batch processing for efficiency
    - _Requirements: 2.2_
  
  - [ ] 21.3 Implement OpenSearch indexing
    - Create `DocumentIndexer` class for OpenSearch bulk indexing
    - Index documents in language-specific indexes
    - Handle indexing errors and retries
    - _Requirements: 2.1, 2.2_
  
  - [ ] 21.4 Create S3 event-triggered Lambda for ingestion
    - Implement Lambda function triggered by S3 uploads
    - Orchestrate: extract → chunk → embed → index
    - Add error handling and dead letter queue
    - _Requirements: 2.1_
  
  - [ ]* 21.5 Write integration tests for document ingestion
    - Test end-to-end ingestion pipeline
    - Test with sample Hindi and English documents
    - Test error handling for malformed documents
    - _Requirements: 2.1, 2.2_

- [ ] 22. Create infrastructure as code
  - [ ] 22.1 Create Terraform modules for AWS resources
    - VPC, subnets, security groups
    - OpenSearch domain with Hindi and English indexes
    - DynamoDB tables (user_profiles, faqs, query_logs)
    - S3 buckets (documents, audio files)
    - Lambda functions with appropriate IAM roles
    - API Gateway with stages (dev, staging, prod)
    - SageMaker endpoints for bi-encoder and cross-encoder
    - _Requirements: All (infrastructure)_
  
  - [ ] 22.2 Create deployment scripts
    - Script to deploy infrastructure (Terraform apply)
    - Script to deploy Lambda functions (zip and upload)
    - Script to configure Lex bot
    - Script to load initial FAQ data
    - _Requirements: All (infrastructure)_
  
  - [ ] 22.3 Create environment configuration
    - Create separate configs for dev, staging, production
    - Configure environment variables for each stage
    - Set up secrets management (AWS Secrets Manager)
    - _Requirements: All (infrastructure)_

- [ ] 23. Create sample data and seed scripts
  - [ ] 23.1 Create sample maternal health documents
    - Collect 50+ sample documents in Hindi and English
    - Cover topics: nutrition, symptoms, exercise, tests, government schemes
    - Organize by trimester
    - _Requirements: 2.1, 6.1_
  
  - [ ] 23.2 Create FAQ seed data
    - Create 100+ FAQs in both Hindi and English
    - Categorize by topic and trimester
    - Include government scheme FAQs
    - _Requirements: 8.1, 8.2, 8.3, 8.5_
  
  - [ ] 23.3 Create data seeding scripts
    - Script to upload documents to S3 and trigger ingestion
    - Script to load FAQs into DynamoDB
    - Script to create sample user profiles for testing
    - _Requirements: 2.1, 8.1_

- [ ] 24. Final checkpoint and documentation
  - [ ] 24.1 Run complete test suite
    - Run all unit tests
    - Run all property-based tests (100+ iterations each)
    - Run all integration tests
    - Generate test coverage report (target: 80%+)
    - _Requirements: All_
  
  - [ ] 24.2 Create deployment documentation
    - Document infrastructure setup steps
    - Document deployment process
    - Document environment configuration
    - Document monitoring and alerting setup
    - _Requirements: All_
  
  - [ ] 24.3 Create API documentation
    - Generate OpenAPI/Swagger documentation
    - Document all endpoints with examples
    - Document error codes and responses
    - Document rate limits and authentication
    - _Requirements: All_
  
  - [ ] 24.4 Create user guide
    - Document how to interact with the system
    - Provide example queries in Hindi and English
    - Document voice interaction usage
    - Document FAQ browsing
    - _Requirements: All_
  
  - [ ] 24.5 Final checkpoint
    - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each property-based test should run minimum 100 iterations
- All property tests should be tagged with: `Feature: maatri-maternal-health-assistant, Property {N}: {property_text}`
- Use mocking for AWS services during unit testing (moto library)
- Use hypothesis library for property-based testing in Python
- Checkpoints ensure incremental validation before proceeding
- Infrastructure tasks (22) can be done in parallel with application development
- Sample data creation (23) should be done early to enable realistic testing

## Testing Configuration

### Property-Based Testing Setup

```python
# conftest.py
from hypothesis import settings, Verbosity

# Configure hypothesis for all tests
settings.register_profile("default", max_examples=100, verbosity=Verbosity.normal)
settings.register_profile("ci", max_examples=200, verbosity=Verbosity.verbose)
settings.load_profile("default")
```

### Test Tagging Example

```python
import pytest
from hypothesis import given, strategies as st

@pytest.mark.property_test
@pytest.mark.tag("Feature: maatri-maternal-health-assistant, Property 1: Language Consistency")
@given(query=st.text(min_size=10), language=st.sampled_from(['hindi', 'english']))
def test_language_consistency(query, language):
    """Property 1: For any query in Hindi or English, response language matches input language"""
    # Test implementation
    pass
```

## Dependencies

Key Python packages to include in `requirements.txt`:

```
boto3>=1.28.0
opensearch-py>=2.3.0
pydantic>=2.0.0
fastapi>=0.100.0
uvicorn>=0.23.0
pytest>=7.4.0
hypothesis>=6.82.0
moto>=4.1.0
python-dotenv>=1.0.0
mypy>=1.4.0
black>=23.7.0
ruff>=0.0.280
```
