# Database

<ul class="nav">
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#getting-started">Getting Started</a></li>
    <li><a href="#creating-and-updating-docs">Creating And Updating Documents</a></li>
    <li><a href="#trashing-and-restoring-docs">Trashing And Restoring Documents</a></li>
    <li><a href="#deleting-docs">Deleting Documents</a></li>
    <li><a href="#querying-db">Querying the DB</a></li>
    <li><a href="#syncing-db">Syncing the DB</a></li>
    <li><a href="#misc-methods">Misc Methods</a></li>
</ul>

## Introduction <a name="introduction"></a>

Koti Cloud provides you with an offline-first JSON database that is synced with Koti Cloud servers. You don't have to solve the data storage and synchronization problem yourself. Besides, since all the data is stored at Koti Cloud, your users will be able to access their data with a single Koti Cloud account no matter what app they are using.

## Getting Started <a name="getting-started"></a>

To use the database, you need to specify your DB's name when initializing a new Koti Cloud app. The name could be absolutely anything and doesn't really matter. It won't interfere with other apps' databases.

```javascript
import { App } from '../../../sdk/v0.4.x/sdk.js';

window.sdk = new App();

sdk.init({
    ...
    // Set the name here
    db: 'my-app-db',
});
```

This is enough for you to start using the database. There's actually a separate database module called `DB`, but you shouldn't use it directly. Instead, you should access the instance of the `DB` class via your global `App` instance: `sdk.db`.

Internally, the DB module is an IndexedDB wrapper (which is a NoSQL DB available in modern browsers) which communicates with the Koti Cloud servers. An IndexedDB without a wrapper is pretty difficult to use and the API is very confusing.

The data is stored in JSON format and grouped into `collections`. The `collections` don't have a fixed schema and there are no database migrations. Thanks to this you can start using the DB right away and adapt its structure on the go. A single object or a "row" of data inside a `collection` is called `document`.

## Creating And Updating Documents <a name="creating-and-updating-docs"></a>

A document is a regular JavaScript object with arbitrary fields. Since there's no fixed schema, each document in a collection can have different fieldset. To create a new document, simply create a JS object, then tell the DB to store it in a specific collection using the `collection` and `create` methods in a chain:

```javascript
const doc = {
    title: 'Grocery List',
    body: 'Onions',
};

doc = sdk.db.collection('notes').create(doc);

console.log(doc._id);
```

The new document will be assigned a unique ID under `_id`, a `_created_at` and `_updated_at` timestamps.

Updating a document is similar. Simply make changes to your JS object and call the `update` method on the DB passing the object:

```javascript
doc.body = doc.body + ' and garlic!';

sdk.db.collection('notes').update(doc);
```

The `_updated_at` timestamp will be updated to the current timestamp. If you want to keep this timestamp intact, set the `timestamps` option to `false`:

```javascript
sdk.db.collection('notes').update(doc, { timestamps: false });
```

There's also a useful `updateOrCreate` method which speaks for itself. If the object has an `_id` - find the document in the DB and update it, otherwise create a new one:

```javascript
doc = sdk.db.collection('notes').updateOrCreate(doc);
```

You can also use pass options for the update here:

```javascript
doc = sdk.db.collection('notes').updateOrCreate(doc, { timestamps: false });
```

## Trashing And Restoring Documents <a name="trashing-and-restoring-docs"></a>

A document can be trashed (temporarily deleted) and restored from trash. To trash a document, do:

```javascript
sdk.db.collection('notes').trash(doc);
```

To restore a trashed document:

```javascript
sdk.db.collection('notes').restore(doc);
```

## Deleting Documents <a name="deleting-docs"></a>

To permanently delete a document, either trashed or not, do the following:

```javascript
sdk.db.collection('notes').delete(doc);
```

## Querying the DB <a name="querying-db"></a>

You can build queries by chaining certain methods after `sdk.db.collection(...)`.

### How Querying Works Internally

There's one thing to note. Koti Cloud DB uses IndexedDB which is pretty hard to query. For each field combination you are going to query there should be an index created. Each index also makes your local database bigger. This is a problem all existing IndexedDB wrappers have to deal with and there's no perfect solution.

We chose not to oblige our app developers with defining any indexes or rigid data schemas. This makes the DB setup and usage a breeze and the locally stored app data smaller.

