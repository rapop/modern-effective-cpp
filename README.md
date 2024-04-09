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

operations on std::atomic variables are often less expensive than mutex
acquisition and release

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

