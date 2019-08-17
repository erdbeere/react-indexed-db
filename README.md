# react-indexed-db

react-indexed-db is a service that wraps IndexedDB database in an "easier to use" service.
It exposes very simple promises API to enable the usage of IndexedDB without most of it plumbing.

## Installation

```js
npm install react-indexed-db
```

## Demo

https://codesandbox.io/s/react-indexed-db-dmmci

## Usage

Create a context with an DB:

```js
import { IndexedDB } from 'react-indexed-db';
import PanelExample from './Panel';

const App: React.FC = () => {
  return (
    <IndexedDB name="MyDB" version={1}>
      <PanelExample />
    </IndexedDB>
  );
};
```

and in any component inside this context you can consume it like bellow:

```js
import { AccessDB } from 'react-indexed-db';

export default function PanelExample() {
  return (
    <AccessDB>
      {db => {
        console.log('MyDB: ', db);
        return <div>{JSON.stringify(db)}</div>;
      }}
    </AccessDB>
  );
}
```

The first argument is the name of your database and the second is the database version.
If you forget the version you the service will default to version 1.

Use the APIs that the ReactIndexedDB service exposes to use indexeddb.
In the API the following functions:

- openDatabase(version, createCallback): opens the database for usage and update it's objectStore/s.
  The first parameter is the version to upgrade the database and the second one is an optional callback that will handle the creation of objectStores for that version if needed.
  **openDatabase** returns a promise that is resolved when the database is open or updated or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    db.openDatabase(1, evt => {
      let objectStore = evt.currentTarget.result.createObjectStore('people', { keyPath: 'id', autoIncrement: true });

      objectStore.createIndex('name', 'name', { unique: false });
      objectStore.createIndex('email', 'email', { unique: true });
    });
    return <div>...</div>;
  }}
</AccessDB>
```

- getByKey(storeName, key): returns the object that is stored in the objectStore by its key.
  The first parameter is the store name to query and the second one is the object's key.
  **getByKey** returns a promise that is resolved when we have the object or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    const [person, setPerson] = useState(null);
    db.getByKey('people', 1).then(
      personFromDB => {
        setPerson(personFromDB);
      },
      error => {
        console.log(error);
      }
    );
    return <div>{person}</div>;
  }}
</AccessDB>
```

- getAll(storeName, keyRange, indexDetails): returns an array of all the items in the given objectStore.
  The first parameter is the store name to query.
  The second parameter is an optional IDBKeyRange object.
  The third parameter is an index details which must include index name and an optional order parameter.
  **getAll** returns a promise that is resolved when we have the array of items or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    const [persons, setPersons] = useState(null);
    db.getAll('people').then(
      peopleFromDB => {
        setPersons(peopleFromDB);
      },
      error => {
        console.log(error);
      }
    );
    return <div>{persons}</div>;
  }}
</AccessDB>
```

- getByIndex(storeName, indexName, key): returns an stored item using an objectStore's index.
  The first parameter is the store name to query, the second parameter is the index and third parameter is the item to query.
  **getByIndex** returns a promise that is resolved when the item successfully returned or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    const [person, setPerson] = useState(null);
    db.getByIndex('people', 'name', 'Dave').then(
      personFromDB => {
        setPerson(peopleFromDB);
      },
      error => {
        console.log(error);
      }
    );
    return <div>{person}</div>;
  }}
</AccessDB>
```

- add(storeName, value, key): Adds to the given objectStore the key and value pair.
  The first parameter is the store name to modify, the second parameter is the value and the third parameter is the key (if not auto-generated).
  **add** returns a promise that is resolved when the value was added or rejected if an error occurred.

Usage example (add without a key):

```js
<AccessDB>
  {db => {
    return (
      <button
        onClick={() => {
          db.add('people', { name: 'name', email: 'email' }).then(
            event => {
              console.log('ID Generated: ', event.target.result);
            },
            error => {
              console.log(error);
            }
          );
        }}>
        Add
      </button>
    );
  }}
</AccessDB>
```

In the previous example I'm using undefined as the key because the key is configured in the objectStore as auto-generated.

- update(storeName, value, key?): Updates the given value in the objectStore.
  The first parameter is the store name to modify, the second parameter is the value to update and the third parameter is the key (if there is no key don't provide it).
  **update** returns a promise that is resolved when the value was updated or rejected if an error occurred.

Usage example (update without a key):

```js
<AccessDB>
  {db => {
    return (
      <button
        onClick={() => {
          db.update('people', { id: 3, name: 'name', email: 'email' }).then(
            () => {
              // Do something after update
            },
            error => {
              console.log(error);
            }
          );
        }}>
        Update
      </button>
    );
  }}
</AccessDB>
```

- delete(storeName, key): deletes the object that correspond with the key from the objectStore.
  The first parameter is the store name to modify and the second parameter is the key to delete.
  **delete** returns a promise that is resolved when the value was deleted or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    return (
      <button
        onClick={() => {
          db.delete('people', 3).then(
            () => {
              // Do something after delete
            },
            error => {
              console.log(error);
            }
          );
        }}>
        Delete
      </button>
    );
  }}
</AccessDB>
```

- openCursor(storeName, cursorCallback, keyRange): opens an objectStore cursor to enable iterating on the objectStore.
  The first parameter is the store name, the second parameter is a callback function to run when the cursor succeeds to be opened and the third parameter is optional IDBKeyRange object.
  **openCursor** returns a promise that is resolved when the cursor finishes running or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    return (
      <button
        onClick={() => {
          db.openCursor(
            'people',
            evt => {
              var cursor = evt.target.result;
              if (cursor) {
                console.log(cursor.value);
                cursor.continue();
              } else {
                console.log('Entries all displayed.');
              }
            },
            IDBKeyRange.bound('A', 'F')
          );
        }}>
        Run cursor
      </button>
    );
  }}
</AccessDB>
```

- clear(storeName): clears all the data in an objectStore.
  The first parameter is the store name to clear.
  **clear** returns a promise that is resolved when the objectStore was cleared or rejected if an error occurred.

Usage example:

```js
<AccessDB>
  {db => {
    return (
      <button
        onClick={() => {
          db.clear('people').then(
            () => {
              // Do something after clear
            },
            error => {
              console.log(error);
            }
          );
        }}>
        Clear Table
      </button>
    );
  }}
</AccessDB>
```

## License

Released under the terms of the [MIT License](LICENSE).