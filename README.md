# **Modern Effective C++**

# **Chapter 1:** Deducing Types

- Why type deduction is important? It makes C++
    software more adaptable, because changing a type at one point in the source code
    automatically propagates through type deduction to other locations.

## Item 1: Understand template type deduction.

```
template<typename T>
void f(ParamType param);

//A call can look like this:
f(expr);
// call f with some expression

```

During compilation, compilers use expr to deduce two types: one for T and one for
ParamType.

These types are frequently different, because ParamType often contains
adornments, e.g., const or reference qualifiers. For example, if the template is
declared like this,

```
template<typename T>
void f(const T& param);
// ParamType is const T&
```

### Case 1: ParamType is a Reference or Pointer, but not a Universal Reference

Passing a const object to a template taking a T& parameter is safe: the constness of the object becomes part of the type deduced for T.
The reference is ignored.

### Case 2: ParamType is a Universal Reference

- Universal reference parameters. Such parameters are declared like rvalue references T&&. 
- behave differently when lvalue arguments are passed in

When universal references are in use, type deduction distinguishes between lvalue arguments and rvalue arguments.

- If expr is an lvalue, both T and ParamType are deduced to be lvalue references. (ex. ParamType and T are both int&)
- If expr is an rvalue, the “normal” (i.e., Case 1) rules apply. (ex. ParamType is int&&)

### Case 3: ParamType is Neither a Pointer nor a Reference

Ignore const and & 

It’s important to recognize that const (and volatile) is ignored only for by-value parameters.

- declaring the function constexpr makes its result available during compilation.

- That makes it possible to declare, say, an array with the same
    number of elements as a second array whose size is computed from a braced initial‐
    izer

```
int keyVals[] = { 1, 3, 7, 9, 11, 22, 35 };// keyVals has
// 7 elements
int mappedVals[arraySize(keyVals)];// so does
// mappedVals

//or 

std::array<int, arraySize(keyVals)> mappedVals;
```

### Function Arguments

Function types can decay into function pointers and same rules applies like for arrays.

## Item 2: Understand auto type deduction.

`auto` plays the role of T

- When the initializer for an auto-declared variable is enclosed in braces, the deduced type is a std::initializer_list.

```
auto x3 = { 27 };// type is std::initializer_list<int>,
// value is { 27 }
auto x4{ 27 };// ditto
```

- treatment of braced initializers is the only way in which auto type deduction and template type deduction differ.

```
auto x = { 11, 23, 9 };// x's type is
// std::initializer_list<int>
template<typename T>
void f(T param);// template with parameter
// declaration equivalent to
// x's declaration
f({ 11, 23, 9 });// error! can't deduce type for T
```

fix 

```
template<typename T>
void f(std::initializer_list<T> initList);
f({ 11, 23, 9 });
// T deduced as int, and initList's
// type is std::initializer_list<int>
```

- in C++14 the use of auto in function return type and lambdas parameter declaration employ template type deduction, not auto type deduction.

## Item 3: Understand decltype.

decltype() returns the type or the name or expression you give it

problematic example:

```
template<typename Container, typename Index>
// C++14;
auto authAndAccess(Container& c, Index i)
// not quite
{
// correct
authenticateUser();
return c[i];
// return type deduced from c[i]
}
```

the ref is striped off

solution:

```
template<typename Container, typename Index>
decltype(auto)
authAndAccess(Container& c, Index i)
{
authenticateUser();
return c[i];
}

```

Now authAndAccess will truly return whatever c[i] returns. In particular, for the
common case where c[i] returns a T&, authAndAccess will also return a T&, and in
the uncommon case where c[i] returns an object, authAndAccess will return an
object, too.

It can also be used in cases like this:

```
Widget w;
const Widget& cw = w;
auto myWidget1 = cw; // auto type deduction:// myWidget1's type is Widget

decltype(auto) myWidget2 = cw;
// decltype type deduction:
// myWidget2's type is
// const Widget&
```

For lvalue expressions of type T other than names, decltype always reports a type of T&.

## Item 4: Know how to view deduced types.

Get the compiler to tell you the type:

Declare a class template that we don't define.

```
template<typename T>
class TD;
// declaration only for TD;
// TD == "Type Displayer"
```

```
TD<decltype(x)> xType;
TD<decltype(y)> yType;
// elicit errors containing
// x's and y's types
```

At runtime

```
std::cout << typeid(x).name() << '\n';
std::cout << typeid(y).name() << '\n';
// display types for
// x and y
```

But results of `std::type_info::name` are not always reliable in complex cases.

Instead use `#include <boost/type_index.hpp>`.

# **Chapter 2:** auto

## Item 5: Prefer auto to explicit type declarations.

The type for `auto` is deducted from it's initializing expression.

Some initializing expressions have types that are neither anticipated nor
desired.

## Item 6: Use the explicitly typed initializer idiom when auto deduces undesired types.

`bool highPriority = features(w)[5];`

features returns a `std::vector<bool>` object, on which operator[] is
invoked. operator[] returns a `std::vector<bool>::reference` object, which is
then implicitly converted to the bool.

`auto highPriority = features(w)[5];`

features returns a `std::vector<bool>` object, and, again, operator[] is
invoked on it. operator[] continues to return a `std::vector<bool>::reference`

highPriority doesn’t have the value of bit 5 of the std::vector<bool>
returned by features at all.

it actually ends up having a dangling pointer!

