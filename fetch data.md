# template to fetch data and data class and data list views
1. Add the http package.
```flutter pub add http```
2. Make a network request using the http package.

3. Convert the response into a custom Dart object.
4. Fetch and display the data with Flutter.

## data class

```dart
class User {
  final int userId;
  final int id;
  final String title;
  final String body;

  const User({
    required this.userId,
    required this.id,
    required this.title,
    required this.body,
  });

   User.fromJson(Map<String, dynamic> json)
  :userId= json['userId'] as int,
  id=json['id'] as int,
  title=json['title'] as String,
  body=json['body'] as String ;

  //or we can use factory constructor that will allow us to return the object ( in normal constructor a return can not be done)

     factory User.fromJson(Map<String, dynamic> json){
    return User(userId: json['userId'],id: json['id'],title: json['title'],body:json['body']);
  }

//this method is invoked automatically when we do jsonEncode(user)

  Map<String, dynamic> toJson() => {
    'userId': userId,
    'id': id,
    'title': title,
    'body': body,
  };
}


```

## fetching method

```dart
Future<Album> fetchAlbum() async {
  final response = await http
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  if (response.statusCode == 200) {

    return Album.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
  } else {

    throw Exception('Failed to load album');
  }
}
```

## the fetching precess i the init method

```dart
class _MyAppState extends State<MyApp> {
  late Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum();
  }
  // ···
}
```

## displayy the data 

```dart
FutureBuilder<Album>(
  future: futureAlbum,
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return Text(snapshot.data!.title);
    } else if (snapshot.hasError) {
      return Text('${snapshot.error}');
    }

    // By default, show a loading spinner.
    return const CircularProgressIndicator();
  },
)
```

# the complete example 

```dart
import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<Album> fetchAlbum() async {
  final response = await http
      .get(Uri.parse('https://jsonplaceholder.typicode.com/albums/1'));

  if (response.statusCode == 200) {
    // If the server did return a 200 OK response,
    // then parse the JSON.
    return Album.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
  } else {
    // If the server did not return a 200 OK response,
    // then throw an exception.
    throw Exception('Failed to load album');
  }
}

class Album {
  final int userId;
  final int id;
  final String title;

  const Album({
    required this.userId,
    required this.id,
    required this.title,
  });

  factory Album.fromJson(Map<String, dynamic> json) {
    return switch (json) {
      {
        'userId': int userId,
        'id': int id,
        'title': String title,
      } =>
        Album(
          userId: userId,
          id: id,
          title: title,
        ),
      _ => throw const FormatException('Failed to load album.'),
    };
  }
}

void main() => runApp(const MyApp());

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late Future<Album> futureAlbum;

  @override
  void initState() {
    super.initState();
    futureAlbum = fetchAlbum();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fetch Data Example',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Fetch Data Example'),
        ),
        body: Center(
          child: FutureBuilder<Album>(
            future: futureAlbum,
            builder: (context, snapshot) {
              if (snapshot.hasData) {
                return Text(snapshot.data!.title);
              } else if (snapshot.hasError) {
                return Text('${snapshot.error}');
              }

              // By default, show a loading spinner.
              return const CircularProgressIndicator();
            },
          ),
        ),
      ),
    );
  }
}
```

# in case the response is a list of objects

## convert the JSON into a list of users

1. Create a parsePhotos() function that converts the response body into a List<Photo>.
2. Use the parsePhotos() function in the fetchPhotos() function.

```dart
// A function that converts a response body into a List<Photo>.
List<Photo> parseUsers(String responseBody) {
  final parsed =
      (jsonDecode(responseBody) as List).cast<Map<String, dynamic>>();

  return parsed.map<User>((json) => User.fromJson(json)).toList();
}
////////////////////////////
Future<List<User>> fetchUsers(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/users'));

  return parseUsers(response.body);
}
```

If you run the `fetchPhotos()` function on a slower device, you might notice the app freezes for a brief moment as it parses and converts the JSON. This is jank, and you want to get rid of it.

You can remove the jank by moving the parsing and conversion to a background isolate using the `compute()` function provided by Flutter. The `compute()` function runs expensive functions in a background isolate and returns the result. In this case, run the `parsePhotos()` function in the background.

```dart
Future<List<User>> fetchUsers(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/users'));

  // Use the compute function to run parsePhotos in a separate isolate.
  return compute(parseUsers, response.body);
}
```


other complete example 
```dart
import 'dart:async';
import 'dart:convert';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));

  // Use the compute function to run parsePhotos in a separate isolate.
  return compute(parsePhotos, response.body);
}

// A function that converts a response body into a List<Photo>.
List<Photo> parsePhotos(String responseBody) {
  final parsed =
      (jsonDecode(responseBody) as List).cast<Map<String, dynamic>>();

  return parsed.map<Photo>((json) => Photo.fromJson(json)).toList();
}

class Photo {
  final int albumId;
  final int id;
  final String title;
  final String url;
  final String thumbnailUrl;

  const Photo({
    required this.albumId,
    required this.id,
    required this.title,
    required this.url,
    required this.thumbnailUrl,
  });

  factory Photo.fromJson(Map<String, dynamic> json) {
    return Photo(
      albumId: json['albumId'] as int,
      id: json['id'] as int,
      title: json['title'] as String,
      url: json['url'] as String,
      thumbnailUrl: json['thumbnailUrl'] as String,
    );
  }
}

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    const appTitle = 'Isolate Demo';

    return const MaterialApp(
      title: appTitle,
      home: MyHomePage(title: appTitle),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
      ),
      body: FutureBuilder<List<Photo>>(
        future: fetchPhotos(http.Client()),
        builder: (context, snapshot) {
          if (snapshot.hasError) {
            return const Center(
              child: Text('An error has occurred!'),
            );
          } else if (snapshot.hasData) {
            return PhotosList(photos: snapshot.data!);
          } else {
            return const Center(
              child: CircularProgressIndicator(),
            );
          }
        },
      ),
    );
  }
}

class PhotosList extends StatelessWidget {
  const PhotosList({super.key, required this.photos});

  final List<Photo> photos;

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
      ),
      itemCount: photos.length,
      itemBuilder: (context, index) {
        return Image.network(photos[index].thumbnailUrl);
      },
    );
  }
}
```