# Json serialization

when we recieve a json we may need not to just decod it with `jsonDecod` method but to go more in depth and make an object from that json for more readability and easy useage

to do so

## install 

```dart pub add dev:json_serializable```

```dart pub add json_annotation```

```dart pub add dev:build_runner```

## use

1. ``import 'package:json_annotation/json_annotation.dart';```

- specify the part `g.dart` of this model

```part 'example.g.dart';```

with the example is the same file name of the data file.

2. above the data class add annotation ``@JsonSerializable()``

3. add the propeties and the constructor

``` dart
import 'package:json_annotation/json_annotation.dart';

part 'example.g.dart';

@JsonSerializable()
class Person {
  /// The generated code assumes these values exist in JSON.
  final String firstName, lastName;

  /// The generated code below handles if the corresponding JSON value doesn't
  /// exist or is empty.
  final DateTime? dateOfBirth;

  Person({required this.firstName, required this.lastName, this.dateOfBirth})
  `````
  
  4. add the methods fromJsona and toJson to be generated later in the `example.g.dart` file.

  ```dart
  import 'package:json_annotation/json_annotation.dart';

part 'example.g.dart';

@JsonSerializable()
class Person {
  /// The generated code assumes these values exist in JSON.
  final String firstName, lastName;

  /// The generated code below handles if the corresponding JSON value doesn't
  /// exist or is empty.
  final DateTime? dateOfBirth;

  Person({required this.firstName, required this.lastName, this.dateOfBirth});

  /// Connect the generated [_$PersonFromJson] function to the `fromJson`
  /// factory.
  factory Person.fromJson(Map<String, dynamic> json) => _$PersonFromJson(json);

  /// Connect the generated [_$PersonToJson] function to the `toJson` method.
  Map<String, dynamic> toJson() => _$PersonToJson(this);
}
  ```

  5. run  ``dart run build_runner build``. to generate the mothods

  ### for more detailes check <a href='https://pub.dev/packages/json_serializable'>this.</a>
  can be helpfull for types like `enum`

  
