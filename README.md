# AI Video Assistant 🎬

An AI-powered video and meeting assistant that turns a YouTube video or local audio/video file into a transcript, summary, and useful meeting insights. It also lets you ask questions about the transcript using Retrieval-Augmented Generation (RAG).

## Features

- **YouTube and local media input** through the Streamlit interface.
- **Audio preprocessing** with `yt-dlp`, PyDub, and FFmpeg: download/extract audio, convert it to mono 16 kHz WAV, and split long recordings into smaller chunks.
- **Speech-to-text with Whisper** running locally for the English transcription route.
- **AI-generated title and summary** using Google Gemini through LangChain.
- **Meeting insight extraction** for action items, owners/deadlines when mentioned, key decisions, and open questions/follow-ups.
- **RAG-based Q&A** using Hugging Face sentence embeddings and ChromaDB to retrieve relevant transcript passages before Gemini generates an answer.
- **Streamlit interface** for analysis results and follow-up questions.

> The current code also contains a Hinglish transcription/translation route using Sarvam. It needs a Sarvam API key. If you intend this project to be English-only, remove the Hinglish option and Sarvam code from `app.py` and `core/transcriber.py` before describing it as English-only.

## Architecture

```text
YouTube URL / Local Audio or Video
              |
              v
      Audio Processing
 Download -> Convert -> Chunk
              |
              v
      Transcription Route
   English: Local Whisper
   Hinglish: Sarvam API (optional)
              |
              v
       Full Transcript
          /       \\
         v         v
  Title + Summary   Action Items / Decisions / Questions
          \\
           v
    Text Chunking + Embeddings
              |
              v
           ChromaDB
              |
         User Question
              |
              v
   Retrieve relevant transcript chunks
              |
              v
       Gemini generates answer
```

## Tech Stack

- **Language:** Python
- **UI:** Streamlit
- **Transcription:** OpenAI Whisper, PyTorch
- **Media processing:** `yt-dlp`, PyDub, FFmpeg
- **LLM and orchestration:** Google Gemini, LangChain, LCEL
- **Embeddings:** Hugging Face Sentence Transformers (`all-MiniLM-L6-v2`)
- **Vector database:** ChromaDB
- **Configuration:** `python-dotenv`
- **Optional Hinglish route:** Sarvam speech-to-text-translate API

## Project Structure

```text
video-rag-assistant/
├── core/
│   ├── extractor.py       # Action items, decisions, and open questions
│   ├── rag_engine.py      # Retrieval + prompt + Gemini answer chain
│   ├── summarizer.py      # Title generation and transcript summarization
│   ├── transcriber.py     # Whisper and optional Sarvam transcription
│   └── vector_store.py    # Embeddings and ChromaDB
├── utils/
│   └── audio_processor.py # Download, convert, and split audio
├── vector_db/             # Local ChromaDB data (generated at runtime)
├── app.py                 # Streamlit interface
├── main.py                # Pipeline orchestration / CLI entry point
├── Requirements.txt       # Python dependencies
├── .env.example           # Safe environment-variable template
├── .gitignore
└── LICENSE
```

## Setup

### 1. Prerequisites

- Python 3.10 or later (Python 3.12 was used during development).
- FFmpeg installed and available on your system `PATH`. PyDub and the YouTube audio conversion step rely on it.
- A Google Gemini API key.
- A Sarvam API key only if you want to use the current Hinglish route.

### 2. Clone the repository

```bash
git clone https://github.com/MohamedAli077/video-rag-assistant.git
cd video-rag-assistant
```

### 3. Create and activate a virtual environment

**Windows PowerShell**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Install dependencies

The dependency file is named `Requirements.txt` (capital **R**):

```bash
python -m pip install -r Requirements.txt
```

The dependency list should explicitly include `langchain-chroma` and `langchain-text-splitters`, which are imported by the code. The companion `Requirements.txt` in this documentation bundle includes both.

### 5. Configure environment variables

Copy `.env.example` to `.env` and add your own keys:

```env
GOOGLE_API_KEY=your_google_gemini_api_key
WHISPER_MODEL=small

# Needed only for the Hinglish/Sarvam route:
SARVAM_API_KEY=your_sarvam_api_key
SARVAM_STT_MODEL=saaras:v2.5
```

Do not commit `.env` or real API keys to GitHub.

### 6. Run the Streamlit app

```bash
streamlit run app.py
```

Open the local URL printed in the terminal (usually `http://localhost:8501`). In the sidebar, enter a YouTube URL or a local audio/video file path, select the language route, and click **Analyse**.

You can also run the command-line pipeline:

```bash
python main.py
```

## How RAG Works in This Project

1. The transcript is split into smaller, overlapping text chunks.
2. A Hugging Face sentence-transformer model converts the chunks into embedding vectors.
3. ChromaDB stores each vector together with its text and metadata.
4. When a user asks a question, the retriever selects the most relevant transcript chunks.
5. The retrieved context and question are passed to Gemini, which generates an answer grounded in the transcript.

## Notes and Limitations

- Whisper runs locally, but title generation, summarization, insight extraction, and RAG answers use the Gemini API and require a valid key and internet access.
- The first Whisper run may download model weights. Smaller models are generally faster on CPU, while transcription quality can vary by model and recording quality.
- YouTube download compatibility can change; keep `yt-dlp` updated and make sure FFmpeg is installed.
- AI-generated summaries and extracted action items/decisions should be checked against the original transcript.
- `vector_db/`, downloaded media, generated WAV files, and Python cache folders are local/runtime artifacts and should not be committed.

## Security

Never publish API keys, credentials, private meeting recordings, or transcripts containing personal or confidential information. Keep `.env`, virtual environments, generated media, Python cache files, and the local vector database out of version control.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
