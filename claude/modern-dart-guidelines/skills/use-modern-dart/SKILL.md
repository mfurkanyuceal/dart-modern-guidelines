---
name: use-modern-dart
description: Apply modern Dart syntax guidelines based on project's Dart SDK version. Use when user ask for modern Dart code guidelines.
---

# Modern Dart Guidelines

## Detected Dart Version

!`grep -rh "sdk:" --include="pubspec.yaml" . 2>/dev/null | grep -v "flutter" | grep -o '[0-9]\+\.[0-9]\+\.[0-9]\+' | head -1 | grep . || echo unknown`

## How to Use This Skill

DO NOT search for pubspec.yaml files or try to detect the version yourself. Use ONLY the version shown above.

**If version detected (not "unknown"):**
- Say: "This project is using Dart X.Y.Z, so I'll stick to modern Dart best practices and freely use language features up to and including this version. If you'd prefer a different target version, just let me know."
- Do NOT list features, do NOT ask for confirmation

**If version is "unknown":**
- Say: "Could not detect Dart SDK version in this repository"
- Use AskUserQuestion: "Which Dart version should I target?" → [2.12] / [2.17] / [3.0] / [3.3] / [3.6] / [3.7] / [3.8] / [3.10]

**When writing Dart code**, use ALL features from this document up to the target version:
- Prefer modern language constructs and library APIs over legacy patterns
- Never use features from newer Dart versions than the target
- Never use outdated patterns when a modern alternative is available

---

## Features by Dart Version

### Dart 2.12+

- **Sound null safety**: All types are non-nullable by default. Use `Type?` for nullable types.

```dart
// Before:
String name = null; // Was allowed, caused runtime crashes

// After:
String name = 'Dart';  // Non-nullable by default
String? name = null;    // Explicitly nullable with ?
```

- **`late` variables**: Defer initialization without making the type nullable. Also enables lazy initialization.

```dart
// Before:
String? _data; // Had to make nullable even though always set before use
void display() => print(_data!); // Forced to use ! everywhere

// After:
late String _data; // Non-nullable, initialized later
void display() => print(_data); // No ! needed

// Lazy initialization:
late final config = expensiveComputation(); // Runs only when first accessed
```

- **`required` named parameters**: Compiler-enforced required named parameters instead of `@required` annotation.

```dart
// Before:
import 'package:meta/meta.dart';
User({@required String name, @required String email}); // Only a hint

// After:
User({required this.name, required this.email}); // Compile-time error if missing
```

- **Flow analysis for type promotion**: Nullable types are automatically promoted after null checks.

```dart
// Before:
void process(String? name) {
  if (name != null) {
    print(name!.toUpperCase()); // Needed ! even after check
  }
}

// After:
void process(String? name) {
  if (name == null) return;
  print(name.toUpperCase()); // Promoted to String automatically
}
```

- **`Never` type**: Bottom type indicating a function never returns normally.

```dart
Never throwError(String msg) => throw Exception(msg);

String describe(int? value) {
  if (value == null) throwError('Value required');
  return 'Value: $value'; // value promoted to int
}
```

### Dart 2.13+

- **Type aliases for any type**: `typedef` now works for any type, not just function signatures.

```dart
// Before (only functions):
typedef Callback = void Function(String);

// After (any type):
typedef StringList = List<String>;
typedef JSON = Map<String, dynamic>;
typedef UserMap = Map<String, List<Map<String, int>>>;
```

### Dart 2.14+

- **Unsigned right shift**: `value >>> 4` instead of manual bit masking. Fills with zeros.

### Dart 2.15+

- **Constructor tear-offs**: Reference constructors as first-class functions without wrapping in a lambda.

```dart
// Before:
final widgets = names.map((name) => Text(name)).toList();
final items = data.map((json) => Item.fromJson(json)).toList();

// After:
final widgets = names.map(Text.new).toList();
final items = data.map(Item.fromJson).toList();
```

