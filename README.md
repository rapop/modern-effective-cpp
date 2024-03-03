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