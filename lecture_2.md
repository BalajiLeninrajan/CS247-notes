# Lecture 2 _May 8th, from Jane_

Last time we observed the `Rational` class and motivations for learning CS 247, as well as why the MIL is more efficient in C++.

## Today: Operator overloading, friends

Recall this as the Object Creation steps:

1. Space is allocated for the object on the stack or heap
2. Call the superclass constructor if needed
3. Initialize fields (MIL)
4. Run constructor body

**Rule**: By step 4, all object fields must be initialized. We can't have uninitialized objects. If you do not provide values for object fields in MIL, then they are initialized first with their default ctor and subsequently overwritten.

```C
struct Course {
    int numStudents;
    Prof prof;
    Course(int n, Prof p): numStudents{n}, prof{p} {}
};

struct Student{
    const int id;
    Uni& myUni;
    Course c;
    Student(int id, Uni& uni, Course c){
        // because these are all considered field reassignments
        this->id = id; // fails because const cannot be reassigned
        this->myUni = uni; // fails because there's no null reference, all references must be initialized rather than reassigned
        this->c = c; // We tried to call the default constructor in step 3 of object creation, but Course has no compiler-provided default constructor
    }
};
```

## Operator overloading

**Overloading**: Providing multiple function/method definitions with differing numbers or types of arguments

E.g.

```C
bool negate(bool b){
    return !b;
}

int negate(int x){
    return -x;
}
```

**Operator overloading**: Gives us syntactic sugar for treating our objects as if they were ordinary types.

To perform operator overloading:

- Write a function with the name "operator" concatenated with the operation you want to define

E.g.

```C
operator+   operator>>  operator<<
```

Note that since these are functions, we need to consider parameter and return types, because C++ has both unary and binary operators.

**Arguments**: Depend on the arity of the operator
Unary operators: !
Binary operators: +, -, \*, /

### Supporting `cin >> r >> q;`

```C
istream& operator>>(istream& in, Rational& r){ // thing on left becomes first argument, right becomes second
    in >> r.num;
    in >> r.denom;
    return in;
}

// This is a standalone function, not a method, just living in a .cc file
```

A few notes on this:

- `istreams` cannot be copied - you need to pass by reference or pointer
- `Rational` is passed by reference so changes are reflected outside of this function. Recall that we want to read into the actual object and not a copy to happen
- `return in` allows for chaining. If we parenthesize, it looks like `(cin >> r) >> q;`
  - Compiler translates this into: `operator>>(operator>>(cin, r), q)`, like functions
- Only assignment operators are implicitly defined
- **Only a limited number of these operator symbols can be defined**
- **This does not actually compile because num and denom are _private_ fields** (defined in `Rational`, Lecture 1)

### Solution 1: Provide accessor and mutator methods

Public methods: `getNum`, `getDenom`, `setNum`, `setDenom`

- Allow us to access and modify num and denom
- Accessors and mutators are better practice because not only are they explicit, we can also **enforce variations and do checks**
- Downside: _What if we have too many fields?_

### Solution 2: Use a friend

```C
class Rational{
    int num, denom;
    // rest over here...

    friend istream& operator>>(istream&, Rational&); // declared inside the class
    // operator>> is still standalone, but it can now access and modify private fields
};
```

With the above, the original code will now compile.

**Note**: This also allows reading from files!

```C
ifstream file{"file.txt"};
Rational r, q;
file >> r >> q; // ifstream (file stream) is a subclass of istream, so this works
```

### Supporting `p = q + r`

Defining `operator+`:

```C
Rational operator+(const Rational& lhs, const Rational& rhs){
    return Rational{lhrs.num * rhs.denom + lhs.denom *rhs.num, lhs.denom * rhs.denom};
}

// q + r is the same as operator+(q, r);
```

It is better practice to use constant references in cases where we don't have expect the arguments to change. References to avoid unnecessary copying and greater efficiency.

**Unfortunately, declaring all these friends can be quite a pain.**

Other option is just doing operator overloading via methods instead of standalone functions.

```C
class Rational{
    int num, denom;
    public:
        // stuff
        Rational operator+(const Rational &rhs){
            // the left hand side becomes this
            return Rational{...}
        }
        // But we're accessing the private fields of rhs! We can access private fields from any object
}
```

Operator `>>` and `<<` are still typically defined as standalone functions! Why?

The thing on the left hand side is typically `istream`, but that class is usually not available to us.

### What about native types?

**Supporting addition with ints:**

```C
class Rational{
    public:
        // This definition supports r + 5 but not 5 + r.
        Rational operator+(int rhs){...}
}
```

`5 + r` case is done with **delegation**: A standalone function that calls the method.

```C
Rational operator+(int lhs, const Rational& rhs){
    return rhs + lhs;
}
```

### Supporting `p = q + r`

The assignment operator works due to compiler provided copy assignment operator.

You can also define it yourself:

```C
class Rational{
    ...
    public:
        Rational& operator=(const Rational& rhs){
            num = rhs.num;
            denom = rhs.denom;
            return *this;
        }
}
```

Return type: `Rational&` - to support chaining with assignment.

E.g. `(a = (b = (c = d)));`

**Note**: `operator=` MUST BE A METHOD.
Similarly: `operator[]`, `operator->`, `operator()`, `operator T where T is a type - conversion operator`

^^ yeah you don't have to memorize this...

```C
public:
    operator float(){...}
```

### Supporting `cout << q / r << endl`

```C
class Rational{
    ...
    friend ostream& operator<<(ostream& out, const Rational& r);
};

ostream& operator<<(ostream& out, const Rational& r){
    return out << r.num << "/" << r.denom;
}
```