`std::vector<bool>::reference` is an example of a proxy class: a class that exists
for the purpose of emulating and augmenting the behavior of some other type.

A visible proxy example is shared_ptr and unique_ptr.

As a general rule, “invisible” proxy classes don’t play well with auto.

Objects of such classes are often not designed to live longer than a single statement.

You therefore want to avoid code of this form:
`auto someVar = expression of "invisible" proxy class type;`

solution:

explicitly typed initializer idiom

`auto highPriority = static_cast<bool>(features(w)[5]);`

can also be used to clearly express intentions for type conversions
`auto ep = static_cast<float>(calcEpsilon());`

# **Chapter 3:** Moving to Modern C++

## Item 7: Distinguish between () and {} when creating objects.

Braced initialization prohibits implicit narrowing conver‐
sions among built-in types as opposed to "()" or "=" initialization.

```
Widget w2();
// most vexing parse! declares a function
// named w2 that returns a Widget!
```

Suprising behavior can happen behavior with braced initializers because of std::initializer_lists, and constructor overload resolution.

If, one or more constructors declare a parameter of type std::initial
izer_list, calls using the braced initialization syntax strongly prefer the overloads
taking std::initializer_lists.

```
Widget(std::initializer_list<long double> il); // added
Widget w2{10, true};// uses braces, but now calls // std::initializer_list ctor
                                                // (10 and true convert to long double)
```

Even what would normally be copy and move construction can be hijacked by
std::initializer_list constructors.

```
operator float() const;
Widget w6{w4};
```


```
Widget w4({});// calls std::initializer_list ctor
// with empty list
```

## Item 8: Prefer nullptr to 0 and NULL.

nullptr’s advantage is that it doesn’t have an integral type. To be honest, it doesn’t
have a pointer type, either, but you can think of it as a pointer of all types. nullptr’s
actual type is std::nullptr_t, and, in a wonderfully circular definition,
std::nullptr_t is defined to be the type of nullptr. The type std::nullptr_t
implicitly converts to all raw pointer types,

## Item 9: Prefer alias declarations to typedefs.

alias declarations may be templatized (in which case they’re called alias templates), while typedefs cannot

```
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;
```

```
template<typename T>
struct MyAllocList {
typedef std::list<T, MyAlloc<T>> type;
};
```

type traits to make transformations
```
std::remove_const<T>::type// yields T from const T
std::remove_reference<T>::type// yields T from T& and T&&
std::add_lvalue_reference<T>::type// yields T& from T
```

you usually apply them to a type parameter inside a template

for each C++11 transformation
std::transformation<T>::type, there’s a corresponding C++14 alias template
named std::transformation_t

## Item 10: Prefer scoped enums to unscoped enums.

```
enum Color { black, white, red };// black, white, red are in same scope as Color
auto white = false;// error! white already declared in this scope
```

```
enum class Color { black, white, red };// black, white, red are scoped to Color
```

no implicit conversion with scoped enums

can be forward-declared also

forward-declaration allows to avoid recompilation

unscoped enums are useful with tuples to associate names to field numbers

```
enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;
auto val = std::get<uiEmail>(uInfo);
```

instead of

```
auto val = std::get<1>(uInfo);
```

what makes this work is the implicit conversion

with scoped enums:

```
auto val = std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>(uInfo);
```

## Item 11: Prefer deleted functions to private undefined ones.

Avoid suprises with the copy ctor and copy assignment.

In c++11

```
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
…
basic_ios(const basic_ios& ) = delete;
basic_ios& operator=(const basic_ios&) = delete;
…
};
```

In C++98 we will just put them as private.

By convention, deleted functions are declared public, not private. because compilers checks for acessibility before deleted status

result in better error messages

Any function can be deleted

```
template<>
void processPointer<void>(void*) = delete;
template<>
void processPointer<char>(char*) = delete;
```

That template specializations must be written at namespace scope, not
class scope. So you cannot make it work with private.

```
class Widget {
public:
…
template<typename T>
void processPointer(T* ptr)
{ … }
…
};
template<>
void Widget::processPointer<void>(void*) = delete;
```

## Item 12: Declare overriding functions override.

Declare overriding functions override.

Member function reference qualifiers make it possible to treat lvalue and
rvalue objects (*this) differently.

## Item 13: Prefer const_iterators to iterators.

const_iterators are the STL equivalent of pointers-to-const. They point to values
that may not be modified. The standard practice of using const whenever possible
dictates that you should use const_iterators any time you need an iterator, yet
have no need to modify what the iterator points to

## Item 14: Declare functions noexcept if they won’t emit exceptions.

- Failure to declare a function noexcept when you know that it won’t emit an
exception is simply poor interface specification.

- it permits compilers to generate better object code.

In a noexcept function, optimizers need
not keep the runtime stack in an unwindable state if an exception would propagate
out of the function, nor must they ensure that objects in a noexcept function are
destroyed in the inverse order of construction should an exception leave the function.

- safe guarantee when you need to be sure no exception will be throw midway of an operation 
ex: during std::vector::push_back or swap

The fact that swapping higher-level data structures can generally be noexcept only if
swapping their lower-level constituents is noexcept should motivate you to offer
noexcept swap functions whenever you can.

program will be terminated if an exception tries to
leave the function

The fact of the matter is that most functions are exception-neutral. Such functions
throw no exceptions themselves, but functions they call might emit one.

