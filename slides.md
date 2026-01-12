---
theme: default
title: Distributed Databases & NoSQL
info: |
  CST363 — Introduction to Database Systems  
  Instructor: Avner Biblarz  
  Topic: MongoDB, Sharding, Replication, NoSQL Concepts
class: text-center
drawings:
  persist: false
transition: slide
mdc: true
---


# NoSQL, Day 2

CST 363


---

## Why was the relational model so dominant?

- Tables, rows, columns.
- Powerful query language (SQL).
- ACID guarantees.
- Decades of optimization.

<br>

## But... only if your data fits the model:

- Uniform rows
- Fixed schema
- Relationships modeled by joins
  - Separate normalized tables vs embedded


---

## Relational Pain Points

Three core limitations

<v-clicks depth="2">


- Rigid Schema
  - Slows agile iterations

- Vertical scaling
  - Hits cost and hardware ceilings --- scaling up by buying bigger servers is expensive and has physical limits

- Complex Joins
  - Degrade performance at scale
  - Normalized data requires expensive join operations, slowing queries as data grows


</v-clicks>


---
layout: center
---

# Core Concepts




---

## The NoSQL Landscape

Four major categories

<div class="p-3">

<v-clicks>

- <carbon-password /> Key-Value --- Simple, fast lookups (e.g., Redis)

- <carbon-document /> Document --- JSON-like, flexible schema

- <carbon-book /> Column --- For large datasets (e.g., Cassandra)

- <carbon-DataVis-1 /> Graph --- For relationships (e.g., Neo4j)

</v-clicks>

</div>


<v-click>

MongoDB leads the document store category, emphasizing:

</v-click>

<div class="p-3">


<v-clicks>

- horizontal scale
- flexible schema
- JSON-like storage

</v-clicks>

</div>

---

## Modern PostgreSQL compared to MongoDB

- PostgreSQL stores `JSONB` within rows of a table, enforcing optional schema. MongoDB uses native `BSON` documents in schema-less collections.

- PostgreSQL uses SQL operators for `JSONB` , while MongoDB uses its own query language with dot notation and operators like `$eq`, `$gt`, etc.

- **Joins:** 
  - PostgreSQL offers fully relational joins
  - MongoDB uses `$lookup` --- considered less powerful.
- **Transactions & ACID:** 
  - PostgreSQL is always fully ACID compliant. 
  - MongoDB offers ACID for single documents and multi-document transactions since version 4.0.



---

## Document Model Basics


<div class="p-3">

<v-click>

The fundamental unit of data in MongodB is a BSON (Binary JSON) document

</v-click>

<v-clicks>

- Composed of field-value pairs, self-describing and nested
- Constrasts with rigid rows and columns of RDBMS
- Supports <b>embedded arrays and sub-documents</b>, removing the need for separate tables and complex joins

</v-clicks>

</div>


---

## <carbon-script /> BSON Document Example

<br />

```js
{
    _id: ObjectId("644c7f2a9c0a4f1234567890"),
    name: "Kipling Peregrine",
    email: "kp@example.com",
    address: {
        street: "123 Main St",
        city: "Anytown"
    },
    hobbies: [
        "reading",
        "gaming"
    ],
    profile : {
        membership: "gold",
        preferences: {
            newsletter: true
        }
    }
}
```


---


## Collections & Databases

Organizing data in MongoDB

<br />

<div grid="~ cols-2 gap-4">


<v-click>

<div class="p-3">

## <carbon-Db2Database /> Database

A physical container for collections. 

A group of collections on disk.

</div>

</v-click>


<v-click>

<div class="p-3">

## <carbon-Folder /> Collection

Analogous to tables, but <b>schema-less</b>. 

A grouping of documents.

</div>

</v-click>


</div>


<br />

<v-click>

Dynamic collection creation and namespaces accelerate development cycles significantly.

</v-click>



---
layout: center
---

# CRUD in Action


---


## CRUD in Action: Create & Read

Concise and readable data operations without joins.


<div grid="~ cols-2 gap-4">

<v-click>

<div>

### Create

Adding new documents is straightforward:


```js
db.users.insertOne({ name: "Alice", age: 30 }); 

db.users.insertMany([
    { name: "Bob", age: 25 },
    { name: "Charlie", age: 35 }
]);
```

</div>

</v-click>


<v-click>

