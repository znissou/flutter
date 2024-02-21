# flutter_bloc package

Widgets that make it easy to integrate blocs and cubits into Flutter. Built to work with package:bloc.

## Installation

```flutter pub add flutter_bloc```

Now in your Dart code, you can use:

``import 'package:flutter_bloc/flutter_bloc.dart';``

## table of content
- BlocProvider
- BlocProvider.value
- BlocBuilder
- BlocListener

## BlocBuilder 

BlocBuilder is a Flutter widget which requires a `bloc` and a `builder` function. BlocBuilder handles building the widget in response to new states. 

- The `builder` function will potentially be called many times and should be a pure function that returns a widget in response to the state.

If the bloc parameter is omitted, BlocBuilder will automatically perform a lookup using BlocProvider and the current BuildContext.

```dart
BlocBuilder<BlocA, BlocAState>(
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

Only specify the bloc if you wish to provide a bloc that will be scoped to a single widget and isn't accessible via a parent BlocProvider and the current BuildContext.

```dart
BlocBuilder<BlocA, BlocAState>(
  bloc: blocA, // provide the local bloc instance
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

For fine-grained control over when the builder function is called an optional buildWhen can be provided. buildWhen takes the previous bloc state and current bloc state and returns a boolean. If buildWhen returns true, builder will be called with state and the widget will rebuild. If buildWhen returns false, builder will not be called with state and no rebuild will occur.

for more control of when the builder will be called we have an optional `buildWhen`can be provided that returns a boolean of true call ``builder`` if not no

```dart
BlocBuilder<BlocA, BlocAState>(
  buildWhen: (previousState, state) {
    // return true/false to determine whether or not
    // to rebuild the widget with state
  },
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

## BlocSelector

- it's like the the BlockBuilder but allows developers to filter updates by selecting a new value based on the current bloc state
- Unnecessary builds are prevented if the selected value does not change.
- If the bloc parameter is omitted, BlocSelector will automatically perform a lookup using BlocProvider and the current BuildContext.

```dart
BlocSelector<BlocA, BlocAState, SelectedState>(
  selector: (state) {
    // return selected state based on the provided state.
  },
  builder: (context, state) {
    // return widget here based on the selected state.
  },
)
```

## BlocProvider

- provides a bloc to its children via `BlocProvider.of<T>(context)`
- a single instance of a bloc can be provided to multiple widgets within a subtree.
- In most cases, `BlocProvider` should be used to create new blocs which will be made available to the rest of the subtree. In this case, since BlocProvider is responsible for creating the bloc, it will automatically handle closing the bloc.

```dart
BlocProvider(
  create: (BuildContext context) => BlocA(),
  child: ChildA(),
);
```

- By default, `BlocProvider` will create the bloc lazily, meaning `create` will get executed when the bloc is looked up via `BlocProvider.of<BlocA>(context)`.

To override this behavior and force `create` to be run immediately, `lazy` can be set to false:
```dart
BlocProvider(
  lazy: false,
  create: (BuildContext context) => BlocA(),
  child: ChildA(),
);
```

In some cases, BlocProvider can be used to provide an existing bloc to a new portion of the widget tree. This will be most commonly used when an existing bloc needs to be made available to a new route. In this case, BlocProvider will not automatically close the bloc since it did not create it.

```dart
BlocProvider.value(
  value: BlocProvider.of<BlocA>(context),
  child: ScreenA(),
);
```

then from either `ChildA`, or `ScreenA` we can retrieve `BlocA` with:

```dart
// with extensions
context.read<BlocA>();

// without extensions
BlocProvider.of<BlocA>(context)
```

## MultiBlocProvider
- merges multiple BlocProvider widgets into one.

instead of this:
```dart
BlocProvider<BlocA>(
  create: (BuildContext context) => BlocA(),
  child: BlocProvider<BlocB>(
    create: (BuildContext context) => BlocB(),
    child: BlocProvider<BlocC>(
      create: (BuildContext context) => BlocC(),
      child: ChildA(),
    )
  )
)
``` 
do this:
```dart
MultiBlocProvider(
  providers: [
    BlocProvider<BlocA>(
      create: (BuildContext context) => BlocA(),
    ),
    BlocProvider<BlocB>(
      create: (BuildContext context) => BlocB(),
    ),
    BlocProvider<BlocC>(
      create: (BuildContext context) => BlocC(),
    ),
  ],
  child: ChildA(),
)
```

in order to get the bloc provided by the block provider under it's tree

```
// with extensions
context.read<BlocA>();

// without extensions
BlocProvider.of<BlocA>(context)
```
## BlocListener

we use it to keep litening to the state to trigger an action like open page or show a snackbar 

example
```dart
BlocListener<BlocA, BlocAState>(
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  child: Container(),
)

// and we can specify the bloc of not provided before

BlocListener<BlocA, BlocAState>(
  bloc: blocA,
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  child: Container()
)

// we can add listenWhen eather

BlocListener<BlocA, BlocAState>(
  listenWhen: (previousState, state) {
    // return true/false to determine whether or not
    // to call listener with state
  },
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  child: Container(),
)


```
we can not use it to render any thing in the page and to do that we need to add a `BlocBuilder` as a child of the `BlockListener` and return the wanted widgets to show from the builder param

 in this case we can use `BlocConsumer` instead of using both.

 ## MultiBlocListener

 check <a href='https://bloclibrary.dev/#/flutterbloccoreconcepts?id=multibloclistener'>here</a>

```dart
MultiBlocListener(
  listeners: [
    BlocListener<BlocA, BlocAState>(
      listener: (context, state) {},
    ),
    BlocListener<BlocB, BlocBState>(
      listener: (context, state) {},
    ),
    BlocListener<BlocC, BlocCState>(
      listener: (context, state) {},
    ),
  ],
  child: ChildA(),
)
```

## BlocConsumer

combine BlocBuilder and BlocListener to one widet that have listen param and build param

```dart
BlocConsumer<BlocA, BlocAState>(
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

```dart
BlocConsumer<BlocA, BlocAState>(
  listenWhen: (previous, current) {
    // return true/false to determine whether or not
    // to invoke listener with state
  },
  listener: (context, state) {
    // do stuff here based on BlocA's state
  },
  buildWhen: (previous, current) {
    // return true/false to determine whether or not
    // to rebuild the widget with state
  },
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)
```

## RepositoryProvider