Some functions, however, have natural implementations that emit no exceptions, and
for a few more—notably the move operations and swap—being noexcept can have
such a significant payoff, it’s worth implementing them in a noexcept manner if at
all possible.4 When you can honestly say that a function should never emit excep‐
tions, you should definitely declare it noexcept.

don't try to twist a function to make it noexcept as it might make code at source less clear and less performant

destructors are implicitly noexcept by design because they deallocate memory 

It’s worth noting that some library interface designers distinguish functions with
wide contracts from those with narrow contracts. A function with a wide contract has
no preconditions. Such a function may be called regardless of the state of the pro‐
gram, and it imposes no constraints on the arguments that callers pass it.5 Functions
with wide contracts never exhibit undefined behavior.

## Item 15: Use constexpr whenever possible.

indicates a value is constant and known at compilation

Values known during compilation are privileged. They may be placed in read-only
memory, for example, and, especially for developers of embedded systems, this can
be a feature of considerable importance.

can be used to set size of std::array or others that need to be known at compilation
(other contexts requiring compile-time constants)

constexpr functions produce compile-time constants only if they are called with compile-time constants

constexpr in front of a function doesn’t say that it returns a const value

the function can also be called in runtime contexts

constexpr functions are limited to taking and returning literal types, which essen‐
tially means types that can have values determined during compilation. In C++11, all
built-in types except void qualify, but user-defined types may be literal, too, because
constructors and other member functions may be constexpr

This is very exciting. It means that the object mid, though its initialization involves
calls to constructors, getters, and a non-member function, can be created in read-
only memory! It means you could use an expression like mid.xValue() * 10 in an
argument to a template or in an expression specifying the value of an enumerator.

some computations traditionally done at runtime can migrate to compile time.

use constexpr whenever possible

By using constexpr whenever possible, you maximize the range of situations in which your
objects and functions may be used.

## Item 16: Make const member functions thread safe.

Operations on `std::atomic` variables are often less expensive than mutex
acquisition and release.

For a single variable or memory location requiring synchroni‐
zation, use of a std::atomic is adequate, but once you get to two or more variables
or memory locations that require manipulation as a unit, you should reach for a
mutex.

ex: caching fonctions that are const

this Item is predicated on the assumption that multiple threads may simultane‐
ously execute a const member function on an object. (they think because its const, its read only)

## Item 17: Understand special member function generation.

special member functions are the ones that C++ is willing to generate on its own.

C++98 has four such functions: the default constructor, the
destructor, the copy constructor, and the copy assignment operator.

As of C++11, the special member functions club has two more inductees: the move
constructor and the move assignment operator.

``` 
Widget(Widget&& rhs); // move constructor
Widget& operator=(Widget&& rhs); // move assignment operator
```

simply remember that a memberwise move consists of move opera‐
tions on data members and base classes that support move operations, but a copy
operation for those that don’t.

The two move operations are not independent. If you declare either, that prevents
compilers from generating the other.

Not the case for copy ctor.

Declaring a move operation (construction or
assignment) in a class causes compilers to disable the copy operations.

Rational:  if something is wrong in the move, its probably not the correct way to do the copy neither.

Thus, the Rule of Three.

if you declare any of a copy constructor, copy assignment operator, or
destructor, you should declare all three.

use = default to say its ok

Performances issues can arise, because if there is no move operator, the unittests will pass the move tests but by actually doing a copy.

Member function templates never suppress generation of special member
functions.

# **Chapter 4:** Smart Pointers

## Item 18: Use std::unique_ptr for exclusive-ownership resource management.

If a raw pointer is small enough and fast
enough for you, a std::unique_ptr almost certainly is, too.

A non-null std::unique_ptr always owns what it points to.

Copying a std::unique_ptr isn’t allowed, because if you could copy a std::unique_ptr, you’d end up with two std::unique_ptrs to the same resource, each thinking it owned (and should therefore destroy) that resource.

Stateless function objects (e.g., from lambda expressions with no captures) incur no size penalty

Factory functions are not the only common use case for std::unique_ptrs. (for polymorphisme) They’re
even more popular as a mechanism for implementing the Pimpl Idiom

## Item 19: Use std::shared_ptr for shared-ownership resource management.

When the last std::shared_ptr pointing to an object
stops pointing there (e.g., because the std::shared_ptr is destroyed or made to
point to a different object), that std::shared_ptr destroys the object it points to.

C++ way of having the best of both worlds:
As with garbage collection, clients need not concern themselves with managing the life‐
time of pointed-to objects, but as with destructors, the timing of the objects’ destruc‐
tion is deterministic.

A std::shared_ptr can tell whether it’s the last one pointing to a resource by con‐
sulting the resource’s reference count,

The existence of the reference count has performance implications:
- 2x size of raw pointers
- Memory for the reference count must be dynamically allocated.
- Increments and decrements of the reference count must be atomic

Moving std::shared_ptrs is faster than copying them: copying requires
incrementing the reference count, but moving doesn’t.

specifying a custom deleter doesn’t
change the size of a std::shared_ptr object. Regardless of deleter, a
std::shared_ptr object is two pointers in size.

the reference count is part of a
larger data structure known as the control block. There’s a control block for each
object managed by std::shared_ptrs. The control block contains, in addition to the
reference count, a copy of the custom deleter

An object’s control block is set up by the function creating the first
std::shared_ptr to the object.

At least that’s what’s supposed to happen

rules:

- std::make_shared always creates a control block.

- A control block is created when a std::shared_ptr is constructed from a
unique-ownership pointer (i.e., a std::unique_ptr or std::auto_ptr).

