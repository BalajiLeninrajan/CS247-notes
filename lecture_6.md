# CS247 Lecture 6 _May 22th_

## `const` constructor and separate compilation

```C
void useList(const List& l) {
  for(auto& s : l) {
    s = "247"; // this modifies the list which should not be allowed
  }
}
```

The `const` keyword in this case only guarantee that the fields of the object do not change.
The strings in the example above are actually part of the object and are mutable.

```C
// recall
class List {
  struct Node;
  Node * head: nullptr;
  ...
}
```

The `const` keyword only gives us physical constness.

- physical constness: are the fields of the object changing, i.e. the bit representation stays the same, what the compiler sees
- logical constness: does the memory associated with the ADT change, what we see

`begin` and `end` are `conrst` methods, but return a non `const` iterator where the operator `*` returns a non-`const` `string&` type.

## `const` Overloading

Creating a new overload of `begin` and `end` that return a different iterators with `const` return types.

### setting up classes

```C
class List {
public:
  class Iterator {
    ...
  }
  class CIterator {
    Node * cur;
    public:
      explicit CIterator(Node * n) : cur(n) {} // constructor

      bool operator!=(const CIterator& other) const {
        return cur != other.cur;
      }

      Cterator & operator++() {
        cur = cur->next;
        return *this;
      }

      const string& operator*(){ // this is now const
        return cur->data;
      }
  }

  Iterator begin() {
    return Iterator(head);
  }
  Iterator end() {
    return Iterator(nullptr);
  }

  // new overloads
  CIterator begin() const {
    return Iterator(head);
  }
  CIterator end() const {
    return Iterator(nullptr);
  }
}

```

```C
void useList(const List& l) {
  for(auto& s : l) { // won't compile - "trying to create non-const reference from const reference"
    s = "247";
  }
  for(const auto& s : l) { // This will work
    s = "247";
  }
  for(const auto& s : l) { // This will also work, however will create copies
    s = "247";
  }
}

void changeList(List& l) { // notice not const
  for(auto& s : l) { // This still works
    s = "247";
  }
}
```

### DRY using a template class

Iterators are essentially identical, we can use templates to avoid code duplication.
Templates are instantiated with a type parameter.

```C
// template class recall
// vector<T> takes in a type T, vector is a template class
vector<int> v1;
vector<string> v1;
```

We will specify the return type of the operator`*` using the template parameter.

```C
class List {
  struct Node {}; // important - will come to this later
public:
  template<typename T> class MyIterator {
    Node * cur;
    public:
      explicit MyIterator<T>(Node * n) : cur(n) {} // constructor

      bool operator!=(const MyIterator<T>& other) const {
        return cur != other.cur; // check if we are at the same node
      }

      MyIterator<T>& operator++() {
        cur = cur->next;
        return *this;
      }

      T operator*(){
        return cur->data;
      }

      Friend class List;
  }
  // create the iterators
  using Iterator = MyIterator<string&>;
  using CIterator = MyIterator<const string&>;
}
```
