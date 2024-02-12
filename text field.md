# Texts and focus

Flutter provides two text fields: `TextField` and `TextFormField`.

## TextField

TextField is the most commonly used text input widget.

To remove the decoration entirely set the decoration to null.

```dart
TextField(
  decoration: InputDecoration(
    border: OutlineInputBorder(),
    hintText: 'Enter a search term',
  ),
),
```

## TextFormField

TextFormField wraps a TextField and integrates it with the enclosing Form. This provides additional functionality, such as validation and integration with other FormField widgets.

```dart
TextFormField(
  decoration: const InputDecoration(
    border: UnderlineInputBorder(),
    labelText: 'Enter your username',
  ),
),
```

# Retrieve the value of a text field
1. Create a TextEditingController.
2. Supply the TextEditingController to a TextField.
3. handle change on the text field

### 1. Create a TextEditingController

To retrieve the text a user has entered into a text field, create a `TextEditingController` and supply it to a `TextField` or `TextFormField`.

**Important:** Call `dispose` of the `TextEditingController` when you’ve finished using it. This ensures that you discard any resources used by the object.

```dart
class _MyCustomFormState extends State<MyCustomForm> {
  // Create a text controller and use it to retrieve the current value
  // of the TextField.
  final myController = TextEditingController();

  @override
  void dispose() {
    // Clean up the controller when the widget is disposed.
    myController.dispose();
    super.dispose();
  }
  ```

  ### 2. Supply the TextEditingController to a TextField

  ```dart
  return TextField(
  controller: myController,
);
```

### 3. handling change
 you can do what ever you want with the `controller.text` to handle the text on an event like the `onChange` in the text field

 ```dart
 class _MyCustomFormState extends State<MyCustomForm> {
  // Create a text controller and use it to retrieve the current value
  // of the TextField.
  final myController = TextEditingController();

  @override
  void initState() {
    super.initState();

    // Start listening to changes.
    myController.addListener(_printLatestValue);
  }

  @override
  void dispose() {
    // Clean up the controller when the widget is removed from the widget tree.
    // This also removes the _printLatestValue listener.
    myController.dispose();
    super.dispose();
  }

  void _printLatestValue() {
    final text = myController.text;
    print('Second text field: $text (${text.characters.length})');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Retrieve Text Input'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              // here you can provide a function to handle the change or just create an anonymos function here
              onChanged: (text) {
                print('First text field: $text (${text.characters.length})');
              },
            ),
            TextField(
              controller: myController,
            ),
          ],
        ),
      ),
    );
  }
}
```

# Focus

To give focus to a text field as soon as it’s visible, use the `autofocus` property.

```dart
TextField(
  autofocus: true,
);
```

### Focus a text field when a button is tapped

1. Create a FocusNode.
```dart
class _MyCustomFormState extends State<MyCustomForm> {
  // Define the focus node. To manage the lifecycle, create the FocusNode in
  // the initState method, and clean it up in the dispose method.
  late FocusNode myFocusNode;

  @override
  void initState() {
    super.initState();

    myFocusNode = FocusNode();
  }

  @override
  void dispose() {
    // Clean up the focus node when the Form is disposed.
    myFocusNode.dispose();

    super.dispose();
  }
  ```
2. Pass the FocusNode to a TextField.

```dart
@override
Widget build(BuildContext context) {
  return TextField(
    focusNode: myFocusNode,
  );
}
```
3. Give focus to the TextField when a button is tapped.
```dart
FloatingActionButton(
  // When the button is pressed,
  // give focus to the text field using myFocusNode.
  onPressed: () => myFocusNode.requestFocus(),
),
```