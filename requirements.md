# Requirements Document

## Introduction

Maatri is an AI-powered maternal healthcare assistant designed to provide verified, personalized pregnancy guidance to expecting mothers in India. The system addresses the critical gap in accessible, language-appropriate maternal health information by offering bilingual support (Hindi and English) through conversational AI, leveraging AWS RAG (Retrieval-Augmented Generation) architecture to ensure accurate, source-backed responses.

## Glossary

- **Maatri_System**: The complete AI-powered maternal healthcare assistant application
- **User**: An expecting mother, family member, or community health worker interacting with the system
- **RAG_Pipeline**: Retrieval-Augmented Generation pipeline consisting of retrieval, re-ranking, and generation components
- **Knowledge_Base**: Collection of verified maternal health documents stored in S3 and indexed in OpenSearch
- **Language_Router**: Component that detects user language and routes to appropriate RAG pipeline
- **Retrieval_Engine**: OpenSearch-based semantic search component using vector embeddings
- **Re_Ranker**: Cross-encoder model that re-scores retrieved passages for relevance
- **Answer_Generator**: LLM-based component that synthesizes natural language responses
- **Safety_Classifier**: Component that detects high-risk symptoms and medical emergencies
- **Voice_Interface**: Speech-to-text and text-to-speech components for voice interaction
- **User_Profile**: Stored user preferences including language, pregnancy stage, and interaction history
- **Source_Citation**: Reference to the original verified document from which information was retrieved
- **Trimester**: Three-month period of pregnancy (first, second, or third trimester)
- **High_Risk_Symptom**: Medical symptom requiring immediate professional attention
- **Government_Scheme**: Indian government maternal health benefit program
- **FAQ_Browser**: Interface for browsing frequently asked questions without conversational input

## Requirements

### Requirement 1: Bilingual Conversational Interface

**User Story:** As an expecting mother in India, I want to interact with the system in my preferred language (Hindi or English), so that I can understand maternal health guidance clearly.

#### Acceptance Criteria

1. WHEN a User submits text input, THE Maatri_System SHALL accept input in Hindi or English
2. WHEN a User submits voice input, THE Voice_Interface SHALL transcribe speech in Hindi or English
3. WHEN the Language_Router detects the input language, THE Maatri_System SHALL route the query to the corresponding language-specific RAG_Pipeline
4. WHEN generating a response, THE Answer_Generator SHALL produce output in the same language as the input
5. WHEN a User switches languages mid-conversation, THE Maatri_System SHALL detect the change and route to the appropriate RAG_Pipeline

### Requirement 2: Verified Knowledge Retrieval

**User Story:** As an expecting mother, I want answers based on verified medical sources, so that I can trust the guidance I receive.

#### Acceptance Criteria

1. THE Knowledge_Base SHALL contain only verified maternal health documents from trusted sources
2. WHEN a User query is received, THE Retrieval_Engine SHALL perform semantic vector search against the language-specific index
3. WHEN retrieving passages, THE Retrieval_Engine SHALL return the top-k most semantically similar passages
4. WHEN passages are retrieved, THE Re_Ranker SHALL re-score them using cross-encoder similarity
5. WHEN generating an answer, THE Answer_Generator SHALL use only the top-ranked passages as context
6. WHEN providing a response, THE Maatri_System SHALL append Source_Citation for every factual claim

### Requirement 3: Natural Conversational Response Generation

**User Story:** As a User with low literacy, I want responses in simple, conversational language, so that I can easily understand the guidance.

#### Acceptance Criteria

1. WHEN generating an answer, THE Answer_Generator SHALL synthesize natural language responses from retrieved passages
2. WHEN the User_Profile indicates low literacy, THE Answer_Generator SHALL use simplified language
3. WHEN the User_Profile contains pregnancy stage information, THE Answer_Generator SHALL provide trimester-aware contextual guidance
4. WHEN no relevant passages are found, THE Maatri_System SHALL respond with a helpful message indicating information unavailability
5. WHERE a User requests short-answer mode, THE Answer_Generator SHALL produce concise responses

### Requirement 4: Voice Interaction Support

**User Story:** As a User with limited literacy, I want to speak my questions and hear responses, so that I can access guidance without reading or typing.

#### Acceptance Criteria

1. WHEN a User provides voice input, THE Voice_Interface SHALL convert speech to text using Amazon Lex
2. WHEN a response is generated, THE Voice_Interface SHALL convert text to speech using Amazon Polly
3. WHEN converting text to speech, THE Voice_Interface SHALL use the appropriate language voice (Hindi or English)
4. WHEN voice quality is poor, THE Voice_Interface SHALL request clarification from the User

### Requirement 5: Safety and Risk Detection

**User Story:** As an expecting mother experiencing concerning symptoms, I want the system to recognize high-risk situations, so that I can seek immediate medical attention.

#### Acceptance Criteria