Internally, all the documents are stored in a single "store" with a single index on the `collection` field. So, when you make a DB query, the filtering actually happens post-factum after all the data for a collection has been retrieved from the DB. This shouldn't be a problem in most cases, but is just something to consider performance-wise in case you have really big collections. You might want to consider splitting your single-collection data into smaller chunks and storing them into multiple related collections.

### Get Single Document by ID

You can get a single document by its ID. Since document IDs are unique between all the collections, so specifying a collection name is optional.

```javascript
sdk.db.collection('notes')
    .getById('4c3c4b5f57859a26ee5c9ea894a2a77011b5852e2169047e276b87c9f1cea1e6');
```

### Get All Results

The `get()` method is used to get all query results:

```javascript
data = await sdk.db.collection('notes').get();

// Array of documents
console.log(data.docs);
// Total number of documents
console.log(data.length);
```

### Get First Result

If you only want to get the first result from the query, use the `first()` method:

```javascript
doc = await sdk.db.collection('notes').first();

console.log(doc);
```

### Filter Results

You can filter the results by any field values. To achieve this use the `where(field, operator, value)` or `where(field, value)` method. The default `operator` value is `=` (equals).

```javascript
data = await sdk.db.collection('notes')
    .where('category', 'work')
    .where('_updated_at', '>=', 1614923810)
    .get();
```

In the example above we will get all the notes from the `work` category which were updated on or before the `1614923810` timestamp.

The supported operators are all the standard operators you could use in a JavaScript `if` condition.

You can have as many `where` clauses as you want. Multiple clauses are joined via `AND` (`&&`) operator. More complex logic is not supported at the moment.

### Sort Results

To sort the results by a field, use the `orderBy(field, dir = 'asc')` method. You can sort the data by multiple fields by applying multiple `orderBy` clauses.

```javascript
data = await sdk.db.collection('notes')
    .orderBy('_updated_at')
    .orderBy('title')
    .get();
```

### Group Results

You can group the results by multiple fields using the `groupBy(field)` method.

```javascript
data = await sdk.db.collection('transactions')
    .groupBy('date')
    .groupBy('account')
    .get();
```

The returned data will have a tree structure. The results for the example query above would look like this:

```javascript
{
    '2021-03-01': {
        'saving-account': [
            ... (array of docs)
        ],
        'credit-card': [
            ... (array of docs)
        ]
    },
    '2021-03-02': {
        'saving-account': [
            ... (array of docs)
        ]
        // No credit-card transactions on this date
    },
    ...
}
```

### Results Subset

You can get a subset of all the results. This is useful for pagination. Use the `from(from)` and `limit(limit)` methods.

```javascript
// Get 20 transactions starting from the 100th transaction (transactions 100-120)
data = await sdk.db.collection('transactions')
    .from(100)
    .limit(20)
    .get();
```

### Trashed Documents

Trashed documents are only marked as trashed and not actually deleted from the DB. By default trashed documents are excluded from the returned query results. To include them, use the `withTrashed()` method.

```javascript
data = await sdk.db.collection('notes')
    .withTrashed()
    .get();
```

You might also want to get only the trashed documents using the `onlyTrashed()` method.

```javascript
data = await sdk.db.collection('notes')
    .onlyTrashed()
    .get();
```

## Syncing the DB <a name="syncing-db"></a>

Syncing the DB is as easy as calling the `sync()` method.

```javascript
sdk.db.sync();
```

There are 3 associated events you can listen to: `syncing`, `synced`, `sync-failed`. For example:

```javascript
sdk.db.on('synced', () => {
    // The DB has just synced. Re-fetch the data.
});
```

The only time the DB is synced automatically is on the app start. In this case expect the `syncing` and `synced` events to be fired twice.

## Misc Methods <a name="misc-methods"></a>

### `move(doc, collection)`

Move a document to a different collection.

```javascript
const doc = await sdk.db.collection('categories')
    .getById('4c3c4b5f57859a26ee5c9ea894a2a77011b5852e2169047e276b87c9f1cea1e6');

sdk.db.collection('categories').move(doc, 'tags');
```

### `getCollections()`

Returns the list of all the existing in the DB collections.

```javascript
sdk.db.getCollections();
```