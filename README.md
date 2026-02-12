# LLM Evaluation & Feedback Platform

A production-style backend system for evaluating, comparing, and improving Large Language Model (LLM) responses through structured human feedback and automated metrics.

This project is inspired by real-world LLM evaluation workflows used in **Evals, Supervised Fine-Tuning (SFT), and Reinforcement Learning with Human Feedback (RLHF)**.

---

## Project Overview

Modern LLM development depends heavily on **systematic evaluation and high-quality feedback loops**.
This platform enables:

* Running prompts against one or more LLMs
* Collecting and storing model responses
* Allowing humans to **rank, score, and justify** response quality
* Generating evaluation metrics to guide model improvement

The system is designed with **scalability, clarity, and clean backend architecture** in mind.

---

## Key Features

### Prompt & Response Pipeline

* Submit prompts via REST API
* Generate responses from one or multiple LLMs
* Store prompt–response pairs with metadata (model, latency, tokens)

### Human Evaluation (Core)

* Rank multiple responses for the same prompt
* Score responses on:
  * Correctness
  * Clarity
  * Relevance
  * Hallucination risk
* Capture **written rationales** for every evaluation

### Evaluation Metrics

* Win-rate per model
* Average quality scores
* Inter-rater agreement
* Latency and response length tracking

### Dataset Generation

* Export evaluation data in JSON format
* Structured for downstream:
  * Supervised Fine-Tuning (SFT)
  * RLHF reward modeling
  * Offline analysis

---

## Why This Project Matters

This project demonstrates hands-on experience with:

* LLM evaluation strategies (Evals)
* Human-in-the-loop feedback systems
* Model alignment and quality benchmarking
* Backend systems for AI workflows

It closely reflects how production LLM systems are **measured, improved, and aligned with user needs**.

---

## Architecture (High-Level)

```
Client / UI / Postman
        ↓
REST API (Backend Service)
        ↓
Evaluation Engine
        ↓
Database (Prompts, Responses, Scores)
```

### Components

* **API Layer** – Handles prompt submission and evaluation requests
* **Evaluation Engine** – Computes metrics and aggregates scores
* **Persistence Layer** – Stores structured evaluation data
* **Export Module** – Generates datasets for training and analysis

---

## Tech Stack

* **Backend:** Java 17 with Spring Boot 3.2.0
* **Database:** MongoDB
* **LLM Integration:** Local LLM models (Ollama, LlamaCPP, or similar)
* **Testing:** JUnit 5 with TestContainers
* **Documentation:** OpenAPI/Swagger (Springdoc)
* **Build Tool:** Maven

---

## Prerequisites

* Java 17 or higher
* Maven 3.8+
* MongoDB 5.0+ (local or containerized)
* Local LLM setup (Ollama, LlamaCPP, or similar)
* IDE: IntelliJ IDEA, Eclipse, or VS Code

---

## Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd llm-evaluation-platform
```

### 2. Configure MongoDB

**Option A: Local MongoDB**
```bash
# macOS with Homebrew
brew install mongodb-community
brew services start mongodb-community
```

**Option B: Docker**
```bash
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

### 3. Set Up Local LLM

**Using Ollama (recommended):**
```bash
# Install Ollama from https://ollama.ai
# Pull a model
ollama pull mistral  # or llama2, neural-chat, etc.
# Start Ollama server (usually runs on http://localhost:11434)
ollama serve
```

### 4. Configure Application Properties

Create or edit `src/main/resources/application.yml`:

```yaml
spring:
  application:
    name: llm-evaluation-platform
  data:
    mongodb:
      uri: mongodb://localhost:27017/llm_eval_db
  
logging:
  level:
    root: INFO
    com.llmeval: DEBUG

server:
  port: 8080
  servlet:
    context-path: /api

llm:
  provider: ollama  # or llamacpp
  base-url: http://localhost:11434
  model-name: mistral  # Configure based on your local model
  timeout-seconds: 60
```

### 5. Build the Project

```bash
mvn clean install
```

### 6. Run the Application

```bash
mvn spring-boot:run
```

The application will start on `http://localhost:8080`

### 7. Access API Documentation

Open Swagger UI in your browser:
```
http://localhost:8080/swagger-ui.html
```

---

## API Endpoints

### Prompts

```
POST   /api/prompts                    - Create a new prompt
GET    /api/prompts                    - Get all active prompts
GET    /api/prompts/{id}               - Get prompt by ID
GET    /api/prompts/category/{category} - Get prompts by category
DELETE /api/prompts/{id}               - Archive a prompt
```

### LLM Responses

```
POST   /api/responses                  - Submit a response for a prompt
GET    /api/responses/{id}             - Get response by ID
GET    /api/responses/prompt/{promptId} - Get all responses for a prompt
GET    /api/responses/model/{modelName} - Get responses from specific model
```

### Evaluations

```
POST   /api/evaluations                - Submit evaluation scores
GET    /api/evaluations/{id}           - Get evaluation by ID
GET    /api/evaluations/prompt/{promptId} - Get evaluations for prompt
GET    /api/evaluations/evaluator/{evaluatorId} - Get evaluations by evaluator
```

### Metrics & Export

```
GET    /api/metrics/model-performance  - Get model performance metrics
GET    /api/metrics/prompt/{promptId}  - Get metrics for specific prompt
GET    /api/export/dataset             - Export evaluation dataset as JSON
```