- When a std::shared_ptr constructor is called with a raw pointer, it creates a
control block.

constructing more than one std::shared_ptr from a single raw pointer is bad (undefined behavior)

```
auto pw = new Widget;
// pw is raw ptr
…
std::shared_ptr<Widget> spw1(pw, loggingDel);
// create control
// block for *pw
…
std::shared_ptr<Widget> spw2(pw, loggingDel);
// create 2nd
// control block
// for *pw!
```

that will ultimately lead to an attempt to destroy *pw twice.

if you must
pass a raw pointer to a std::shared_ptr constructor, pass the result of new directly
instead of going through a raw pointer variable.

often happens with the "this" pointer

```
void Widget::process()
{
…
processedWidgets.emplace_back(this);
}
```

fix: std::enable_shared_from_this

That’s a template for a base class you inherit from if
you want a class managed by std::shared_ptrs to be able to safely create a
std::shared_ptr from a this pointer.

```
class Widget: public std::enable_shared_from_this<Widget> {
public:
…
void process();
…
};
```

std::enable_shared_from_this defines a member function that creates a
std::shared_ptr to the current object, but it does it without duplicating control
blocks.

```
void Widget::process()
{
// as before, process the Widget
…
// add std::shared_ptr to current object to processedWidgets
processedWidgets.emplace_back(shared_from_this());
}
```

## Item 20: Use std::weak_ptr for std::shared_ptr-like pointers that can dangle.

`std::weak_ptr` is like a `shared_prt` that doesn't affect reference count.

It can dangle (point to something that has been destroyed)

You can check if it has expired with `expired()`

`std::weak_ptrs` lack dereferencing operations

between the call to expired and the dereferencing action,
another thread might reassign or destroy the last std::shared_ptr pointing to the
object, thus causing that object to be destroyed. In that case, your dereference would
yield undefined behavior

you need is an atomic operation that checks to see if the `std::weak_ptr` has
expired and, if not, gives you access to the object it points to.

This is done by creating a `std::shared_ptr` from the `std::weak_ptr`.
with `lock()`

or with ctor

```
std::shared_ptr<Widget> spw3(wpw);
// if wpw's expired,
// throw std::bad_weak_ptr
```

weak_ptrs can be useful for caching, Observer design pattern or circular dependencies (A points to B and B points to A).

From an efficiency perspective, the `std::weak_ptr` story is essentially the same as
that for `std::shared_ptr`

## Item 21: Prefer std::make_unique and std::make_shared to direct use of new.

```
processWidget(std::shared_ptr<Widget>(new Widget),
computePriority());
// potential
// resource
// leak!
```

Because functions arguments must be evaluated before the function runs, but the compiler choose in which order the arguments are genereate when it produces the source code.

disadvantage: none of the make functions permit the specification of custom deleters

Also, we cannot brace initialize with `make`, we need to use `new` if we want to. Or this trick.

```
// create std::initializer_list
auto initList = { 10, 20 };
// create std::vector using std::initializer_list ctor
auto spv = std::make_shared<std::vector<int>>(initList);
```

using make functions to create objects of types with class-specific versions of operator new and operator delete is typically a poor idea.

With a direct use of new, the memory for the ReallyBigType object can be released
as soon as the last std::shared_ptr to it is destroyed.

This is not the case with `make` since the count for weak_ptr is in the control block is in the same control block as the object. It will wait till the last weak_ptr is destroyed even if it points to null.

With `new`, only the memory for the control block remains allocated.

## Item 22: When using the Pimpl Idiom, define special member functions in the implementation file.

Pimpl (“pointer to implementation”) Idiom - for better build times

version with unique ptr:

```
class Widget {
public:
Widget();
…
// in "widget.h"
private:
struct Impl;
std::unique_ptr<Impl> pImpl;
};


#include "widget.h"
#include "gadget.h"
#include <string>
#include <vector>// in "widget.cpp"
struct Widget::Impl {
std::string name;
std::vector<double> data;
Gadget g1, g2, g3;
};// as before
Widget::Widget()
: pImpl(std::make_unique<Impl>())
{}

```

Because Widget no longer mentions the types std::string, std::vector, and
Gadget, Widget clients no longer need to #include the headers for these types. That
speeds compilation, and it also means that if something in these headers changes,
Widget clients are unaffected.

# **Chapter 5:** Rvalue References, Move Semantics, and Perfect Forwarding

perfect forwarding = write function templates that take arguments and forwards them to other function such that they receive th exact same arguments

a parameter is always an lvalue
`void f(Widget&& w);`

A useful heuristic to determine whether an expression is an lvalue is to ask if you can
take its address. If you can, it typically is.

## Item 23: Understand std::move and std::forward.

std::move unconditionally casts its argument to an rvalue, while
std::forward performs this cast only if a particular condition is fulfilled.

don’t declare objects const if you want to be able to move from them because Move requests on const objects are silently transformed into copy operations.

std::forward is a conditional cast: it casts to an rvalue only if its argument was initialized with an
rvalue.

param is always a l-value since its a function param.

```
void process(const Widget& lvalArg);
void process(Widget&& rvalArg);// process lvalues
template<typename T>
void logAndProcess(T&& param)
{
    auto now =
    std::chrono::system_clock::now();// template that passes
    makeLogEntry("Calling 'process'", now);
    process(std::forward<T>(param));
}
```

