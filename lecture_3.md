# CS247 Lecture 3 _May 13th_

## Takeaways

- Interface driven development (IDD)
  - Writing out the client code and design interface first before doing implementation.
  - The idea is that we don't force an ugly interface on the client.
- Operator overloading gave us a convenient syntax, allows us to treat our own classes like built-ins
- Good ADT design:
  - protects invariants
  - resistant to tampering and forgery

## Example: Linked List

### Correctness

```C
struct Node {
  string data;
  Node* next;
};

Node n{"a", new Node{"b", new Node{"c", nullptr}}};
Node p{n}; // runs copy constructor (field by field copy)
p.next->data = "z";
cout << p; // expected: a, z, c
cout << n; // expected: a, b, c
```

![shallow copy diagram](https://cdn.discordapp.com/attachments/1352286409003761715/1371855166902632568/image0.jpg?ex=6824a72f&is=682355af&hm=c401d9c99e974af2f519ded1b8a2fead9092557be7b80682d4c2d55590be1789&)

This is a shallow copy, data is shared between variables
We can write a custom copy constructor to do a deep copy instead

```C
Node(const Node& other): data{other.data} next{other.next ? new Node{*other.next} : nullptr} {}
```

![deep copy diagram](https://cdn.discordapp.com/attachments/1352286409003761715/1371856671520854116/image0.jpg?ex=6824a896&is=68235716&hm=7184314b6848f8324829ca9d6d33cfcca6b796e2176ecf764d5c251d97454118&)

```C
// dtor for linked list
// recursive destructor
// base case: `delete nullptr`, this is safe
~Node(){delete next;}
```

### Efficiency

```C
Node getAlphabet() {
  return Node{"a", new Node{"b", new Node{"c", ...}}};
}

Node getAlphabet = getAlphabet();
```

Steps taken by program:

![alphabet diagram](https://cdn.discordapp.com/attachments/1352286409003761715/1371859660167974964/image0.jpg?ex=6824ab5e&is=682359de&hm=d08ae8447189274917b95dcfd9fcde0b2a7f28062c17ad13dc04aca10bf8d16d&)

- Create list inside `getAlphabet`stack frame
- To get this list in to the caller's stack frame,
  copy construct the alphabet from the `getAlphabet`'s return value
- temporary list made in `getAlphabet` is destroyed

**This is obviously inefficient, why don't we reuse the list?**

#### Value categories

Every expression in C++ has w independant properties:

- Type: how and expression can be used
- Value category: tells us about expression's lifetime

  - We will cover 2 types of value categories:

    - lvalue: has an address, can be assigned to

    ```C
    int x = 5;
    f(x); // x is an lvalue, we take address of x in the expr
    ```

    - rvalue: temporary values, will be destroyed "soon", we can't assign to them

    ```C
    f(5); // 5 is an rvalue, we don't take address of 5 in the expr

    string g() {return "Hello";}
    string s = g(); // return value of g() is an rvalue
    // g() destroyed after the assignment, &g() does not work

    ```

  - examples:

  ```C
  // valid
  int x = 5;
  int &y = x;

  // invalid, can't bind rvalue
  int &x = 5;

  // invalid, can't bind rvalue
  void f(int& x);
  f(5);

  // valid, special case: can bind const lvalue to rvalue
  void g(const int&x);
  g(x);
  ```

  - we can also create rvalue references: extended lifetime of an rvalue to the lifetime of the reference

  ```C
  string f() {return "Hello";}
  string& s = f(); // invalid, f() is destroyed
  // T&& in c++ is not reference to reference, it's instead used for rvalue references
  string&& s = f(); // valid, s is an rvalue reference
  // we can use s till it's out of scope
  ```

  - commonly used for function overloading:

  ```C
  void f(const string& s) {cout << "1";}
  void f(string&& s) {cout << "2";}
  string s{"Hello"};
  f(s); // prints 1
  f("Hello"); // prints 2
  ```

#### Back to Linked List example

We copied 26 nodes instead of using the originals

```C
// This required a copy, otherwise data would be shared (shallow copy)
Node n{"a", new Node{"b", new Node{"c", nullptr}}};
Node p{n}; // lvalue, copy constructor called

// Copy is not needed here
Node alphabet =
    getAlphabet(); // this is a rvalue, we need a move constructor
```

#### Move constructor

copy constructor for rvalues

```C
Node(Node&& other) : data{other.data}, next{other.next} {}
```

![move constructor diagram](https://cdn.discordapp.com/attachments/1352286409003761715/1371867271881822218/image0.jpg?ex=6824b275&is=682360f5&hm=cf877780eef2c86fb1fc888724aacc9d950b6cabdaf956df841d0e90ab998dd4&)
**But the destructor will destroy the original!**

```C
// properly "steal" the values by taking it away from other
Node(Node&& other) : data{other.data}, next{other.next} {
  other.next = nullptr; // set the original to nullptr
}
```

**RLAVUES are rarely used outside of uses like this**

Efficiency gain from move constructor: O(n) -> O(1)

#### One other bit of efficiency

- `std::string` has a move constructor
- is other.data a lvalue or rvalue?
  - lvalue
    - we can get address
    - it has a name
    - it can be scoped
  - even though type of other is rvalue reeference
  - recall that type and value category are different
- we invoke the copy constructor in the example, but we can use the move constructor

```C
Node(Node&& other) : data{std::move(other.data)}, next{other.next} {}
```

`std::move`: forces any values to be treated as rvalues, in the example above it causes the move constructor to be called instead of the copy constructor for strings
