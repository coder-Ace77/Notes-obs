 
---


**LangChain** is an **open-source framework** designed to make it easier to build applications powered by **large language models (LLMs)** by **connecting them with external data, tools, and workflows**. 

The document we want our llm to be aware of is sliced into chunks and chunks are stored in vector db in form of embeddings. The vector db can be queired to find the relevant part from text and this is passed with the question/instruction to the language as prompt.

## Langchain components

#### LLM wrappers

Allow us to connect to llms to code.

#### Prompt template

Allow us to create dynamic templates which are inputs to the llms

#### Index

They halp us in extracting the relevant information to allow us to get the context.
#### Memory

Storing and retrieving data in between a conversation. There are two kinds of memory short term memory refers to memory in the conversation and long term memory allow us to retrive the information between conversations.

#### Chains

They allow us to combine multiple components together to solve the specific task and build entire llm

#### Agent

Facilitate the interaction between llm and apis.

### Chat gpt completion api

It simulates a **chat between roles** (user, system, assistant) — giving structure to the conversation.Essentially, it’s how you send **a conversation history** to the model and get back a **response**.

Unlike the older **Completions API** (which took plain text prompts), the **Chat API** expects a **list of messages**.

Each message has:
- a **role** (system / user / assistant / tool)
- a **content** (the text of the message)

Example:

```python
[
  {"role": "system", "content": "You are a helpful assistant."},
  {"role": "user", "content": "Explain LangChain in simple terms."}
]
```
`
 - system -  defines the assistant’s personality or behavior (e.g., “You are a math tutor.”)
 - user - represents user queries or input
 - assisatant - represents AI's previous responses

The model considers **all messages together** — not just the last one — to maintain conversation context.

One can also stream the responses live using param stream=True

```python
for chunk in client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Write a short poem on AI"}],
    stream=True,):
    print(chunk.choices[0].delta.content or "", end="")
```

Structure of response structure:

```python
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1698424442,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "LangChain is a framework for connecting LLMs..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 32,
    "total_tokens": 57
  }
}
```

example of the api

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    temperature=0.3,
    max_tokens=150,
    messages=[
        {"role": "system", "content": "You are a concise assistant."},
        {"role": "user", "content": "Explain how photosynthesis works."}
    ]
)
print(response.choices[0].message.content)
```

Now with langchain api we have to use `invoke` method. Mainly with langchain api we can 

```python
res = llm.invoke(PROMPT
res.content # text in here
```

Also with streaming we can create streaming object and flush the outputs

```python
llm = ChatOpenAI(
		model="gpt-5-nano",
		api-key=OPENAI_API_KEY,
		streaming=True
	)
	
for chunk in llm.stream("Explain the autgrad in 10 words?")
	print(chunk.content,end="",flush=True)	
```

Now to use `messages` in the langchain we have to import them first 

```python
from langchain.schema import (SystemMessage,HumanMessage,AIMessage)

messages = [
    SystemMessage("You are physicist and respond only in german"),
    HumanMessage("Explain quantum mechanics in one sentence?")
]

output = llm.invoke(messages)

```

AIMessage is equivalent to assistant message. HumanMessage is user message. 

### Caching the calls

We can cache the api calls so that if same request is send again and again we get stored output.
Now there are two caches which langchain provides us--

1. In memory cache
2. Sql lite cache

Memory cache
```python
from langchain.globals import set_llm_cache
from langchain_openai import OpenAI
llm = OpenAI(model_name='')
```

```python
from langchain.cache import InMemoryCache
set_llm_cache(InMemoryCache())
prompt = ""

llm.invoke(prompt)
```

SqlLite chace

```python
from langchain.cache import SQLiteCache
set_llm_cache(SQLiteCache(database_path=""))
llm.invoke("")  # cached
```

## Prompt templates

Prompt template is simply a piece of text in which we can inject the user's input. 
1. PromptTemplates
2. ChatPromptTemplates

