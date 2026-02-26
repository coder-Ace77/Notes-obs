
---

## Graph databases

A **Graph Database** is a NoSQL database that prioritizes the **relationships** between data points as much as the data itself. Unlike a traditional SQL database that uses tables and "Foreign Keys" to link data, a graph database stores these connections physically, allowing you to traverse them at lightning speed.

In a SQL database, finding a "friend of a friend of a friend" requires joining the same table multiple times. As you add more "hops," the performance drops off a cliff.

In a Graph DB, there are no joins. The database uses **Index-Free Adjacency**, meaning each node physically "points" to its neighbors. Moving from one node to the next is a constant-time operation (O(1)), no matter how large the total dataset is.

Top graph databases are - 

- Neo4j - The most mature and widely used "native" graph database. It uses the **Cypher** query language (which looks like ASCII art).
- **AWS Neptune:** A fully managed service that is great for teams already in the Amazon ecosystem.

Mostly used in Social media networks and recommendation engines. 

## Vector databases

In the context of the modern AI revolution, **Vector Databases** have become the "Long-term Memory" for Large Language Models (LLMs). While traditional databases store strings and numbers, vector databases store the **mathematical meaning** of data.

To understand these databases, you must first understand **Embeddings**. An embedding is a process where a piece of unstructured data (a sentence, an image, or an audio clip) is converted into a long list of numbers, called a **Vector**.

For example, the word "King" might be represented as:`[0.12,−0.59,0.22,...,0.88]`. In a vector database, these numbers aren't random. They represent **dimensions of meaning**. Words like "King" and "Queen" will have vectors that are numerically very close to each other, while "King" and "Toaster" will be very far apart.

Vector dbs can allow for semantic search by doing the similarity check and finding the close relative to some vector. 