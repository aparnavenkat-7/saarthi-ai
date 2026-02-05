# Design Document: Saarthi AI

## Overview

Saarthi AI is a multilingual conversational AI assistant that helps Indian citizens and students discover government schemes, scholarships, internships, and opportunities based on eligibility. The system combines a modern web/mobile frontend with a FastAPI backend, integrating LLM capabilities through a RAG (Retrieval-Augmented Generation) pipeline, speech services for voice interaction, and a structured database of schemes.

### Key Design Principles

1. **User-Centric Simplicity**: Design for users with varying digital literacy levels
2. **Multilingual First**: Support regional languages at every layer
3. **Accuracy Over Creativity**: Prioritize factual scheme information from database over LLM hallucinations
4. **Conversational Flow**: Natural dialogue that guides users step-by-step
5. **Visual Accessibility**: Modern glassmorphism UI with clear visual feedback
6. **Scalability**: Architecture that grows with user base and scheme database

### Technology Stack

- **Frontend**: React with TypeScript, TailwindCSS for styling, Framer Motion for animations
- **Backend**: FastAPI (Python) for REST APIs
- **Database**: PostgreSQL for structured scheme data, Redis for caching
- **AI Layer**: OpenAI GPT-4 or similar LLM, LangChain for RAG pipeline
- **Speech Services**: Google Cloud Speech-to-Text and Text-to-Speech APIs
- **Translation**: Google Cloud Translation API or similar
- **Hosting**: Cloud platform (AWS/GCP/Azure) with containerized deployment


## Architecture

### System Architecture

The system follows a three-tier architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  React Frontend (Web) / Flutter (Mobile - Optional)  │  │
│  │  - Glassmorphism UI Components                       │  │
│  │  - Voice Interaction Interface                       │  │
│  │  - Multilingual Display                              │  │
│  │  - Animated Avatar & Waveforms                       │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕ HTTPS/WebSocket
┌─────────────────────────────────────────────────────────────┐
│                      Application Layer                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              FastAPI Backend Services                 │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  API Gateway & Request Router                  │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  User Profile Service                          │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Recommendation Engine                         │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Conversation Manager                          │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │  Admin Service                                 │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕ Internal APIs
┌─────────────────────────────────────────────────────────────┐
│                         AI Layer                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  RAG Pipeline                                         │  │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │  │
│  │  │  Retriever │→ │ LLM Engine │→ │  Response    │  │  │
│  │  │  (Vector   │  │ (GPT-4)    │  │  Generator   │  │  │
│  │  │   Search)  │  │            │  │              │  │  │
│  │  └────────────┘  └────────────┘  └──────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Speech Services                                      │  │
│  │  ┌──────────────────┐  ┌──────────────────────────┐ │  │
│  │  │  Speech-to-Text  │  │  Text-to-Speech          │ │  │
│  │  │  (Multi-language)│  │  (Multi-language)        │ │  │
│  │  └──────────────────┘  └──────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Translation Service                                  │  │
│  │  (Regional Language Support)                          │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕ Database Queries
┌─────────────────────────────────────────────────────────────┐
│                        Data Layer                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  PostgreSQL Database                                  │  │
│  │  - Schemes Table                                      │  │
│  │  - User Profiles Table                                │  │
│  │  - Saved Schemes Table                                │  │
│  │  - Conversation History Table                         │  │
│  │  - Admin Users Table                                  │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Redis Cache                                          │  │
│  │  - Session Data                                       │  │
│  │  - Frequently Accessed Schemes                        │  │
│  │  - User Preferences                                   │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Vector Database (Pinecone/Weaviate)                 │  │
│  │  - Scheme Embeddings for Semantic Search             │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **User Query Flow**:
   - User submits text/voice query in regional language
   - Frontend sends query to API Gateway
   - Conversation Manager extracts intent and entities
   - RAG Pipeline retrieves relevant schemes from vector database
   - LLM generates natural language response using retrieved context
   - Translation Service converts response to user's language
   - Response sent back to frontend with scheme recommendations

2. **Voice Interaction Flow**:
   - User activates voice input
   - Frontend captures audio stream
   - Audio sent to Speech-to-Text service
   - Transcribed text processed through normal query flow
   - Response text sent to Text-to-Speech service
   - Audio response streamed back to frontend

3. **Recommendation Flow**:
   - User profile loaded from database
   - Recommendation Engine queries schemes matching eligibility criteria
   - Schemes ranked by relevance score
   - Results cached in Redis for quick access
   - Recommendations displayed with eligibility explanations


