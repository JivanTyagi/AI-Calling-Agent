# 🤖 ☎️ AI Dia* **🤖 Text-to-text LLM:** understands the phone conversation, can make "function calls" and steers the conversation towards accomplishing specific tasks specified through a "system" message (OpenAI GPT-4o or Anthropic Claude Sonnet 3.5)
* **🧠 RAG (Retrieval Augmented Generation):** stores and retrieves conversation history per phone number using Qdrant vector database, enabling personalized conversations based on previous interactions
* **🔈 Text-to-Speech Service:** converts the LLM response to high-quality speechr 


## Summary
A full stack app for interruptible, low-latency and near-human quality AI phone calls built from stitching LLMs, speech understanding tools, text-to-speech models, and Twilio’s phone API

![UI Screenshot](examples/screenshot.png)

[Listen to example call](examples/sample.m4a)

## Features
The following components have been implemented and wrangled together in *streaming fashion* to achieve the tasks of *low-latency* and *interruptible* AI calls:
* **☎️ Phone Service:** makes and receives phone calls through a virtual phone number (Twilio)
* **🗣️ Speech-to-Text Service:** converts the caller’s voice to text (so that it can be passed to LLMs) and understands speech patterns such as when the user is done speaking and interruptions to facilitate interruptibility (Deepgram)
* **🤖 Text-to-text LLM:** understands the phone conversation, can make “function calls” and steers the conversation towards accomplishing specific tasks specified through a “system” message (OpenAI GPT-o or Anthropic Claude Sonnet 3.5)
* **🔈 Text-to-Speech Service:** converts the LLM response to high-quality speech
* **⚙️ Web Server:** A FastAPI-based web-server that provides end-points for:
   * Answering calls using Twilio’s Markup Language (Twilio ML),
   * Enabling audio streaming to/from Twilio through a per-call WebSocket
   * Interacting with the basic Steamlit web UI
* **📊 Frontend UI:** Simple Streamlit frontend to see initiate/end calls and view call progress in real-time in a browser


## Installation

### 1. Install dependencies:
(You might want to create a Python Virtual Environment to minimize the chance of conflicts.)
   ```
   pip install -r requirements.txt
   ```
### 2. Set up Qdrant Vector Database (for RAG functionality)

The system uses Qdrant for storing and retrieving conversation history per phone number. You can either:

**Option A: Use Docker (Recommended)**
```bash
# Start Qdrant using Docker Compose
docker-compose -f docker-compose.qdrant.yml up -d

# Verify Qdrant is running
curl http://localhost:6333/health
```