<div>

### Read

Flexible querying with operators:

```js
db.users.find(
  { age: { $gt: 25 } }
); 

db.users.find(
  { age: { $in: [ 25, 35 ] } },
  { name: 1, _id: 0 }
).pretty();
```

</div>

</v-click>

</div>

<v-click>

<div class="p-3">

Query operators like <code>\$gt</code> , <code>\$in</code> filter data efficiently, keeping code clean and readable.

</div>

</v-click>


---

# CRUD in Action: Update & Delete

Modifying data safely and atomically.

<div grid="~ cols-2 gap-4">


<div>

### Update

Use modifiers like <code>\$set</code> and <code>\$unset</code> :


```js
db.users.updateOne( 
  { name: "Alice" }, { $set: { age: 31 } } 
); 

db.users.updateMany( 
  { age: { $lt: 30 } }, { $set: { status: "young" } } 
);
```

</div>


<div>

### Delete

Remove documents with precision:

```js
db.users.deleteOne({ name: "Bob" }); 
  
db.users.deleteMany({ status: "inactive" });
```
</div>

</div>



---

## More MongoDB Usage Examples

- Create collection and insert a document

```js
db.users.insert({name: "Jack", email: "jack@example.com"});
```


- Retrieve all/some documents

```js
db.users.find();
db.users.find({name: "Jack"});
```

- Update

```js
db.users.update({name: "Jack"}, {$set: {hobby: "cooking"}});
updateOne, updateMany, replaceOne
```

- Delete

```js
db.users.remove({name: "Alex"});
deleteOne, deleteMany
```



---

## More on BSON

<div class="p-2">

- Binary superset of JSON (extra types: `Date`, `Decimal128`, `ObjectId`, etc.)
- Stored as flexible schema per document
- Each collection = table analogue

</div>

<img src="/images/bson.webp" class="w-[40%] mx-auto" />

---

# Common BSON Types (1 of 2)

<div class="p-2">

- **MinKey**: Internal marker; sorts lower than all other types
- **Null**: Explicit null value
- **Numbers**: 
  - `int32`, `int64`, `double`; all converted to a common type for comparisons
- **String**: Standard UTF-8 string
- **Symbol**: Legacy JavaScript symbol type (rarely used)
- **Object**: Embedded document (`{ key: value }`)
- **Array**: Ordered list of values (`[1, 2, 3]`)

</div>

---

## Common BSON Types (2 of 2)

<div class="p-2">

- **Binary**: Raw binary data (e.g. files, buffers)
- **ObjectId**: 12-byte unique identifier (used by default for `_id`)
- **Boolean**: `true` or `false`
- **Date**, **Timestamp**: 
  - `Date`: millisecond precision
  - `Timestamp`: used internally for replication
- **RegExp**: Regular expression pattern
- **MaxKey**: Internal marker; sorts higher than all other types

</div>


---

## Anatomy of an `ObjectId`

<br>

##### `ObjectId` is a 12-byte unique identifier generated by MongoDB by default:

<div class="text-sm">

| Section         | Bytes | Description                     |
|-----------------|-------|----------------------------------|
| **Timestamp**   | 4     | Seconds since Unix epoch        |
| **Machine ID**  | 5     | Host identifier (usually a hash)|
| **Process ID**  | 2     | ID of generating process/thread |
| **Counter**     | 3     | Incrementing value (per ObjectId) |

</div>

Example: 
```js
ObjectId("652ec2c3d5b6a532bcf4aabc")
```


---

## Comparison Examples

<br>

- Use `$exists: true` to check for presence

```js
db.test.find({ tags: { $gt: "b" } }) 
// Compares "b" against all elements in array 'tags'
```

<br>

- Object Comparison

```js
{ a: 1, b: 2 } < { a: 1, b: 3 }   // true
{ b: 2, a: 1 } != { a: 1, b: 2 }  // order matters!
```

MongoDB compares objects by walking their key–value pairs in order.


---

## Sample Order Document

<br>

```js
{
  _id: ObjectId("644c..."),
  customerId: ObjectId("644b..."),
  status: "SHIPPED",
  items: [
    { sku: "ABC123", qty: 2, price: 49.99 },
    { sku: "XYZ987", qty: 1, price: 19.99 }
  ],
  shipping: {
    address: { city: "Denver", state: "CO" },
    carrier: "UPS"
  },
  createdAt: ISODate("2025‑04‑20T19:12:00Z")
}
```