## Components and Interfaces

### Frontend Components

#### 1. ChatInterface Component
- **Purpose**: Main conversational UI with message display and input
- **Props**: 
  - `userId: string`
  - `selectedLanguage: string`
  - `onSchemeSelect: (schemeId: string) => void`
- **State**:
  - `messages: Message[]`
  - `isTyping: boolean`
  - `suggestions: string[]`
- **Key Methods**:
  - `sendMessage(text: string): Promise<void>`
  - `handleVoiceInput(): Promise<void>`
  - `updateSuggestions(context: string): void`

#### 2. VoiceInterface Component
- **Purpose**: Voice input/output with visual feedback
- **Props**:
  - `isActive: boolean`
  - `language: string`
  - `onTranscript: (text: string) => void`
- **State**:
  - `isRecording: boolean`
  - `audioLevel: number`
  - `waveformData: number[]`
- **Key Methods**:
  - `startRecording(): void`
  - `stopRecording(): Promise<string>`
  - `playAudioResponse(audioUrl: string): void`

#### 3. SchemeCard Component
- **Purpose**: Display scheme information with glassmorphism styling
- **Props**:
  - `scheme: Scheme`
  - `eligibilityMatch: number`
  - `onSave: () => void`
  - `onViewDetails: () => void`
- **Styling**: Glassmorphism with backdrop blur, transparency, and Indian flag color accents

#### 4. ProfileManager Component
- **Purpose**: User profile creation and editing
- **Props**:
  - `userId: string`
  - `onProfileUpdate: (profile: UserProfile) => void`
- **Fields**:
  - Age, Location (State/District), Student Status, Income Range, Education Level, Field of Study

#### 5. Dashboard Component
- **Purpose**: Display saved schemes and application tracking
- **Props**:
  - `userId: string`
- **Features**:
  - Saved schemes list with status indicators
  - Deadline countdown timers
  - Application progress tracking

#### 6. AnimatedAvatar Component
- **Purpose**: AI assistant visual representation
- **Props**:
  - `isThinking: boolean`
  - `isSpeaking: boolean`
- **Animations**: Idle breathing, thinking pulse, speaking mouth movement

#### 7. SuggestionCards Component
- **Purpose**: Floating contextual suggestions
- **Props**:
  - `suggestions: Suggestion[]`
  - `onSelect: (suggestion: Suggestion) => void`
- **Styling**: Floating cards with smooth hover transitions

### Backend Services

#### 1. User Profile Service
- **Endpoints**:
  - `POST /api/profiles` - Create user profile
  - `GET /api/profiles/{userId}` - Retrieve profile
  - `PUT /api/profiles/{userId}` - Update profile
  - `DELETE /api/profiles/{userId}` - Delete profile
- **Database**: PostgreSQL `user_profiles` table
- **Validation**: Age range (0-120), valid state/district codes, income brackets

#### 2. Conversation Manager Service
- **Endpoints**:
  - `POST /api/chat/message` - Process user message
  - `GET /api/chat/history/{userId}` - Retrieve conversation history
  - `POST /api/chat/voice` - Process voice input
- **Key Functions**:
  - `extractIntent(message: string): Intent`
  - `extractEntities(message: string): Entity[]`
  - `maintainContext(userId: string, message: string): Context`
  - `generateResponse(intent: Intent, context: Context): Response`

#### 3. Recommendation Engine Service
- **Endpoints**:
  - `GET /api/recommendations/{userId}` - Get personalized recommendations
  - `POST /api/recommendations/filter` - Filter schemes by criteria
  - `GET /api/schemes/{schemeId}/eligibility/{userId}` - Check eligibility
- **Key Functions**:
  - `matchEligibility(profile: UserProfile, scheme: Scheme): EligibilityResult`
  - `rankSchemes(schemes: Scheme[], profile: UserProfile): RankedScheme[]`
  - `calculateRelevanceScore(scheme: Scheme, profile: UserProfile): number`

#### 4. RAG Pipeline Service
- **Components**:
  - **Retriever**: Vector similarity search in scheme embeddings
  - **LLM Engine**: GPT-4 API integration with custom prompts
  - **Response Generator**: Combines retrieved context with LLM output
- **Key Functions**:
  - `retrieveRelevantSchemes(query: string, topK: number): Scheme[]`
  - `generateEmbedding(text: string): Vector`
  - `generateResponse(query: string, context: Scheme[], language: string): string`
  - `validateResponse(response: string, context: Scheme[]): boolean`

