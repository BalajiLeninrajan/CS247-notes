# CS247 Lecture 5 _May 20th_

## (Cont. from last lecture)

Invariants for linked lists:

- no cyclic lists
- all next pointers are heap allocated or `nullptr`
- no sharing (assuming that we implement the list big 5)

Our list class is more safe but it is slower due to all the checks of invariants.

```C
List l;
l.push("foo");
l.push("bar");
l.push("bash");

// to print list
for (for i = 0; i < 3; i++) {
    std::cout << l.ith(i) << std::endl;
}
```

## Fixing efficiency with iterators

### why is `l.ith()` so slow?

`l.ith()` is slow because it has to traverse the list every time we call it. This is O(i) for each call, resulting in quadratic time complexity for printing the list.
Traversal is slow and allowing clients to access the underlying nodes is representation exposure, the solution is the iterator design pattern.

The iterator design pattern allows us efficient iteration without representation exposure.

### The iterator design pattern

The iterator models a pointer like abstraction: fast access w/o representation exposure.

```C
// Example: ptr to go through an array
int arr[5] = {1, 2, 3, 4, 5};
for (int *p = arr; p != arr + 5; p++) {
  cout << *p << endl;
}
```

Our iterator modeled on this type of for loop using a pointer, Namely:

- initialize the pointer `int * p`
- get corresponding data `*p`
- ending pointer `arr + 5`
- Conditional to check if we reached the end `p != arr + 5`

```C
// With iterators
for (List::iterator p = l.begin(); p != l.end(); p++) {
    std::cout << *p << std::endl;
}
```

### implementation of List

```C
class List {
  struct Node;
  Node * head: nullptr;
  public:
    // same public methods

  class Iterator {
    Node * cur; // where we are in the linked list
    public:
      explicit Iterator(Node * n) : cur(n) {} // constructor
      // operator overloads
      // not equals for conditional check
      bool operator!=(const Iterator & other) const {
        return cur != other.cur; // check if we are at the same node
      }

      // to go to next node
      Iterator & operator++() {
        cur = cur->next; // go to next node
        return *this;
      }

      //dereference operator
      string operator*(){
        return cur->data; // return the data of the current node
      }
    }
  }

  Iterator begin() {
    return Iterator(head); // return the iterator pointing to the head
  }

  Iterator end() {
    return Iterator(nullptr); // return the iterator pointing to nullptr
  }
}

// Example usage
for (List::iterator it = l.begin(); it != l.end(); it++) {
    std::cout << *it << std::endl;
}
```

Every method we just created is in constant time, hence overall time complexity is O(n) for printing the list.
C++ has built in support for the iterator design pattern.
If we have a container class that implements:

- `begin()`
- `end()`
  and a iterator class that implements:
- `++()`
- `*()`
- `!=()`
  Then we can use the range based for loop.

```C
for (auto x : l) { // implicit call to iterator, uses *() to deduce type
    std::cout << x << std::endl;
}
```

If we want to loop through the list and modify the strings, `*()` returns a reference to the string.

```C
for (auto & x : l) { // explicit ask for reference
    x = "Hello"; // modifies the string in the list
}
```

The iterators can allow for forgery

- Clients should not create their own iterators
  - they should just use begin and end
  - `auto it = List::Iterator(nullptr);` should not be done by the client
  - in cases for an iterator for an array or vector this could cause a segmentation fault if used incorrectly

**Solution**: Make iterator's constructor private and make List a friend class.

```C
class List {
  struct Node;
  Node * head: nullptr;
  public:
    // same public methods

  class Iterator {
    Node * cur;
    expliost Iterator(Node * n) : cur(n) {} // constructor
    public:
      friend List; // make List a friend class (can access private members of Iterator)
      // rest is the same as before
  }
}
```

- Forgery is prevented and clients can only use the iterators through the List class.
- Friends weaken encapsulation and make it more difficult handle invariants.
- Make friends only when they can do something for you

### Const correctness

Variables which won't be modify are labeled `const`, otherwise they are mutable.

- Looking for the maximum usage of the word `const`, go full  on this shit
- easier to reason about Variables

```C
void printList(const List& l) { // pass by reference
    for (auto x : l) {
        std::cout << x << std::endl;
    }
}
```

This will fail compilation as the compiler is worried that the ranged based for loop using the iterator will mutate the object.
C++ will only let you call methods that are defined to be const.
`const` methods are checked to make sure they do not modify the fields of `this`.

```C
Iterator List:begin() const { // const keyword tells compiler that we aren't modifying the list
  // same as before
}
```

## Next time: fixing iterators to handle `const` correctness properly
