# Chaning Flutter Project Name

Fllow these steps.

## 1. Change `android:label`
`android:label` in the `android/app/src/main/AndroidManifest.xml`

```
<application
    android:name="io.flutter.app.FlutterApplication"
    android:label="New Name"
    android:icon="@mipmap/ic_launcher">  
```

## 2. Change `CFBundleName`
`CFBundleName` in the `ios/Runner/Info.plist`

```
<key>CFBundleName</key>
<string>New Name</string>
```

## 3. Change the title in MaterialApp

```dart
return new MaterialApp(
  title: 'New Name'
  ...);
  ```