#### 5. Speech Service
- **Endpoints**:
  - `POST /api/speech/transcribe` - Convert speech to text
  - `POST /api/speech/synthesize` - Convert text to speech
- **Integration**: Google Cloud Speech APIs
- **Supported Languages**: Hindi, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam, Punjabi, English

#### 6. Translation Service
- **Endpoints**:
  - `POST /api/translate` - Translate text between languages
- **Key Functions**:
  - `translate(text: string, sourceLang: string, targetLang: string): string`
  - `detectLanguage(text: string): string`
- **Caching**: Frequently translated phrases cached in Redis

#### 7. Admin Service
- **Endpoints**:
  - `POST /api/admin/schemes` - Create new scheme
  - `PUT /api/admin/schemes/{schemeId}` - Update scheme
  - `DELETE /api/admin/schemes/{schemeId}` - Delete/deactivate scheme
  - `POST /api/admin/schemes/bulk-import` - Import multiple schemes
  - `GET /api/admin/analytics` - View usage analytics
- **Authentication**: JWT-based admin authentication
- **Validation**: Comprehensive scheme data validation

#### 8. Notification Service
- **Endpoints**:
  - `POST /api/notifications/send` - Send notification
  - `GET /api/notifications/{userId}` - Get user notifications
  - `PUT /api/notifications/{notificationId}/read` - Mark as read
- **Channels**: In-app, Email, SMS
- **Triggers**: Deadline reminders, new scheme matches, scheme updates


## Data Models

### User Profile Model
```typescript
interface UserProfile {
  userId: string;              // UUID
  age: number;                 // 0-120
  state: string;               // Indian state code
  district: string;            // District name
  isStudent: boolean;
  educationLevel: EducationLevel;  // 'high_school' | 'undergraduate' | 'postgraduate' | 'doctorate'
  fieldOfStudy?: string;       // Optional, for students
  incomeRange: IncomeRange;    // 'below_1L' | '1L_3L' | '3L_5L' | '5L_10L' | 'above_10L'
  employmentStatus: EmploymentStatus;  // 'student' | 'employed' | 'unemployed' | 'self_employed'
  preferredLanguage: string;   // ISO language code
  createdAt: Date;
  updatedAt: Date;
}
```

### Scheme Model
```typescript
interface Scheme {
  schemeId: string;            // UUID
  name: string;
  nameTranslations: Record<string, string>;  // Language code -> translated name
  description: string;
  descriptionTranslations: Record<string, string>;
  category: SchemeCategory;    // 'scholarship' | 'internship' | 'government_scheme' | 'hackathon' | 'competition'
  eligibilityCriteria: EligibilityCriteria;
  benefitAmount?: number;      // In INR
  benefitDescription: string;
  applicationDeadline: Date;
  applicationLink: string;
  requiredDocuments: Document[];
  applicationSteps: ApplicationStep[];
  targetAudience: string[];    // ['students', 'women', 'farmers', etc.]
  geographicScope: GeographicScope;  // 'national' | 'state' | 'district'
  applicableStates?: string[]; // If state-specific
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
  sourceUrl: string;           // Official government source
}
```

### Eligibility Criteria Model
```typescript
interface EligibilityCriteria {
  minAge?: number;
  maxAge?: number;
  requiredStates?: string[];
  requiredEducationLevel?: EducationLevel[];
  maxIncomeRange?: IncomeRange;
  requiredEmploymentStatus?: EmploymentStatus[];
  isStudentOnly?: boolean;
  requiredFieldOfStudy?: string[];
  additionalCriteria?: Record<string, any>;  // Flexible for scheme-specific criteria
}
```

### Conversation Message Model
```typescript
interface Message {
  messageId: string;
  userId: string;
  role: 'user' | 'assistant';
  content: string;
  language: string;
  timestamp: Date;
  intent?: Intent;
  entities?: Entity[];
  schemeReferences?: string[];  // Scheme IDs mentioned
  metadata?: Record<string, any>;
}
```

### Saved Scheme Model
```typescript
interface SavedScheme {
  savedSchemeId: string;
  userId: string;
  schemeId: string;
  savedAt: Date;
  applicationStatus: ApplicationStatus;  // 'not_started' | 'in_progress' | 'submitted' | 'completed'
  notes?: string;
  reminderSent: boolean;
}
```

