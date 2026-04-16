# Java AI Agent with Spring AI & Amazon Bedrock

An enterprise-ready, stateful AI agent built with Java 25 and Spring Boot. This project demonstrates how to move beyond basic, stateless LLM interactions by implementing a fully featured AI assistant capable of memory, document grounding (RAG), and real-time tool calling.

## 🚀 Overview

Foundation models are powerful, but relying on them in isolation presents several challenges: they lack business context, have no memory of past interactions, cannot access proprietary data, and cannot trigger real-time external actions. 

This project solves these limitations by utilizing the **Spring AI** framework to build a comprehensive agent that features:
* **Custom Personas:** Tailored agent behavior and constraints via system prompts.
* **Persistent Conversation Memory:** Stateful, multi-turn conversations stored in PostgreSQL.
* **Retrieval-Augmented Generation (RAG):** Grounded responses using your own data stored in a `pgvector` database.
* **Tool Calling (Function Calling):** Seamless integration with external APIs for real-time data retrieval (e.g., dynamic weather and time zone information).

## 🛠️ Architecture & Tech Stack

* **Language:** Java 25
* **Framework:** Spring Boot 3.5.9
* **AI Orchestration:** Spring AI
* **LLM Provider:** Amazon Bedrock (Anthropic Claude Sonnet 4)
* **Embedding Model:** Amazon Titan (`amazon.titan-embed-text-v2:0`)
* **Database & Vector Store:** PostgreSQL with `pgvector` extension (Amazon Aurora Serverless)
* **Streaming:** Spring WebFlux (Reactor) for real-time token streaming

## 📋 Prerequisites

Before running this application, ensure you have the following:
* Java 25 installed
* Maven installed
* An AWS Account with access to **Amazon Bedrock** (specifically Claude Sonnet 4 and Titan Embedding models).
* AWS CLI configured with appropriate credentials.
* A running **PostgreSQL** instance with the `pgvector` extension installed.

## ⚙️ Getting Started

### 1. Clone the Repository
```bash
git clone <your-repository-url>
cd aiagent

2. Configure Environment Variables
The application relies on a PostgreSQL database for both conversation memory and vector storage. Set the following environment variables (in a production AWS environment, these are fetched from AWS Systems Manager and Secrets Manager):

Bash
export SPRING_DATASOURCE_URL=jdbc:postgresql://<your-db-host>:5432/<your-database>
export SPRING_DATASOURCE_USERNAME=<your-db-username>
export SPRING_DATASOURCE_PASSWORD=<your-db-password>

3. Build and Run
Start the Spring Boot application using the Maven wrapper:

Bash
./mvnw spring-boot:run
Note: The application will automatically initialize the required JDBC schemas for chat memory and pgvector on startup.

💻 API Usage
The application exposes REST endpoints to interact with the AI agent and manage the knowledge base.

Chatting with the Agent
The /invocations endpoint accepts a JSON payload and returns a streaming response (text/plain). It automatically tracks conversations based on the Authorization header.

Bash
curl -N -X POST http://localhost:8080/invocations \
  -H "Content-Type: application/json" \
  -H "Authorization: alex_user" \
  -d '{"prompt": "Hi, my name is Alex. What is the weather like in Las Vegas tomorrow?"}'
Persona: The agent is pre-configured to act as an assistant for "Unicorn Rentals".

Memory: If you send a follow-up request using the same Authorization header asking "What is my name?", the agent will remember "Alex".

Tools: The agent will automatically recognize the request for weather, utilize the built-in WeatherTools and DateTimeTools to fetch live data from the Open-Meteo API, and synthesize the final answer.

Loading Documents into the Knowledge Base (RAG)
You can ingest external knowledge into the PostgreSQL vector store using the /load endpoint. The text is automatically chunked, embedded using Amazon Titan, and saved.

Bash
curl -X POST http://localhost:8080/load \
  -H "Content-Type: text/plain" \
  -d "Unicorns have origins in different cultures: Chinese Qilin, Indian unicorn seals, Greek accounts"
Once loaded, subsequent queries about "unicorn origins" will be grounded in this specific text using the QuestionAnswerAdvisor.

📂 Project Structure
Plaintext
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
