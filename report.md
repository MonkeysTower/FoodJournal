## Debagging and fixing report
This file contains errors encountered while working with the application.


### Error applying the method to the wrong element.
In the database.js file in the source code, we tried to call the execAsync method on the transaction object (tx), which is passed to the withTransactionAsync function. However, in the expo-sqlite library, the execAsync method exists only on the database object (db), not on the transaction object. As a consequence, you should remove all passed parametres and replace all ‘tx’ with ‘db’.


Initial code:
```
await db.withTransactionAsync(async (tx) => {
      await tx.execAsync(
        `CREATE TABLE IF NOT EXISTS users (
          id INTEGER PRIMARY KEY AUTOINCREMENT, 
          email TEXT UNIQUE, 
          password TEXT
        );`
      );
      
  await tx.execAsync(
    `CREATE TABLE IF NOT EXISTS journals (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    userId INTEGER, 
    image TEXT, 
    description TEXT, 
    date TEXT, 
    category TEXT, 
    FOREIGN KEY(userId) REFERENCES users(id)
    );`
  );
}
```


Final code:
```
await db.withTransactionAsync(async () => {
  await db.execAsync(
    `CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    email TEXT UNIQUE, 
    password TEXT
    );`
  );
    
  await db.execAsync(
    `CREATE TABLE IF NOT EXISTS journals (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    userId INTEGER, 
    image TEXT, 
    description TEXT, 
    date TEXT, 
    category TEXT, 
    FOREIGN KEY(userId) REFERENCES users(id)
    );`
  );
}
```

### Fixes executeSQL function due to errors in other parts of the code
In the declared function executeSql in the file database.js, for the correct data transfer should be removed withTransactionAsync, and change the structure of the function so that for SELECT queries used method getAllAsync, and for all other (INSERT,DELETE,UPDATE) - runAsync designed for this.

Initial code:
```
const executeSql = async (query, params = []) => {
  try {
    if (!isInitialized) {
      await initDatabase();
    }
    
    return await db.withTransactionAsync(async (tx) => {
      return await tx.execAsync(query, params);
    });
  } catch (error) {
    console.error('SQL execution error:', error);
    throw error;
  }
};
```


Final code:
```
const executeSql = async (query, params = []) => {
  try {
    if (!isInitialized) {
      await initDatabase();
    }

    // Определяем тип запроса
    const isSelectQuery = query.trim().toUpperCase().startsWith('SELECT');

    if (isSelectQuery) {
      // Для SELECT используем allAsync (возвращает все строки)
      return await db.getAllAsync(query, params);
    } else {
      // Для INSERT, UPDATE, DELETE используем runAsync
      return await db.runAsync(query, params);
    }
  } catch (error) {
    console.error('SQL execution error:', error);
    throw error;
  }
};
```

### Removing incorrect formatting and adding error handling

In the homeScreen.js file in the loadJournals function you should do the following:
- Remove .rows._array since it is already an array of data.
- Add error.message output (if it exists) for more informative logging

### Changing the working items of the library (upgrading to a new version)
The homeScreen.js file needs to be updated to work with the camera, as the current version of the expo-camera library does not have a Camera method. Instead, CameraView and useCameraPermissions are used instead. Also, the way ref is passed has changed - it is now implemented via useRef.

