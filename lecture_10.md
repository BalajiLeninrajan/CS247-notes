# CS247 Lecture 10 _June 5rd_

## Why not virtual everything?

How regular function calls are compiled:
At compile time, we know which function to jump to, `jalr` in mips. Recall `jalr` sets `PC` to execute the assembly associated with that function.
In dynamic dispatch, the address we want to jump to cannot be hard codded, it may depend on user input.

```C++
struct vec2 {
  int x, y;
  virtual void f() {
    cout << "vec2";
  }
};

struct vec3 : vec2 {
  int z;
  void f() override {
    cout << "vec3";
  }
};
```

![Memory diagram of the above classes](https://media.discordapp.net/attachments/1352286409003761715/1380190009377816628/image0.jpg?ex=6842f99c&is=6841a81c&hm=d84b994a3f866bfe7b6c2c327283852599131d89e0032ddf96287b7e0fc9e5a5&=&format=webp)

**Example**

```C++
vec2 *v = new vec2();
v->f();
```

The compiler will produce

1. start at the top of the object's memory layout, follow the pointer to vtable
2. access the pointer to `f()` inside the vtable
3. emit a `jalr` instruction to jump to the address of `f()`

Takeaway: virtual functions are not free, they require more space in memory and more instructions to execute.

C++ philosophy state that you should not pay for things unless you ask for them, you get the fastest thing by default. Other programming languages may have virtual as default.

## how to shoot yourself in the foot 101

### `virtual` destructors

```C++
struct X {
  int * a;
  X(int n) : a {new int[n]} {}
  ~X() {delete[] a; } // this line will hurt us
};

struct Y : public X {
  int * b;
  Y(int n, int m) : X{n}, b {new int[m]} {}
  ~Y() { delete[] b; }
};

X x{5};
X *px = new X{5};
Y y{5, 10};
Y *py = new Y{5, 10};
X *xPtrToy = new  Y{5, 10};

delete px;
delete py;
delete xPtrToy; // memory leak, considered undefined behavior
```

Think about the deletion process:

1. destructor is run
2. object fields run their destructors
3. superclass destructor is run
4. space is reclaimed

Because the destructor of `X` is not virtual, the destructor of `Y` is never called when we delete `xPtrToy`. This leads to a memory leak because the memory allocated for `b` in `Y` is never freed.

**if there is a chance for a class to be sub-classed, declare destructor as virtual**
If a class should not be sub-classed, declare it as `final`:

### polymorphic arrays

```C++
struct vec2 {
  int x, y;
  vec2(int x, int y) : x{x}, y{y} {}
};

struct vec3 : vec2 {
  int z;
  vec3(int x, int y, int z) : vec2{x, y}, z{z} {}
};

void f(vec2 * a) {
  a[0] = vec2{7, 8};
  a[1] = vec2{9, 10};
}

vec3 myArray[2] = {{1, 2, 3}, {4, 5, 6}};

f(myArray); // compiles!
```

![Memory diagram of the issue](https://media.discordapp.net/attachments/1352286409003761715/1380199402500526161/image0.jpg?ex=6843025c&is=6841b0dc&hm=847e376378a6e580c915b6b527c8ca1a229734a7e5dbae719fb3f1d9b084a585&=&format=webp)

There are 2 implicit conversions happening here:

- Arrays can become a pointer by taking the address of the first element.
- `Vec3 *` can be converted to `Vec2 *` because `Vec3` is a subclass of `Vec2`.

**Don't use arrays to objects polymorphically, instead use arrays of pointers to objects**

## Pure virtual methods

```C++
class Shape {
public:
  virtual float area() const;
};

class Square : public Shape {
  float length;
  public:
  float area() const override {
    return length * length;
  }
};

class Circle : public Shape {
  float radius;
  public:
  float area() const override {
    return 3 * radius * radius;
  }
};
```

The above program won't compile because `Shape` has a non-implemented method `area()`.
We could give it a dummy value like `0`, but this won't prevent the user from calling `area()` on a `Shape` object, which is not what we want.
