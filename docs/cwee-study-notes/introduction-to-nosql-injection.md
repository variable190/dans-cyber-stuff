# Introduction to NoSQL Injection

## Types of NoSQL DB

| Type | Description | Top 3 Engines (as of November 2022) |
| --- | --- | --- |
| Document-Oriented Database | Stores data in documents which contain pairs of fields and values These documents are typically encoded in formats such as JSON or XML | MongoDB, Amazon DynamoDB, Google Firebase - Cloud Firestore |
| Key-Value Database | A data structure that stores data in key:value pairs, also known as a dictionary | Redis, Amazon DynamoDB, Azure Cosmos DB |
| Wide-Column Store | Used for storing enormous amounts of data in tables, rows, and columns like a relational database, but with the ability to handle more ambiguous data types | Apache Cassandra, Apache HBase, Azure Cosmos DB |
| Graph Database | Stores data in nodes and uses edges to define relationships | Neo4j, Azure Cosmos DB, Virtuoso |

## MongoDB

### Basic Commands

```bash
mongosh mongodb://127.0.0.1:27017 # connect to DB on default TCP port
show databases
use academy # databases created once data first stored
show collections # list all collections in a database
db.apples.insertOne({type: "Granny Smith", price: 0.65}); # collection when a document first inserted into it
db.apples.insertMany([{type: "Golden Delicious", price: 0.79}, {type: "Pink Lady", price: 0.90}]); # add multiple documents to collection
db.apples.find({type: "Granny Smith"}); # selecting data
db.apples.find({}); # list all documents in a collection
```

### Query Operators

| Type | Operator | Description | Example |
| --- | --- | --- | --- |
| Comparison | $eq | Matches values which are equal to a specified value | type: {$eq: "Pink Lady"} |
| Comparison | $gt | Matches values which are greater than a specified value | price: {$gt: 0.30} |
| Comparison | $gte | Matches values which are greater than or equal to a specified value | price: {$gte: 0.50} |
| Comparison | $in | Matches values which exist in the specified array | type: {$in: ["Granny Smith", "Pink Lady"]} |
| Comparison | $lt | Matches values which are less than a specified value | price: {$lt: 0.60} |
| Comparison | $lte | Matches values which are less than or equal to a specified value | price: {$lte: 0.75} |
| Comparison | $nin | Matches values which are not in the specified array | type: {$nin: ["Golden Delicious", "Granny Smith"]} |
| Logical | $and | Matches documents which meet the conditions of both specified queries | $and: [{type: 'Granny Smith'}, {price: 0.65}] |
| Logical | $not | Matches documents which do not meet the conditions of a specified query | type: {$not: {$eq: "Granny Smith"}} |
| Logical | $nor | Matches documents which do not meet the conditions of any of the specified queries | $nor: [{type: 'Granny Smith'}, {price: 0.79}] |
| Logical | $or | Matches documents which meet the conditions of one of the specified queries | $or: [{type: 'Granny Smith'}, {price: 0.79}] |
| Evaluation | $mod | Matches values which divided by a specific divisor have the specified remainder | price: {$mod: [4, 0]} |
| Evaluation | $regex | Matches values which match a specified RegEx | type: {$regex: /^G.*/} |
| Evaluation | $where | Matches documents which satisfy a JavaScript expression | $where: 'this.type.length === 9' |

```bash
# select all apples whose type starts with a 'G' and whose price is less than 0.70
db.apples.find({
    $and: [
        { type: { $regex: /^G/ } },
        { price: { $lt: 0.70 } }
    ]
});

# same but with where clause
db.apples.find({$where: `this.type.startsWith('G') && this.price < 0.70`});

#select the top two apples sorted by price in descending order
db.apples.find({}).sort({price: -1}).limit(2)
```

### Updating Documents

```bash
db.apples.updateOne({type: "Granny Smith"}, {$set: {price: 1.99}}) # update operations take a filter and an update operation
db.apples.updateMany({}, {$inc: {quantity: 1, "price": 1}}) # increase the prices of all apples at the same time
db.apples.replaceOne({type:'Pink Lady'}, {name: 'Pink Lady', price: 0.99, color: 'Pink'}) # completely replace the document
db.apples.remove({price: {$lt: 0.8}}) # remove matched documents
```