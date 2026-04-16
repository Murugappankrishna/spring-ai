# 🚀 AI Agent - Build, Run, and Usage Guide

## 🛠️ 3. Build and Run

Start the Spring Boot application using the Maven wrapper:

```bash
./mvnw spring-boot:run
```

**Note:**
The application will automatically initialize the required JDBC schemas for:

* Chat memory
* pgvector (for embeddings)

---

## 💻 API Usage

The application exposes REST endpoints to interact with the AI agent and manage the knowledge base.

---

### 🤖 Chatting with the Agent

The `/invocations` endpoint:

* Accepts a JSON payload
* Returns a **streaming response** (`text/plain`)
* Tracks conversations using the `Authorization` header

#### Example:

```bash
curl -N -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -H "Authorization: alex_user" \
  -d '{"prompt": "Hi, my name is Alex. What is the weather like in Las Vegas tomorrow?"}'
```

#### 🧠 Features

* **Persona**
  The agent acts as an assistant for **"Unicorn Rentals"**

* **Memory**
  If you send a follow-up request with the same `Authorization` header:

  ```text
  What is my name?
  ```

  The agent will remember:

  ```text
  Alex
  ```

* **Tools Integration**
  The agent automatically:

  * Detects intent (e.g., weather queries)
  * Uses:

    * `WeatherTools`
    * `DateTimeTools`
  * Fetches real-time data from APIs (like Open-Meteo)
  * Synthesizes a final response

---

### 📚 Loading Documents into Knowledge Base (RAG)

Use the `/load` endpoint to ingest external knowledge into PostgreSQL vector store.

#### Example:

```bash
curl -X POST http://localhost:8080/load \
  -H "Content-Type: text/plain" \
  -d "Unicorns have origins in different cultures: Chinese Qilin, Indian unicorn seals, Greek accounts"
```

#### ⚙️ What Happens Internally

* Text is **chunked**
* Converted into **embeddings** (Amazon Titan)
* Stored in **pgvector**
* Used later for **retrieval-augmented generation (RAG)**

#### 🔍 Result

Future queries like:

```text
What are the origins of unicorns?
```

Will be grounded using the stored data via `QuestionAnswerAdvisor`.

---

## 📂 Project Structure

```plaintext
aiagent/
├── src/main/java/com/example/agent/
│   ├── AgentApplication.java      # Spring Boot entry point
│   ├── ChatService.java           # Core AI logic (Client, Advisors, Memory, RAG)
│   ├── InvocationController.java  # REST APIs for chat and document ingestion
│   ├── InvocationRequest.java     # Request DTO
│   ├── DateTimeTools.java         # @Tool for live time-zone retrieval
│   └── WeatherTools.java          # @Tool for live weather API retrieval
├── src/main/resources/
│   ├── application.properties     # Bedrock & DB configuration
│   └── static/                    # Frontend Web UI assets (HTML/CSS/JS)
└── pom.xml                        # Maven dependencies
```

---

