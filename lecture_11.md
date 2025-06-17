# CS247 Lecture 11 _June 10rd_

## Polymorphic copy/move/assignment

### Let's consider the book hierarchy

```c++
Text t{"Polymorphism", "Author", 300, "Topic"};
Text t2 = t; // invokes copy constructor
```

This does work, we are given a compiler provided implementation of the copy, move constructor and assignment operator. They do what we expect, they copy/move the individual fields.

### Compiler provided copy/move constructors

#### Constructors

```C++
// copy constructor
Text::Text(const Text& other) : Book(other), topic(other.topic) {}
```

> [!NOTE]
> there is a implicit conversion of other from `const Text&` to `const Book&`
> when calling the Book copy constructor

```C++
// copy constructor
Text::Text(Text&& other) : Book(std::move(other)), topic(std::move(other.topic)) {}
```

> [!NOTE]
> we use `std::move` to indicate that we are moving the object, not copying it.
> even though other is a rvalue reference, it's value category is lvalue
> if `std::move` isn't invoked we would copy instead of moving

#### Assignment operators

```C++
// copy assignment operator
Text& Text::operator=(const Text& other) {
  // *this = other; won't work since this is a recursive call

  // calls book copy assignment operator
  // implicit conversions to `const Book&`
  // uses implicit this: this->Book::operator=(other);
  Book::operator=(other); // call base class assignment operator

  topic = other.topic; // copy topic field
  return *this;
}
```

```C++
// move assignment operator
Text& Text::operator=(const Text&& other) {
  Book::operator=(std::move(other)); // call base class assignment operator

  topic = std::move(other.topic); // move topic field
  return *this;
}
```

Usually the compiler provided copy/move constructors and assignment operators are sufficient, but sometimes we need to implement our own, i.e. when doing manual memory management.

### Issues with the compiler provided copy/move constructors and assignment operators

```C++
Text t{"Polymorphism", "Ross", 1000, "C++"};
Text t2{"Programming for babies", "LaurierProf", 30, "Python"};

Book& br1 = t1;
Book& br2 = t2;

br1 = br2; // performing assignment on the underlying objects
cout << t1; // {"Programming for babies", "LaurierProf", 30, "C++"}
```

The topic field remains unchanged since it is not part of the book class.
Calling `br1 = b2`: the copy assignment operator is non-virtual we use the static type to resolve the method call, which is `Book&`, hence the `topic` field is never assigned.
This is called the **partial assignment problem** because only part(the super class part) of the object is reassigned.

```C++
Text t{"Polymorphism", "Ross", 1000, "C++"};
Text t2{"Programming for babies", "LaurierProf", 30, "Python"};

Book& br1 = t1;
Text& br2 = t2;

br1 = br2; // will still call Book's operator
// the line above === br1.operator=(br2); // calls Book's operator
```

#### Virtual big 5

The natural attempt to solve the partial assignment problem is to make the big 5 virtual, but this is not a good idea.

```C++
class Book {
  string title, author;
  it length;
public:
  ...
  virtual Book& operator=(const Book& other) {
    title = other.title;
    author = other.author;
    length = other.length;
    return *this;
  }
}
```

The usual signature for `Text::opperator=` is:

```C++
Text& Text::operator=(const Text& other);
```

What is stopping us from slapping `override` on the end of the signature? The signatures are different!
The return and argument types are different. The return type being different is actually ok in overrides(C++ allows covariant return types), but the argument type is not.
**Covariant return types**: superclass returns `A&` or `A*` in a virtual method, subclasses can override it to return `B&` or `B*` where `B` is a subclass of `A`.

However any chance to the arguments in a subclass method makes it an invalid override. For this a valid override we need the following signature:

```C++
Text& Text::operator=(const Book& other);
```

Problems:

1. Usually, our `Text::operator=` would contain the line `topic = other.topic;` but we can't access the `topic` field of `other`
2. Now it's legal to perform assignment between sibling classes, this is called the **mixed assignment problem**

```C++
Comic c{...};
Textbook t{...};
t = c; // compiles, but doesn't do what we want
```

> [!WARNING]
> ... life stuff happened for a bit, notes incomplete
