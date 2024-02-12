# assets and images

Flutter uses the pubspec.yaml file, located at the root of your project, to identify assets required by an app.

Here is an example:

```
flutter:
  assets:
    - assets/my_icon.png
    - assets/background.png

```

To include all assets under a directory, specify the directory name with the / character at the end:

```
flutter:
  assets:
    - directory/
    - directory/subdirectory/

```

**Note: Only files located directly in the directory are included**

## to display 

```
return const Image(image: AssetImage('assets/background.png'));```