1. WHEN a User describes symptoms, THE Safety_Classifier SHALL analyze the input for High_Risk_Symptom indicators
2. IF a High_Risk_Symptom is detected, THEN THE Maatri_System SHALL immediately display an emergency alert with medical facility contact information
3. WHEN providing any medical guidance, THE Maatri_System SHALL include a disclaimer that it does not provide medical diagnosis
4. THE Maatri_System SHALL NOT make diagnostic claims or prescribe treatments
5. WHEN uncertain about safety, THE Maatri_System SHALL err on the side of caution and recommend professional consultation

### Requirement 6: Government Scheme Information

**User Story:** As an expecting mother from a low-income background, I want to learn about government maternal health schemes, so that I can access available benefits.

#### Acceptance Criteria

1. THE Knowledge_Base SHALL include information about Indian government maternal health schemes
2. WHEN a User asks about financial assistance or government programs, THE Retrieval_Engine SHALL retrieve relevant Government_Scheme information
3. WHEN providing Government_Scheme information, THE Maatri_System SHALL include eligibility criteria and application procedures
4. WHEN Government_Scheme information is provided, THE Maatri_System SHALL include Source_Citation with official government sources

### Requirement 7: User Profile and Personalization

**User Story:** As a returning User, I want the system to remember my language preference and pregnancy stage, so that I receive personalized guidance without repeating information.

#### Acceptance Criteria

1. WHEN a User first interacts with the system, THE Maatri_System SHALL create a User_Profile
2. WHEN a User specifies language preference, THE Maatri_System SHALL store it in the User_Profile
3. WHEN a User provides pregnancy stage information, THE Maatri_System SHALL store the Trimester in the User_Profile
4. WHEN a returning User interacts, THE Maatri_System SHALL retrieve the User_Profile from DynamoDB
5. WHEN generating responses, THE Answer_Generator SHALL use User_Profile information for personalization

### Requirement 8: FAQ Browsing Interface

**User Story:** As a User exploring maternal health topics, I want to browse frequently asked questions, so that I can learn without formulating specific queries.

#### Acceptance Criteria

1. THE Maatri_System SHALL provide an FAQ_Browser interface
2. WHEN a User accesses the FAQ_Browser, THE Maatri_System SHALL display categorized frequently asked questions
3. WHEN displaying FAQs, THE FAQ_Browser SHALL organize questions by Trimester and topic
4. WHEN a User selects an FAQ, THE Maatri_System SHALL display the pre-generated answer with Source_Citation
5. THE FAQ_Browser SHALL support both Hindi and English languages

### Requirement 9: Low Bandwidth Optimization

**User Story:** As a User in a rural area with limited internet connectivity, I want the system to work on slow connections, so that I can access guidance despite network constraints.

#### Acceptance Criteria

1. WHEN network bandwidth is limited, THE Maatri_System SHALL prioritize text responses over voice
2. WHEN loading resources, THE Maatri_System SHALL use compressed data formats
3. WHEN the User_Profile indicates low bandwidth preference, THE Maatri_System SHALL reduce response verbosity
4. THE Maatri_System SHALL implement progressive loading for the FAQ_Browser interface
5. WHEN network errors occur, THE Maatri_System SHALL cache recent responses for offline access

### Requirement 10: Scalability and Availability

**User Story:** As a system administrator, I want the system to handle varying loads and remain available, so that Users can access guidance at any time.

#### Acceptance Criteria

1. THE Maatri_System SHALL use AWS Lambda for stateless, auto-scaling compute
2. THE Maatri_System SHALL use Amazon OpenSearch with appropriate instance sizing for query load
3. WHEN query volume increases, THE Maatri_System SHALL automatically scale compute resources
4. THE Maatri_System SHALL maintain 99.5% uptime availability
5. WHEN a component fails, THE Maatri_System SHALL implement graceful degradation and error handling

### Requirement 11: Modular Architecture

**User Story:** As a system architect, I want clear separation between interaction, retrieval, generation, and safety components, so that the system is maintainable and extensible.

#### Acceptance Criteria

1. THE Maatri_System SHALL implement API Gateway for all external interactions
2. THE Language_Router SHALL be independently deployable as a Lambda function
3. THE Retrieval_Engine SHALL expose a REST API for passage retrieval
4. THE Re_Ranker SHALL be independently deployable as a SageMaker endpoint
5. THE Answer_Generator SHALL be accessible through Amazon Bedrock API
6. THE Safety_Classifier SHALL be independently testable and deployable
7. WHEN any component is updated, THE other components SHALL continue functioning without modification

### Requirement 12: Data Security and Privacy

**User Story:** As an expecting mother, I want my personal health information to be secure, so that my privacy is protected.

#### Acceptance Criteria

1. WHEN storing User_Profile data, THE Maatri_System SHALL encrypt data at rest in DynamoDB
2. WHEN transmitting data, THE Maatri_System SHALL use TLS encryption
3. THE Maatri_System SHALL NOT store personally identifiable medical diagnoses
4. WHEN a User requests data deletion, THE Maatri_System SHALL remove all User_Profile information within 30 days
5. THE Maatri_System SHALL implement access controls for administrative functions
