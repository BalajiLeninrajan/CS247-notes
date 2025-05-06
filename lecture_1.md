# CS 247 Lecture 1

## Whats the point???

### Vibe coding works

- AI is useful but you must be able to detect bullshit
- You can only do that by getting a deep knowledge base
  - makes you more employable
- Don't be a prompt engineer, everyone is a prompt engineer

---

### Why C++

- C++ is close to hardware but all these details exist in higher level languages as well
- Learning these details makes you better at higher level languages

## C++ refresher

### ADT: Abstarct Data Type

- C++: Bundle collection of values along w/ operations and error handling
- Interface: How clients use your ADT
- Implementation: underlying details
  - Clients should not need to know implementation to use ADT

### How to prevent misuse of ADT at compile time

- Detecting issues at compile time prevents a whole class of bugs from reaching the user
- Types of misuse:

  - Forgery: Creating an invalid instantiation of an ADT

  ```c
  ADT1 v1;
  ADT2 *v2 = malloc(sizeof(ADT2));
  v2 = &v1;
  ```

  - Tampering: Modifying an valid ADT to become invalid in an unexpected way

### Example

```c
#include <iostream>
using namespace std;

int main() {
  Rational r, p, q; // define default ctor
  cin >> r >> q;
  p = q + r; // define addition
  cout << q/r << endl;
  Rational z{q};
}

Class Rational {
  int num, denom;
  public:
    Rational(): num {0}, denom {1} {}
    explicit Rational(int n): num {n}, denom {1} {} // this is an example
    Rational(int n, int d): num {n}, denom {d} {}
};
```

### What's the deal with `explicit`?

```c
// this is an example of implicit conversion
void f(std::string s) {}
f("Hello"); // "Hello" is of type const char* (string literal)
```

The `explicit` keyword prevents implicit conversion from `int` to `Rational` in the constructor

- will aid in catching mistakes which occur due to implicit conversions
- generally single argument constructors should be `explicit`

### Does the Rational class have to be a class?

ADT: not tied to a particular implementation - could also use a c style struct as long as it satisfies the ADT requirements

- Reasons to use classes instead of structs:
  - contructors/destructors is garunteed
  - can enforce access specifiers (public/private/protected)
  - Eg: rational class - we may wish to ensure denom != 0
    - prevents forgery in the constructor
    - prevent tempering with private denom field

> [!IMPORTANT]
> Invariant: property of a class that should always be true

### Default constructors

- we gave rationLal a default constructor `Rational()`
  - a default constructor is a constructor that takes no arguments
- Not all classes should have a default constructor
  - Eg: for a `Student` class, there is no sensible default for fields like name or birthday
- If you do not write a default constructor, the compiler will generate one for you
  - object fields are initialized with their default constructors
  - primitive fields are uninitialized - garbage memory
- Writing any constructor will prevent the compiler from generating a default constructor
  - if you want to use the default constructor, you must explicitly define it
  - copy and move constructors are still generated

### Default parameters

```c
#include <iostream>
using namespace std;

int main() {
  Rational r, p, q; // define default ctor
  cin >> r >> q;
  p = q + r; // define addition
  cout << q/r << endl;
  Rational z{q};
}

Class Rational {
  int num, denom;
  public:
    // using explicit here is valid, will come into play when any value is given
    Rational(int n = 0, int d = 1): num {n}, denom {d} {}
};

```

Note: Default parameters appear in declaration, not in the definition

```c
// Rational.h
Class Rational {
  int num, denom;
  public:
    // using explicit here is valid, will come into play when any value is given
    Rational(int n = 0, int d = 1);
}

// Rational.cc
// adding default parameters would throw a compile error
Rational::Rational(int n, int d): num {n}, denom {d} {}

```

Reason: The caller is responsible for pushing args onto the stack

- If default args are provided solely in the .cc file then the callee would need to push the default args onto the stack
- This is simply not possible
- **The caller must know the defaults so that it can push them and the caller imports .h files not .cc files**
- will cause problems if the same .cc file is imported in multiple places

### MIL: member initialization list

```c
// MIL
Rational::Rational(int n, int d): num {n}, denom {d} {}

// Alternative
Rational::Rational(int n, int d) {
  this->num = n;
  this->denom = d;
}
```

- MIL is considered best practice due to the object construction process
- 4 steps to creating an object:
  1. Memory is allocated (stack or head)
  2. Call super class constructor
  3. Initialize fields (usually done via MIL)
  4. Constructor body is executed
- Rule: By step 4, all object fields must be initialized
  - there's no such thing as uninitialized objects
  - primitive types can be uninitialized because of C backwards compatibility
- if not provided with MIL they are initialized first with default constructors then overwritten
  Example:

```c
struct C {
  String s;
  // suboptimal approach
  C(string s) { // initializes s with default constructor
    this->s = s; // overwrites s
  }
}
```

- MIL is significantly faster if the super class constructor is slow

**cases where MIL is needed**

- const fields
- reference fields
- object fields w/o default constructor
- super class w/o default constructor