### Application Guide Model
```typescript
interface ApplicationGuide {
  schemeId: string;
  steps: ApplicationStep[];
  requiredDocuments: Document[];
  estimatedTime: string;       // e.g., "30 minutes"
  tips: string[];
}

interface ApplicationStep {
  stepNumber: number;
  title: string;
  description: string;
  actionRequired: string;
  externalLink?: string;
}

interface Document {
  documentType: string;        // 'aadhar' | 'income_certificate' | 'marksheet' | etc.
  description: string;
  isMandatory: boolean;
  format: string[];            // ['pdf', 'jpg', 'png']
  maxSize?: string;            // e.g., "2MB"
}
```

### Recommendation Result Model
```typescript
interface RecommendationResult {
  scheme: Scheme;
  relevanceScore: number;      // 0-100
  eligibilityMatch: EligibilityMatch;
  reasonsForRecommendation: string[];
  daysUntilDeadline: number;
}

interface EligibilityMatch {
  isEligible: boolean;
  matchedCriteria: string[];
  unmatchedCriteria: string[];
  explanation: string;
}
```

### Admin User Model
```typescript
interface AdminUser {
  adminId: string;
  username: string;
  email: string;
  passwordHash: string;
  role: 'super_admin' | 'content_manager';
  createdAt: Date;
  lastLogin: Date;
}
```

### Notification Model
```typescript
interface Notification {
  notificationId: string;
  userId: string;
  type: NotificationType;      // 'deadline_reminder' | 'new_scheme' | 'scheme_update'
  title: string;
  message: string;
  schemeId?: string;
  isRead: boolean;
  sentAt: Date;
  channels: NotificationChannel[];  // ['in_app', 'email', 'sms']
}
```

### Database Schema

#### PostgreSQL Tables

**user_profiles**
- Primary Key: `user_id` (UUID)
- Indexes: `state`, `is_student`, `education_level`

**schemes**
- Primary Key: `scheme_id` (UUID)
- Indexes: `category`, `application_deadline`, `is_active`, `geographic_scope`
- Full-text search index on `name` and `description`

**saved_schemes**
- Primary Key: `saved_scheme_id` (UUID)
- Foreign Keys: `user_id`, `scheme_id`
- Indexes: `user_id`, `scheme_id`, `application_status`

**conversation_history**
- Primary Key: `message_id` (UUID)
- Foreign Key: `user_id`
- Indexes: `user_id`, `timestamp`

**admin_users**
- Primary Key: `admin_id` (UUID)
- Unique: `username`, `email`

**notifications**
- Primary Key: `notification_id` (UUID)
- Foreign Keys: `user_id`, `scheme_id`
- Indexes: `user_id`, `is_read`, `sent_at`

#### Vector Database (Pinecone/Weaviate)

**scheme_embeddings**
- Vector dimension: 1536 (OpenAI ada-002 embedding size)
- Metadata: `scheme_id`, `category`, `target_audience`, `geographic_scope`
- Used for semantic search and RAG retrieval


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, I identified several areas of redundancy:

1. **Language consistency properties (2.1, 2.3, 2.4)**: Properties 2.3 and 2.4 both test context/history preservation across language switches. These can be combined into a single comprehensive property.

2. **Display field properties (4.3, 8.3)**: Both test that required fields are present in scheme displays. These can be combined into one property about scheme display completeness.

3. **Filtering properties (4.4, 8.2)**: Both test category filtering functionality. Can be combined into a single property about category filtering.

4. **Notification deadline properties (9.3, 19.1)**: These are identical requirements. Only one property needed.

5. **Suggestion display properties (3.3, 10.5)**: Both test waveform display during voice interaction. Only one property needed.

The following properties represent the unique, non-redundant correctness properties for Saarthi AI:

### Core Properties

**Property 1: Profile Validation**
*For any* submitted user profile data, the system should accept and store only profiles where all required fields (age, location, student status, income range, education level) are present and valid, and reject profiles with missing or invalid data.
**Validates: Requirements 1.2**

**Property 2: Profile Update Triggers Recommendation Refresh**
*For any* user profile update, the system should generate different recommendations when eligibility criteria change, ensuring recommendations reflect the current profile state.
**Validates: Requirements 1.3**

**Property 3: Missing Profile Data Prompts**
*For any* user profile with missing required fields, when the user requests recommendations or eligibility checks, the system should request the specific missing information before proceeding.
**Validates: Requirements 1.5**

