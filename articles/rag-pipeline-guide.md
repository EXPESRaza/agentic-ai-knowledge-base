# 🧠 RAG Pipeline Guide – From Load to LLM

Welcome to this comprehensive, beginner-friendly guide on **Retrieval-Augmented Generation (RAG)**. This guide breaks down the complete RAG architecture in a clear, easy-to-follow sequence — from ingesting documents to generating intelligent responses with Large Language Models (LLMs).

---

## 🚀 What is RAG?

**RAG (Retrieval-Augmented Generation)** is a framework where language models are enhanced by retrieving relevant data from external knowledge sources like documents, PDFs, or databases. This helps generate **factual**, **context-aware**, and **up-to-date** answers — without relying solely on a model’s memory.

---

## 🧭 The Big Picture – Step-by-Step RAG Workflow

```
┌────────────────┐
│ Your Raw Data  │ (PDFs, CSVs, Text, etc.)
└──────┬─────────┘
       ▼
┌────────────────────┐
│ Document Loaders   │ → Converts data into LangChain docs
└──────┬─────────────┘
       ▼
┌────────────────────┐
│ Text Splitters     │ → Breaks data into manageable chunks
└──────┬─────────────┘
       ▼
┌────────────────────┐
│ Embedding Models   │ → Converts text chunks into numerical vectors
└──────┬─────────────┘
       ▼
┌────────────────────┐
│ Vector Database    │ → Stores embeddings for fast retrieval
└──────┬─────────────┘
       ▼
┌────────────────────┐
│ Similarity Search  │ → Finds relevant chunks for a user’s question
└──────┬─────────────┘
       ▼
┌───────────────────────┐
│ LLM (OpenAI, Gemini)  │ → Generates final answer using retrieved context
└───────────────────────┘
```

---

## 1. 📥 Data Ingestion

Use document loaders to read data.

```python
from langchain_community.document_loaders import PyPDFLoader

loader = PyPDFLoader("data/llama2.pdf")
pages = loader.load()
```

👉 You can use `.txt`, `.md`, `.csv`, `.docx`, `.html`, and more!

---

## 2. ✂️ Text Chunking

Split large text into small, overlapping chunks using a text splitter.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
split_docs = splitter.split_documents(pages)
```

🔍 **Why chunking?** Because LLMs have token limits and vector search works best with atomic pieces of text.

---

## 3. 🔢 Embedding Text

Embeddings turn text into vectors (numerical form) that capture semantic meaning.

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-large")
query_result = embeddings.embed_query("This is a tutorial on embeddings")
```

📏 Output: A vector of 3072 dimensions!

💡 **Alternatives**: HuggingFace, Cohere, Azure, etc.

```python
from langchain_huggingface import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
query_result = embeddings.embed_query("This is a tutorial on embeddings")
```

📏 Output: A vector of 384 dimensions!

---

## 4. 💾 Storing in Vector DB

Use FAISS, Chroma, Weaviate, or others.

```python
from langchain.vectorstores import FAISS
from langchain.docstore import InMemoryDocstore
import faiss

index = faiss.IndexFlatIP(384)
vector_store = FAISS(
    embedding_function=embeddings,
    index=index,
    docstore=InMemoryDocstore(),
    index_to_docstore_id={},
)

vector_store.add_documents(split_docs)
```

🧠 **Vector DB** stores vectors and helps in fast nearest-neighbor search.

---

## 5. 🔍 Similarity Search

Retrieve similar documents based on your question.

```python
retriever = vector_store.as_retriever(search_kwargs={"k": 5})
retrieved_docs = retriever.invoke("What is LLaMA model?")
```

🔢 Uses **cosine similarity**, **L2 distance**, or **dot product** to find best matches.

---

## 6. 🧠 Final Answer via LLM

Use OpenAI, Google Gemini, or Claude to generate the final answer from retrieved context.

```python
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain import hub
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

model = ChatGoogleGenerativeAI(model="gemini-1.5-flash")
prompt = hub.pull("rlm/rag-prompt")

def format_docs(docs):
    return "

".join(doc.page_content for doc in docs)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

rag_chain.invoke("What is the LLaMA model?")
```

---

## 🔁 Bonus: Why Not SQL/NoSQL?

You can’t run cosine similarity in SQL or NoSQL.

| Feature           | SQL/NoSQL | Vector DB |
| ----------------- | --------- | --------- |
| Similarity Search | ❌        | ✅        |
| Fast retrieval    | ✅        | ✅        |
| Vector support    | ❌        | ✅        |

---

## 📚 RAG FAQs

**Q: Can I use RAG without OpenAI?**  
A: Absolutely! HuggingFace models and local vector DBs like FAISS make it open-source friendly.

**Q: Is embedding model size important?**  
A: Yes. OpenAI models like `text-embedding-3-large` offer 3072 dimensions, which is more contextually rich than smaller ones.

**Q: What is the best vector database?**  
A: It depends on use case:

- For local → FAISS, Chroma
- For production → Pinecone, Weaviate, Qdrant

---

## ✅ Summary

**Retrieval-Augmented Generation (RAG)** enhances LLMs by giving them **contextual knowledge** retrieved from your private data using **vector databases**. You now know how to:

1. Load and chunk your data
2. Embed with models
3. Store and query vector DBs
4. Generate final responses with an LLM

---

Happy RAG-ing! 🧠⚙️  
Want more guides? [Follow the repo!](https://github.com/EXPESRaza/agentic-ai-knowledge-base)
