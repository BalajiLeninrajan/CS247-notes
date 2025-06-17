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