```python
from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

template = """You are experienced in {language} and are at {postiion} in the company. Give answer on my behalf."""

prompt_template = PromptTemplate.from_template(template=template)

prompt = prompt_template.format(language="language",position="position")

# now prompt is the prompt object having the text with value

output = llm.invoke(prompt)
```

## Simple chains

A **Chain** in LangChain is simply a **sequence of steps** that connect different components (like LLMs, prompts, tools, retrievers, etc.) together into a **workflow**.

Each step in the chain takes some input → performs an operation (like calling an LLM or retrieving documents) → and passes the output to the next step.

Think of it like a **data pipeline** for reasoning with an LLM. 

#### 1. **LLMChain**

- Simplest and most common chain.
    
- Combines a **prompt** and an **LLM model**.
    
- You give it input variables, and it runs the LLM on a formatted prompt.

```python
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

llm = ChatOpenAI(model="gpt-4o-mini")

prompt = PromptTemplate(
    input_variables=["topic"],
    template="Write a short poem about {topic}."
)

chain = LLMChain(llm=llm, prompt=prompt)

result = chain.invoke({"topic": "rain"})
print(result["text"])
```

#### . **SequentialChain**

- Used when you want to **combine multiple chains** in sequence.
- Output of one chain becomes input of the next.

```python
from langchain.chains import SimpleSequentialChain

# Step 1: Generate a title
title_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate(
        input_variables=["topic"],
        template="Generate a catchy title about {topic}."
    )
)

# Step 2: Write a short blog post based on that title
blog_chain = LLMChain(
    llm=llm,
    prompt=PromptTemplate(
        input_variables=["title"],
        template="Write a 100-word blog post titled '{title}'."
    )
)

# Combine
overall_chain = SimpleSequentialChain(chains=[title_chain, blog_chain])
print(overall_chain.invoke("Artificial Intelligence"))
```

#### **RetrievalQAChain**

- Used for **retrieval-augmented generation (RAG)**.
- It connects a **retriever** (that fetches documents) with an **LLM** that answers questions based on those documents.

```python
from langchain.chains import RetrievalQA
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
db = FAISS.load_local("vector_index", embeddings)

retriever = db.as_retriever()
qa_chain = RetrievalQA.from_chain_type(llm=llm, retriever=retriever)

response = qa_chain.invoke({"query": "What is LangChain?"})
print(response["result"])
```

## Agents

If **chains** are _fixed pipelines_, then **agents** are _intelligent decision-makers_ that dynamically choose **what to do next** based on the situation.

An **Agent** in LangChain is a system that uses an **LLM to decide which actions to take**, and in what order, to accomplish a goal.

Unlike a chain (which follows a _predefined sequence_), an **agent decides the sequence of steps at runtime** — based on its reasoning.

## How It Works — Core Idea

An **agent loop** typically looks like this:

1. **Receive a user query**
2. **Reason** (using the LLM) about what to do
3. **Select a tool** (e.g., search engine, calculator, database)
4. **Use the tool**
5. **Observe the result**
6. **Decide next step**
7. Repeat until done

This loop is called **ReAct** (Reasoning + Acting).

Example:

```python
from langchain.agents import initialize_agent, load_tools
from langchain_openai import ChatOpenAI

# Step 1: Initialize LLM
llm = ChatOpenAI(model="gpt-4o-mini")

# Step 2: Load tools
tools = load_tools(["serpapi", "llm-math"], llm=llm)

# Step 3: Create agent
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent_type="zero-shot-react-description",
    verbose=True
)

# Step 4: Run
response = agent.invoke({"input": "Who is the current president of India and what is their age in 2025?"})
print(response["output"])
```

|Agent Type|Description|Typical Use|
|---|---|---|
|`zero-shot-react-description`|The default. LLM decides actions using descriptions of tools.|General purpose|
|`structured-chat-zero-shot-react-description`|Like above, but with structured JSON output.|More reliable output format|
|`chat-conversational-react-description`|Keeps memory of past interactions.|Conversational assistants|
|`self-ask-with-search`|Special agent for question answering with web search.|QA bots|
|`openai-functions`|Uses OpenAI’s function-calling paradigm.|Robust and structured actions|
exmaple with custom agent