**Property 4: Language Consistency**
*For any* user query after language selection, all system responses should be in the selected regional language, regardless of the query language.
**Validates: Requirements 2.1**

**Property 5: Context Preservation Across Language Switches**
*For any* conversation, switching languages mid-conversation should preserve all conversation context and history, allowing references to previous messages in the new language.
**Validates: Requirements 2.3, 2.4**

**Property 6: Translation Preserves Critical Data**
*For any* scheme information translated to a regional language, critical fields (eligibility criteria numbers, deadline dates, benefit amounts, document requirements) should remain identical to the source data after translation.
**Validates: Requirements 2.5**

**Property 7: Eligibility Matching**
*For any* user profile and scheme recommendation request, all returned schemes should match the user's eligibility criteria (age range, location, student status, income range, education level).
**Validates: Requirements 4.1**

**Property 8: Recommendation Ranking**
*For any* set of recommended schemes, schemes should be ordered by relevance score calculated from profile match percentage, deadline proximity, and benefit amount, with higher scores appearing first.
**Validates: Requirements 4.2**

**Property 9: Scheme Display Completeness**
*For any* displayed scheme (recommendation or search result), the rendered output should contain all required fields: name, description, eligibility summary, deadline, benefit amount, and category.
**Validates: Requirements 4.3, 8.3**

**Property 10: Category Filtering**
*For any* scheme filter request by category (scholarship, internship, government scheme, hackathon, competition), all returned schemes should belong to the requested category.
**Validates: Requirements 4.4, 8.2**

**Property 11: Eligibility Determination**
*For any* user profile and scheme, the eligibility check should return a boolean determination (eligible/ineligible) with an explanation listing which criteria are met and which are not met.
**Validates: Requirements 5.1, 5.2, 5.3**

**Property 12: Alternative Scheme Suggestions**
*For any* eligibility check where the user is ineligible, the system should suggest at least one alternative scheme that the user is eligible for, if such schemes exist in the database.
**Validates: Requirements 5.5**

**Property 13: Application Guide Completeness**
*For any* scheme, the generated application guide should include all application steps from the scheme data, all required documents with descriptions, the application deadline, and the application portal link.
**Validates: Requirements 6.1, 6.2, 6.3, 6.4**

**Property 14: RAG Retrieval Relevance**
*For any* user query about schemes, the RAG pipeline should retrieve only schemes that are semantically relevant to the query based on vector similarity, with relevance scores above a minimum threshold.
**Validates: Requirements 7.1**

**Property 15: Source Citation**
*For any* response containing scheme details, the response should include a reference to the source (scheme ID or official URL) for the information provided.
**Validates: Requirements 7.4**

**Property 16: Student Profile Filtering**
*For any* user profile with student status set to true, scheme recommendations should include only schemes where the target audience includes students or the category is scholarship/internship/hackathon/competition.
**Validates: Requirements 8.1, 8.4**

**Property 17: Deadline Prioritization**
*For any* set of recommended schemes, schemes with deadlines within 30 days should rank higher than schemes with later deadlines, all else being equal.
**Validates: Requirements 8.5**

**Property 18: Save Scheme Persistence**
*For any* scheme saved by a user, the scheme should appear in the user's dashboard with the correct status (deadline approaching, application open, closed) based on the current date and deadline.
**Validates: Requirements 9.1, 9.2**

**Property 19: Deadline Notification Trigger**
*For any* saved scheme where the deadline is within 7 days and no reminder has been sent, the system should create a notification for the user.
**Validates: Requirements 9.3, 19.1**

**Property 20: Dashboard Scheme Removal**
*For any* scheme removed from a user's dashboard, subsequent dashboard queries should not include that scheme.
**Validates: Requirements 9.4**

**Property 21: Contextual Suggestion Updates**
*For any* conversation, as the conversation progresses through different topics (profile setup → scheme discovery → application guidance), the suggestion cards should update to reflect the current conversation context.
**Validates: Requirements 11.2**

**Property 22: Suggestion Language Consistency**
*For any* displayed suggestion cards, all suggestion text should be in the user's selected regional language.
**Validates: Requirements 11.4**

**Property 23: Irrelevant Suggestion Hiding**
*For any* conversation state where no contextual suggestions are applicable, the suggestion cards should be hidden or empty.
**Validates: Requirements 11.5**

**Property 24: Admin Scheme Creation**
*For any* admin-submitted scheme with all required fields (name, description, eligibility criteria, deadline, documents, application link), the scheme should be created and immediately available in user-facing recommendations if active.
**Validates: Requirements 12.2, 12.3**

