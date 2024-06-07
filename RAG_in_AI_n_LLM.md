### RAG in AI/LLM 

#### Components of a RAG
- Embeddings 
	- Floating point vectors that represent text or other data. Embeddings capature semantic meaning and context which results in text with smilar meanings having closer embeddings.
	- Input --> Text Embeddings Model --> Embeddings Values 
	- capturing relationships on the context

- An embedding code example 
```python

import google.generativeai as genai 

model = "models/embedding-001"
embedding = genai.embed_content(model=model, context="Your information to be embedded")

print(f'{embedding}')
```

- Vector Search 
	- Store data in a speciallised vector database, optimized for fast lookups 
	- Query --> search --> Rank
	- Types of Search
		- Brute force search 
		- Approximate nearest neighbor search
- LLM
	- Send retrieved document chucks to LLMs like the Gemini models to summarise a response