```python
from langchain.agents import Tool, initialize_agent

def get_square(x: int) -> int:
    return x * x

square_tool = Tool(
    name="Square Calculator",
    func=get_square,
    description="Calculates the square of a number."
)

agent = initialize_agent(
    tools=[square_tool],
    llm=llm,
    agent_type="zero-shot-react-description"
)

result = agent.invoke({"input": "Find the square of 23"})
print(result["output"])
```

The LLM will be prompted in a **ReAct** style (Reasoning + Acting). “Zero-shot” means it’s not given examples — it just gets tool descriptions and the question.

The LLM will:

1. **THINK** — reason in natural language
2. **ACT** — pick a tool to use
3. **OBSERVE** — see the result
4. Repeat until done

Excellent point — the **LLM itself** decides _what to do next_.

## Text embeddings

**Text embeddings** are **numerical representations of text** — words, sentences, or even whole documents — that capture their **meaning** in a form computers can understand.

An embedding is a **vector (list of numbers)** that represents the _semantic meaning_ of text.


- Each piece of text → becomes a point in an **n-dimensional space** (like 1536 dimensions for OpenAI embeddings).
- The **distance** or **angle** between two points (vectors) tells you how semantically related they are.

**Cosine similarity** = how similar two vectors are based on their direction.

If cosine similarity ≈ 1 → texts are very similar  
If cosine similarity ≈ 0 → unrelated  
If cosine similarity ≈ -1 → opposite meaning (rare in embeddings)

Usage of embeddings

|Use Case|Explanation|
|---|---|
|**Semantic Search**|Find text/documents that mean the same thing even if wording differs.|
|**RAG (Retrieval-Augmented Generation)**|Retrieve contextually relevant documents for LLMs.|
|**Clustering**|Group similar texts (e.g., grouping support tickets or reviews).|
|**Recommendation Systems**|Suggest similar content or queries.|
|**Classification**|Represent text for downstream ML models.|

A **vector DB** lets you find pieces of data (like text, images, or audio) that are _semantically similar_ — even if they don’t contain the same words.


Let’s imagine:

- You have **10,000 documents** (or customer reviews, support tickets, etc.).
- You want to find all documents **similar in meaning** to the query:
		  “Who leads India?”

But a **vector DB** stores **embeddings** (numeric representations of meaning).  
So you can compute **similarity** (via cosine similarity or dot product) to find the most semantically relevant items.

✅ It allows **semantic search** — _search by meaning_, not just by keyword.


## How Similarity Works

The **heart** of a vector database is its **similarity search algorithm**.

Most systems use one of:

- **Cosine similarity** — angle between two vectors
- **Dot product** — magnitude-weighted similarity
- **Euclidean distance** — geometric distance between two points

Then they use **approximate nearest neighbor (ANN)** algorithms for fast lookup:

- HNSW (Hierarchical Navigable Small World)
- IVF (Inverted File Index)
- PQ (Product Quantization)

These make it possible to search **millions of embeddings in milliseconds**.

### Pinecone

**Pinecone** is a **cloud-based vector database** — designed to store, index, and search through **embeddings** (vectors) efficiently at scale.

You send it vectors (like sentence embeddings), and it lets you **instantly find similar ones** — even among millions of records.

Local libraries like **FAISS** or **Chroma** are great for prototypes, but they:
- Run only on one machine
- Don’t scale easily to millions/billions of vectors
- Don’t provide automatic indexing, backups, or APIs

✅ **Pinecone solves that**:

- Fully managed cloud service
- Instant scaling
- Real-time similarity search API
- Metadata filtering
- Low latency (~10–50 ms)
- Built-in security and persistence

### Pinecone indexes