```
Widget w;
logAndProcess(w); // call with lvalue
logAndProcess(std::move(w)); // call with rvalue
```

Neither std::move nor std::forward do anything at runtime.

## Item 24: Distinguish universal references from rvalue references.

“T&&” has two different meanings

- r-value reference, binds to r-values, exists to identify objects that may be moved from.
- either rvalue reference or lvalue reference, look like rvalue references in the source code (i.e., “T&&”), but they can bind to anything (we call them universal references)

** universal references should almost always have std::forward applied to them, and as
this book goes to press, some members of the C++ community have started referring to universal references
as forwarding references

Universal references arise in two contexts. The most common is function template
parameters

the second is auto declaration

`auto&& var2 = var1;`

What these contexts have in common is the presence of type deduction.

**if you see “T&&” without type deduction, you’re looking at an rvalue reference**

Because universal references are references, they must be initialized. The initializer
for a universal reference determines whether it represents an rvalue reference or an
lvalue reference. If the initializer is an rvalue, the universal reference corresponds to
an rvalue reference. If the initializer is an lvalue, the universal reference corresponds
to an lvalue reference.

```
template<typename T>
void f(T&& param); // param is a universal reference
Widget w;
f(w); // lvalue passed to f; param's type is
        // Widget& (i.e., an lvalue reference)


f(std::move(w)); // rvalue passed to f; param's type is
            // Widget&& (i.e., an rvalue reference)
```

presence of a const qualifier is enough to disqualify a reference from
being universal

```
template<typename T>
void f(const T&& param);
// param is an rvalue reference
```

this entire Item—the foundation of universal references—is a lie…
er, an “abstraction.” The underlying truth is known as reference collapsing

## Item 25: Use std::move on rvalue references, std::forward on universal references.

rvalue references should be unconditionally cast to rvalues (via std::move)
when forwarding them to other functions, because they’re always bound to rvalues

universal references should be conditionally cast to rvalues (via std::forward)
when forwarding them, because they’re only sometimes bound to rvalues

when used multiple times inside a function, you’ll want to apply std::move (for rvalue
references) or std::forward (for universal references) to only the final use of the
reference.

If you’re in a function that **returns by value**, and you’re returning an object bound to
an rvalue reference or a universal reference, you’ll want to apply std::move or
std::forward when you return the reference.


```
Matrix
operator+(Matrix&& lhs, const Matrix& rhs)
{
    lhs += rhs;
    return std::move(lhs);
}
```

lhs will be moved into the function’s return value location

if not, the fact that lhs is an lvalue would force compilers to instead copy it into the return
value location.

works only if `Matrix` type supports move construction

Never apply std::move or std::forward to local objects if they would other‐
wise be eligible for the return value optimization.

## Item 26: Avoid overloading on universal references.

Avoid ineficiencies here:

```
std::multiset<std::string> names;
void logAndAdd(const std::string& name)
{
auto now = std::chrono::system_clock::now();
log(now, "logAndAdd");
names.emplace(name);
}
```

with:

```
template<typename T>
void logAndAdd(T&& name)
{
auto now = std::chrono::system_clock::now();
log(now, "logAndAdd");
names.emplace(std::forward<T>(name));
}
```

Functions taking universal references are the greediest functions in C++.

```
class Person {
public:
template<typename T>
explicit Person(T&& n)
: name(std::forward<T>(n)) {}

explicit Person(int idx)
: name(nameFromIdx(idx)) {}
…

private:
std::string name;
};
```

passing an integral type other than int (e.g.,
std::size_t, short, long, etc.) will call the universal reference constructor over‐
load instead of the int overload

Perfect-forwarding constructors are especially problematic, because they’re
typically better matches than copy constructors for non-const lvalues, and
they can hijack derived class calls to base class copy and move constructors.

## Item 27: Familiarize yourself with alternatives to overloading on universal references.

Simply use different names for the would-be overloads.

revert to C++98 and replace pass-by-universal-reference with
pass-by-lvalue-reference-to-const, (const T&) but design is less efficient

pass by value

use tag dispatch:

ex:

```
template<typename T>
void logAndAdd(T&& name)
{
    logAndAddImpl(
    std::forward<T>(name),
    std::is_integral<typename std::remove_reference<T>::type>()
    );
}

template<typename T>
void logAndAddImpl(T&& name, std::false_type)
{
auto now = std::chrono::system_clock::now();
log(now, "logAndAdd");
names.emplace(std::forward<T>(name));
}
```

`std::enable_if` gives you a way to force compilers to behave as if a particular tem‐
plate didn’t exist.

`std::decay<T>::type` is the same as T, except that references and cv-qualifiers (i.e.,
const or volatile qualifiers) are removed.

```
class Person {
public:
template<
typename T,
typename = typename std::enable_if<!std::is_same<Person, typename std::decay<T>::type>::value>::type>
explicit Person(T&& n);
…
};
```

`std::is_base_of<T1, T2>::value` is true if T2 is derived from T1

```
static_assert(
std::is_constructible<std::string, T>::value,
"Parameter n can't be used to construct a std::string"
);
```

Universal reference parameters often have efficiency advantages, but they typ‐
ically have usability disadvantages.

ex: produce long error messages.

## Item 28: Understand reference collapsing.

If a reference to a reference arises in a context where this is per‐
mitted (e.g., during template instantiation), the references collapse to a single refer‐
ence according to this rule:

