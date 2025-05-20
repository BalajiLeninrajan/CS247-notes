# CS247 Lecture 4 _May 15th_

## Assignment operators and the safe list

```C
// copy assignment operator for linked list
Node& Node::operator=(const Node& other) {
  if (this == &other) return *this; // check for self-assignment
  // For an assignment operator, data is not created but overwritten
  delete next; // free old memory
  data = other.data;
  // recursive call to copy constructor
  next = other.next ? new Node{*other.next} : nullptr; // for deep copy
  return *this; // recall: to support chaining
}
```

![Copy constructor stack diagram](https://cdn.discordapp.com/attachments/1352286409003761715/1372581897716236299/image0.jpg?ex=68274c01&is=6825fa81&hm=f7cc45644ff0aeab9018e5096522d0937773b40cbeb45c09b9c15d83e1d27ad0&)

In a low memory environment, new, which request memory from the OS, may fail. In this case an exception is thrown and we "bail out" of the function (don't run anymore lines).
Currently we delete all our own nodes before attempting to copy from other. If `new` fails, we destroy our own list and can't recover.
A better approach is to request new memory before deleting our own nodes. If `new` fails, we can just return without doing anything. Will temporarily increase memory pressure but is overall safer

```C
// copy assignment operator for linked list
Node& Node::operator=(const Node& other) {
  if (this == &other) return *this; // check for self-assignment
  // call new first
  Node* tmp = other.next ? new Node{*other.next} : nullptr; // for deep copy
  // For an assignment operator, data is not created but overwritten
  delete next; // free old memory
  data = other.data;
  // recursive call to copy constructor
  next = tmp;
  return *this; // recall: to support chaining
}
```

![Stack diagram with `tmp`](https://cdn.discordapp.com/attachments/1352286409003761715/1372586510527889619/image0.jpg?ex=6827504d&is=6825fecd&hm=f9397420ed633330c21ad90d6bba4e087b76bd306b0847eb68fd96754b3c491e&)

We generally need all of the big 5:

- constructor
- destructor
- move constructor
- copy constructor
- assignment operator

### Copy/Move elision

Optimization which skips copy/move in safe cases, but this could effect the behaviour of the program
This works by instead writing the data directly to a variable instead of into main stack frame.
Elision is mandatory C++14 onwards, **elision won't be tested in bs ways in this course**

### Safe list

Tampering:

```C
Node n {"abc", nullptr}; n->next = &n; // this is bad
```

Crashes when n goes out of scope, deleting stack memory, causing an infinite loop.
ADTs have representation in memory, clients should not be able to tamper with this representation, otherwise we will have a _representation exposure_
The fix is encapsulation: provide an interface to the client that upholds the invariants of the ADT.
