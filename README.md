# Learning MongoDB

Author: Lin Dong

Date: Sun Mar  8 13:36:08 PDT 2015

Lets talk about Mongo, [src code](https://github.com/tutsplus/learning-mongodb)

## 1
Intro

## 2 install
Install MongoDB and DB directory

1. `mkdir -p /data/db`
2. `sudo !!`,  `!!` refers to the last command
3. `sudo chown \`id -u\` /data/db`,  `id -u` gives the current user id

Boot up the server and connect to it

1. `mongod`, to start the server
2. `mongo`, to connect to the server


My personal notes for development, just create a directory under current user.

1. `mkdir -p ~/data/db`
2. `mongod --dbpath ~/data/db`

## 3 tools
1. `mongoimport`, import json
2. `mongoexport`, export json
3. `mongodump`, export binary
4. `mongorestore`, import binary
5. `bsondump`, convert binary to json
6. `mongostat`, show status of current running mongo

## 4 terms
NoSQL Database Terms

1. Database: Like a database in RDBMS(relational database management system)
2. Collection: Like RDBMS Table, no schema
3. Document: Like a RDMBS Record, or Row
4. Field: Like a RDBMS Column {key, value}

## 5 create
1. `show dbs`, show list of DBs that are available
2. `use DB_NAME`, i.e. `use test`, `use bookmarks`
3. `db`, show current DB name
4. `db.links.count()`,
5. `db.links.insert({ title: 'Tut+', url: 'http://tutsplus.com', comments: 'some comments', tags: ['tutorials', 'dev'], saved_on: new Date()})`
6. `db.links.save(doc)`, save doc, save will automatically invoke update, insert or whichever one is appropriate.
7. `db.links.find()`, will return all collections
8. `db.links.find().forEach(printjson)` for **PrettyPrint**
9. `show collections`, show current collections

## 6 objectIDs
1. `_id`, immutable and unique. `12bytes` contains date, process id, and server name. So we can get the date of creation by `db.links.find()[0]._id.getTimestamp()`

## 7 relations
* Normalization, reduce the amount of dependency and redundancy
* De-normalization, the other way

1. Use the same `objectID` for users and links
2. have to update the collections manually

## 8 queries 1
Set up the DB

1. rename `08 - bookmarks.js` to `bookmarks.js`
2. `mongo 127.0.0.1/bookmarks bookmarks.js`
3. `mongo bookmarks`

Queries

Find All

1. `db.users.find().forEach(printjson)`
2. `db.links.find().forEach(printjson)`
3. `db.users.find({FILEDNAME: VAL})`, i.e. `db.users.find({email: 'johndoe@gmail.com'})`

`find()` will return a **cursor**, or an array of objects.

Find One:

1. `db.users.findOne({FILEDNAME: VAL})`


`findOne()` will return a **document**, or one object


Find fields and only show specific attributes:

1. `db.links.find({favorites: 100}, {title:1, url: true})`
2. `db.links.find({favorites: 100}, {title:0})`
3. cannot mix and combine `1` and `0`, with exception of `objectID`

i.e.

```mongo
var john = db.users.findOne({'name.first': 'John'});
db.links.find({userId: john._id}, {title: 1, _id: 0});
```

## 9 queries 2

For integers

```mongo
// >
db.links.find({favourites: {$gt: 150}}, {title: 1, favourites: 1, _id: 0});
// <
db.links.find({favourites: {$lt: 150}}, {title: 1, favourites: 1, _id: 0});
// <=
db.links.find({favourites: {$lte: 150}}, {title: 1, favourites: 1, _id: 0});
// >=
db.links.find({favourites: {$gte: 150}}, {title: 1, favourites: 1, _id: 0});

// combintions
db.links.find({favourites: {$gt: 100, $lt: 300}}, {title: 1, favourites: 1, _id: 0});
```

For in ranges

```mongo
db.users.find({'name.first': {$in: ['John', 'Jane'] }});

// either John and Jane
db.users.find({'name.first': {$in: ['John', 'Jane'] }}, {'first': 1});

// not John and not Jane
db.users.find({'name.first': {$in: ['John', 'Jane'] }}, {'first': 1});

// for array
db.links.find({tags: {$in: ['marketplace','code']}}, {title: 1, tags:1, _id: 0});

// for all matched criteria
db.links.find({tags: {$all: ['marketplace','code']}}, {title: 1, tags:1, _id: 0});
```

## 10 queries 3
More operators:

```mongo
// not equal
db.links.find({tags: {$ne: 'code'}}, {title: 1, tags:1});

// not equal for array
db.links.find({tags: {$ne: ['code']}}, {title: 1, tags:1});

// return all users whose first name is John or whose last name is Wilson
db.users.find({$or: [{'name.first': 'John'}, {'name.last': 'Wilson'}]}, {name: 1});

// return all users whose first name is neither John and last name is not Wilson
db.users.find({$nor: [{'name.first': 'John'}, {'name.last': 'Wilson'}]}, {name: 1});

// return both
db.users.find({$and: [{'name.first': 'Bob'}, {'name.last': 'Smith'}]}, {'name.first': 1});

// check if fileds exist
db.users.find({email: {$exists: true}}, {name:1, _id:0});

// mod
db.links.find({favourites: {$mod: [5, 0]}}, {title:1, favourites:1, _id:0});

// not
db.links.find({favourites: {$not: {$mod: [5, 0]}}}, {title:1, favourites:1, _id:0});

// match
db.users.find({ logins: {$elemMatch: {minutes: 20} } });
db.users.find({ logins: {$elemMatch: {at : { $lt: new Date(2012, 3, 30) }}}});

// where
db.users.find({ $where: "this.name.first == 'John'"});
db.users.find({ $where: 'this.name.first == "John"'});
db.users.find({ $where: "this.name.first == 'John'", age: 30});

// shortcuts for using where ONLY
db.users.find( 'this.name.first == "John"' );

var f = function() {return this.name.first === 'John'}
db.users.find(f);
db.users.find({ $where: f });
```

## 11 queries 4
Getting distinct elements

```mongo
db.links.distinct('favourites');
db.links.distinct('url');
```

Grouping

```mongo
db.links.group({
  key: {userId: true},
  initial: {favCount: 0},
  reduce: function(doc, o) {
    o.favCount += doc.favourites
  },
  finalize: function(o) {
    o.name = db.users.findOne( {_id: o.userId}).name;
  }
});
```

Regex

```
db.links.find({title: /tuts\+$/ });
db.links.find({title: /tuts\+$/ }, {title: 1});

// combine regex and other operators
db.links.find({title: {$regex: /tuts\+$/, $ne: 'Mobiletuts+' }}, {title: 1});
```

## 12 queries 5
Count

```mongo
db.users.find({'name.first' : 'John'}).count();
db.users.count({'name.first' : 'John'});
db.users.count();
db.links.count();
```

Sorting

```mongo
db.links.find({}, {title: 1, _id: 0});

// assending order
db.links.find({}, {title: 1, _id: 0}).sort({title: 1});

// descending order
db.links.find({}, {title: 1, _id: 0}).sort({title: -1});
db.links.find({}, {title: 1, favourites: 1,  _id: 0}).sort({favourites: -1, title: 1});

// max
db.links.find({}, {title: 1, favourites: 1, _id: 0}).sort({favourites: -1}).limit(1)

// min
db.links.find({}, {title: 1, favourites: 1, _id: 0}).sort({favourites: 1}).limit(1)
```

Soring can be used for **paging**

```mongo
db.links.find({}, {title: 1, _id: 0}).skip(0 * 3).limit(3);

db.links.find({}, {title: 1, _id: 0}).skip(1 * 3).limit(3);
```

## 13 update 1

Update by replacement

```mongo
db.users.update({'name.first': 'John'}, { job: 'developer'});
```

Upsert, update and insert

```mongo
// will fail silently
db.users.update({name: 'Kate Wills'}, {name: 'Kate Wills', job: 'Lisp Developer'});
db.users.find({name: 'Kate Wills'});

// will insert
db.users.update({name: 'Kate Wills'}, {name: 'Kate Wills', job: 'Lisp Developer'}, true);
```

Updated by modification

```mongo
var n = { title: 'Nettuts+'};
db.links.fink(n);

db.links.update(n, {$inc: {favourites: 5}});
db.links.find(n);

// update
var q = {name: 'Kate Wills'};
db.users.find(q);
db.users.update(q, {$set: {job: 'Web Developer'}});
db.users.update(q, {$set: {email: 'kw@gmail.com'}});

// unset
db.users.update(q, {$unset: {job: 'Web Dev'}});
```

Update by modification for bathing, 3rd parameter to be false,  4th parameter to be true.

```mongo
db.users.insert({name: {first: 'Jane'}});
db.users.find({'name.first':'Jane'});

// this will only update the first match record
db.users.update({ 'name.first': 'Jane' }, {$set: {Job: 'Web developer'}});

// this will update all matched records
db.users.update({ 'name.first': 'Jane' }, {$set: {Job: 'Web developer'}}, false, true);
```

Save method

```mongo
var bob = db.users.findOne({ 'name.first': 'Bob'});
bob.job = 'Server Admin';
db.users.save(bob);
db.users.find({ 'name.first': 'Bob'});
```

findAndModify
```mongo
db.users.findAndModify({
  query: {'name': 'Kate Wills'},
  update: { $set: {age: 30}},
  new: true // default is false
});

// return the last record BEFORE modification
db.users.findAndModify({ query: {'name': 'Kate Wills'}, update: { $set: {age: 40}}});
db.users.find({ name: 'Kate Wills'});


db.links.findAndModify({
  query: {favourites: 100},
  update: { $inc: { favourites: 10}},
  fields: {title: 1, favourites: 1, _id: 0},
  new: true // default is false
});
```

## 14 update 2

Add: `push`, `pushAll`, `addToSet`

```mongo
// Push
var n = {title: 'Nettuts+'};
db.links.findOne(n);
db.links.update(n, {$push: {tags: 'blog'}});

// PushAll
db.links.update(n, {$pushAll: {tags: ['blog', 'text', 'python']}});


// addToSet, add single elelemt, no duplicates will occur
db.links.update(n, {$addToSet: {tags: 'blog'}});

// adToSet, avoid duplicates in existing elements
db.links.update(n, {$addToSet: {tags: { $each: ['blog', 'text', 'python']}}});
```

Remove: `pull` and `pullAll`

```mongo
// pull
db.links.update(n, {$pull : {tags: 'python'}});

// pullAll
db.links.update(n, {$pullAll : {tags: ['code', 'text']}});
```

Pop: remove either the first or last element

```mongo
// remove from the end
db.links.update(n, {$pop : {tags: 1}});

// remove from the beginning
db.links.update(n, {$pop : {tags: -1}});
```

Position operator

```mongo
db.users.update({'logins.minutes':20 }, {$inc: {'logins.$.minutes': 10}}, false, true);
db.users.find({'logins.minutes': 30}).forEach(printjson);

db.users.update({'logins.minutes': 30}, {$set: {random: true}}, false, true)
db.users.find({'logins.minutes': 30}).forEach(printjson);


db.users.update({'logins.minutes': 30}, {$set: {'logins.$.location': 'unknown'}}, false, true)
```

Rename operator

```mongo
db.users.update({ random: true}, {$rename: {random: 'something else'}}, false, true);
```

## 15 delete
Delete contents

```mongo
db.users.remove(); // it will delete the contents of users, not the users itself
db.users.count();

db.users.remove({'name.first': 'John'});

db.users.findAndModify({
  query: {'name.first': /B/},
  remove: true
});

db.users.drop(); // it will remove the container as well

use other
db.test.insert({});
show collections
show dbs
db
db.dropDatabase();  // drop the current database
show dbs
```

## 16 indexes
Without Indexing

```mongo
use bookmarks;
db.links.find({'title': 'Nettuts+'});
db.links.find( {title: 'Nettuts+'}).explain();

// id is indexed
db.links.find({_id: ObjectId("54fcbcaba08a3053ccac10e8")});
db.links.find({_id: ObjectId("54fcbcaba08a3053ccac10e8")}).explain();
```

With Indexing

```mongo
db.links.ensureIndex({title: 1});
db.system.indexes.find();
db.links.find( {title: 'Nettuts+'}).explain();

// ensure the index without the same
db.links.ensureIndex({title: 1}, {unique: true, dropDups: true});

// sparse
db.links.ensureIndex({title: 1}, {sparse: true});

// compound index, i.e. cookbook
db.links.ensureIndex({title: 1, url: 1});

// drop index that no longer need
db.system.indexes.find();
db.links.dropIndex('INDEX_NAME');

// i.e.
db.links.dropIndex('title_1_url_1');
```

Rule of thumb: indexing the fields that query most frequent.

## 17 Php

No notes available.

##18 Node
[Source code](https://github.com/tutsplus/learning-mongodb/tree/master/18%20-%20mongo-js)

```js
//package.json
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "nodemon app.js"
  },
  "dependencies": {
    "express": "3.0.0",
    "jade": "*",
    "mongodb": "*",
    "nodemon": "*"
  }
}
```

## 19 NoSQL vs SQL

1. NoSQL: good at incredibly huge amounts of data
2. Designed for clusters of servers