**Option B: Install Qdrant locally**
Follow the [official Qdrant installation guide](https://qdrant.tech/documentation/guides/installation/)

**Option C: Use Qdrant Cloud**
Sign up for [Qdrant Cloud](https://cloud.qdrant.io/) and get your API key and endpoint.

### 3. Set up `ngrok`
Twilio requires an externally accessible server to be able to route calls. To do this while running a local instance, you need to expose the server to the outside world. One way to do this is through using [`ngrok`](https://ngrok.com)

Run `ngrok` to get an external URL that forwards traffic to your local web server:

```
ngrok http 3000
```

Copy the URL that ngrok gives you (e.g. `1bf0-157-131-155-236.ngrok-free.app`) without the `https://` at the beginning and set that as your `SERVER` variable in the following section.

### 4. Configure .env file

Make a copy of the `.env.example` file and rename it to `.env`. Then set the required credentials and configurations.

Please note that you have a choice between `anthropic` and `openai` for the LLM service, and between `deepgram` and `elevenlabs` for the TTS service.

```
# Server Configuration
SERVER=your_server_here
# port number if you are running the server locally
PORT=3000

# Service API Keys

# Twlio
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token

# AI Services
## LLM
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key

## Speech Understanding/TTS
DEEPGRAM_API_KEY=your_deepgram_api_key

## TTS
ELEVENLABS_API_KEY=your_elevenlabs_api_key
ELEVENLABS_MODEL_ID=eleven_turbo_v2
ELEVENLABS_VOICE_ID=XrExE9yKIg1WjnnlVkGX

# Which service to use for TTS
TTS_SERVICE=elevenlabs

# Which service to use for LLM
LLM_SERVICE=openai

# When you call a number, what should the caller ID be?
APP_NUMBER=your_app_number

# When UI launches, what number should it call by default
YOUR_NUMBER=your_number

# When a call needs to be transferred, what number should it be transferred to?
TRANSFER_NUMBER=your_transfer_number

# AI Configuration
SYSTEM_MESSAGE="You are a representative called Sarah from El Camino Hospital. Your goal is to obtain a prior authorization for a patient called John Doe for a knee surgery. Be brief in your correspondence."
INITIAL_MESSAGE="Hello, my name is Sarah, and I'm calling from El Camino Hospital. I need to discuss a prior authorization for a patient. Could you please direct me to the appropriate representative?"

# Should calls be recorded? (this has legal implications, so be careful)
RECORD_CALLS=false

# Qdrant Vector Database Configuration (for RAG)
QDRANT_HOST=localhost
QDRANT_PORT=6333
QDRANT_API_KEY=  # Leave empty for local Qdrant, set for Qdrant Cloud
QDRANT_COLLECTION_NAME=phone_conversations
QDRANT_VECTOR_SIZE=384

# RAG Configuration
RAG_ENABLED=true
RAG_MAX_CONTEXT_CHUNKS=5
RAG_SIMILARITY_THRESHOLD=0.7
RAG_CHUNK_SIZE=500
RAG_CHUNK_OVERLAP=50
```

### 5. Initialize Qdrant Database

Set up the Qdrant collection for RAG functionality:

```bash
# Test Qdrant setup and RAG functionality
python setup_qdrant.py setup

# Optional: Run tests to verify everything works
python setup_qdrant.py test
```

### 6. Configure the Twilio end point
Assuming that you have created a [Twilio phone number](https://www.twilio.com/docs/phone-numbers) and installed Twilio's CLI, run the following to configure Twilio to use your app's endpoint:

```
twilio phone-numbers:update YOURNUMBER --voice-url=https://NGROKURL/incoming
```

### 7. Run the FastAPI server
```
python app.py
```

### 8. Run the Frontend server
```
streamlit ui/streamlit_app.py
```

## RAG (Retrieval Augmented Generation) Features

This implementation includes a sophisticated RAG system using Qdrant vector database that provides:

### 🧠 **Personalized Conversations**
- **Phone Number-based Context**: Each caller's conversation history is stored and retrieved based on their phone number
- **Cross-call Memory**: The AI remembers previous conversations and can reference them in new calls
- **Contextual Responses**: Relevant past interactions are automatically included in the AI's context

### 📚 **Smart Context Retrieval**
- **Semantic Search**: Uses sentence transformers to find semantically similar past conversations
- **Configurable Similarity**: Adjustable similarity thresholds to control context relevance
- **Chunked Storage**: Large conversations are intelligently split into overlapping chunks for better retrieval

### ⚙️ **Configurable Parameters**
- `RAG_ENABLED`: Enable/disable RAG functionality
- `RAG_MAX_CONTEXT_CHUNKS`: Maximum number of relevant contexts to retrieve (default: 5)
- `RAG_SIMILARITY_THRESHOLD`: Minimum similarity score for context inclusion (default: 0.7)
- `RAG_CHUNK_SIZE`: Size of conversation chunks for storage (default: 500 characters)
- `RAG_CHUNK_OVERLAP`: Overlap between chunks for better continuity (default: 50 characters)

### 🔧 **Management Tools**
```bash
# Setup and test RAG functionality
python setup_qdrant.py setup    # Initialize Qdrant collection
python setup_qdrant.py test     # Test RAG with sample data
python setup_qdrant.py info     # Show current configuration
```

### 💾 **Data Storage**
- **Conversation Turns**: Each user-assistant interaction is stored with metadata
- **Call Metadata**: Additional information about calls (duration, status, etc.)
- **Automatic Cleanup**: Configurable retention policies for old conversations

## Contribution
Contributions are welcome! Please feel free to submit a Pull Request.




## Acknowledgement 
This project would have not happened without [this great TypeScript example from Twilio Labs](https://github.com/twilio-labs/call-gpt). Claude Sonnet 3.5, GPT-4o, and [Aider](https://aider.chat) also provided ample help in writing parts of this code base 🦾

twilio phone-numbers:update +16419384316 --voice-url=https://102b-14-139-234-179.ngrok-free.app/incoming