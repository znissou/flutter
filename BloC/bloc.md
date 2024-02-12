# what is BloC
the BLoC (Business Logic Component) pattern is a design pattern which helps separate the presentation layer from the business logic.

`Bloc makes it easy to separate presentation from business logic, making your code fast, easy to test, and reusable.`

# Installation

```
flutter pub add bloc
```
now we can import it like this 
```
import 'package:bloc/bloc.dart';
```

**`if you forgot about dart Stream this is a good time to check it again`**

# Cubit

`Cubit` is the core component of the bloc package.

<img src='cubit_architecture_full.png'>

- A Cubit is a class which extends BlocBase and can be extended to manage any type of state.

- A Cubit can expose functions which can be invoked to trigger state changes.

- States are the output of a Cubit and represent a part of your application's state. UI components can be notified of states and redraw portions of themselves based on the current state.

### Creating a Cubit

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
}
```
we need to define the type of state which the Cubit will be managing in this case it is `int` but in more complex cases we could use class types.

###  initial state

```dart
class CounterCubit extends Cubit<int> {
    //we can use an external value like tha example or a static one
  CounterCubit(int initialState) : super(initialState);
}
```
in this case we can instantiate CounterCubit instances with different initial states.

### State Changes
Each Cubit has the ability to output a new state via emit.

the emit method can be called only inside the cubit.

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
}
```

###  Access the state

we can access the current state of the Cubit via the state getter.

```dart
  final cubit = CounterCubit();
  print(cubit.state); // 0
  cubit.increment();
  print(cubit.state); // 1
  //to close the internal state stream.
  cubit.close();
  ```

  ### Stream Usage

  Cubit exposes a Stream which allows us to receive real-time state updates:

  ```dart
  Future<void> main() async {
  final cubit = CounterCubit();
  final subscription = cubit.stream.listen(print); // 1
  cubit.increment();
  // this delay added for this example to avoid canceling the subscription immediately.
  await Future.delayed(Duration.zero);
  await subscription.cancel();
  await cubit.close();
}
```

## Observing a Cubit

When a Cubit emits a new state, a `Change` occurs. We can observe all changes for a given Cubit by overriding `onChange`.

A `Chang`e occurs just before the state of the Cubit is updated. A Change consists of the `currentState` and the `nextState`.

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }
}
```
so when we do this 
```dart
  CounterCubit()
    ..increment()
    ..close();

    //the output is: Change { currentState: 0, nextState: 1 }

```

### Error Handling

check <a href='https://bloclibrary.dev/#/coreconcepts?id=error-handling'>here</a>


# Bloc

A Bloc is a more advanced class which relies on events to trigger state changes rather than functions.

Bloc also extends BlocBase which means it has a similar public API as Cubit.

 However, rather than calling a function on a Bloc and directly emitting a new state, Blocs receive events and convert the incoming events into outgoing states.

 <img src='bloc_architecture_full.png'>

 ### Creating a Bloc

  defin the state that we'll be managing and define the event that the Bloc will be able to process. (na3rfo input li howa l event w output li ra7 tkoun state)

  ```dart
  sealed class CounterEvent {}

final class CounterIncrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
    //the value passed to the super construcor is the initial state
  CounterBloc() : super(0);
}

```

### State Changes

- register event handlers via the `on<Event>` API
- An event handler is responsible for converting any incoming events into zero or more outgoing states.

```dart
sealed class CounterEvent {}

final class CounterIncrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<CounterIncrementPressed>((event, emit) {
      // handle incoming `CounterIncrementPressed` event
    });
  }
}

```

```dart
 on<CounterIncrementPressed>((event, emit) {
      emit(state + 1);
    });
```

In the above snippet, we have registered an EventHandler to manage all CounterIncrementPressed events. For each incoming CounterIncrementPressed event we can access the current state of the bloc via the state getter and emit(state + 1).

Since the Bloc class extends BlocBase, we have access to the current state of the bloc at any point in time via the state getter

-Both blocs and cubits will ignore duplicate states. If we emit `State nextState` where s`tate == nextState`, then no state change will occur.

### Using a Bloc

**Basic Usage**

```dart
Future<void> main() async {
  final bloc = CounterBloc();
  print(bloc.state); // 0
  bloc.add(CounterIncrementPressed());
  // added to ensure we wait for the next event-loop iteration (allowing the EventHandler to process the event).
  await Future.delayed(Duration.zero);
  print(bloc.state); // 1
  //  to close the internal state stream.
  await bloc.close();
}

```

**Stream Usage**

Bloc is a special type of Stream, which means we can also subscribe to a Bloc for real-time updates to its state:

```dart
Future<void> main() async {
  final bloc = CounterBloc();
  final subscription = bloc.stream.listen(print); // 1
  bloc.add(CounterIncrementPressed());
  //added for this example to avoid canceling the subscription immediately.
  await Future.delayed(Duration.zero);
  //calling cancel on the subscription when we no longer want to receive updates and closing the Bloc.
  await subscription.cancel();
  await bloc.close();
}
```

### Observing a Bloc

we can observe all state changes for a Bloc using `onChange`.

```dart
sealed class CounterEvent {}

