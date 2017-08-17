
# MongoDB cheat sheet

## Overview

Overview

MongoDB is a document database that provides high performance, high availability, and easy scalability.

* Document Database
* Documents (objects) map nicely to programming language data types.
* Embedded documents and arrays reduce need for joins.
* Dynamic schema makes polymorphism easier.
* High Performance
* Embedding makes reads and writes fast.
* Indexes can include keys from embedded documents and arrays.
* Optional streaming writes (no acknowledgments).
* High Availability
* Replicated servers with automatic master failover.
* Easy Scalability
* Automatic sharding distributes collection data across machines.
* Eventually-consistent reads can be distributed over replicated servers.

---

## Use cases

NoSQL products (and among them MongoDB) should be used to meet challenges. If you have one of the following challenges, you should consider MongoDB:

#### You Expect a High Write Load

MongoDB by default prefers high insert rate over transaction safety. If you need to load tons of data lines with a low business value for each one, MongoDB should fit. Don't do that with $1M transactions recording or at least in these cases do it with an extra safety.

#### You need High Availability in an Unreliable Environment (Cloud and Real Life)

Setting replicaSet (set of servers that act as Master-Slaves) is easy and fast. Moreover, recovery from a node (or a data center) failure is instant, safe and automatic

#### You need to Grow Big (and Shard Your Data)

Databases scaling is hard (a single MySQL table performance will degrade when crossing the 5-10GB per table). If you need to partition and shard your database, MongoDB has a built in easy solution for that.

#### Your Data is Location Based

MongoDB has built in spacial functions, so finding relevant data from specific locations is fast and accurate.

Your Data Set is Going to be Big (starting from 1GB) and Schema is Not Stable

Adding new columns to RDBMS can lock the entire database in some database, or create a major load and performance degradation in other. Usually it happens when table size is larger than 1GB (and can be major pain for a system like BillRun that is described bellow and has several TB in a single table). As MongoDB is schema-less, adding a new field, does not effect old rows (or documents) and will be instant. Other plus is that you do not need a DBA to modify your schema when application changes.

#### Examples:

* Log Aggregation
* Event Bases Systems
* Hierarchical Aggregation
* Product Catalgos
* Session Storage

## When not to use MongoDB

* You need ACID Transactions.
* You have a structured schema
 

---

## Links

- UseCases -http://docs.mongodb.org/ecosystem/use-cases/
- CRUD - http://docs.mongodb.org/manual/applications/crud/
- Data Models - http://docs.mongodb.org/manual/data-modeling/

## Mapping SQL to MongoDB

### Converting to MongoDB Terms

- MYSQL EXECUTABLE ORACLE EXECUTABLE MONGODB EXECUTABLE
- mysqld oracle mongod
- mysql sqlplus mongo

### SQL MONGODB

