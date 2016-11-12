# debug_assert

debug_assert is a simple, C++11, header-only library that provides a very flexible `DEBUG_ASSERT()` macro.
How many times did you write an assertion macro yourself, because `assert()` is controlled globally and cannot be enabled for certain parts of the program only?
This library solves the problem by providing a flexible, modular assertion macro.

## Features

* No dependencies. It only requires `std::abort()` and - unless `DEBUG_ASSERT_NO_STDIO` is defined - `std::fprintf()`.
* Single, small header file that just needs to be copied into your own project.
* Customizable assertion handling - assertion failure will call a user-defined function, with user-defined arguments.
* Modular - enable or disable assertions for different parts of the same program.
* Support for levels - give levels to your assertion macro and only enable certain levels of assertions.
* Little preprocessor use - just a single assertion macro which is needed to get the stringified expression and source location. Enabling/Disabling is controlled by compile time programming instead of preprocessor conditionals.
* Fast - even though a disabled assertion will still expand to something,
there is no overhead with even basic optimizations enabled and very little without optimization (just the code to read `__FILE__` and `__LINE__`). To be precise: It will only evaluate the assertion expression if the assertion is enabled!

## Overview

The basic usage of the library is like so:

```cpp
DEBUG_ASSERT(1 + 1 == 2, my_module{}); // basic
DEBUG_ASSERT(1 + 1 == 2, my_module{}, debug_assert::level<2>{}); // with level
```

Where `my_module` is a user-defined tag type that will both control the assertion level and the handler code.
It looks like this:

```cpp
struct my_module
: debug_assert::default_handler, // use the default handler
  debug_assert::set_level<-1> // level -1, i.e. all assertions, 0 would mean none, 1 would be level 1, 2 level 2 or lower,...
{};
```

A module handler must have `static` function `handle()` that takes a `debug_assert::source_location`, the stringified expression and any additional arguments you pass to `DEBUG_ASSERT()` (besides the `debug_assert::level`).

