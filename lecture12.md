# Lecture 12

_June 12, 2025_

Last time: Polymorphic big 5

This time: Polymorphic big 5 cont., Exceptions

## From Last Lecture: Does this work?

1. **Does this solution address mixed assignment?**

   Compiler provided `operator=` for `Text` takes in a `const Text&`, so not assignment with siblings.

2. **What about partial assignment?**

```cpp
Text t1{...};
Text t2{...};
AbstractBook& br1 = t1;
AbstractBook& br2 = t2;
br1 = br2; // This will not compile because the operator= in AbstractBook is a protected method
```

An Exercise: Why doesn't the protected solution work with the _old class hiearchy?_

### One minor caveat about having a pure virtual destructor:

Pure virtual destructors still require implementations.

```C++
AbstractBook::~AbstractBook(){} // even as simple as this.
```

Why? Recall the steps of object destruction sequence:
Step 3 was to call the superclass destructor. Any method you'd call requires an implementation.

Note: Giving a pure virtual method an implmentation doesn't make your class concrete - only subclasses with pure virtual methods can be concrete.

## Exceptions

Example: `stl::vector`. As we know, these are dynamically reallocated arrays, which are generally safer to use. But what about the following?

```cpp
vector<int> v;
v.push_back(247);
cout << v[1324234] << endl; // out of bounds
cout << v.at(100000) << endl; // "slower but safer", performs bounds checking and throws an instance of std::out_of_range
```

What can vector do to signal to us that an out-of-bounds access has occurred?

**Option 1**: Sentinel values - reserve return values to signal errors.
E.g. for functions returning positive ints, you can use values like -1. `INT_MIN`, `INT_MAX`.

- But this restricts return values (Is -1 a true -1 or an error?). For other generic types `T` in vector, what values should be reserved? Not always clear.

**Option 2**: Use a global variable. E.g. C has `errno`. Also not ideal-limited types of errors are supposed. If you have multiple errors, we can only get the most recent one. Easy to ignore too.

**Option 3**: Bundle in a struct:

```C++
template<typename ET, typename DT> struct ReturnType{
    ET error;
    DT data;
};
```

Best we've seen - but this is so damn tedious. What if we never use the error variable? We'll inflate our space. _And_ it's easy to ignore.

These approaches are useful for langauges without built-in error handling support, but C++ has **exceptions.**

`vector::at`, if given an out-of-bounds index, will throw an **exception.** If unhandled, the program crashes. But we can address it via `try-catch`.

```C++
try{
    // code that has possibility of exception
    cout << v.at[100000] << endl;
} catch (std::out-of-range& r){
    cout << "Range error" << r.what() << endl;
} // r here is a reference to an object of class type std::out-of-range defined in <stdexcept>. The what method returns a string describing the error.
```

Helps us address the "non-locality of error handling." `vector` knows an error happened but doesn't know what to do about it. We, on the other hand don't know when the error will happen but know what to do if it happens. `try-catch` lets these two sides "talk" to each other.

To raise an exception ourselves, use the `throw` keyword.

In C++, any type can be thrown, but `<stdexcept>` has a large number of objects for common scenarios:

- `out-of-range`, `logic-error`, `invalid-argument`, etc.

When an exception is thrown, we look through the stack frames to find an encompassing `catch` block with a type that matches the thrown exception. If none found, program crashes.

Example:

```C++
void f(){
    throw logic-error{"called f, bozo"};
    cout << "done f";
}

void g(){
    f();
    cout << "done g";
}

void h(){
    g();
    cout << "done h";
}

int main(){
    try{
        h();
    } catch(logic-error& e){
        cout << e.what() << endl; // outputs "called f, bozo"
    }
}
```

When an exception is thrown, control flow is immediately passed to the exception and jumps to the catch block. **None of the print statements run**.

This process is called "stack unwinding".

This process is called stack unwinding. If there are functions we jump out of, the compiler makes sure taht the destructors of any objects stored on the stack of these functions are run.

For pointers to objects stored on the heap, you might just leak memory.

**We can also throw from within a catch block.**

```C++
void calculation(DataStructure& ds){...}
void controller(Datastructure& ds){
    try{
        calculation(ds);
    } catch(ds_error& dse){
        throw prompt-input-error{...} // responding an error with a catch
    }
}

int main(){
    while(cin >> s){
        try{
            controller(ds);
        } catch(prompt_input_error& pie){
            cout << "Bad input, try again" << endl; // To escalate it higher and higher
        }
    }
}
```

Also demonstrates separating error recovery into different parts of the program.

We can also rethrow exceptions:

```C++
try{...}
catch(std::exception& s){
    ... // maybe we can only fix half of the errors
    throw; // re-raises the exception for higher-up functions to deal with
}
```

Note: If re-raising an exception, use `throw` and not `throw s`;

1. `throw s` performs an extra copy
2. `throw s` will then search for `catch` blocks according to the _static type_ of `s` (but `s` could actually have a different dynamic type!)

Can also catch anything in C++ using the following

```C++
try{
    doSomething();
} catch (...){ // LITERALLY these three dots
    doSomethingElse();
}
```