- **Explicit generic instantiation**: `id<int>` instead of `(int x) => id<int>(x)` when tearing off generic functions.

### Dart 2.17+

- **Enhanced enums**: Enums can have fields, constructors, methods, and implement interfaces.

```dart
// Before:
enum Color { red, green, blue }
extension ColorExt on Color {
  String get hex {
    switch (this) {
      case Color.red: return '#FF0000';
      case Color.green: return '#00FF00';
      case Color.blue: return '#0000FF';
    }
  }
}

// After:
enum Color {
  red('#FF0000'),
  green('#00FF00'),
  blue('#0000FF');

  final String hex;
  const Color(this.hex);
}
```

- **Super-initializer parameters**: Forward constructor parameters to superclass without repetition.

```dart
// Before:
class Dog extends Animal {
  final String breed;
  Dog(String name, int age, this.breed) : super(name, age);
}

// After:
class Dog extends Animal {
  final String breed;
  Dog(super.name, super.age, this.breed);
}
```

- **Named arguments anywhere**: Named arguments can be freely interleaved with positional arguments.

```dart
// Before:
createUser('123', name: 'Alice', age: 30); // Named must be last

// After:
createUser(name: 'Alice', '123', age: 30); // Named args can appear before positional args
```

### Dart 2.18+

- **Enhanced type inference**: Type information flows between arguments in generic function calls. `buildMap(['a', 'b'], (key) => key.length)` works without explicit type arguments.

### Dart 3.0+

- **Records**: Anonymous, immutable value types. Prefer over single-use data classes.

```dart
// Before:
class Point { final int x, y; Point(this.x, this.y); }
Point getOrigin() => Point(0, 0);

// After:
(int, int) getOrigin() => (0, 0);
final (x, y) = getOrigin(); // Destructuring

// Named fields:
({String name, int age}) getUser() => (name: 'Alice', age: 30);
```

- **Switch expressions**: Switches as expressions with `=>` syntax.

```dart
// Before:
String describe(Shape shape) {
  switch (shape) {
    case Shape.circle: return 'Circle';
    case Shape.square: return 'Square';
    default: return 'Unknown';
  }
}

// After:
String describe(Shape shape) => switch (shape) {
  Shape.circle => 'Circle',
  Shape.square => 'Square',
  _ => 'Unknown',
};
```

- **If-case patterns**: Pattern matching in if statements for concise type checks and destructuring.

```dart
// Before:
if (json.containsKey('user') && json['user'] is List) {
  var list = json['user'] as List;
  if (list.length >= 2 && list[0] is String && list[1] is int) {
    var name = list[0] as String;
    var age = list[1] as int;
    print('$name: $age');
  }
}

// After:
if (json case {'user': [String name, int age]}) {
  print('$name: $age');
}
```

- **Object patterns**: Destructure class instances in patterns.

```dart
// Before:
if (shape is Circle) {
  final circle = shape as Circle;
  print(circle.radius);
}

// After:
if (shape case Circle(:final radius)) {
  print(radius);
}
```

- **Guard clauses (`when`)**: Add boolean conditions to pattern cases.

```dart
String classify(int n) => switch (n) {
  0 => 'zero',
  int v when v > 0 && v < 10 => 'small positive',
  int v when v < 0 => 'negative',
  _ => 'large',
};
```

- **Logical patterns**: Combine patterns with `||` (or) and `&&` (and).

```dart
bool isWeekend(String day) => switch (day) {
  'Saturday' || 'Sunday' => true,
  _ => false,
};
```

- **List/map destructuring with rest patterns**:

```dart
// Before:
var first = items.first;
var rest = items.sublist(1);

// After:
var [first, ...rest] = items;

// Map destructuring:
var {'name': name, 'age': age} = json;
```

- **For-in pattern destructuring**:

