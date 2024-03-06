# SQLite
SQLite is a solution for local data storage that offer basicaly every thing we need

SQLite does not have a separate server process which means that it essentially stores the data in a file

we can use the package `sqflite` to implement it in flutter

# sqflite Package
## Installation
To install run:
```flutter pub add sqflite```

To use import:
```import 'package:sqflite/sqflite.dart';```

## Creating a Database
The first step in using SQLite with sqflite is to create a database.
in this example we will create a simple database with a single table named `tasks`

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static Database? _database;

  Future<Database> get database async {
    if (_database != null) {
      return _database!;
    }

    // If _database is null, initialize it
    _database = await initDatabase();
    return _database!;
  }

  Future<Database> initDatabase() async {
    // Get the path to the database file
    String path = join(await getDatabasesPath(), 'tasks.db');

    // Open or create the database at the specified path
    return await openDatabase(
      path,
      version: 1,
      onCreate: (Database db, int version) async {
        // Create the tasks table
        await db.execute('''
          CREATE TABLE tasks(
            id INTEGER PRIMARY KEY,
            title TEXT,
            description TEXT,
            completed INTEGER
          )
        ''');
      },
    );
  }
}
```

## CRUD Operations
Now that we have our database set up, let's implement CRUD (Create, Read, Update, Delete) operations.
### Inserting Data

```dart
Future<void> insertTask(Task task) async {
  final Database db = await database;

  await db.insert(
    'tasks',
    task.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace,
  );
}
```
### Querying Data

```dart
Future<List<Task>> getTasks() async {
  final Database db = await database;

  final List<Map<String, dynamic>> maps = await db.query('tasks');

  return List.generate(maps.length, (i) {
    return Task(
      id: maps[i]['id'],
      title: maps[i]['title'],
      description: maps[i]['description'],
      completed: maps[i]['completed'],
    );
  });
}
```
### Updating Data

```dart
Future<void> updateTask(Task task) async {
  final Database db = await database;

  await db.update(
    'tasks',
    task.toMap(),
    where: 'id = ?',
    whereArgs: [task.id],
  );
}
```
### Deleting Data
```dart
Future<void> deleteTask(int id) async {
  final Database db = await database;

  await db.delete(
    'tasks',
    where: 'id = ?',
    whereArgs: [id],
  );
}
```

## Model Class
We also need a model class to represent the data we are working with. In this case, it's the Task class.

```dart
class Task {
  final int? id;
  final String title;
  final String description;
  final int completed;

  Task({
    this.id,
    required this.title,
    required this.description,
    required this.completed,
  });

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'title': title,
      'description': description,
      'completed': completed,
    };
  }
}
```

# Putting It All Together

```dart
void main() async {
  // Initialize the database
  final dbHelper = DatabaseHelper();
  final database = await dbHelper.database;

  // Insert a task
  Task newTask = Task(
    title: 'Learn Flutter',
    description: 'Explore Flutter development and its features.',
    completed: 0,
  );
  await dbHelper.insertTask(newTask);

  // Query all tasks
  List<Task> tasks = await dbHelper.getTasks();
  for (Task task in tasks) {
    print('Task: ${task.title}, Completed: ${task.completed}');
  }

  // Update a task
  Task taskToUpdate = tasks[0];
  taskToUpdate.completed = 1;
  await dbHelper.updateTask(taskToUpdate);

  // Query all tasks after update
  tasks = await dbHelper.getTasks();
  for (Task task in tasks) {
    print('Task: ${task.title}, Completed: ${task.completed}');
  }

  // Delete a task
  await dbHelper.deleteTask(taskToUpdate.id!);

  // Query all tasks after delete
  tasks = await dbHelper.getTasks();
  for (Task task in tasks) {
    print('Task: ${task.title}, Completed: ${task.completed}');
  }
}
```