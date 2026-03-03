
---

Querying and updating in MongoDB is centered around the concept of "MQL" (MongoDB Query Language), which uses a JSON-like structure to interact with data.MongoDB uses documents to define what you are looking for and how you want to change it.

In the `mongosh` which is a javascript based terminal. We do following

- use "db" - to create db/use this db from on. `db` keyword will point to this db.
- General structure of a command is as follows

```javascript
db.collection_name.operation(doc)
```

Here collection name should be from the same db. Common operations are `find, findOne, inseert,update` along with counter parts `insertOne` etc. 

### find()

It takes two optional arguments. First positional arg is filter and second is projection

```javascript
db.users.find({name:"adil"})
```

- if no argument is passed it is same as find all. To refine this, you use equality matches like `{ "name": "Alice" }` or comparison operators such as `$gt` (greater than), `$lt` (less than), and `$in` (matches any value in an array).
- Projection means to get the subset of fields from docs. It is again a doc and we give value corresponding to fields we want and 0 corresponding to fields we don't want. 1 and 0 can not be mixed except `_id`:0  can be used with other `1`. 

```js
db.users.find({}, { "email": 1, "_id": 0 })
```

Anatomy of filter - 

The filter is an doc object where every key is either top level field or nested field written using dot operator which must be wrapped in dot notation. Eg `add.city`

