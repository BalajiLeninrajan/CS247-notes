# CS247 Lecture 13 _June 17rd_

## Exceptions

Never let a destructor throw and exception, destructors are tagged `noexcept`by default, if we do call a exception from a destructor, the program will call `std::terminate` which will crash the program.
We can get around this with

```C++
struct A{
  ~A() noexcept(false) {} // not recommended
}
```

During stack unwinding destructors for stack allocated objects are run. If one of these destructors run, then we have multiple active unhandled exceptions, this also calls `std::terminate` and there's no way to bypass this.

If `new` fails, `std::bad_alloc` is thrown.

## Exception Safety

`virtual` functions and exceptions significantly change the flow of our program

- `virtual` functions when calling a method, it is no longer obvious at compile time where we will go.
- Exceptions: can jump out of many functions once without returning

```C++
// easy to reason about in a world without exceptions
void f() {
  MyClass mc; // stack unwinding will handle this in any case
  MyClass *p = new myClass;
  g(); // it is no longer clear if g will return
  delete p; // if g throws, we will never reach this line, that will cause a memory leak
}
```

```C++
// easy to reason about in a world without exceptions
void f() {
  MyClass mc; // stack unwinding will handle this in any case
  MyClass *p = new myClass;
  try {
    g(); // it is no longer clear if g will return
  } catch (...) {
    delete p; // clean stuff on the heap
    throw;
  }
  delete p; // code duplication, inelegant and will probably shoot you in the foot with bigger programs
}
```

**Goal**: ensure that `p` is freed at the end of `f()`
Some other languages have a `finally` block, runs on function exit, but C++ does not have this, only stack unwinding.
_Simple approach_: Don't use pointers if you don't have to, put everything on stack.
Unfortunately this is not always possible, without pointers we can't have dynamic dispatch and polymorphism.
In this case we can wrap a resource in a stack allocated object that will free the resource when it goes out of scope.

> C++ idiom: **RAII**: Resource Acquisition Is Initialization

Example: files

```C++
// in C++
{
  ifstream f{"file.txt"}; // RAII, file is opened
  // do stuff with f
} // file is closed when f goes out of scope

// in C resource acquisition and freeing is manually done
{
  FILE *f = fopen("file.txt", "r");
  // do stuff with f
  fclose(f); // if we forget this, we leak memory
}
```

### Smart Pointers :D

```C++
std::unique_ptr<T> - from #include <memory>
// wraps a pointer to an object of type T, when the unique_ptr goes out of scope, the object is deleted
```

There are operator overload with `*` and `->` so we can use it just like a normal pointer.

```C++
void f() {
  MyClass mc; // stack unwinding will handle this in any case
  unique_ptr<MyClass> p{new myClass}; // also on stack, no memory leak
  g(); // can still throw an exception
} // implicitly calls delete on p, no memory leak
```

**Make sure `unique_ptr`s are actually unique**.

```C++
{
  C* c = new C;
  unique_ptr<C> p{c}; // p now owns c
  unique_ptr<C> q{c}; // oh no
} // p and q will both try to delete c, which is undefined behavior
```

To prevent these, we can use `std::make_unique<T>()` which calls `new T` for us and returns a `std::unique_ptr<T>` to it.
Hides the new, this means the C++ adage for a `delete` for a `new` still holds (use `make_unique` instead of `new`)

```C++
auto p = std::make_unique<MyClass>();
unique_ptr<MyClass> q{p}; // trying to call the copy constructor
```

`unique_ptr`'s copy constructor is invokes.

- the compiler provided copy constructor will do a shallow copy, which would cause a double delete.
- doing a deep copy defeats the point of having a pointer in the first place, confusing semantics.
- both these options are straight ass so we just send the copy constructor of `unique_ptr` to the shadow realm.
- the copy constructor is `delete`d

Our shitty implementation of a `unique_ptr`.

```C++
template <typename T> class unique_ptr {
  T *ptr;

public:
  unique_ptr(T *p = nullptr) : ptr(p) {}
  ~unique_ptr() {
    delete ptr; // Automatically delete the managed object
  }

  unique_ptr(const unique_ptr &) = delete; // Disable copy constructor
  unique_ptr &operator=(const unique_ptr &) = delete; // Disable copy assignment

  unique_ptr(unique_ptr<T> &&other) : ptr(other) { other.ptr = nullptr; }
  unique_ptr<T> &operator=(unique_ptr<T> &&other) {
    swap(ptr, other.ptr);
    return *this;
  }

  T &operator*() {
    return *ptr; // Dereference operator
  }

  T *get() {
    return ptr; // Get the raw pointer
  }
};
```

`unique_ptr`s can also be used for ownership/owns-a relationships, since owns-a mean the object should be freed once it is done being used.

### `shared_ptr`s

`shared_ptr`s keep track to how many pointers point to a object in memory and the resource is freed when all the pointers are out of scope.
These are not bulletproof, cyclic references can cause memory leaks, so use them with care.
