* [C++]
    * [Indentation]
    * [Namespace]
    * [Includes]
    * [Header and implementation]
    * [Declaring variables]
    * [Whitespace]
    * [Braces]
    * [Parentheses]
    * [Line breaks]
    * [Casts]
        * [`static_cast`]
            * [Fundamental types]
            * [Composite types]
        * [`dynamic_cast`]
        * [`reinterpret_cast`]
    * [`auto`]
    * [General exceptions]
* [Qml]
* [CMake]
* [Bash]
* [Python]

To keep the source consistent, readable, modifiable, and easy to merge, we use a fairly rigid coding style. All patches are expected to conform to this style, with the exception of third-party modules, or patches to existing files that are already using a different style.

Our C++ style and conventions are mostly based on the [Qt code style](https://wiki.qt.io/Qt_Coding_Style) and [conventions](https://wiki.qt.io/Coding_Conventions), but there are some differences. The guidelines for other languages are based on common practice in those languages, defaulting to a similar style to our C++ style when no consensus is available.

Guidelines for the contribution process itself can be found [here](https://github.com/audacity/audacity/blob/master/CONTRIBUTING.md) for Audacity and [here](https://github.com/musescore/MuseScore/blob/main/CONTRIBUTING.md) for MuseScore Studio. Generally, the same advice applies when submitting changes to the Muse Framework.

## C++
[C++]: #c

The style described here is valid for the `master` branch (and all branches derived from it) since MuseScore 4.0. The `3.x` branch (and its derivatives) use the [old coding style](https://musescore.org/en/handbook/developers-handbook/finding-your-way-around/musescore-coding-rules).

On the master branch, this style is enforced in pull requests by the [codestyle CI check](https://github.com/musescore/MuseScore/wiki/Fix-the-CI-checks#Codestyle), which uses [Uncrustify](https://github.com/uncrustify/uncrustify) to format the code according to this [config file](https://github.com/musescore/MuseScore/blob/master/tools/codestyle/uncrustify_musescore.cfg), and gives an error if any style violations were detected.

You can [run Uncrustify on your own machine](https://github.com/musescore/MuseScore/wiki/Formatting-the-source-code) to have your code formatted to this style automatically.

### Indentation
[Indentation]: #indentation

* 4 spaces are used for indentation
* Spaces, not tabs!

### Namespace
[Namespace]: #namespace

* All namespaces must be child to the global namespace `mu`
* Namespaces names must be lowercase  so they can be easily distinguished from class names

```C++
 // Wrong
 namespace Cool {
     ...
 }

 // Correct
 namespace mu::cool {
     ...
 }
```

### Includes
[Includes]: #includes

To keep compile times to a minimum, you should only `#include` what you actually use.

> [!TIP]
> If you delete code, check whether the remaining includes are still needed. Qt Creator highlights includes that are not used.

> [!IMPORTANT]
> Wherever practicable, avoid using `#include` in header files. Instead, use a forward declaration in the header and put the `#include` in the implementation file. This helps because headers may themselves be included in other files, causing compile times to grow exponentially.

Includes should be specified in this order:

1. Corresponding header file (only relevant in implementation files)
2. System
3. Qt
4. Framework
5. IOC
    1. Framework
    2. Other modules
    3. Current module
6. Current module _(Note: files from other modules should only be included via framework or IOC)_
7. Current class

Leave a blank line between each category of include. Includes in the same category should be specified in **alphabetical order** to reduce the likelihood of merge conflicts.

```C++
// System
#include <string>

// Qt
#include <QObject>
#include <QRect>

// Framework
#include "actions/actionable.h"
#include "async/asyncable.h"

// IOC
#include "ioc.h"
#include "global/isomeclass.h"      // Framework
#include "context/iglobalcontext.h" // Other module
#include "ianotherclass.h"          // Current module

// Current module
#include "types.h"
```

### Header and implementation
[Header and implementation]: #header-and-implementation

* Header files use the `.h` extension. Source/implementation files use `.cpp`.
* Each header and implementation file must have license text at the top.
* The header file must have a `#pragma once` guard immediately below the license.
* Header files should only contain _declarations_, not _definitions/implementations_, except when necessary (e.g. function-template definitions).

> [!IMPORTANT]
> Don't put a [singleton](https://en.wikipedia.org/wiki/Singleton_pattern)'s implementation in the header file.

The class declaration should be in this order:  

1. Licence
2. Guard (we use `#pragma once`)
3. [Includes]
4. Forward declarations
5. Namespace
6. Class declaration
7. Qt macro (Q_OBJECT, Q_PROPERTY, Q_ENUMS...)
8. Public methods (first constructor and destructor if they public)
9. Public slots
10. Signals
11. Protected
12. Private slots
13. Private methods
14. Private members

```C++
 // Correct
{LICENCE}

#pragma once

{INCLUDES}

namespace mu::cool {
class Rect : public QObject
{
    Q_OBJECT
public:
    explicit Rect(QObject* parent = nullptr);
    ~Rect();

    int width() const;
    int height() const;

    int square() const;

public slots:
    void setWidth(int w);
    void setHeight(int h);

signals:
    void squareChanged(int sq);

private:
    void updateSquare();

    QRect m_rect;
    int m_square = 0;
};
}

```

```C++
// Correct

{LICENCE}

#include "rect.h"

Rect::Rect(QObject* parent)
    : QObject(parent)
{
}

Rect::~Rect()
{
}

void Rect::updateSquare()
{
    int s = width() * height();
    if (s != m_square) {
        m_square = s;
        emit squareChanged(s);
    }
}

void Rect::setWidth(int w)
{
    m_rect.setWidth(w);
    updateSquare();
}

int Rect::width() const
{
    return m_rect.width();
}

void Rect::setHeight(int h)
{
    m_rect.setHeight(h);
    updateSquare();
}

int Rect::height() const
{
    return m_rect.height();
}

int Rect::square() const
{
    return m_square;
}
```



* If there is initialization in constructors, always put a colon on a new line

```C++
// Wrong
ClassName::ClassName(int arg) : Base(), m_param(arg) {}

ClassName::ClassName(int arg) : Base(), m_param(arg)
{}

ClassName::ClassName(int arg) :
    Base(), m_param(arg)
{
}

// Correct
ClassName::ClassName(int arg)
    : Base(), m_param(arg)
{  
}
```

### Declaring variables
[Declaring variables]: #declaring-variables

* Declare each variable on a separate line
* Avoid short or meaningless names (e.g. "a", "rbarr", "nughdeget")
* Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious
* Wait when declaring a variable until it is needed
* For pointers or references, no space between the '*' or '&' and type (__different from Qt__)
* Always initialize the variable with the initial value.
* Use `nullptr`, rather than `0`, to initialize pointers.

```C++
 // Wrong
 int a, b;
 char *c, *d;
 const Type &val = ...
 void func(const Type &val);

 // Correct
 int height = 0;
 int width = 0;
 char* nameOfThis = nullptr;
 char* nameOfThat = nullptr;
 const Type& val = ...
 void foo(const Type& val);
```

* Variables and functions start with a lower-case letter. Each consecutive word in a variable's name starts with an upper-case letter (camelCase)
* Constants all uppercase letters  and underscore to separate words
* Avoid abbreviations

```C++
 // Wrong
 short Cntr;
 int panel_height = 0;
 char ITEM_DELIM = ' ';

 // Correct
 short counter = 0;
 int panelHeight = 0;
 const char ITEM_DELIMITER = ' ';
```

* Classes and structures always start with an upper-case letter.
* Acronyms are camel-cased (e.g. XmlReader, not XMLReader).
* Data members of classes are named like ordinary nonmember variables, but with a prefix `m_`.
* Public members of struct (POD) are named like ordinary nonmember variables, with out any prefix.

```C++
 // Wrong
 class counter
 {
 ...
 private:
     int count;
 }

 class HTMLParser
 {
 ...
 private:
     char* _data;
 }

 // Correct
 class Counter
 {
 ...
 private:
     int m_count = 0;
 }

 class HtmlParser
 {
 ...
 private:
     char* m_data = nullptr;
 }
```

### Whitespace
[Whitespace]: #whitespace

* Use blank lines to group statements together where suited
* Always use only one blank line
* Always use a single space after a keyword and before a curly brace:

```C++
 // Wrong
 if(foo){
     ...
 }

 // Correct
 if (foo) {
     ...
 }
```

* Surround binary operators with spaces
* Leave a space after each comma
* No space after a cast, avoid C-style casts when possible

```C++
 // Wrong
 char *blockOfMemory = (char* ) malloc(data.size());

 // Correct
 char* blockOfMemory = reinterpret_cast<char*>(malloc(data.size()));
```

Use safer type conversion when possible:

```C++
// Wrong
Chord* chord = static_cast<Chord*>(e);

// Correct
Chord* chord = toChord(e);
```

### Braces
[Braces]: #braces

* Use attached braces: The opening brace goes on the same line as the start of the statement. If the closing brace is followed by another keyword, it goes into the same line as well:

```C++
 // Wrong
 if (codec)
 {
     ...
 }
 else
 {
     ...
 }

 if (codec) {
     ...
     }

 // Correct
 if (codec) {
     ...
 } else {
     ...
 }
```

* Exception: Function implementations (but not lambdas) and class declarations always have the left brace on the start of a line:

```C++
 static void foo(int g)
 {
     ...
 }

 class Moo
 {
     ...
 };
```

* Do not put multiple statements on one line
* If the body of the expression consists of one line, add braces anyway (__different from Qt__)

```C++
 // Wrong
 if (foo) bar();

 if (foo)
     bar();

 // Correct
 if (foo) {
     bar();
 }

```

* Do not put 'else' after jump statements:

```C++
 // Wrong
 if (thisOrThat)
     return;
 else
     somethingElse();

 // Correct
 if (thisOrThat) {
     return;
 }

 somethingElse();
```

### Parentheses
[Parentheses]: #parentheses

* Use parentheses to group expressions:

```C++
 // Wrong
 if (a && b || c)

 // Correct
 if ((a && b) || c)

 // Wrong
 a + b & c

 // Correct
 (a + b) & c
```

### Line breaks
[Line breaks]: #line-breaks

* Keep lines shorter than 120 characters; wrap if necessary
* Commas go at the end of wrapped lines; operators start at the beginning of the new lines. An operator at the end of the line is easy to miss if the editor is too narrow.

```C++
 // Wrong
 if (longExpression +
     otherLongExpression +
     otherOtherLongExpression) {
 }

 // Correct
 if (longExpression
     + otherLongExpression
     + otherOtherLongExpression) {
 }
```

### Casts
[Casts]: #casts

> [!IMPORTANT]
> Never use C-style casts in C++ code.
> ```C++
> int foo = (int)5.3;      // Wrong
> Dog* dog = (Dog*)animal; // Wrong
> ```

#### `static_cast`
[`static_cast`]: #static_cast

##### Fundamental types
[Fundamental types]: #fundamental-types

Always use `static_cast` when converting between fundamental data types, such as:

* Integral types (`bool`, `char`, `int`, `long`, `size_t`, etc.)
* Floating-point types (`float`, `double`, etc.)

Also use `static_cast` with:

* Aliases of fundamental types (e.g. [`qreal`](https://doc.qt.io/qt-6/qttypes.html#qreal-typedef) and [`staff_idx_t`](https://github.com/musescore/MuseScore/blob/aa5d5c59ee72b136118ca75b99bb27576ecd23f9/src/engraving/types/types.h#L47))
* Enumerated types (`enum` and `enum class`)

```C++
auto foo = static_cast<int>(5.3); // Valid: foo is 5
auto bar = static_cast<float>(2); // Valid: bar is 2.0
```

> [!IMPORTANT]
> Be careful when casting to a smaller or lower-precision type, such as from `int` to `char`, or from `double` to `float`. The conversion will succeed, but some information is lost in the process.
>
> Similarly, be careful when converting from `signed` to `unsigned`, or from `unsigned` to `signed`. These cover different ranges of possible values, so conversion can give unexpected results in either direction, although technically no information is lost (assuming both types are otherwise the same).
>
> Converting from `float` to `int` discards the fractional portion like in the example above. Also, the integer portion may overflow or underflow since floating-point types can represent a greater range of values than integral types with the same bit width. Doing the reverse, i.e. converting from `int` to `float`, may also lose information if the integer in question is of sufficient magnitude. A 32-bit floating point number can only represent integers continuously in the range ±2^24.

##### Composite types
[Composite types]: #composite-types

`static_cast` is also the preferred option for converting pointers of composite types (i.e. `class` and `struct`). Use it whenever you can be certain that it will succeed.

At compile time, `static_cast` gives an error if neither type is an ancestor (via inheritance) of the other type.

```C++
auto dog = new Dog();
auto cat = static_cast<Cat*>(dog); // Compiler error!
// Error because Cat class is neither an ancestor nor a descendent of Dog class.
```

The error is useful, but its absence doesn't guarantee success. The program can give very strange behavior at runtime if you down-cast to a derived type when the object in question isn't actually an instance of the derived type.

```C++
// Dog class inherits from Animal class (directly or indirectly).
// This means every Dog is an Animal, but not every Animal is a Dog.

auto dog1 = getRandomDog();
auto animal1 = static_cast<Animal*>(dog1); // Up-cast, always safe.
animal1->move(); // Calls Animal::move() instead of Dog::move().

auto animal2 = getRandomAnimal();
auto dog2 = static_cast<Dog*>(animal2); // Down-cast, potentially unsafe.
dog2->fetch(); // Dog::fetch(), gives strange result if animal2 isn't a Dog.
```

The strange behavior can be predicted (see [`reinterpret_cast`]) but it's probably not what you want. Instead, you can avoid it by manually checking one of the object's properties prior to conversion.

```C++
if (animal2->sound() == "Woof!") {          // Definitely a Dog, so now
    auto dog2 = static_cast<Dog*>(animal2); // it's safe to down-cast and
    dog2->fetch();                          // access Dog-specific members.
}
```

`static_cast` performs no checks at runtime, so it has little to no impact on performance. However, manual checks may impact performance.

> [!NOTE]
> If the manual check involves calling a virtual method, or anything that's slow to compute, you might as well use `dynamic_cast` instead.

#### `dynamic_cast`
[`dynamic_cast`]: #dynamic_cast

If the type conversion isn't guaranteed to succeed, you could use `static_cast` with a manual check as mentioned above, or one of the following options.

When converting to a more specific subclass of MuseScore's `EngravingObject`, use the dedicated `isType()` and `toType()` functions:

```C++
if (chordrest->isChord()) {
    Chord* ch = toChord(chordrest);
    // Do something with `ch`
}
```

When converting to a more specific subclass of `QObject`, use Qt's dedicated [`qobject_cast`](https://doc.qt.io/qt-6/qobject.html#qobject_cast), which returns `nullptr` at runtime if the cast is not possible:

```C++
if (auto *dialog = qobject_cast<QDialog*>(widget)) {
    // Do something with `dialog`
}
```

Otherwise, there is `dynamic_cast`, which performs a runtime check via [RTTI] and returns `nullptr` if the cast is not possible.

[RTTI]: https://www.geeksforgeeks.org/cpp/rtti-run-time-type-information-in-cpp/

```C++
if (auto dog = dynamic_cast<Dog*>(animal)) {
    // Do something with `dog`
}
```

However, RTTI is slow, so the other options are preferred where possible. Also, RTTI relies on the types in question having at least one virtual method. Fortunately, the compiler gives an error if this condition is not satisfied. It also gives errors in all situations that would give errors with `static_cast`.

#### `reinterpret_cast`
[`reinterpret_cast`]: #reinterpret_cast

Use `reinterpret_cast` when neither type is an ancestor of the other type. It performs no checks and has no performance overhead.

```C++
auto dog = new Dog();
auto cat = reinterpret_cast<Cat*>(dog); // No error.
std::cout << cat->sound(); // Could print "Woof!", "Meow!", junk data, or crash.
// Technically, the actual outcome could be predicted if we
// had complete knowledge of how both types are implemented.
```

`reinterpret_cast` is equivalent to a C-style cast, but it sends a different message to other developers:

* Using a C-style cast means *"I'm a C programmer and I don't know how to write C++ code."*

* Whereas using `reinterpret_cast` means *"Trust me, I know what I'm doing. I tried all other options and none of them work here."*

Most programmers will never have to use `reinterpret_cast`, and nor should they! It's really best avoided.

### `auto`
[`auto`]: #auto

When the type is repeated in the line (e.g. after a cast), use the `auto` keyword to avoid repetition.

```C++
// Wrong (too verbose)
AudioPluginsScannerMock* mock = dynamic_cast<AudioPluginsScannerMock*>(scanner.get());

// Correct (more readable)
auto mock = dynamic_cast<AudioPluginsScannerMock*>(scanner.get());
```

When the type is long (30+ chars?), e.g. due to long nesting and templateisation: 
If it's recurrent, create a descriptive alias with "using" keyword. 

```C++
using ClipConstHolders = std::vector<std::shared_ptr<const ClipInterface> >;
```
If it only happens once, or e.g. if it is a verbose iterator: 

```C++
IteratorRange<IntervalIterator<WaveClipChannel> >
```
auto will keep the code more readable:

```C++
for (const auto& clip : Intervals()) {
(...)
}
```


### General exceptions
[General exceptions]: #general-exceptions

* When strictly following a rule makes your code look bad, feel free to break it.
* If there is a dispute in any given code, create a pull request with the proposed changes to the style code.

> [!TIP]
> **Don't fight the [automatic code formatter](https://github.com/musescore/MuseScore/wiki/Formatting-the-source-code)** (i.e. Uncrustify for C++ code). Either reconfigure it in a PR to give the desired output, or learn to deal with its current output. It's rarely worth sacrificing the overall consistency and readibility these tools provide, particularly for viewing diffs and resolving merge conflicts, in order to micromanage the few places where they fall short.

## Qml
[Qml]: #qml

In general, the same style as for C++

* Property names as well as C ++ variable names
* Private property must be a separate object

```javascript
 // Wrong
 Item {
     property var cntr
     property var internal_data

     ...
 }

 // Correct
 Item {

     property var counter: 0

     QObject {
         id: prv
         property var data: null
     }

     ...
 }
```

* The opening brace goes on the same line as the start of the statement (with no exceptions)

```javascript
 // Wrong
 Rectangle
 {
     width: 40
     height: 20
     ...
 }

 function do_something()
 {
     ...
 }

 // Correct
 Rectangle {

     width: 40
     height: 20
     ...
 }

 function doSomeThing() {
     ...
 }
```

* This is the order of properties in a QML item
```javascript
Object {
    id: someID

    // newline here
    
    anchors.someProperty: ...
    anchors.another: ...
    // Or:
    Layout.someProperty: ...

    // newline here

    someRegularProperty: ...
    anotherOne: ...

    // newline here

    navigation.someProperty: ...

    // newline here

    onSomeSignal: {
        doSomething()
    }

    onSomeSignalWithParameter: function(param) {
        soSomethingWith(param)
    }
}
```

## CMake
[CMake]: #cmake

//! TODO need to write like [this](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)

## Bash
[Bash]: #bash

In general, a mix of our C++ style and [Google's shell style](https://google.github.io/styleguide/shellguide.html), with some modifications.

```Bash
#!/usr/bin/env bash

((${BASH_VERSION%%.*} >= 4)) || { echo >&2 "$0: Error: Please upgrade Bash."; exit 1; }

set -euxo pipefail          # omit 'x' in most scripts (it's mainly for debugging)

readonly FOO_BAR="Foo Bar"  # global shell constant
export BAR_BAZ="Bar Baz"    # environment variable

my_num=3                    # global shell variable

if (($# >= 1)); then
    # Remember to print messages on STDERR.
    echo >&2 "Do something"
    my_num=7                # modify the variable
fi

function print_something()
{
    local arg1="$1"     # local shell variable
    local -r arg2="$2"  # local shell constant
    shift 2
    # Prefer 'printf' to 'echo' when arguments are unknown.
    printf >&2 '%s\n' "${arg1}" "${arg2}" "$@"
}

print_something "one" "two" "three"
```

## Python
[Python]: #python

Use `PascalCase` for class names and `snake_case` for everything else, including methods and variables within classes, i.e. `FooBar.baz_qux()`.