**Property 25: Inactive Scheme Exclusion**
*For any* scheme marked as inactive, the scheme should not appear in user recommendations, search results, or new notifications.
**Validates: Requirements 12.4**

**Property 26: Bulk Import Scheme Creation**
*For any* valid structured data file imported by an admin, each scheme entry in the file should create a corresponding scheme record in the database.
**Validates: Requirements 12.5**

**Property 27: Intent and Entity Extraction**
*For any* natural language user query, the system should extract at least one intent (search_schemes, check_eligibility, get_application_guide, etc.) and relevant entities (scheme names, categories, locations).
**Validates: Requirements 13.1**

**Property 28: Paraphrase Robustness**
*For any* set of queries with the same semantic meaning but different phrasing, the extracted intent should be identical across all variations.
**Validates: Requirements 13.2**

**Property 29: Ambiguity Clarification**
*For any* query that matches multiple intents with similar confidence scores (difference < 0.2), the system should respond with a clarifying question rather than assuming an intent.
**Validates: Requirements 13.3**

**Property 30: Context Maintenance**
*For any* multi-turn conversation, references to entities mentioned in previous messages (e.g., "that scheme", "the scholarship I asked about") should correctly resolve to the appropriate entity from conversation history.
**Validates: Requirements 13.4**

**Property 31: Missing Information Prompts**
*For any* user request that requires profile information not yet provided (e.g., eligibility check without age), the system should prompt for the specific missing information.
**Validates: Requirements 13.5**

**Property 32: Error Recovery Guidance**
*For any* error response (invalid input, service failure, not found), the response should include at least one specific recovery action the user can take.
**Validates: Requirements 14.4**

**Property 33: Data Deletion Completeness**
*For any* user data deletion request, all personal information (profile, conversation history, saved schemes) associated with that user ID should be removed from the database.
**Validates: Requirements 16.2**

**Property 34: Conversation Anonymization**
*For any* stored conversation message, sensitive personal information patterns (phone numbers, email addresses, Aadhar numbers, income amounts) should be replaced with anonymized placeholders.
**Validates: Requirements 16.5**

**Property 35: Cross-Device Data Sync**
*For any* user accessing the system from multiple devices, profile data, saved schemes, and conversation history should be consistent across all sessions.
**Validates: Requirements 17.4**

**Property 36: Search Query Matching**
*For any* search query, all returned schemes should contain the query text (or semantically similar text) in their name, description, or category fields.
**Validates: Requirements 18.1**

**Property 37: Multi-Filter Application**
*For any* scheme search with multiple filters applied (e.g., location=Maharashtra AND category=scholarship AND deadline<2024-12-31), all returned schemes should satisfy all filter criteria simultaneously.
**Validates: Requirements 18.2**

**Property 38: Keyword Highlighting**
*For any* search results, the displayed scheme text should include highlighting markup around words that match the search query.
**Validates: Requirements 18.3**

**Property 39: Empty Search Suggestions**
*For any* search query that returns zero results, the response should include at least one suggestion for a broader search term or similar scheme.
**Validates: Requirements 18.5**

**Property 40: New Scheme Notifications**
*For any* newly added active scheme, users whose profiles match the scheme's eligibility criteria should receive a notification about the new scheme.
**Validates: Requirements 19.3**

**Property 41: Notification Preference Respect**
*For any* user with notification preferences configured (e.g., email only, no SMS), notifications should be sent only through the enabled channels.
**Validates: Requirements 19.4**

**Property 42: Scheme Update Notifications**
*For any* scheme update (deadline change, eligibility change, document change), all users who have saved that scheme should receive a notification about the update.
**Validates: Requirements 19.5**

**Property 43: Feedback Collection**
*For any* user interaction with scheme recommendations (view, save, apply), the system should log the interaction for analytics purposes.
**Validates: Requirements 20.2, 20.4**


## Error Handling

### Error Categories and Handling Strategies

#### 1. User Input Errors
- **Invalid Profile Data**: Return validation errors with specific field issues and acceptable value ranges
- **Malformed Queries**: Use NLU to interpret intent; if confidence is low, ask clarifying questions
- **Empty or Whitespace-Only Input**: Prompt user to provide meaningful input
- **Unsupported Language**: Inform user of supported languages and default to English