In Pinecone, an **index** is the **core data structure** that stores your vectors (embeddings).
Think of it like a **database table**, or more accurately:

🗂️ An **index = a semantic search engine** built specifically for your vectors.
Each index can contain **millions (or even billions)** of vectors, and is optimized for **fast similarity search**.

Every Pinecone index stores items like this:

|Field|Description|
|---|---|
|`id`|Unique string ID for the vector|
|`values`|The actual embedding (list of floats)|
|`metadata`|Optional extra info like text, tags, source, etc.|

Parameter when creating index

```python
pc.create_index(
    name="my-index",
    dimension=1536,         # embedding size (depends on model)
    metric="cosine",        # similarity metric (cosine / dotproduct / euclidean)
    spec={"pod_type": "p1"} # performance tier
)
```

dimention - The **length of your embedding vectors**.
metric - defines how similarity is measured

|Metric|Meaning|
|---|---|
|`"cosine"`|measures angle (best for semantic similarity)|
|`"dotproduct"`|measures raw vector alignment|
|`"euclidean"`|measures straight-line distance|

Defines the **performance/capacity tier**.  
Pinecone offers pods like:

- `"s1"` — starter tier (free)
- `"p1"` — production tier (faster, more memory)
- `"p2"` — high-performance tier

Once created index can be connected with and we can put vectors in it.

```python
index = pc.Index("my-index")

index.upsert([
    {
        "id": "doc1",
        "values": [0.11, 0.23, -0.54, ...],
        "metadata": {"category": "tech"}
    },
    {
        "id": "doc2",
        "values": [0.22, -0.13, 0.65, ...],
        "metadata": {"category": "science"}
    }
])
```

After inserting we can also query them

```python
query_vector = [0.09, -0.42, 0.33, ...]  # your query embedding

results = index.query(
    vector=query_vector,
    top_k=3,
    include_metadata=True
)
```

Returns most similar vectors

```python
{
  "matches": [
    {"id": "doc2", "score": 0.93, "metadata": {"category": "science"}},
    {"id": "doc1", "score": 0.78, "metadata": {"category": "tech"}}
  ]
}
```

Each index can have **multiple namespaces** — like folders inside the same index.

```python
index.upsert(vectors, namespace="finance")
index.upsert(vectors, namespace="tech")

results = index.query(vector=query_vector, namespace="finance")
```

This lets you keep separate logical datasets in one index (e.g., per user or topic).

```python

# listing all index
pc.list_indexes()

# describing index
pc.describe_index("my-index")

# deleting an index
pc.delete_index("my-index")
```

```python
from pinecone import Pinecone

# connect
pc = Pinecone(api_key="YOUR_API_KEY")

# create index if it doesn’t exist
if "demo-index" not in [i["name"] for i in pc.list_indexes()]:
    pc.create_index(name="demo-index", dimension=1536, metric="cosine")

index = pc.Index("demo-index")

# upsert
index.upsert([
    {"id": "1", "values": [0.1, 0.2, 0.3, ...], "metadata": {"topic": "AI"}}
])

# query
results = index.query(vector=[0.1, 0.2, 0.25, ...], top_k=3, include_metadata=True)
print(results)
```

## End to end project to search a text file

LLMs (and embedding models) have **token limits** — they can’t handle huge documents at once.

So before creating embeddings, we must:

1. **Split large text/documents** into **smaller, meaningful chunks**
2. Create embeddings **for each chunk**
3. Store them in a **vector database** (like Pinecone or Chroma)
4. Later, retrieve only the **most relevant chunks** for a given query.

That’s what **text splitting** in LangChain does.

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,        # max characters per chunk
    chunk_overlap=100,     # overlap to maintain context
)
```

```python
text = """
Artificial Intelligence (AI) is the simulation of human intelligence processes by machines...
It is used in applications like natural language processing, computer vision, and robotics...
"""