<b>Note:</b> <br />
`customerId: ObjectId("644b...")` is a reference to the customer (an `_id` in a `customers` collection)


---

## Indexing & Querying


```js
// compound index on status + createdAt
 db.orders.createIndex({ status: 1, createdAt: -1 });

// containment‑style query
db.orders.find({ status: "SHIPPED", "items.sku": "ABC123" });
```

- **Text Index**
  - Enables full-text search on string fields
  - Removes stop words and stems words to their root

- **Hashed Index**
  - Uses a hash of the field value
  - Great for **sharded clusters** with equality lookups
  - Not suitable for range queries



---

## Explain & Analyze in MongoDB

<br>

- Use `.explain()` to inspect how a query is executed:

```js
db.orders.find({
  status: "SHIPPED",
  "items.sku": "ABC123"
}).explain("executionStats");
```
<br>

- `executionStats` shows:
  - Whether indexes were used
  - Number of documents examined
  - Number of index keys scanned
  - Execution time

<br>

Great for debugging slow queries and verifying index effectiveness




---

## MongoDB Aggregation Framework

<br>

- A powerful way to process data using pipelines --- like chaining stages in Unix or SQL
- Each stage transforms the documents as they pass through

```js
db.orders.aggregate([
  { $match: { status: "SHIPPED" } },
  { $unwind: "$items" },
  { $group: {
      _id: "$items.sku",
      totalQty: { $sum: "$items.qty" }
  }}
]);
```
<br>

What this does:
- Filters to shipped orders
- Flattens the items array
- Groups by `sku` and sums quantities across all orders




---
layout: center
---

# Scale & Resilience



---


## Review of Replication Concept

Ensuring high availability and data safety.

<v-clicks>


- Replica Sets maintain multiple data copies across nodes.

- Automatic Failover and consensus elections ensure no single point of failure.

- Write Concern and Read Preference let developers balance consistency vs. availability.

</v-clicks>

<v-click>

  <img src="/images/repl.png"  class="w-3/5 mx-auto">

</v-click>


---

## Sharding Strategy

Scaling out horizontally to handle petabyte workloads.

<v-clicks depth="2">

- `mongos` Router --- the query router
- `config servers` --- Store metadata and config.
- `shard` Clusters --- Hold partitioned data.
- Data is partitioned via a Shard Key.
  - Strategies include range and hashed sharding, with automatic balancing for linear scale-out.

</v-clicks>

<v-click>

<img src="/images/cluster.png" class="w-[50%] mx-auto" />

</v-click>


---
layout: center
---


## Distributed Systems Review & CAP Theorem

---

<v-clicks depth="2">

- **NoSQL** databases are non-relational and designed for

  - High scalability, Flexible schema, High availability

- **Sharding**: Horizontal partitioning of data across multiple machines (*shards*).

  - Each shard holds part of the dataset.
  - Improves scalability by distributing load.

- **Replication**
  - Copies data across multiple servers (*replica sets*).
  - Provides fault tolerance and high availability.
  - One **primary** accepts writes; **secondaries** replicate and handle reads.
  - Automatic failover if primary goes down.

</v-clicks>


---


## CAP Theorem Definitions

<br>

##### Given many nodes which contain replicas of partitions of the data, a few definitions:

<v-clicks depth="2">

- **C**onsistency
  - All replicas contain the same version of data --- client always has the same view of the data (no matter what node)

- **A**vailability
  - System remains operational on failing nodes --- all clients can always read and write

- **P**artition Tolerance
  - The system continues to function despite network partitions (i.e., lost or delayed messages between nodes) and tolerates communication failures.

</v-clicks>

---


## CAP Theorem (Brewer’s Theorem)

<br>

<v-click>

- In a distributed system, you can only fully guarantee two of these three properties at the same time:
  - Consistency (`C`): Every read sees the most recent write or an error.
  - Availability (`A`): Every request receives a (non-error) response, without guarantee that it contains the most recent write.

</v-click>

<br>

<v-click>

  - Partition Tolerance (`P`): The system continues to operate despite arbitrary network partitions between nodes.
</v-click>

--- 


<img src="/images/cap.png" class="w-[50%] mx-auto" />

---