#### 2. Service Errors
- **LLM API Failures**: Retry with exponential backoff (3 attempts); fall back to template-based responses if all retries fail
- **Speech Service Failures**: Offer text input alternative; display friendly error message
- **Translation Service Failures**: Fall back to English; notify user that translation is temporarily unavailable
- **Database Connection Errors**: Use cached data if available; display maintenance message if cache miss

#### 3. Data Errors
- **Scheme Not Found**: Suggest similar schemes based on semantic search; offer to browse by category
- **No Eligible Schemes**: Explain why no matches found; suggest profile updates that would expand eligibility
- **Expired Deadlines**: Filter out expired schemes from recommendations; show in dashboard with "closed" status
- **Missing Scheme Data**: Display available fields; mark missing fields as "Information not available"

#### 4. Authentication and Authorization Errors
- **Unauthenticated Access**: Redirect to login; preserve intended action for post-login redirect
- **Unauthorized Admin Access**: Return 403 Forbidden with message about insufficient permissions
- **Session Expiration**: Prompt for re-authentication; preserve conversation context

#### 5. Rate Limiting and Resource Errors
- **API Rate Limits**: Queue requests; inform user of wait time; prioritize critical operations
- **Database Query Timeouts**: Return partial results if available; suggest narrowing search criteria
- **Memory/Storage Limits**: Implement pagination; limit conversation history retention

### Error Response Format

All error responses should follow a consistent structure:

```typescript
interface ErrorResponse {
  error: {
    code: string;              // Machine-readable error code
    message: string;           // User-friendly error message in selected language
    details?: string;          // Additional technical details (optional)
    recoveryActions: string[]; // Specific actions user can take
    timestamp: Date;
  };
}
```

### Graceful Degradation

- **Voice Input Unavailable**: Fall back to text input; maintain full functionality
- **Translation Service Down**: Operate in English only; notify users of temporary limitation
- **Vector Database Unavailable**: Use SQL full-text search as fallback; may have reduced relevance
- **Cache Unavailable**: Query database directly; accept slower response times
- **Notification Service Down**: Queue notifications for later delivery; log failures

### Logging and Monitoring

- Log all errors with severity levels (ERROR, WARNING, INFO)
- Include request ID, user ID (anonymized), timestamp, and stack trace
- Monitor error rates and alert on anomalies
- Track error recovery success rates
- Implement distributed tracing for multi-service errors


## Testing Strategy

### Dual Testing Approach

Saarthi AI requires both unit testing and property-based testing to ensure comprehensive coverage:

- **Unit Tests**: Verify specific examples, edge cases, error conditions, and integration points
- **Property Tests**: Verify universal properties across all inputs through randomized testing

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across a wide input space.

### Property-Based Testing Configuration

**Library Selection**: 
- **Python Backend**: Use `hypothesis` for property-based testing
- **TypeScript Frontend**: Use `fast-check` for property-based testing

**Test Configuration**:
- Minimum 100 iterations per property test (due to randomization)
- Each property test must reference its design document property
- Tag format: `# Feature: saarthi-ai, Property {number}: {property_text}`

**Example Property Test Structure** (Python with hypothesis):

```python
from hypothesis import given, strategies as st
import pytest

# Feature: saarthi-ai, Property 1: Profile Validation
@given(
    age=st.integers(min_value=-10, max_value=150),
    state=st.text(),
    income_range=st.sampled_from(['below_1L', '1L_3L', '3L_5L', '5L_10L', 'above_10L', 'invalid']),
)
def test_profile_validation_property(age, state, income_range):
    """
    For any submitted user profile data, the system should accept and store 
    only profiles where all required fields are present and valid.
    """
    profile_data = {
        'age': age,
        'state': state,
        'income_range': income_range,
        # ... other fields
    }
    
    result = profile_service.create_profile(profile_data)
    
    # If all fields are valid, profile should be created
    if is_valid_profile(profile_data):
        assert result.success is True
        assert result.profile_id is not None
    else:
        # If any field is invalid, profile should be rejected
        assert result.success is False
        assert result.error is not None
```

### Unit Testing Strategy

**Focus Areas for Unit Tests**:
1. **Specific Examples**: Test known good inputs and expected outputs
2. **Edge Cases**: Empty strings, boundary values, special characters
3. **Error Conditions**: Invalid inputs, service failures, missing data
4. **Integration Points**: API endpoints, database operations, external service calls

**Avoid Over-Testing with Unit Tests**:
- Don't write dozens of unit tests for variations of the same input
- Use property tests to cover input variations
- Focus unit tests on specific scenarios that demonstrate correct behavior

