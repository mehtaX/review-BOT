# Flipkart Product Review Chatbot

An intelligent, RAG (Retrieval-Augmented Generation) based conversational chatbot designed to analyze customer reviews of Flipkart products. The application enables users to query review datasets interactively, receiving context-aware product recommendations and summary analysis through a clean, web-based chat interface.

---

## 🚀 Key Features

*   **RAG Architecture**: Combines vector retrieval with generative Large Language Models to answer queries accurately based on actual product reviews.
*   **Conversational Memory**: Retains session history for contextual, multi-turn dialogue (e.g., answering follow-up questions).
*   **Vector Database Integration**: Uses **DataStax Astra DB** as a scalable cloud vector database.
*   **High Performance LLM & Embeddings**: Uses **Groq API** (Llama 3.1 70B) for fast text generation and **Hugging Face Inference API** (`BAAI/bge-base-en-v1.5`) for high-quality text embeddings.
*   **Interactive Web UI**: A lightweight, responsive front-end built using Flask, HTML/CSS, and jQuery.

---

## 🛠️ Technology Stack

*   **Backend Framework**: [Flask](https://flask.palletsprojects.com/)
*   **AI Orchestration**: [LangChain](https://www.langchain.com/)
*   **Vector Store**: [DataStax Astra DB](https://www.datastax.com/products/astra-db)
*   **Embeddings Provider**: [Hugging Face](https://huggingface.co/)
*   **LLM Provider**: [Groq Cloud](https://groq.com/)
*   **Frontend**: HTML5, Vanilla CSS, Bootstrap 4, jQuery

---

## 📂 Project Structure

```text
Flipkart-product-review-chatbot/
│
├── data/
│   └── flipkart_product_review.csv  # CSV dataset containing Flipkart product titles and reviews
│
├── flipkart/                         # Core package folder
│   ├── __init__.py                  # Packages package init file
│   ├── data_converter.py            # Script to parse CSV reviews into LangChain Document objects
│   ├── data_ingestion.py            # Inserts document vectors and queries the Astra DB vector store
│   └── retrieval_generation.py      # Establishes history-aware retrieval and conversational RAG chains
│
├── static/
│   └── style.css                    # Styling sheet for the Chatbot UI
│
├── templates/
│   └── index.html                   # HTML template containing the chat interface
│
├── .dockerignore                    # List of files ignored during Docker image builds
├── .gitignore                       # List of files untracked by Git (.env, venv, etc.)
├── app.py                           # Flask server and main application entry point
├── aws.md                           # AWS cloud manual deployment documentation
├── Dockerfile                       # Instructions to build a Docker container for the app
├── requirements.txt                 # List of python dependency packages
├── setup.py                         # Packages python project locally as 'flipkart'
└── template.py                      # Initialization script for codebase files/directories
```

---

## ⚙️ Local Setup and Installation

### Prerequisites

*   Python 3.10 or higher
*   [Anaconda / Miniconda](https://www.anaconda.com/download) (Recommended for package management)
*   API keys for:
    *   **Groq Cloud**
    *   **DataStax Astra DB**
    *   **Hugging Face**

---

### Step 1: Create a Conda Environment

Open your terminal or command prompt and create a clean environment:

```bash
conda create -n flipkart-bot python=3.10 -y
```

Activate the environment:

```bash
conda activate flipkart-bot
```

---

### Step 2: Install Project Dependencies

Install the requirements along with the local `flipkart` module package in editable mode:

```bash
pip install -r requirements.txt
```

---

### Step 3: Configure Environment Variables

Create a file named `.env` in the root of the project directory and fill in your API credentials:

```env
GROQ_API_KEY=gsk_your_actual_groq_api_key
ASTRA_DB_API_ENDPOINT=https://your-database-id.apps.astra.datastax.com
ASTRA_DB_APPLICATION_TOKEN=AstraCS:your_actual_astra_token
ASTRA_DB_KEYSPACE=default_keyspace
HF_TOKEN=hf_your_actual_huggingface_token
```

---

## 🏃 Running the Application

### 1. Ingest Data (First-Time Run Only)

To embed the product reviews and upload them to your Astra DB vector store, execute the data ingestion module:

```bash
python flipkart/data_ingestion.py
```

*Note: In `data_ingestion.py`, setting the status argument of `data_ingestion(None)` will parse and insert the CSV records. Once loaded, the script passes `"done"` to bypass repeating ingestion.*

---

### 2. Launch the Web Interface

Start the Flask server:

```bash
python app.py
```

Once running, the application will be hosted on `http://localhost:5000`. Navigate to this URL in your web browser to start chatting with the bot about Flipkart product reviews!

---

## 🐳 Docker Containerization

To package and run the application locally inside a Docker container:

1. **Build the Docker Image**:
   ```bash
   docker build -t flipkart-chatbot .
   ```

2. **Run the Docker Container**:
   Make sure to pass your environment variables:
   ```bash
   docker run -d -p 5000:5000 \
     --name chatbot-container \
     -e GROQ_API_KEY="your_groq_api_key" \
     -e ASTRA_DB_API_ENDPOINT="your_astra_db_endpoint" \
     -e ASTRA_DB_APPLICATION_TOKEN="your_astra_token" \
     -e ASTRA_DB_KEYSPACE="default_keyspace" \
     -e HF_TOKEN="your_hf_token" \
     flipkart-chatbot
   ```

3. Access the application in your browser at `http://localhost:5000`.

---

## ☁️ Production Deployment

To deploy this application to **AWS EC2** using **Amazon ECR** manually, please refer to the step-by-step instructions in the [AWS Cloud Deployment Guide](file:///c:/Users/hp/Desktop/everything/projectsss/Flipkart-product-review-chatbot/aws.md).
