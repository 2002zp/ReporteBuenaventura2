# Backend Testing Instructions

## Context
Built a comprehensive citizen complaint platform for Buenaventura with the following backend features:

### Core Features Implemented:
1. **User Authentication System** - JWT-based auth with email/password registration and login
2. **Complaint CRUD Operations** - Create, read, update, delete complaints with proper user associations
3. **AI-Powered Classification** - ChatGPT integration using Emergent LLM Key for automatic complaint categorization
4. **Statistics Endpoints** - Analytics for complaint tracking and performance metrics
5. **Database Integration** - MongoDB with proper UUID handling and timezone-aware operations

### Test Focus Areas:
1. **Authentication Flow** - Register new user, login, JWT token verification
2. **Complaint Operations** - Create complaints, list user complaints, update status, ratings
3. **AI Classification** - Test automatic categorization with various complaint types
4. **Error Handling** - Proper HTTP status codes and error messages
5. **Database Operations** - User and complaint data persistence

### Configuration:
- Backend URL: Uses REACT_APP_BACKEND_URL from environment
- All API endpoints prefixed with '/api'
- Emergent LLM Key configured for ChatGPT integration
- MongoDB with proper error handling

### Key Test Scenarios:
1. User registration with valid data
2. User login and token generation
3. Create complaint with location data and verify AI classification
4. List user complaints and verify data structure
5. Update complaint status (simulate admin action)
6. Submit complaint rating and feedback
7. Get statistics overview

### Known Dependencies:
- MongoDB connection required
- Emergent LLM Key for ChatGPT classification
- All models use UUIDs (not ObjectIDs) for JSON serialization

Please test comprehensively focusing on the high-priority tasks marked in test_result.md.