final class CounterIncrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<CounterIncrementPressed>((event, emit) => emit(state + 1));
  }

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }
}

//to run it 
void main() {
  CounterBloc()
    ..add(CounterIncrementPressed()) // Change { currentState: 0, nextState: 1 }
    ..close();
}
```

because Bloc is event-driven, we are also able to capture information about what triggered the state change.

We can do this by overriding `onTransition`.

- The change from one state to another is called a Transition. A Transition consists of the current state, the event, and the next state.

```dart
sealed class CounterEvent {}

final class CounterIncrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<CounterIncrementPressed>((event, emit) => emit(state + 1));
  }

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }

  @override
  void onTransition(Transition<CounterEvent, int> transition) {
    super.onTransition(transition);
    print(transition);
  }
}

// when adding an event this outputs: 
//Transition { currentState: 0, event: Instance of 'CounterIncrementPressed', nextState: 1 }
//Change { currentState: 0, nextState: 1 }
```

**Note:** onTransition is invoked before onChange and contains the event which triggered the change from currentState to nextState.

**BlocObserver**

Just as before, we can override onTransition in a custom BlocObserver to observe all transitions that occur from a single place.

```dart
class SimpleBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print('${bloc.runtimeType} $transition');
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    print('${bloc.runtimeType} $error $stackTrace');
    super.onError(bloc, error, stackTrace);
  }
}
```
We can initialize the SimpleBlocObserver just like before:

```dart
void main() {
  Bloc.observer = SimpleBlocObserver();
  CounterBloc()
    ..add(CounterIncrementPressed())
    ..close();  
}

//CounterBloc Transition { currentState: 0, event: Instance of 'CounterIncrementPressed', nextState: 1 }
//Transition { currentState: 0, event: Instance of 'CounterIncrementPressed', nextState: 1 }
//CounterBloc Change { currentState: 0, nextState: 1 }
//Change { currentState: 0, nextState: 1 }

```
**Note:** `onTransition` is invoked first (local before global) followed by `onChange`.

**onEvent method**

Bloc instances allow us to override `onEvent` which is called whenever a new event is added to the Bloc. Just like with `onChange` and `onTransition`, `onEvent` can be overridden locally as well as globally.

**in Bloc**
```dart
sealed class CounterEvent {}

final class CounterIncrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<CounterIncrementPressed>((event, emit) => emit(state + 1));
  }

  @override
  void onEvent(CounterEvent event) {
    super.onEvent(event);
    print(event);
  }

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }

  @override
  void onTransition(Transition<CounterEvent, int> transition) {
    super.onTransition(transition);
    print(transition);
  }
}
```
**in BlocObserver**

```dart
class SimpleBlocObserver extends BlocObserver {
  @override
  void onEvent(Bloc bloc, Object? event) {
    super.onEvent(bloc, event);
    print('${bloc.runtimeType} $event');
  }

  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    print('${bloc.runtimeType} $change');
  }

  @override
  void onTransition(Bloc bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print('${bloc.runtimeType} $transition');
  }
}
```

the output of the same main as before is:

```CounterBloc Instance of 'CounterIncrementPressed'
Instance of 'CounterIncrementPressed'
CounterBloc Transition { currentState: 0, event: Instance of 'CounterIncrementPressed', nextState: 1 }
Transition { currentState: 0, event: Instance of 'CounterIncrementPressed', nextState: 1 }
CounterBloc Change { currentState: 0, nextState: 1 }
Change { currentState: 0, nextState: 1 }
```

**Note:** `onEvent` is called as soon as the event is added. The local `onEvent` is invoked before the global `onEvent` in BlocObserver.

### Error Handling

Bloc has an `addError` and `onError` method. We can indicate that an error has occurred by calling `addError` from anywhere inside our Bloc. We can then react to all errors by overriding `onError`.

```dart
sealed class CounterEvent {}

final class CounterIncrementPressed extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<CounterIncrementPressed>((event, emit) {
      addError(Exception('increment error!'), StackTrace.current);
      emit(state + 1);
    });
  }

  @override
  void onChange(Change<int> change) {
    super.onChange(change);
    print(change);
  }

  @override
  void onTransition(Transition<CounterEvent, int> transition) {
    print(transition);
    super.onTransition(transition);
  }

  @override
  void onError(Object error, StackTrace stackTrace) {
    print('$error, $stackTrace');
    super.onError(error, stackTrace);
  }
}
```

**Note:** The local onError is invoked first followed by the global onError in BlocObserver.

### Advanced Event Transformations

check <a href='https://bloclibrary.dev/#/coreconcepts?id=advanced-event-transformations'>here</a>