---

## Example Evaluation Flow

### 1. Create a Prompt

```bash
curl -X POST http://localhost:8080/api/prompts \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Explain quantum computing in simple terms",
    "category": "science",
    "createdBy": "user123"
  }'
```

**Response:**
```json
{
  "id": "prompt_001",
  "content": "Explain quantum computing in simple terms",
  "category": "science",
  "createdAt": "2026-02-08T10:30:00",
  "responseCount": 0,
  "status": "ACTIVE"
}
```

### 2. Generate Responses from Models

```bash
curl -X POST http://localhost:8080/api/responses \
  -H "Content-Type: application/json" \
  -d '{
    "promptId": "prompt_001",
    "modelName": "gpt-4",
    "content": "Quantum computers use quantum bits...",
    "latencyMs": 245.5,
    "tokenCount": 120
  }'
```

### 3. Submit Evaluation

```bash
curl -X POST http://localhost:8080/api/evaluations \
  -H "Content-Type: application/json" \
  -d '{
    "promptId": "prompt_001",
    "responseIds": ["response_001", "response_002"],
    "evaluatorId": "evaluator_john",
    "scores": [
      {
        "responseId": "response_001",
        "correctnessScore": 5,
        "clarityScore": 4,
        "relevanceScore": 5,
        "hallucationRiskScore": 5,
        "comment": "Excellent explanation"
      },
      {
        "responseId": "response_002",
        "correctnessScore": 3,
        "clarityScore": 3,
        "relevanceScore": 4,
        "hallucationRiskScore": 4,
        "comment": "Good but less detailed"
      }
    ],
    "rankedOrder": "response_001,response_002",
    "rationale": "Response 1 provides better clarity and depth"
  }'
```

### 4. Retrieve Metrics

```bash
curl http://localhost:8080/api/metrics/model-performance
```

---

## 🧪 Testing

### Run Unit Tests

```bash
mvn test
```

### Run Integration Tests with TestContainers

```bash
mvn test -Dgroups=integration
```

### Test Coverage

```bash
mvn jacoco:report
# Coverage report generated at: target/site/jacoco/index.html
```

---

## Project Structure

```
llm-evaluation-platform/
├── src/
│   ├── main/
│   │   ├── java/com/llmeval/
│   │   │   ├── model/              # Data models (Prompt, Response, Evaluation)
│   │   │   ├── repository/         # MongoDB repositories
│   │   │   ├── service/            # Business logic
│   │   │   ├── controller/         # REST endpoints
│   │   │   ├── dto/                # Data transfer objects
│   │   │   ├── exception/          # Custom exceptions
│   │   │   ├── util/               # Utility classes
│   │   │   └── DemoApplication.java # Entry point
│   │   └── resources/
│   │       ├── application.yml     # Application configuration
│   │       └── logback-spring.xml  # Logging configuration
│   └── test/
│       ├── java/com/llmeval/
│       │   ├── service/            # Service layer tests
│       │   └── controller/         # API endpoint tests
│       └── resources/
│           └── application-test.yml
├── pom.xml                          # Maven dependencies
└── README.md                        # This file
```

---

## Engineering Best Practices

* ✅ Clean, modular, and well-documented code
* ✅ Clear separation of concerns (Model → Repository → Service → Controller)
* ✅ Input validation and error handling
* ✅ Meaningful logging for debugging and monitoring
* ✅ Comprehensive API documentation (Swagger/OpenAPI)
* ✅ Unit and integration tests
* ✅ Consistent naming conventions
* ✅ Dependency injection with Spring
* ✅ Database indexing for query performance

---

## Error Handling

The API returns standardized error responses:

```json
{
  "timestamp": "2026-02-08T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Invalid evaluation scores",
  "path": "/api/evaluations"
}
```

---

## Performance Considerations

* MongoDB indexes on frequently queried fields (promptId, modelName, evaluatorId)
* Pagination support for large result sets
* Batch operations for bulk evaluations
* Caching for metric calculations

---

## Security Considerations

* Input validation on all endpoints
* SQL/NoSQL injection prevention through parameterized queries
* Rate limiting for API endpoints (recommended for production)
* Authentication/Authorization (recommended implementation)
* CORS configuration for frontend integration

---

## Future Enhancements

* Automated hallucination detection using semantic analysis
* Multi-evaluator consensus scoring with inter-rater agreement metrics
* Prompt versioning and A/B testing framework
* Role-based access control (annotator, reviewer, admin)
* Support for additional local LLM providers (GPT4All, LM Studio, etc.)
* Real-time evaluation dashboard
* Advanced filtering and search capabilities
* Batch evaluation workflows
* Webhooks for external integrations
* GraphQL API alongside REST

---

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Create a feature branch (`git checkout -b feature/new-feature`)
2. Commit changes with clear messages (`git commit -m "Add new feature"`)
3. Push to the branch (`git push origin feature/new-feature`)
4. Open a pull request with detailed description

---

## Support

For issues, questions, or feature requests, please open an issue on the repository.

---

## Author

Built to demonstrate practical experience in **LLM evaluation, backend engineering, and AI alignment workflows**.

---

## License

MIT License

See [LICENSE](LICENSE) file for details.

---

## Acknowledgments

* Inspired by production LLM evaluation systems
* Community contributions and feedback
* Open-source libraries and frameworks

---

**Last Updated:** February 9, 2026