```dart
// Before:
for (var entry in map.entries) {
  print('${entry.key}: ${entry.value}');
}

// After:
for (var MapEntry(:key, :value) in map.entries) {
  print('$key: $value');
}

for (var (x, y) in points) {
  print('$x, $y');
}
```

- **`sealed` classes**: Closed type hierarchies with exhaustive pattern matching.
  ALWAYS use sealed for discriminated unions. The compiler enforces all subtypes are handled.

```dart
// Before:
abstract class Result {}
class Success extends Result { final String data; Success(this.data); }
class Error extends Result { final String message; Error(this.message); }
// No compile-time exhaustiveness check

// After:
sealed class Result {}
class Success extends Result { final String data; Success(this.data); }
class Error extends Result { final String message; Error(this.message); }

// Compiler enforces exhaustiveness -- no default needed:
String handle(Result r) => switch (r) {
  Success(:final data) => 'Got: $data',
  Error(:final message) => 'Error: $message',
};
```

- **`final` class modifier**: Cannot be extended or implemented outside its library.

```dart
final class SecurityToken {
  final String value;
  SecurityToken(this.value);
}
```

- **`base` class modifier**: Can be extended but not implemented outside its library.

```dart
base class DatabaseConnection {
  void query(String sql) { /* ... */ }
}
```

- **`interface` class modifier**: Can be implemented but not extended outside its library.

```dart
interface class Repository {
  Future<User> findById(String id) => throw UnimplementedError();
}
```

- **`mixin class`**: Explicitly declare a class usable as both a class and a mixin.

```dart
// Before (any class could be used as mixin):
class Logging { void log(String msg) => print(msg); }

// After (must be explicit):
mixin class Logging { void log(String msg) => print(msg); }
class MyService with Logging {}
```

- **`break` is now optional in switch statements**: Non-empty cases have never fallen through in Dart. In Dart 3.0+, the previously-required trailing `break` is unnecessary and flagged as redundant by the `unnecessary_breaks` lint.

```dart
// Before:
switch (command) {
  case 'start':
    handleStart();
    break; // Was required even though no fall-through
  case 'stop':
    handleStop();
    break;
}

// After:
switch (command) {
  case 'start':
    handleStart(); // Implicit break, no fall-through
  case 'stop':
    handleStop();
}
```

### Dart 3.2+

- **Private final field promotion**: Private final fields are type-promoted after null checks.

```dart
// Before:
class Foo {
  final String? _name;
  Foo(this._name);
  void greet() {
    final name = _name; // Local copy needed
    if (name != null) print(name.toUpperCase());
  }
}

// After:
class Foo {
  final String? _name;
  Foo(this._name);
  void greet() {
    if (_name != null) print(_name.toUpperCase()); // Promoted directly
  }
}
```

### Dart 3.3+

- **Extension types**: Zero-cost wrappers that add a static type interface without runtime allocation.
  Prefer over wrapper classes when you need type-safe identifiers or domain types.

```dart
// Before (wrapper class -- allocates at runtime):
class UserId {
  final int value;
  const UserId(this.value);
}

// After (zero-cost wrapper -- compiled away):
extension type const UserId(int value) {
  bool get isValid => value > 0;
}

void fetchUser(UserId id) { /* ... */ }
fetchUser(UserId(42));  // Type-safe, zero overhead at runtime
```

### Dart 3.4+

- **Improved type inference**: Better context type flow through conditional expressions (`e1 ? e2 : e3`), if-null expressions (`e1 ?? e2`), and switch expressions. Fewer explicit type annotations needed in these contexts.

### Dart 3.6+

- **Digit separators**: Use underscores in numeric literals for readability.
  ALWAYS use digit separators for numbers with five or more digits.

```dart
// Before:
const population = 8000000000;
const hexColor = 0xFF5733AB;

// After:
const population = 8_000_000_000;
const hexColor = 0xFF_57_33_AB;
```

### Dart 3.7+