- `CREATE TABLE users (name VARCHAR(128), age 
NUMBER)` - `db.createCollection("users")`
- `INSERT INTO users VALUES ('Bob', 32)` - `db.users.insert({_id:'UsefulID', name: "Bob", age: 32, parent: 'ID', children: []})`
- `SELECT * FROM users` - `db.users.find()`
- `SELECT name, age FROM users` - `db.users.find({}, {name: 1, age: 1, _id:0})`
- `SELECT name, age FROM users WHERE age = 33` - `db.users.find({age: 33}, {name: 1, age: 1, _id:0})`
- `SELECT name FROM users WHERE _id = 'UsefulID'` - `db.users.findOne({_id: 'UsefulID'}).name` or `db.users.find({_id: 'UsefulID'})[0]].name`
- `SELECT * FROM users WHERE age > 33` - `db.users.find({age: {$gt: 33}})`
- `SELECT * FROM users WHERE age <= 33` - `db.users.find({age: {$lte: 33}})`
- `SELECT * FROM users WHERE age > 33 AND age < 40` - `db.users.find({age: {$gt: 33, $lt: 40}})`
- `SELECT * FROM users WHERE age = 32 AND name = ‘Bob’` - `db.users.find({age: 32, name: “Bob”})`
- `SELECT * FROM users WHERE age = 33 OR name = ‘Bob’` - `db.users.find({$or:[{age:33}, {name: “Bob”}]})`
- `SELECT * FROM users WHERE age = 33 ORDER BY name ASC` - `db.users.find({age: 33}).sort({name: 1})`
- `SELECT * FROM users ORDER BY name DESC` - `db.users.find().sort({name: -1})`
- `SELECT * FROM users WHERE name LIKE '%Joe%'` - `db.users.find({name: /Joe/})`
- `SELECT * FROM users WHERE name LIKE 'Joe%'` - `db.users.find({name: /^Joe/})`
- `SELECT * FROM users LIMIT 10 SKIP 20` - `db.users.find().skip(20).limit(10)`
- `SELECT * FROM users LIMIT 1` - `db.users.findOne()`
- `SELECT DISTINCT name FROM users` - `db.users.distinct("name")`
- `SELECT COUNT(*) FROM users` - `db.users.count()`
- `SELECT COUNT(*) FROM users WHERE AGE > 30` - `db.users.find({age: {$gt: 30}}).count()`
- `SELECT COUNT(AGE) FROM users` - `db.users.find({age: {$exists: true}}).count()`
- `UPDATE users SET age = 33 WHERE name = 'Bob'` - `db.users.update({name: "Bob"}, {$set: {age: 33}}, {multi: true})`
- `UPDATE users SET age = age + 2 WHERE name = 'Bob'` - `db.users.update({name: "Bob"}, {$inc: {age: 2}}, {multi: true})`
- `DELETE FROM users WHERE name = 'Bob'` - `db.users.remove({name: "Bob"})`
- `CREATE INDEX ON users (name ASC)` - `db.users.ensureIndex({name: 1})`
- `CREATE INDEX ON users (name ASC, age DESC)` - `db.users.ensureIndex({name: 1, age: -1})`
- `EXPLAIN SELECT * FROM users WHERE age = 32` - `db.users.find({age: 32}).explain()`

## Queries and What They Match

### Queries

- `{a: 10}` - Docs where a is 10, or an array containing the value 10.
- `{a: 10, b: "hello"}` -  Docs where a is 10 and b is "hello".
- `{a: {$gt: 10}}` -  Docs where a is greater than 10. Also available: `$lt (<), $gte (>=), $lte (<=), and $ne (!=)`
- `{a: {$in: [10, "hello"]}}` -  Docs where a is either 10 or "hello". 
- `{a: {$all: [10, "hello"]}}` -  Docs where a is an array containing both 10 and "hello".
- `{"a.b": 10}` -  Docs where a is an embedded document with b equal to 10.
- `{a: {$elemMatch: {b: 1, c: 2}}}` -  Docs where a is an array that contains an element with both b equal to 1 and c equal to 2. 
- `{$or: [{a: 1}, {b: 2}]}` -  Docs where a is 1 or b is 2.
- `{a: /^m/}` -  Docs where a begins with the letter m.
- `{a: {$mod: [10, 1]}}` -  Docs where a mod 10 is 1.
- `{a: {$type: 2}}` -  Docs where a is a string (see bsonspec.org for more)

The following queries cannot use indexes as of MongoDB v2.0. These query forms should normally be 
accompanied by at least one other query term which does use an index:

- `{a: {$nin: [10, "hello"]}}` -  Docs where a is anything but 10 or "hello".
- `{a: {$size: 3}}` -  Docs where a is an array with exactly 3 elements.
- `{a: {$exists: true}}` -  Docs containing an a field.
- `{a: /foo.*bar/}` -  Docs where a matches the regular expression foo.*bar.
- `{a: {$not: {$type: 2}}}` -  Docs where a is not a string. $not negates any of the 
other query operators.

## Update Modifiers

- `{$inc: {a: 2}}` -  Increment a by 2.
- `{$set: {a: 5}}` -  Set a to the value 5.
- `{$unset: {a: 1}}` -  Delete the a key.
- `{$push: {a: 1}}` -  Append the value 1 to the array a.
- `{$push: {a: {$each: [1, 2]}}}` -  Append both 1 and 2 to the array a.
- `{$addToSet: {a: 1}}` -  Append the value 1 to the array a (if the value doesn’t already exist).
- `{$addToSet: {a: {$each: [1, 2]}}}` -  Append both 1 and 2 to the array a (if they don’t already exist).
- `{$pop: {a: 1}}` -  Remove the last element from the array a.
- `{$pop: {a: -1}}` -  Remove the first element from the array a.
- `{$pull: {a: 5}}` -  Remove all occurrences of 5 from the array a.
- `{$pullAll: {a: [5, 6]}}` -  Remove all occurrences of 5 or 6 from the array a
