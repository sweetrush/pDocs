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


#### Multimodel RAG
	- input --> Embeddings Model --> storage --> Retrieval + Generation



#### Using RAG 
- Code Examples support

Extracting data from PDF Code Snipit 
```python


from unstructured.partition.pdf import partition_pdf
pdf_file_name = "NameOfthePDFFile.pdf"

# Extracting the tables, chunk Text from input PDF 

raw_elements = partition_pdf(
	filename=pdf_file_name,
	chucking_strategy="by_title",
	infer_table_structure=True,
	max_characters=1000,
	new_after_n_chars=1500,
	combine_text_under_n_chars=250,
	strategy="hi_res"
	);

# Extract images from input PDF 
pdf_file_path="/path/to/the/imagefile.pdf"
extract_images_from_pdf(pdf_file_path)

```

Organize text from document

tables = []
texts = []
for element in raw_elements:
	if "unstructured.document.elements.Table" in str(type(element)):
		tables.append(str(element))
	elif "unstructured.documents.elements.CompositeElement" in str(type(element)):
	    texts.append(str(element))
