MedRAG — Retrieval-Augmented Generation for Clinical Guidelines

A full-stack RAG system that grounds medical question answering in indexed NICE clinical guidelines. Built end-to-end: from PDF ingestion and vector storage, through semantic retrieval, to local LLM inference — all served through a lightweight Streamlit interface.

Why This Project

Clinical guideline documents are long, dense, and hard to query. Standard LLMs hallucinate when asked specific medical questions. This system solves both problems: it retrieves the most semantically relevant chunks from real NICE PDFs before prompting the model, grounding every answer in the indexed source.

The design rationale for choosing llama.cpp over higher-level inference frameworks was intentional: it surfaces low-level inference parameters — temperature, repetition penalties, Mirostat sampling — and makes the mechanics of LLM inference legible, not abstracted away.

System Architecture
High-level workflow:

Download medical guideline PDFs
Extract text from PDFs
Clean and preprocess text
Chunk documents
Generate embeddings
Store vectors in FAISS
Retrieve relevant chunks per query
Build RAG prompt
Generate answer using local LLM
Display results in Streamlit UI
This forms a full RAG lifecycle: Documents → Vectors → Retrieval → Prompt → LLM → UI.

Setup

1. Clone the repository
git clone https://github.com/<your-username>/medrag.git
cd medrag
2. Download the model
Download Meta-Llama-3.1-8B-Instruct-Q4_K_M.gguf from Hugging Face:
https://huggingface.co/joshnader/Meta-Llama-3.1-8B-Instruct-Q4_K_M-GGUF
Place it in the llm/ directory. A Hugging Face account and access token are required.

3. Create and activate a virtual environment
python3 -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate

4. Install dependencies
pip install -r requirements.txt

5. Run the app
streamlit run scripts/streamlit_app.py

Data Pipeline (one-time setup)
If building the FAISS index from scratch:

python scripts/extract_pdf_text.py        # Extract text from PDFs
python scripts/clean_text.py              # Clean and normalize
python scripts/clean_text_chunking.py     # Chunk into segments
python scripts/combine_chunks_embeddings.py  # Build embeddings + FAISS index

Example Queries
"How is sepsis detected during pregnancy?"
"What are the symptoms of depression?"
"What are the NICE recommendations for managing type 2 diabetes?"

Design Decisions

Why llama.cpp over Ollama or vLLM?

Deliberate choice. llama.cpp exposes the full inference parameter surface — temperature, repetition penalty, Mirostat — making the mechanics of token generation explicit. This was a learning goal, not just a technical means to an end.

Why FAISS over a managed vector DB?

FAISS runs entirely in-process, no external service required. For a single-machine learning setup with thousands of chunks, it's the right tradeoff between simplicity and performance.

Single-turn context window

The current implementation is one-prompt, one-response. Prior turns are not fed back into the LLM prompt. This is a known constraint and a candidate for future iteration using a conversational memory buffer.


Limitations


Not multi-turn: each query is independent; no conversation history is maintained.
Source coverage is limited to the NICE guidelines listed in sources.md.
The LLM is quantized (Q4_K_M) and runs on CPU — responses may be slow on lower-spec hardware.
Not for clinical use.



What I Learned


End-to-end RAG implementation from raw PDFs to a working assistant
Trade-offs in chunking strategy (token overlap, sentence boundaries) and their downstream impact on retrieval quality
FAISS index construction and similarity search mechanics
Prompt engineering for grounded, source-cited responses
LLM inference internals through llama.cpp: temperature scaling, sampling strategies, and how parameter choices affect output quality and diversity



Future Work


 Add conversational memory (multi-turn context)
 Hybrid retrieval: combine BM25 sparse retrieval with dense vector search
 Reranking layer (cross-encoder) to improve retrieval precision
 Evaluation pipeline: faithfulness and answer relevance scoring (RAGAS)
 Containerise with Docker for reproducible deployment



Contributions

Personal notes and comments are embedded throughout the scripts to support readability — both for others learning RAG systems and as a reference for myself. Feedback and suggestions are welcome.


License

MIT