If either reference is an lvalue reference, the result is an lvalue reference.
Otherwise (i.e., if both are rvalue references) the result is an rvalue refer‐
ence.

calling func which takes an universal reference will collapse from this to this

`void func(Widget& && param);`

`void func(Widget& param);`

Reference collapsing can also happen with `auto`, `typedef` and `decltype`

A universal reference isn’t a new kind of reference, it’s actually an rvalue ref‐
erence in a context where two conditions are satisfied:
    • Type deduction distinguishes lvalues from rvalues. Lvalues of type T are
    deduced to have type T&, while rvalues of type T yield T as their deduced type.
    • Reference collapsing occurs.

## Item 29: Assume that move operations are not present, not cheap, and not used.

- pointer reassignment is what makes move faster than copy.
- this is possible for types that are dynamically stored like `std::vector` which has a pointer to its data on the heap.

Often, you dont know if the move operator is really gonna be invoked and thus the copy operator might be.
You need to beconservative with your copy operations also when you don't konw, for example in templates.

## Item 30: Familiarize yourself with perfect forwarding failure cases.

Perfect forwarding means we don’t just forward objects, we also forward their salient
characteristics: their types, whether they’re lvalues or rvalues, and whether they’re
const or volatile.

only universal reference parameters encode information about the
lvalueness and rvalueness of the arguments that are passed to them

```
template<typename T>
void fwd(T&& param)
{
f(std::forward<T>(param));
}
```

perfect-forwarding happens is calling f or fwd with the same arguments does the same thing.

Several kind or arguments makes this fail.

**Braced initializers** are one example.
In a direct call to f, the type is found by comparison by the compiler. But, for fwd the compiler try to deduce it instead of comparing whats passed to fwd and f.

Perfect-forwarding can fail if compilers are unable to deduce a type or compilers deduce the “wrong” type.

In some cases compilers are forbidden from deducing a type.

**0 or NULL as null pointers** are noth well deduced, thus cannot be perfect-forwarded.

**Declaration-only integral static const data members** 

`static const std::size_t MinVals = 28;`

because complier don't create memory for them, they just replace them with their value. If someone takes their address, compilation works but not linking.

For examaple, fwd’s parameter is a
universal reference, and references, in the code generated by compilers, are usually
treated like pointers.

For some compilers, linking might work.

The fix is to provide a definitioin.

`const std::size_t Widget::MinVals; // in Widget's .cpp file`

**Overloaded function names and template names**

and **bitfields**

# **Chapter 6:** Lambda Expressions

A lambda expression is just that: an expression. It’s part of the source code.

A closure is the runtime object created by a lambda.

A closure class is a class from which a closure is instantiated. Each lambda causes
compilers to generate a unique closure class. The statements inside a lambda
become executable instructions in the member functions of its closure class.

## Item 31: Avoid default capture modes.

Using capture by reference can lead to dangling references if the lambda exists after the local scope where its created.

Make it explicit. `[&divisor]` instead of default `[&]`

Default by value is not the solution since you could copy pointers that could go dangling.

Captures apply only to non-static local variables (including parameters) visible in
the scope where the lambda is created.

Example, here is `this` that is captured:

```
void Widget::addFilter() const
{
filters.emplace_back(
[=](int value) { return value % divisor == 0; }
);
}
```

Compilers do something like this:
```
void Widget::addFilter() const
{
auto currentObjectPtr = this;
filters.emplace_back(
[currentObjectPtr](int value)
{ return value % currentObjectPtr->divisor == 0; }
);
}
```

## Item 32: Use init capture to move objects into closures.

C++14 feature.

```
auto func = [pw = std::move(pw)]
                { return pw->isValidated()
                && pw->isArchived(); };
```

// C++11 emulation of init capture
```
auto func =
std::bind(
    [](const std::vector<double>& data)
    { /* uses of data */ },
    std::move(data)
    );
```

std::bind produces function objects.

The first argument to std::bind is a
callable object. Subsequent arguments represent values to be passed to that object.

A bind object contains copies of all the arguments passed to std::bind. For each
lvalue argument, the corresponding object in the bind object is copy constructed. For
each rvalue, it’s move constructed.

## Item 33: Use decltype on auto&& parameters to std::forward them.

In our lambda, if x is bound to an lvalue, decltype(x) will yield
an lvalue reference.

if x is bound to an
rvalue, decltype(x) will yield an rvalue reference instead of the customary non-
reference.

```
auto f =
[](auto&& param)
{
return
func(normalize(std::forward<decltype(param)>(param)));
};
```

## Item 34: Prefer lambdas to std::bind.

most important reason: lambdas are more readable

using `std::bind`, the call to the function takes place through a function pointer and compilers are less likely to inline function that are called like that, thus the code might be slower

std::bind always copies its arguments, but callers can achieve the effect of having an argument stored by
reference by applying std::ref to it. The result of
auto compressRateB = std::bind(compress, std::ref(w), _1);

is that compressRateB acts as if it holds a reference to w, rather than a copy.

all arguments passed to bind objects are passed by reference, because the func‐
tion call operator for such objects uses perfect forwarding.

In C++14, there are no reasonable use cases for
std::bind.

In C++11, however, std::bind can be justified in two constrained situa‐
tions:

- Move capture.

- Polymorphic function objects.


# **Chapter 7:** The Concurrency API

# Item 35: Prefer task-based programming to thread-based.

thread-based approach:
`std::thread t(doAsyncWork);`

task-based approach:
`auto fut = std::async(doAsyncWork);`