See `example.cpp` for more information and [read the blogpost](https://foonathan.github.io/blog/2016/09/16/assertions.html).

### CMake

For convenience you can also use CMake to setup the include directory, set the C++11 flag and have options that map to the customizable macros.
Simple call `add_subdirectory(path/to/debug_assert)` and then `target_link_libraries(my_target PUBLIC debug_assert)`.
It will not actually build something, only setup the flags.
The options are named like the macros.

## Documentation

> Generated by [standardese](https://github.com/foonathan/standardese).

``` cpp
#define DEBUG_ASSERT_MARK_UNREACHABLE 

#define DEBUG_ASSERT_ASSUME(Expr)

#define DEBUG_ASSERT_FORCE_INLINE inline

#define DEBUG_ASSERT_CUR_SOURCE_LOCATION debug_assert::source_location{__FILE__, __LINE__}

#define DEBUG_ASSERT(Expr, ...) 

#define DEBUG_UNREACHABLE(...) 

namespace debug_assert
{
    struct source_location;
    
    template <unsigned Level>
    struct level;
    
    template <unsigned Level>
    struct set_level;
    
    struct no_handler;
    
    struct default_handler;
}
```

## Macro `DEBUG_ASSERT_MARK_UNREACHABLE`<a id="DEBUG_ASSERT_MARK_UNREACHABLE"></a>

``` cpp
#define DEBUG_ASSERT_MARK_UNREACHABLE 
```

Hint to the compiler that a code branch is unreachable. Define it yourself prior to including the header to override it.

----

## Macro `DEBUG_ASSERT_ASSUME`<a id="DEBUG_ASSERT_ASSUME"></a>

``` cpp
#define DEBUG_ASSERT_ASSUME(Expr)
```

Hint to the compiler that a condition is `true`. Define it yourself prior to including the header to override it.

----

## Macro `DEBUG_ASSERT_FORCE_INLINE`<a id="DEBUG_ASSERT_FORCE_INLINE"></a>

``` cpp
#define DEBUG_ASSERT_FORCE_INLINE inline
```

Strong hint to the compiler to inline a function. Define it yourself prior to including the header to override it.

----

## Macro `DEBUG_ASSERT_CUR_SOURCE_LOCATION`<a id="DEBUG_ASSERT_CUR_SOURCE_LOCATION"></a>

``` cpp
#define DEBUG_ASSERT_CUR_SOURCE_LOCATION debug_assert::source_location{__FILE__, __LINE__}
```

Expands to the current [source\_location]().

-----

## Macro `DEBUG_ASSERT`<a id="DEBUG_ASSERT"></a>

``` cpp
#define DEBUG_ASSERT(Expr, ...) debug_assert::detail::do_assert([&] { return Expr; }, DEBUG_ASSERT_CUR_SOURCE_LOCATION, #Expr, __VA_ARGS__)
```

The assertion macro. Usage: \`DEBUG\_ASSERT(\<expr\>, \<handler\>, \[\<level\>\], \[\<handler-specific-args\>\]. Where:

  - `<expr>` - the expression to check for, the expression `!<expr>` must be well-formed and contextually convertible to `bool`.
  - `<handler>` - an object of the module specific handler
  - `<level>` (optional, defaults to `1`) - the level of the assertion, must be an object of type [debug\_assert::level\<Level\>](README.md#debug_assert::level).
  - `<handler-specific-args>` (optional) - any additional arguments that are just forwarded to the handler function.

It will only check the assertion if `<level>` is less than or equal to `Handler::level`. A failed assertion will call: `Handler::handle(location, expression, args)`. `location` is the [source\_location]() at the macro expansion, `expression` is the stringified expression and `args` are the `<handler-specific-args>` as-is. If the handler function returns, it will call \[std::abort()\].

*Notes*: Define `DEBUG_ASSERT_DISABLE` to completely disable this macro, it will expand to nothing. This should not be necessary, the regular version is optimized away completely.

-----

## Macro `DEBUG_UNREACHABLE`<a id="DEBUG_UNREACHABLE"></a>

``` cpp
#define DEBUG_UNREACHABLE(...) debug_assert::detail::do_assert([&] { return false; }, DEBUG_ASSERT_CUR_SOURCE_LOCATION, "", __VA_ARGS__)
```

Marks a branch as unreachable.

Usage: `DEBUG_UNREACHABLE(<handler>, [<level>], [<handler-specific-args>])` Where:

  - `<handler>` - an object of the module specific handler
  - `<level>` (optional, defaults to `1`) - the level of the assertion, must be an object of type [debug\_assert::level\<Level\>](README.md#debug_assert::level).
  - `<handler-specific-args>` (optional) - any additional arguments that are just forwarded to the handler function.

It will only check the assertion if `<level>` is less than or equal to `Handler::level`. A failed assertion will call: `Handler::handle(location, "", args)`. `location` is the [source\_location]() at the macro expansion and `args` are the `<handler-specific-args>` as-is. If the handler function returns, it will call \[std::abort()\].

*Notes*: Define `DEBUG_ASSERT_DISABLE` to completely disable this macro, it will expand to `DEBUG_ASSERT_MARK_UNREACHABLE`. This should not be necessary, the regular version is optimized away completely.

-----

## Class `debug_assert::source_location`<a id="debug_assert::source_location"></a>

``` cpp
struct source_location
{
    const char* file_name;
    
    unsigned line_number;
};
```

Defines a location in the source code.

-----

## Class template `debug_assert::level<Level>`<a id="debug_assert::level"></a>

``` cpp
template <unsigned Level>
struct level{};
```

Tag type to indicate the level of an assertion.

-----

## Class template `debug_assert::set_level<Level>`<a id="debug_assert::set_level"></a>

``` cpp
template <unsigned Level>
struct set_level
{
    static const unsigned level = Level;
};
```

Helper class that sets a certain level. Inherit from it in your module handler.

-----

## Class `debug_assert::no_handler`<a id="debug_assert::no_handler"></a>

``` cpp
struct no_handler
{
    template <typename ... Args>
    static void handle(const source_location&, const char*, Args&&...) noexcept;
};
```

Does not do anything to handle a failed assertion (except calling [std::abort()](http://en.cppreference.com/mwiki/index.php?title=Special%3ASearch&search=std%3a%3aabort%28%29)). Inherit from it in your module handler.

### Function template `debug_assert::no_handler::handle<Args...>`<a id="debug_assert::no_handler::handle(const debug_assert::source_location &,const char *,Args &&...)"></a>

``` cpp
template <typename ... Args>
static void handle(const source_location&, const char*, Args&&...) noexcept;
```

*Effects*: Does nothing.

*Notes*: Can take any additional arguments.

-----

-----

## Class `debug_assert::default_handler`<a id="debug_assert::default_handler"></a>

``` cpp
struct default_handler
{
    static void handle(const source_location& loc, const char* expression, const char* message = nullptr) noexcept;
};
```

The default handler that writes a message to `stderr`. Inherit from it in your module handler.

### Function `debug_assert::default_handler::handle`<a id="debug_assert::default_handler::handle(const debug_assert::source_location &,const char *,const char *)"></a>

``` cpp
static void handle(const source_location& loc, const char* expression, const char* message = nullptr) noexcept;
```

*Effects*: Prints a message to `stderr`.

*Notes*: It can optionally accept an additional message string.

*Notes*: If `DEBUG_ASSERT_NO_STDIO` is defined, it will do nothing.

-----

-----