Now the value is another nested doc but whose keys start with `$` these are called operators and must be used like that. If we omit operator it is by default `$eq. Now the operator side can have multiple filters separated by commas. Finally the value of doc is the data we are comparing against. 

When you provide multiple fields within a single filter document, MongoDB treats them as an `AND` operation by default.

```js
{age:{$eq:20}}
or
{age:20}
```


```js
db.orders.find(
  { "status": "Shipped" },           // Query
  { "customerName": 1, "date": 1, "_id": 0 } // Projection
)
```

|**Operator**|**Description**|**Example**|
|---|---|---|
|`$eq`|Matches values that are **equal** to a specified value.|`{ "age": { "$eq": 25 } }`|
|`$ne`|Matches all values that are **not equal**.|`{ "status": { "$ne": "deleted" } }`|
|`$gt` / `$gte`|**Greater than** / Greater than or equal to.|`{ "price": { "$gt": 50 } }`|
|`$lt` / `$lte`|**Less than** / Less than or equal to.|`{ "stock": { "$lte": 10 } }`|
|`$in`|Matches any of the values specified in an **array**.|`{ "tags": { "$in": ["tech", "news"] } }`|
|`$nin`|Matches **none** of the values specified in an array.|`{ "code": { "$nin": [404, 500] } }`|

```js
db.orders.find({
  "total": { "$gte": 100, "$lte": 500 }
}).sort({ "total": -1 })
```

These combine multiple query expressions.

For logical operations like `$or`, the anatomy changes. The operator becomes the **Key**, and the value becomes an **Array** of filter documents.

- **`$and`**: Joins query clauses with a logical AND (all conditions must match). Note: MongoDB provides an implicit AND when you list multiple fields in one object.
- **`$or`**: Joins query clauses with a logical OR (at least one condition must match).
- **`$not`**: Inverts the effect of a query expression.
- **`$nor`**: Joins query clauses with a logical NOR (fails to match both).

```js
db.orders.find({
  "$or": [
    { "customerType": "VIP" },
    { "status": "Urgent" }
  ]
})
```

These deal with the "shape" or existence of the fields themselves.
- **`$exists`**: Matches documents that have (or don't have) a specific field. `{ "phone": { "$exists": true } }`.
- **`$type`**: Selects documents if a field is of a specific BSON type (e.g., String, Double, Array).

These allow for more complex logic, like Regex or JavaScript execution.
- **`$regex`**: Provides regular expression capabilities for pattern matching strings.
- **`$mod`**: Performs a modulo operation on the value of a field.
- **`$text`**: Performs a text search (requires a text index).

```js
db.orders.find({
  "customerName": { "$regex": "^Alex", "$options": "i" }
})
```

Since MongoDB is document-based, it has specialized operators for querying arrays.

For the arrays we can directly do value match which gives which filters out docs with atleast one element of that form. 

- **`$all`**: Matches arrays that contain all elements specified in the query.
- **`$elemMatch`**: Selects documents if at least one element in the array matches all the specified criteria. This is vital for querying arrays of sub-documents.
- **`$size`**: Selects documents if the array field is a specific length.

```cpp
db.orders.find({
  "items": {
    "$elemMatch": { "name": "Laptop", "price": { "$gt": 1000 } }
  }
})
```

Technically, `find()` does not return the documents immediately; it returns a **Cursor**. You then chain methods to that cursor to modify the result set:

- `.sort({ field: 1 })`: 1 for ascending, -1 for descending.
- `.limit(n)`: Restricts the result set to n documents.
- `.skip(n)`: Skips the first n documents (useful for pagination).
- `.count()`: Returns the number of documents (though `countDocuments()` is preferred in modern drivers).

When using these together, the order of execution in the database is always **Sort -> Skip -> Limit**, regardless of the order you type them in your code.

### Insert

There are two variations of insert.

- insert
- insertOne

Both are simple as we just take the object we want to insert and that's it.

### Update

Update call in mongodb has two parts -

- filter - same as find
- update action - A document that uses update operations starting with `$` to define how to define change. 

If you provide a plain document like `{ price: 20 }` as the second argument, MongoDB will **replace** the entire document with that new one. To change specific fields, you **must** use operators.
Note that each update doc part starts with an opertor and then inside takes the inc on what to update and by what. For example `inc` `{$inc:{filed:val}}`

`$set` is used to change the value of a field or create it if it doesn’t exist. `{ $set: { "stock": 20, "onSale": true } }`

`$unset` is used to remove a specific field from a document. `{ $unset: { "oldReferenceCode": "" } }`

`$inc` Increments (or decrements) a numeric value by a specific amount. `{ $inc: { "stock": -1 } }`

`$push` and `$pull` — Array Managers. `$push` is used to add an item to array while pull removes the item from array.

```js
db.collection.updateOne(
   { <query> },
   { $push: { <field>: <value> } }
)
```

Example

```js
db.users.updateOne(
   { name: "Alex" },
   { $push: { interests: "hiking" } }
)
```

We can also add multiple items using each 

```js
db.users.updateOne(
   { name: "Alex" },
   { $push: { interests: { $each: ["cooking", "traveling"] } } }
)
```

Pull removes all the instances from array.

```js
db.collection.updateOne(
   { <query> },
   { $pull: { <field>: <condition> } }
)
```

Eg - 

```js
db.users.updateOne(
   { name: "Alex" },
   { $pull: { interests: "hiking" } }
)
```

We can also remove based on condition. 

```js
db.students.updateOne(
   { name: "Sam" },
   { $pull: { grades: { $lt: 70 } } }
)
```

We also have updateMany counterpart which can also be used. 

#### The upsert

Upsert means inserting a doc if not exists. 

```js
db.books.updateOne(
  { title: "The Silmarillion" }, 
  { $set: { price: 28.00 } }, 
  { upsert: true }
)
```

This upsert can be passed as the third arg.

Now array updates - You match the specific array element in the **filter**, and then use `$` in the **update** to represent "the element that matched."

Eg - 

```js
db.books.updateOne(
  { title: "Dune", "reviews.user": "Alice" }, // Filter matches the specific review
  { $set: { "reviews.$.score": 10 })
```

What if you want to update **multiple** elements in an array that meet a condition? For example, "Increase the score of every review that is currently below 6."

```js
db.books.updateOne(
  { title: "Dune" },
  { $set: { "reviews.$[elem].score": 6 } },
  { arrayFilters: [ { "elem.score": { $lt: 6 } } ] }
)
```
#### Deletion

Deletion is pretty simple as it just needs filter and the deletion will be applied. Note we have deleteOne and deleteMany variants as well. 

```js
db.books.deleteOne(filter);
```


### Aggregation

The **Aggregation Pipeline** is MongoDB’s version of a multi-stage assembly line. Instead of just finding documents, you pass them through a sequence of stages that transform, filter, and calculate data to produce a final result.

If `find()` is like a simple filter, `aggregate()` is like a data processing factory.

The basic syntax of Pipeline is `db.collection.aggregate([ { stage1 }, { stage2 }, { stage3 } ])`.

The input to each stage is the output of the previous stage. The most common stages are:

- `$match` - Filters documents (uses the same anatomy as the `find()` filter).
- **`$group`**: Groups documents by a specific key and performs "accumulator" operations (like sum, average, etc.).
- **`$project`**: Reshapes the document (adding fields, removing fields, or calculating new ones).
- - **`$sort`**, **`$limit`**, **`$skip`**: Identical to the cursor methods but inside the pipeline.
- **`$unwind`**: Deconstructs an array field from the input documents to output a document for _each_ element.

Note if filed name comes in the value part it is written in commas with `$` eg - `{_id:"$author"}`
#### match stage

This is usually your first stage. It functions exactly like the "Filter" in `find()`. Its job is to decrease the number of documents moving down the line to save memory and processing power. It uses standard query operators like find in `find`

```js
db.books.aggregate([{$match:{prices:{$gt30}}},..])
```

#### project stage

This stage reshapes each document. You can use it to include fields, exclude fields, or create entirely new "computed" fields.

```js
{ $project: { user_name: "$name" } }

{ $project: { total_cost: { $multiply: ["$price", "$quantity"] } } }

{ $project: { slug: { $toLower: "$title" } } }
```
#### group stage

This is where the magic happens. You must define an `_id` (the field you are grouping by).

```js
{
  $group: {
    _id: "$author",          // Group by author
    totalBooks: { $sum: 1 }, // Count how many books each author has
    avgPrice: { $avg: "$price" } // Calculate average price per author
  }
}
```

Other operators - `$sum`, `$avg`, `$max`, `$push`.
#### unwind stage

**`$unwind`** is the "Array Flattener." While most stages take one document and output one document (or zero if filtered), `$unwind` is unique because it can take **one** document and turn it into **many**. It is essential because most aggregation operators (like `$group` or `$sort`) cannot "see" inside an array—they treat the whole array as a single value.

When you "unwind" a field, you are essentially telling MongoDB: _"For every item in this array, give me a standalone copy of the parent document containing just that item."_

`{ $unwind: "$fieldName" }`

Example 

```js
{
  "_id": 101,
  "title": "Dune",
  "genres": ["Sci-Fi", "Epic", "Adventure"]
}
```

Output of unwind will be 

```js
{ "_id": 101, "title": "Dune", "genres": "Sci-Fi" }
{"_id": 101, "title": "Dune", "genres": "Epic" }
{ "_id": 101, "title": "Dune", "genres": "Adventure" }
```

```js
db.books.aggregate([
  // 1. Flatten the arrays so every genre is its own document
  { $unwind: "$genres" },

  // 2. Now we can group by the individual genre strings
  { 
    $group: { 
      _id: "$genres", 
      count: { $sum: 1 } 
    } 
  }
])
```

Example 

```js
db.books.aggregate([
  // 1. Filter for Sci-Fi only (The Bouncer)
  { $match: { genres: "Sci-Fi" } },

  // 2. Group by Author and calculate average (The Accountant)
  { 
    $group: { 
      _id: "$author", 
      avgPrice: { $avg: "$price" },
      bookCount: { $sum: 1 }
    } 
  },
  // 3. Filter the groups (Only expensive authors)
  { $match: { avgPrice: { $gt: 20 } } },
  // 4. Sort by highest price (The Organizer)
  { $sort: { avgPrice: -1 } }
])
```

Each individual stage in a pipeline is limited to **100MB of RAM**. Use `{ allowDiskUse: true }` in your aggregate options. This allows MongoDB to spill data to temporary files on the hard drive, though it is significantly slower than RAM. There are two kinds of stages in pipelines - 

Stages like match , project and unwind are called streaming stages. They are fast and have a low memory footprint because they don't need to "see" the whole dataset to work.

Stages like `$sort` and `$group` must "block" the pipeline until they have seen **every single document** to determine the final order or total. These are the most resource-heavy.

Aggregation pipelines can only use indexes if the **very first stage** is a `$match` or a `$sort`.Once you hit a stage that changes the document structure (like `$project`, `$unwind`, or `$group`), indexes can no longer be used for the rest of the pipeline.