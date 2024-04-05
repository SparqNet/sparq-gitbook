# The DBService class and its members

The class that abstracts the database itself and its operations is called DBService. The constructor requires a file system path, it opens the database (if it exists) or creates it on the spot (if it doesn't exist) at that moment.

To close the database, call the `DBService::close` function. Aside from using the structures above, it also uses an internal pointer to a `leveldb::DB` object and a group of member functions that abstract the main CRUD operations for LevelDB:

* `DBService::has` - checks if a key exists in a given database prefix.
* `DBService::get` - gets a value from a key in a given database prefix, if it exists.
* `DBService::put` - inserts a key and value in a given database prefix. Due to the way LevelDB works, updating an entry is the same as inserting a different value in a key that already exists.
* `DBService::del` - deletes a key in a given database prefix.
* `DBService::writeBatch` - same as put + del but batched. The function requires a `WriteBatchRequest` struct, therefore, all operations in it are done in one go.
* `DBService::readBatch` - same as get but returns all entries in a given database prefix. This function has two overloads - the first one returns all entries and requires only the prefix, and the second one returns only values from specific keys and requires both the prefix and a key list.
* `DBService::removeKeyPrefix` - helper function that removes the prefix from a given key (e.g. key `"0003abc"` with value `"123"`, after going through this function it would return `"abc"`).