Why its superior:
- produces a return value because it provides a `get` function
- `get` provides access to the exception if it throws
- frees you from the details of thread management

Software threads are limited. This can throw if no more threads are available.
`std::thread t(doAsyncWork);`

*Oversubscription* can also happen. That's when you have more software threads ready to run than hardware threads available.

You will avoid this with tasks.

If using `std::async` the scheduler won't create a thread right away.

Situations when using `threads` is appriopriate:
- Access to the API of the underlying threading implementation (pthreads or Windows’ Threads)
- Optimize thread usage.
- Implement advance threading technologies.

`threads` allow us to set its scheduling priority via its API, `tasks` don't.

`std::thread` objects aren't copyable.

Invoking `join` or `detach` on an unjoinable thread yields undefined
behavior.

# Item 36: Specify std::launch::async if asynchronicity is essential.

- `std::launch::async` launch policy means that f must be run asynchro‐
nously, i.e., on a different thread

- `std::launch::deferred` launch policy means that f may run only when
get or wait is called on the future returned by std::async.

    When get or wait is invoked, f will
    execute synchronously, i.e., the caller will block until f finishes running

The default is `std::launch::async | std::launch::deferred`

This avoids oversubscription for example.

Cautious, if f is defered, it will always return `std::future_status::deferred`.

Example,
```
while (fut.wait_for(100ms) !=
std::future_status::ready)
{
…
}
```

might never finish, and the bug will be apparent only under heavy loads.

Other issues may arise.

# Item 37: Make std::threads unjoinable on all paths.

`.join()` will wait for the content of the `thread` to finish executing.

`detach()` = the connection between the thread and their underlying software thread has been
severed. The thread will still continue tho.

Destruction of a joinable thread is dangerous, thus it was banned. The standard now says that **destruction of a joinable thread causes program termination**.

Thus, we need to make unjoinable on every path out of the scope in which it’s defined.

Any time you want to perform some action along every path out of a block, the nor‐
mal approach is to put that action in the destructor of a local object. Such objects are
known as RAII objects, and the classes they come from are known as **RAII** classes.
(RAII itself stands for “Resource Acquisition Is Initialization).

RAII is implemented on most STL containers, but not for thread. But, we can implement it ourselves. 

We also saw earlier that doing a join could lead
to performance anomalies (that, to be frank, could also be unpleasant to debug), but
given a choice between undefined behavior (which detach would get us), program
termination (which use of a raw std::thread would yield), or performance anoma‐
lies, performance anomalies seems like the best of a bad lot.

Using ThreadRAII to perform a join on
std::thread destruction can sometimes lead not just to a performance anomaly, but
to a hung program. The “proper” solution to these kinds of problems would be to
communicate to the asynchronously running lambda that we no longer need its work
and that it should return early.

# Item 38: Be aware of varying thread handle destructor behavior.

Futures from `std::async` block in their destructors.

Future destructors normally just destroy the future’s data members.

The final future referring to a shared state for a non-deferred task launched
via std::async blocks until the task completes.

# Item 39: Consider void futures for one-shot event communication.

Case: Sometimes it’s useful for a task to tell a second, asynchronously running task that a
particular event has occurred.

One possibility is to use a condition variable.


```
void detect 
{
    ...
    cv.notify_one();
    ...
}

void react 
{
    std::unique_lock<std::mutex> lk(m);
    cv.wait(lk);

    ...
}
```

Note: Locking a mutex before waiting on a condition variable is typical for threading libraries. The
need to lock the mutex through a std::unique_lock object is simply part of the
C++11 API.

Code smell here: There is a need for a mute even though, there is not necessarilly a need for shared data.

Even without this, there are still 2 problems:
- If the detecting task happens to execute the notification before the reacting task executes the wait, the reacting task will miss the notification, and it will wait forever.
- Spurious wakeups: A fact of life in threading APIs (in many languages—not just C++) is that code waiting on a condition variable may be awakened even if the condvar wasn’t notified. Such awakenings are known as spurious wakeups. Proper code deals with them by confirming that the condition being waited for has truly occurred, and it does
this as its first action after waking.

Fix for spurious : 
```
cv.wait(lk,
[]{ return whether the event has occurred; });
```

But it doesn't really work since it condv doesn't know if the event it's waiting for has occurred in the first place.

Next trick: shared flag.

`std::atomic<bool> flag(false);`

The problem here is that the task is blocked but still running in a background thread and thus incurs costs.

That’s an advantage of the condvar-based approach, because a task in a wait call is truly blocked.
It’s common to combine the condvar and flag-based designs.

This approach works but isn't clean.

```
std::condition_variable cv;
std::mutex m;// as before
bool flag(false);
{
    std::lock_guard<std::mutex> g(m);
    flag = true;

    cv.notify_one();
}

And here’s the reacting task:
{
    …
    // prepare to react
    {
    std::unique_lock<std::mutex> lk(m);
    cv.wait(lk, [] { return flag; });// use lambda to avoid
    // spurious wakeups
    …// react to event
    // (m is locked)
    }
    …
    // continue reacting
    // (m now unlocked)
}
```

An alternative is to avoid condition variables, mutexes, and flags by having the reacting task wait on a future that’s set by the detecting task.

```
std::promise<void> p;

...
detecting task
p.set_value();

...
reacting task
p.get_future().wait();
```