- **`package:web` and `dart:js_interop` over deprecated web libraries**: Use `package:web` for DOM access and `dart:js_interop` for JS interop.
  NEVER use `dart:html`, `dart:js`, `dart:js_util`, `dart:svg`, `dart:indexed_db`, `dart:web_audio`, or `dart:web_gl`.

```dart
// Before:
import 'dart:html';
document.querySelector('#my-id')?.text = 'Hello';

import 'dart:js' as js;
js.context.callMethod('alert', ['hello']);

// After:
import 'package:web/web.dart';
document.querySelector('#my-id')?.text = 'Hello';

import 'dart:js_interop';
@JS()
external void alert(String message);
```

- **Wildcard variables**: Local variables and parameters named `_` are now non-binding — they cannot be referenced and multiple `_` declarations in the same scope don't conflict.

```dart
// Before (Dart < 3.7): _ was a regular variable
void process(List<int> items) {
  var _ = items.length;  // Bound to a variable
  var _ = items.first;   // ERROR: _ already declared
}

// After (Dart 3.7+): _ is a wildcard, non-binding
void process(List<int> items) {
  var _ = items.length;  // Not bound, value discarded
  var _ = items.first;   // Fine — no collision
}

// Useful in callbacks and patterns:
list.fold(0, (sum, _) => sum + 1);  // _ parameter discarded
switch (record) {
  case (int _, String _, bool flag):
    print('Flag: $flag');  // Only flag is bound
}
```

### Dart 3.8+

- **Null-aware elements in collections**: Use `?` prefix to conditionally include elements only when non-null.

```dart
// Before:
final items = <String>[
  'Header',
  if (title != null) title,
  if (subtitle != null) subtitle,
  'Footer',
];

// After:
final items = <String>[
  'Header',
  ?title,
  ?subtitle,
  'Footer',
];

// Maps (each ? independently guards entry inclusion):
String? maybeKey = getKey();
String? maybeValue = getValue();
final map = <String, String>{
  ?maybeKey: 'fallback',   // Entry omitted if maybeKey is null
  'fixed': ?maybeValue,    // Entry omitted if maybeValue is null
};
```

### Dart 3.10+

- **Dot shorthand syntax**: Omit the type name when Dart can infer it from context.
  Works with enum values, static members, and constructors.

```dart
// Before:
Color background = Color.blue;
TextAlign alignment = TextAlign.center;

switch (status) {
  case Status.loading: showSpinner();
  case Status.success: showContent();
}

// After:
Color background = .blue;
TextAlign alignment = .center;

switch (status) {
  case .loading: showSpinner();
  case .success: showContent();
}

// Works with constructors too:
final size = Size.square(100);  // Before
final size = .square(100);      // After (when context type is Size)

// Enum values in function arguments:
Text('Hello', textAlign: .center);
```

Note: `.shorthand` resolves against the **static interface of the context type**. If the context type is abstract, use the full concrete type name (e.g. `EdgeInsets.all(16)` not `.all(16)`).

---

## Cross-Version Best Practices

- **Prefer `final` local variables** when the value won't be reassigned.

```dart
// Old:
var name = 'Alice';
// Modern:
final name = 'Alice';
```

- **Use collection `if` and `for`** instead of imperative add/addAll.

```dart
// Old:
var items = <Widget>[];
items.add(Header());
if (showBody) items.add(Body());
for (var item in list) items.add(ItemWidget(item));

// Modern:
final items = <Widget>[
  Header(),
  if (showBody) Body(),
  for (var item in list) ItemWidget(item),
];
```

- **Use spread operator** instead of `addAll`.

```dart
// Old:
var combined = <int>[];
combined.addAll(list1);
combined.addAll(list2);

// Modern:
final combined = [...list1, ...list2];
final withNullable = [...list1, ...?nullableList];
```

- **Use trailing commas** for multi-line constructs to improve formatting.
