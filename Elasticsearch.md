
---

Elastic search is a database engine that powers the search bar. At its core, it is a **distributed, open-source search and analytics engine** built on top of Apache Lucene. It is designed to handle "unstructured" data (like text) and return results in milliseconds, even when searching through billions of records.

It can handle - 

- Fuzzy matches - Finds "Apple" even if you type "Aple."
- Stemming - Searching for "running" will also find results for "run" or "ran."
- Returns synonyms - Searching for "laptop" can return results for "notebook."

Elasticsearch is the "E" in the **ELK Stack** (Elasticsearch, Logstash, Kibana). It is the world's most popular tool for:

- **Log Analytics:** Watching server logs in real-time to spot errors.
- **Security (SIEM):** Detecting cyber threats by analyzing millions of network events per second.
- **Infrastructure Monitoring:** Tracking CPU, memory, and application health.

Modern AI apps use Elasticsearch as a **Vector Database**. It can store "embeddings" (mathematical representations of meaning) to perform **Semantic Search**. This allows a user to search for _"warm winter clothes"_ and find a "Heavy Down Jacket" even if the word "winter" or "warm" isn't in the description.

To achieve its legendary speed, Elasticsearch uses a completely different architecture than a traditional relational database (RDBMS).

If you want to find a word in a book, you don't read every page; you go to the **Index** at the back. Elasticsearch does the same.

When you "index" a document, Elasticsearch breaks the text into individual words (**tokens**) and creates a map: Eg - `"Pizza" → **Found in Documents:** [1, 5, 12, 102] , "Pizza" → **Found in Documents:** [1, 5, 12, 102]`. 

When you search for "Pizza," it doesn't scan the data; it simply looks at the "Pizza" entry in the index and immediately knows which documents to return.

### Elastic search is designed to be distributed. 

Elasticsearch is designed to be **distributed**. It doesn't live on one server; it lives on a **Cluster**.
**Indices:** Think of an "Index" like a database table.
**Shards:** An index is broken into smaller pieces called "Shards." These shards are spread across different servers (**Nodes**).
**Replicas:** Every shard has a copy (replica) on a different server. If one server catches fire, the cluster stays online because the replica takes over.

When you get 1,000,000 results, why is the best one at the top? Elasticsearch uses an algorithm called **BM25** (Best Matching 25) to calculate a **Relevance Score**. It looks at:

- **Term Frequency:** How many times does the word appear in this document?
- **Inverse Document Frequency:** How "unique" is this word across all documents? (Common words like "the" get less weight than rare words like "algorithm").

### Under the hood

The primary reason Lucene is fast is that it **never scans the actual documents** during a search.
When you index a document, Lucene breaks the text into "tokens" (terms) and builds a map. Instead of `Document -> Terms`, it stores `Term -> [Document IDs]`. To find "Apple," Lucene doesn't read any documents; it just jumps to the "A" section of its dictionary and finds the list of IDs associated with "Apple."

**The Performance Win:** Because segments never change once written, Lucene doesn't need "locks" for reading. Multiple threads can read the same segment simultaneously at full speed without worrying about a "write" operation changing the data mid-search.

Elastic serach using skip lists to skip large chunk of document base from scanning. Document IDs are just integers (102,105,110...). To save space and speed up transfer, Lucene uses **Delta Encoding**. Instead of storing the full numbers, it stores the _difference_ between them (102,3,5...). These small numbers are then compressed using "Bit Packing." This allows Lucene to fit thousands of Doc IDs into a single CPU cache line, making the "intersection" of lists incredibly fast at the hardware level.

The Inverted Index is great for finding _which_ docs have a word, but it's terrible for _sorting_ those docs by "Price" or "Date." For this, Lucene uses **Doc Values**. This is a **columnar storage** format (similar to what you'd find in a data warehouse like Snowflake). It stores values for a single field together in a way that is highly compressed and optimized for sequential access, which is what the CPU needs for fast sorting and aggregations.

For filters (e.g., "Show only products where `in_stock = true`"), Lucene uses **Bitsets**. A Bitset is just a string of 1s and 0s representing every document in the index.
- **The Speed:** Modern CPUs can perform "AND" or "OR" operations on these bitsets using specialized instructions (**SIMD**).
- Comparing two bitsets to filter a million documents can take just a few **microseconds**.

