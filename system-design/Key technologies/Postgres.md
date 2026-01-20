
---

Read the `Database/Realtional Database` 
## Core capabilities of postgres

Most system design discussions about PostgreSQL will center around its read performance, write capabilities, consistency guarantees, and schema flexibility.
### Read performance

First up is read performance - this is critical because in most applications, reads vastly outnumber writes. In our social media example, users spend far more time browsing posts and profiles than they do creating content.Without proper indexing, PostgreSQL would need to scan every row in the posts table to find matching posts - a process that gets increasingly expensive as our data grows.

#### Basic indexes

The most fundamental way to speed up reads in PostgreSQL is through indexes.By default, PostgreSQL uses B-tree indexes, which work great for:

- Exact matches (WHERE email = '[user@example.com](mailto:user@example.com)')
- Range queries (WHERE created_at > '2024-01-01')
- Sorting (ORDER BY username if the ORDER BY column match the index columns' order)

A common trap in interviews is to suggest adding indexes for every column. Remember that each index:

- Makes writes slower (as the index must be updated)
- Takes up disk space
- May not even be used if the query planner thinks a sequential scan would be faster

### Full text searches

**Full-Text Search** using GIN indexes. Postgres supports full-text search out of the box using GIN (Generalized Inverted Index) indexes.

```sql
-- Add a tsvector column for search
ALTER TABLE posts ADD COLUMN search_vector tsvector;
CREATE INDEX idx_posts_search ON posts USING GIN(search_vector);

-- Now you can do full-text search
SELECT * FROM posts 
WHERE search_vector @@ to_tsquery('postgresql & database');        
```

This capability means we may not want the elastic search.JSONB columns with GIN indexes are particularly useful when you need flexible metadata on your posts. For example, in our social media platform, each post might have different attributes like location, mentioned users, hashtags, or attached media. Rather than creating separate columns for each possibility, we can store this in a JSONB column (giving us the flexibility to add new attributes as needed just like we would in a NoSQL database!).

```sql
-- Add a JSONB column for post metadata
ALTER TABLE posts ADD COLUMN metadata JSONB;
CREATE INDEX idx_posts_metadata ON posts USING GIN(metadata);

-- Now we can efficiently query posts with specific metadata
SELECT * FROM posts 
WHERE metadata @> '{"type": "video"}' 
  AND metadata @> '{"hashtags": ["coding"]}';

-- Or find all posts that mention a specific user
SELECT * FROM posts 
WHERE metadata @> '{"mentions": ["user123"]}';
```




