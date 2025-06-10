# CS247 Lecture 9 _June 3rd_

## UML

**UML**: unified modeling language, a standard way to visualize the design of a system.

### Modelling a class

```
-------------
| ClassName |
--------------------- <- fields (optional)
| - field1: Integer |
| # field2: Boolean |
| + field3: String  |
-------------------------- <- methods (optional)
| - getField1(): Integer |
| - helper(): Void       |
--------------------------
```

Note: In the diagram above:

- `-` denotes private
- `#` denotes protected
- `+` denotes public

### Composition and Aggregation

If A "owns-a" B object (Composition):

- The B object does not have an independent existence outside of A.
- A destroyed => B destroyed
- A is copied => B is copied (deep copy)

![example UML for SLL](https://media.discordapp.net/attachments/1352286409003761715/1379465785046728764/image0.jpg?ex=68405720&is=683f05a0&hm=063cf8d08f35a7319977c097b46cf202595858272fb60543968f164d1fa3cb8d&=&format=webp)

If A "has-a" B object (Aggregation):

- The B object can exist independently of A.
- A destroyed, B lives on
- A is copied => B is not copied (shallow copy)

![example UML with aggregation](https://media.discordapp.net/attachments/1352286409003761715/1379465938478301325/image0.jpg?ex=68405744&is=683f05c4&hm=d58ca240991d88bb2ecb83a2cadfbe393cfd057f5b0202459aeb4146c92e201c&=&format=webp)

Implementation in C++:
Composition: object field or a owning pointer (you are responsible for the lifetime of the object)
Aggregation: non-owning pointer or reference

```C
class Student {
  University* university; // Aggregation
  ... // other fields
};

int main() {
  University u;
  Student s1{&u}; // can access u independently of s1
}
```

Specialization relationships ("is-a"):

- one class is a Specialization of the other
- If B is a specialization of A, then B "is-a" A and we can substitute B for A.
- In C++: public inheritance

![Example UML for specialization](https://media.discordapp.net/attachments/1352286409003761715/1379469577813102622/image0.jpg?ex=68405aa8&is=683f0928&hm=5ef7654d780c356254b691ff500ea4ebea59a1b469b322f4bc70ed4a48ec2697&=&format=webp)

```C
class Book {
  String title, author;
  int lenght;
  protected:
  int getLength() const { return length; }
  public:
  Book(String t, String a, int l) : title(t), author(a), length(l) {}
  virtual bool isHeavy() const {
    return length > 200;
  }
}

class Textbook: public Book {
  String topic;
  public:
  // Why `Book(t, a, l), topic(topic)` instead of doing each field assignment manually?
  // fields are private and the MIL can only be used for direct fields of the class
  // MIL doesn't work with inherited fields (public or private)
  Textbook(String t, String a, int l, String topic)
    : Book(t, a, l), topic(topic) {}
  bool isHeavy() const override {
    return getLength() > 500;
  }
}
```

## Dynamic Dispatch

Polymorphic Types: A type that can represent multiple different objects, eg. Book\* can be both a Book and a Textbook.
Dynamic dispatch: the process of selecting which method to call at runtime based on the actual type of the object.

```C
Book* b;
String choice;
cin >> choice;
if(choice == "book") {
  b = new Book("Title", "Author", 300);
} else if(choice == "textbook") {
  b = new Textbook("Title", "Author", 600, "Topic");
}
// static type of b: Book*
// dynamic type of b: Book or Textbook pointer
```

Types of types:

- Static type: the type of the pointer/reference at compile time.
- Dynamic type: the underlying type of the object pointed to at runtime.
  - Compiler generally does not know the dynamic type of a pointer/reference.

How to figure out which `isHeavy()` is called?

- Calling the method directly: **ALWAYS** use static type of the pointer/reference.

```C
Book b{...};
Textbook t{...};
b.isHeavy(); // calls Book::isHeavy()
t.isHeavy(); // calls Textbook::isHeavy()
Book bt = Textbook{...}; // compiles!
b.isHeavy(); // calls Book::isHeavy() - bt is a copy of t, not a reference
```

- is the method virtual in the static type's methods (or in any superclass of the static type)? Use dynamic type of the pointer/reference.
- Non-virtual in the static types method or in superclasses' methods? Use static type of the pointer/reference.

The `override` keyword has no effect on the executable itself, nevertheless it's useful for catching bugs

### Why not use `virtual` everywhere?

Because Java does that and Java is suck.

1. Sometimes we will wish to control how our subclasses override methods.
2. performance 🏎️

```C
Struct Vec{
  int x, y;
  void doSmth() {};
}

Struct Vec2{
  int x, y;
  virtual void doSmth() {};
}

Vec v1{1, 2}; // 8 bytes
Vec2 v2; // 16 bytes
```

Adding virtual to a method add a virtual pointer which allows dynamic dispatch and inheritance, but it adds extra memory overhead and performance cost.