chunks = text_splitter.split_text(text)
print(chunks)
```

and out put 

```json
[
 "Artificial Intelligence (AI) is the simulation of human intelligence processes by machines...",
 "It is used in applications like natural language processing, computer vision, and robotics..."
]
```

Once chunks are created we can form the embeddings

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

text = "Artificial Intelligence enables machines to think like humans."
vector = embeddings.embed_query(text)

print(len(vector))  # → 1536 (vector dimension)
```

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
)

text = """Long article or document text..."""
# Step 1: Split text
texts = text_splitter.split_text(text)

# Step 2: Create embeddings for each chunk
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectors = [embeddings.embed_query(chunk) for chunk in texts]
```

Now we can store the embedding in the vector store

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

texts = ["AI enables...", "ML is a subset...", "Deep learning uses neural networks..."]

embeddings = OpenAIEmbeddings()

vectorstore = Chroma.from_texts(
    texts,
    embedding=embeddings,
    collection_name="my_docs"
)

# “Take my list of text documents, turn them into embeddings using the given embedding model, and store them inside a local **Chroma** vector database collection called `my_docs`.”
```

What happens internally:

- For each text in `texts`, LangChain calls `embeddings.embed_query(text)` (or an equivalent method)
- That produces a **numerical vector** (a list of ~1536 floats if using OpenAI embeddings)
- So you end up with one embedding per text chunk.


Then LangChain:

- Stores each embedding vector inside the collection
- Associates it with the **original text**
- Optionally stores metadata (if provided)

Now after this content is there. vectorstore object can do similarity searches and other things as well. 

🔍 _“Given a query, find the most semantically similar text chunks from my database.”_

Similarity search is not a keyword matching rather it is meaning based

When we do - 

```python
results = vectorstore.similarity_search("What is deep learning?", k=2)
```

LangChain:

1. Converts your query into an **embedding vector**
2. Compares it with all stored embeddings using the **similarity metric** (e.g., cosine similarity)
3. Returns the **top `k` most similar chunks**

## Using as retriver

You can directly use this vectorstore as a **retriever** for an LLM:

```python
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")
retriever = vectorstore.as_retriever(search_kwargs={"k": 2}) # 

qa = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever
)

query = "Explain deep learning."
print(qa.invoke({"query": query}))
```

- `vectorstore` → previously created Chroma vectorstore containing your embedded texts.
- `.as_retriever()` → converts the vectorstore into a **Retriever object**, which `RetrievalQA` can use.
- `search_kwargs={"k": 2}` → tells the retriever to:
    - Fetch the **top 2 most relevant chunks** for each query
    - These chunks will act as **context** for the LLM
- The Retriever handles:
    1. Converting your query to an embedding
    2. Searching the vectorstore for **top-k similiar chunks**
    3. Returning those chunks to the chain

```python
qa = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever
)
```

This is like passsing following prompts

```
Use the following context to answer the question:

Context:
{retrieved_docs}

Question:
{query}
```


What it does internally:

1. Takes the **retriever** (which fetches relevant documents)
2. Passes retrieved text chunks as **context** to the **LLM**
3. Uses a **default prompt template** like:

**RetrievalQA** in LangChain is a specialized chain designed to combine the power of **vector-based retrieval** with a **language model** to answer questions accurately using relevant context. Instead of relying solely on the LLM’s internal knowledge, RetrievalQA first uses a **retriever** (which could be a vector store like Chroma, Pinecone, or FAISS) to search for the most semantically relevant pieces of text—these could be chunks from documents, articles, or any other knowledge base. Once these relevant chunks are retrieved, the LLM is provided with this context and the user’s query, allowing it to generate an informed, precise answer. This process effectively implements a **Retrieval-Augmented Generation (RAG)** workflow: the retriever provides memory, and the LLM produces the answer using that memory. The number of retrieved chunks can be controlled (e.g., top-k), and optional metadata filters can refine which pieces are considered, making the system highly flexible. In short, RetrievalQA bridges **semantic search** and **generative AI**, ensuring the LLM answers are grounded in actual data rather than just its pre-trained knowledge.