Like the approach using a flag, this design requires no mutex, works regardless of
whether the detecting task sets its std::promise before the reacting task waits, and
is immune to spurious wakeups. (Only condition variables are susceptible to that
problem.) Like the condvar-based approach, the reacting task is truly blocked after
making the wait call, so it consumes no system resources while waiting.

Problems:
- Between a std::promise and a future is a shared state, and shared states are typically dynamically allocated. You should therefore assume that this design incurs the cost of heap-based allocation
and deallocation.
- std::promise may be set only once. The communications channel between a std::promise and a future is a one-shot mechanism: it can’t be used repeatedly.

Assuming you want to suspend a thread only once (after creation, but before it’s running its thread function), a design using a void future is a reasonable choice.

```
std::promise<void> p;
void react();

void detect()
{
    std::thread t([]
    {
    p.get_future().wait();
    react();
    });
    // here, t is suspended
    // prior to call to react
    … // here function can hang.
    p.set_value();
    ...

    t.join();
}
```

We could make use of RAII here.

There is still a problem if an exception is thrown at the `...`, the function can hang.
The thread running the lambda will never finish.

Key: Use std::shared_futures instead of a std::future in the react code. Because std::future’s share member function transfers ownership of its shared state to the std::shared_future object produced by share.

```
std::promise<void> p;// as before
void detect()
{
    auto sf = p.get_future().share();
    std::vector<std::thread> vt;

    for (int i = 0; i < threadsToRun; ++i) {
    vt.emplace_back([sf]{ sf.wait();
    // wait on local
    react(); });
    // copy of sf; see
    }

    …// detect hangs if
    // this "…" code throws!
    p.set_value();// unsuspend all threads
    …
    for (auto& t : vt) {
    t.join();
    }
}
```

# Item 40: Use std::atomic for concurrency, volatile for special memory.

Once a std::atomic object has been constructed, operations on it behave as if they were inside a mutex-protected critical section, but the operations are generally implemented using special machine instructions that are more efficient than would be the case if a mutex were employed.

Don't use `volatile` for concurrent programming. There is no guarantee that there will not be data races.

The Standard’s decree that data races cause undefined behavior means that compilers may generate code to do literally anything.

As a general rule, **compilers are permitted to reorder such unrelated assignments**. That is, given this sequence of assignments (where a, b, x, and y correspond to independent variables),
a = b;
x = y;
compilers may generally reorder them as follows:
x = y;
a = b;

Even if compilers don’t reorder them, the underlying hardware might do it.

Solution: Using `std:atomic` imposes a restriction that the code running after will not run before the atomic operation.

Compilers also simplify code:

```
auto y = x;
y = x;
// read x
// read x again
x = 10;
x = 20;
// write x
// write x again
```

Might end like this:

```
auto y = x;// read x
x = 20;// write x
```

Even if we don't write code like this directly,  compilers take reasonable-looking source code and perform template instantiation, inlining, and various common kinds of reordering optimizations, it’s not
uncommon for the result to have redundant loads and dead stores that compilers can
get rid of.

Such optimizations are valid only if memory behaves normally. “Special” memory
doesn’t. Probably the most common kind of special memory is memory used for
**memory-mapped I/O**.

`volatile` is the way we tell compilers that we’re dealing with special memory. Its
meaning to compilers is “Don’t perform any optimizations on operations on this
memory.”

Copy operations for `std::atomic` are deleted. 

```
auto y = x; // error!
y = x; // error!
```

In order for the copy construction of y from x
to be atomic, compilers would have to generate code to read x and write y in a single
atomic operation. Hardware generally can’t do that, so copy construction isn’t sup‐
ported for std::atomic types.

Solution:

```
std::atomic<int> y(x.load());// read x
y.store(x.load());// read x again
```

Given that code, compilers could “optimize” it by storing x’s value in a register
instead of reading it twice:

```
register = x.load();// read x into register
std::atomic<int> y(register);// init y with register value
y.store(register);// store register value into y
```

The result, as you can see, reads from x only once, and that’s the kind of optimization
that must be avoided when dealing with special memory. (The optimization isn’t per‐
mitted for volatile variables.)

Thus,
- `std::atomic` is useful for concurrent programming, but not when dealing with special memory.
- `volatile` is useful when working with special memory (when reads and writes should not be optimized), but not for concurrent programming.


# **Chapter 8:** Tweaks

## Item 41: Consider pass by value for copyable parameters that are cheap to move and always copied.

So, as I said, when parameters are copied via assignment, analyzing the cost of pass
by value is complicated. Usually, the most practical approach is to adopt a “guilty
until proven innocent” policy, whereby you use overloading or universal references
instead of pass by value unless it’s been demonstrated that pass by value yields
acceptably efficient code for the parameter type you need.

Pass by value can also lead to the slicing problem.

## Item 42: Consider emplacement instead of insertion.

Writting `vs.push_back("xyzzy");` is like writting `vs.push_back(std::string("xyzzy"));`.

Insertion functions take objects to be inserted, while
emplacement functions take constructor arguments for objects to be inserted. This dif‐
ference permits emplacement functions to avoid the creation and destruction of tem‐
porary objects that insertion functions can necessitate.

With current implementations of the Standard Library, there are situations where, as expected, emplacement outperforms insertion, but, sadly, there are also situations where the insertion functions run faster.

Depends on a lot of things, thus it needs to be benchmarked.

When you use an emplacement function, be especially careful to make sure you’re passing the correct arguments, because even explicit constructors will be considered by compilers as they try to find a way to interpret your code as valid.