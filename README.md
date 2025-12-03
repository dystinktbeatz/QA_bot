## Project overview

This repository contains a Retrieval-Augmented Generation (RAG) Question-Answering (QA) bot that:

* Loads one or more PDF documents.
* Splits document text into chunks for embedding.
* Generates embeddings using IBM watsonx embedding models.
* Stores embeddings in a Chroma vector database.
* Retrieves relevant chunks using similarity search.
* Uses an IBM watsonx LLM (via LangChain) to generate grounded answers.
* Serves a simple web UI using Gradio where users can upload a PDF and ask questions.

The project demonstrates a complete RAG pipeline using `langchain`, `langchain-ibm`, `langchain-community`, `ibm-watsonx-ai`, and `chroma`.

---

## Goals & learning objectives

By following this project you (or a reader) will be able to:

* Combine document loaders, text splitters, embedding models and vector databases to build a QA bot.
* Understand how retrieval augments generation to produce grounded answers from documents.
* Configure and use IBM watsonx embeddings and LLMs inside LangChain.
* Deploy a minimal Gradio front-end to expose the RAG system.

---

## Repository contents (recommended structure)

```
qabot/                         # root project folder
├─ qabot.py                    # main application (RAG pipeline + Gradio app)
├─ requirements.txt            # pinned deps (optional)
├─ README.md                   # this file
├─ examples/                   # (optional) sample PDFs for quick testing
│   └─ sample_document.pdf
├─ notebooks/                  # (optional) helper notebooks / tests
└─ scripts/                    # (optional) helper scripts (embedding tests, etc.)
```

---

## Prerequisites

* Python 3.11 (recommended) or compatible Python 3.x.
* An IBM watsonx account and credentials (API keys/credentials) to access watsonx LLM and embedding models.
* (Optional) Hugging Face token if you use any HF-hosted models or helpers.
* Basic familiarity with pip, virtual environments, and running Python scripts.

---

## Installation

1. **Clone the repository and enter the project directory**

```bash
git clone <your-repo-url>
cd qabot
```

2. **Create and activate a virtual environment**

```bash
pip install virtualenv
virtualenv my_env
source my_env/bin/activate
```

3. **Install the required packages** (versions suggested by the course)

```bash
python3.11 -m pip install \
  gradio==4.44.0 \
  ibm-watsonx-ai==1.1.2 \
  langchain==0.2.11 \
  langchain-community==0.2.10 \
  langchain-ibm==0.1.11 \
  chromadb==0.4.24 \
  pypdf==4.3.1 \
  pydantic==2.9.1 \
  huggingface_hub==0.23.0
```

> ⚠️ Pin package versions if you need reproducibility. Versions above were used in the course materials — you may update them carefully.

---

## Configuration / Credentials

1. **IBM watsonx credentials**: follow the `ibm-watsonx-ai` documentation to set the required environment variables or local credential files. Typical items:

   * API key / IAM credentials
   * Service URL (e.g. `https://us-south.ml.cloud.ibm.com`)
   * Project ID (course used `skills-network`)

2. **Hugging Face token** (optional): store with `huggingface_hub.login()` or by setting `HUGGINGFACE_HUB_TOKEN` if you plan to use HF-hosted artifacts.

Keep secrets out of the repo. Use environment variables, `.env` files (gitignored), or secret stores.

---

## How it works (component summary)

1. **Document loader**: `PyPDFLoader` loads PDF pages and turns them into LangChain `Document` objects.
2. **Text splitter**: `RecursiveCharacterTextSplitter` chops long documents into chunks; default parameters used in this project are `chunk_size=1000`, `chunk_overlap=200`, `length_function=len`.
3. **Embedding model**: `WatsonxEmbeddings` (IBM slate model) converts each chunk into a numeric vector.
4. **Vector DB**: `Chroma.from_documents` stores vectors and supports similarity search.
5. **Retriever**: `vectordb.as_retriever()` returns an object able to find relevant chunks for a query.
6. **QA Chain**: `RetrievalQA.from_chain_type` composes the retriever with the LLM (via `WatsonxLLM`) to answer user queries. The course example uses `chain_type='stuff'` and `return_source_documents=True`.
7. **Frontend**: Gradio (`gr.Interface`) provides a simple UI with: file upload + textbox → response box. The app is launched on `0.0.0.0:7860` for cloud IDE accessibility.

---

## Running the application

1. Make sure credentials are set and virtual environment is active.
2. Run the main application file (example name `qabot.py`):

```bash
python qabot.py
```

3. The script launches a Gradio UI by default at `http://0.0.0.0:7860` (or a local preview URL provided by your cloud IDE). Upload a PDF, type your question, and submit.

---

## Example usage / quick checks

* **Test embedding** (quick script):

  ```python
  from qabot import watsonx_embedding
  emb = watsonx_embedding()
  vectors = emb.embed_documents(["This is a test sentence."])
  print(vectors[0][:5])  # first five numbers of the embedding
  ```

* **Test pipeline manually**:

  1. Load sample PDF with `document_loader()`
  2. Split with `text_splitter()`
  3. Build vectordb with `vector_database()`
  4. Create retriever and use `RetrievalQA` as in `retrainer_qa()`

---

## Recommended improvements & tips

* **Persist Chroma DB**: by default Chroma may be in-memory. Configure a persistent directory if you want to reuse embeddings across runs and avoid re-embedding each time.
* **Chain types**: if your documents + retrieved chunks exceed LLM context, use `map_reduce` or `refine` instead of `stuff`.
* **Chunk sizing**: tune `chunk_size` and `chunk_overlap` for your documents: code/manuals may need shorter chunks for precise answers.
* **Rate limits & costs**: monitor API usage & LLM generation tokens. Use lower `MAX_NEW_TOKENS` for tests.

---

## Troubleshooting

* **No response / API errors**: verify IBM credentials, endpoint, and project_id are correctly configured.
* **Slow startup**: embedding large docs can take time on first run — consider adding progress logging.
* **Wrong or hallucinated answers**: ensure retriever retrieves correct context; verify returned source documents are relevant and tune retriever or chunking.

---

## Screenshots & course deliverables

The original course requested screenshots for the following steps (optional but useful for records):

* `pdf_loader` — screenshot showing the code used to load PDFs.
* `code_splitter.png` — screenshot showing the code used to split documents.
* `embedding.png` — screenshot showing a sample embedded sentence and the first 5 numbers of the vector.
* `vectordb.png` — screenshot showing the code that builds the Chroma DB.
* `retriever.png` — screenshot showing the retriever function.

Place these screenshots under `docs/screenshots/` if you plan to submit or archive the course lab work.

---

## Security & privacy notes

* Do **not** commit API keys or credentials. Use environment variables or gitignored `.env` files.
* Uploaded PDFs may contain sensitive information — do not deploy this app publicly without access controls.

---

## License & attribution

Add your preferred license file (e.g., `LICENSE`). If this code is based on Coursera course materials, include an acknowledgment such as:

> Parts of this project follow the IBM "RAG and Agentic AI" Coursera course materials. This repository is the author's implementation for learning and demonstration purposes.

---

## Contact / contributions

If you want to improve this project:

* Open a GitHub Issue describing the problem or enhancement.
* Create a PR with small focused changes and tests where appropriate.

---

