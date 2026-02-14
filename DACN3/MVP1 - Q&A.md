## Pgvector in RAG (Retrieval-Augmented Generation)

Used heavily with LLM systems.

Typical flow:

1. User asks question
    
2. Convert question → embedding
    
3. Query pgvector
    ![[Pasted image 20260214175712.png]]
4. Send top results to LLM

Dataflow:
	User Query
	     ↓
	Embedding Model
	     ↓
	Postgres + pgvector
	     ↓
	Top K Results
	     ↓
	LLM

Embedding model: Jina model
https://medium.com/@intuitivedl/the-ultimate-guide-to-using-pgvector-76239864bbfb