**Example Unit Test Structure**:

```python
def test_eligibility_check_specific_scholarship():
    """Test eligibility check for a specific scholarship scenario."""
    profile = UserProfile(
        age=20,
        state='Maharashtra',
        is_student=True,
        education_level='undergraduate',
        income_range='below_1L'
    )
    
    scheme = Scheme(
        name='Merit Scholarship',
        eligibility_criteria=EligibilityCriteria(
            min_age=18,
            max_age=25,
            required_states=['Maharashtra'],
            is_student_only=True,
            max_income_range='1L_3L'
        )
    )
    
    result = recommendation_engine.check_eligibility(profile, scheme)
    
    assert result.is_eligible is True
    assert 'age' in result.matched_criteria
    assert 'state' in result.matched_criteria
    assert len(result.unmatched_criteria) == 0
```

### Test Coverage Goals

- **Backend Services**: 80%+ code coverage
- **Frontend Components**: 70%+ code coverage
- **Critical Paths** (eligibility matching, RAG pipeline, profile validation): 95%+ coverage
- **Property Tests**: All 43 correctness properties implemented

### Integration Testing

**API Integration Tests**:
- Test all REST endpoints with valid and invalid inputs
- Verify authentication and authorization
- Test error responses and status codes
- Verify response schemas match OpenAPI specification

**Service Integration Tests**:
- Test RAG pipeline end-to-end (retrieval → LLM → response)
- Test speech services with sample audio files
- Test translation service with known translations
- Test database operations with test database

**UI Integration Tests**:
- Test user flows (profile creation → scheme discovery → save scheme)
- Test voice interaction flow
- Test language switching
- Test responsive design on different screen sizes

### End-to-End Testing

**Critical User Journeys**:
1. New user creates profile → receives recommendations → saves scheme → gets reminder
2. Student searches for scholarships → checks eligibility → views application guide
3. User asks question in Hindi → receives response in Hindi → switches to English
4. Admin adds new scheme → scheme appears in user recommendations
5. User provides voice input → receives voice response

**E2E Test Tools**:
- **Frontend**: Playwright or Cypress for browser automation
- **Backend**: pytest with requests library for API testing
- **Mobile** (if applicable): Appium for mobile app testing

### Performance Testing

**Load Testing**:
- Simulate 1000+ concurrent users
- Measure response times under load
- Identify bottlenecks in database queries and API calls
- Test auto-scaling behavior

**Stress Testing**:
- Test system behavior at 2x-5x expected load
- Verify graceful degradation
- Test recovery after service failures

**Performance Benchmarks**:
- Text query response: < 3 seconds (p95)
- Voice query response: < 5 seconds (p95)
- Database queries: < 500ms (p95)
- Page load time: < 2 seconds (p95)

### Security Testing

**Vulnerability Testing**:
- SQL injection prevention
- XSS prevention in user inputs
- CSRF protection
- Authentication bypass attempts
- Authorization boundary testing

**Data Privacy Testing**:
- Verify encryption at rest and in transit
- Test data deletion completeness
- Verify anonymization of conversation logs
- Test access control for admin endpoints

### Continuous Integration

**CI Pipeline**:
1. Run linters and formatters (pylint, black, eslint, prettier)
2. Run unit tests with coverage reporting
3. Run property tests (100 iterations minimum)
4. Run integration tests
5. Build Docker images
6. Deploy to staging environment
7. Run E2E tests on staging
8. Generate test reports

**Quality Gates**:
- All tests must pass
- Code coverage must meet thresholds
- No critical security vulnerabilities
- Performance benchmarks must be met

### Test Data Management

**Synthetic Test Data**:
- Generate realistic user profiles with faker library
- Create diverse scheme database for testing
- Generate multilingual test content
- Create audio samples for speech testing

**Test Data Isolation**:
- Use separate test database
- Reset database state between test runs
- Use transactions for test isolation
- Mock external services (LLM, speech, translation)

### Monitoring and Observability

**Production Monitoring**:
- Track error rates by service and endpoint
- Monitor response times and latency
- Track user engagement metrics
- Monitor LLM token usage and costs
- Alert on anomalies and threshold breaches

**Logging Strategy**:
- Structured logging with JSON format
- Include request IDs for tracing
- Log all errors with context
- Anonymize PII in logs
- Centralized log aggregation (ELK stack or similar)