## CAP in Practice (MongoDB Example 1 of 2)

- Partition tolerance (P) is non‑negotiable
  - MongoDB replica sets always allow writes/reads during node failures, so P is assumed.

- Choosing between C and A
  - Write concern `w:1` → Prioritize **Availability** (fast primary *ack*)
  - Write concern `w:"majority"` → Prioritize **Consistency** (wait for most nodes)

--- 

## CAP in Practice (MongoDB Example 2 of 2)

- Read preferences
  - `primary` (strong consistency)
  - `secondaryPreferred` (higher availability, possible stale reads)

- When to pick which
  - A‑leaning for logging or analytics where occasional data lag is fine
  - C‑leaning for financial transactions or user profile updates where correctness is critical

---


<br />


 Product | Default Stance | Tunable |
|---------|----------------|---------|
| PostgreSQL | **CA** on single node;<br>loses A under split | Limited (synchronous replica) |
| MongoDB | **CP** with majority writes;<br>can relax to AP via read prefs | Yes (`w`, `j`, readPreference) |



---


## Geospatial & GIS: A Quick Overview

- **Geospatial data** refers to data that has a location — on Earth’s surface.
  - Coordinates: Latitude/Longitude
  - Shapes: Points, lines, polygons (e.g., roads, zones, cities)

- **GIS** = Geographic Information Systems
  - Software + databases + tools to store, analyze, and visualize geospatial data
  - Common in urban planning, logistics, maps, agriculture, public health, etc.

- Geospatial queries rely heavily on indexes, or they’d be painfully slow.
<br>

#### Examples:
- " Find all restaurants within 2 miles of me" 
- “Which fire zones intersect this zip code?”
- “Where should we build a new delivery hub?”




---

## Geospatial Indexes in MongoDB

- **2d Index**
  - Legacy option for **flat (Euclidean)** space --- Uses **geohashing** to map 2D → 1D
  - Limited use today (e.g., indoor maps, games)

- **2dsphere Index**
  - Supports queries on a **spherical model** of Earth
  - Required for `$geoWithin`, `$geoIntersects`, `$near`
  - Internally uses **S2 Geometry Library**, not geohashing
  - Common for real-world coordinates (WGS84)

- **Multikey Support**
  - Works if a document has **multiple location points** (e.g., pickup & dropoff)
  - Each location in the array can be indexed


---


## Aside: PostgreSQL's PostGIS

- PostgreSQL supports rich geospatial queries via the PostGIS extension  
  - `GEOGRAPHY` (spherical) and `GEOMETRY` (flat) types and functions like `ST_DWithin()`.
- Not automatically provided in default image
  - At the `psql` prompt can install with `CREATE EXTENSION postgis;`

<br>

Example:
```sql
SELECT name
FROM places
WHERE ST_DWithin(location, ST_MakePoint(-122.4, 37.8)::geography, 5000);
```

- Finds all landmarks within 5 km of a point in San Francisco.

---

## Similar example using MongoDB's `near()`

- Create a 2dsphere index:

```js
db.landmarks.createIndex({ location: "2dsphere" });
```

- Run the query:

```js
db.landmarks.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [ -122.4, 37.8 ] // [lng, lat]
      },
      $maxDistance: 5000 // in meters
    }
  }
});
```

##### Note: 
  - This return documents within a radius, but also sorted from closest to farthest (extra overhead). 
  - `$near` does not support polygons, use `$geoWithin` for this


---

## Summary: NoSQL, MongoDB & JSONB

<br>

- **Relational model** excels with uniform, structured data — uses **joins** to model relationships
- **MongoDB** is a flexible, schema-less **document store**, optimized for nested and evolving data
- **PostgreSQL’s JSONB** offers hybrid flexibility with SQL — ideal when you need both structure and unstructured fields
- **CAP Theorem**: In distributed systems, you can only fully guarantee **two of**: Consistency, Availability, Partition Tolerance
- **MongoDB** offers tunable trade-offs with **write concerns** and **read preferences**
- **Indexing** is essential for performance: Mongo uses **B-tree–based** indexes for efficient query access
- **Geospatial support** is built-in via `2dsphere` indexes (Mongo) and **PostGIS** (PostgreSQL)
- **Aggregation Framework** in MongoDB replaces SQL-style queries with **pipeline stages** for filtering, transforming, and grouping data

