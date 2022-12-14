# CPP Manual

Few of the notes from personal projects, books, online material and trial and errors.

**auto**

`auto` provides type deduction if compiler can guess the type. For example when using range based for loops.

```cpp
for (auto it : vec) {
    // use it
}
```

the type `auto` here derives the constant iterator for it.

**decltype**

`decltype` specifier is used when deriving complex types. An example of it is as follows.

```cpp
template <typename P, typename Q>
auto add(P p, Q q) -> decltype(p + q) {
	return p + q;
}
```

The above function derives return type of add from `p` and `q`.

For example calling

```cpp
auto val = add(1.1, 2);

printf("result %f\n", val);
```

is perfectly valid and compiler produces valid code. The resulting output will be in `double`.

## Classes

Class is a container of data and implementation. Data is in the form of variables pertianing to that class and implementation is a set of
functions that use and manipulate the data to produce some outcome.

declaring a class makes a class object. For example consider a `class test`.

```cpp
class test {
	public:
		void run();
		
	private:
		int timer;
};
```

declaring variable of type `test` eventually produces class object. `test t`, where `t` is the class object.


Delete copy and move constructors, copy and move assignment operators when creating any static functions / variables within the class. Keeping them and having the default copy and assignment operators can produce undefined behavior.

For example, when a singleton class to be defined and used, it is done as follows.

```cpp

class singleton {
    public:
        ~singleton() = default;
	singleton(singleton &) = delete;
	singleton &operator=(const singleton &) = delete;
	singleton(singleton &&) = delete;
	singleton &&operator=(const singleton &&) = delete;
	
	static singleton *instance() {
	    static singleton s;
	    return &s;
	}
	
    private:
        explicit singleton() = default;
};

```

A class constructor can call another constructor with in the same class.

```cpp
#include <iostream>

class example {
    public:
        example() {
            printf("Example class is called\n");
        }
        example(int f) : example() {
            f_ = f;
            printf("Example class with value %d is called\n");
        }

    private:
        int f_;
};

int main()
{
    example e(1);
}
```

**Enum class**

Enum class is defined as

```cpp
enum class fruits {
	APPLE,
	ORANGE,
};
```

To define a enumerated types,

```cpp
fruits f = fruits::APPLE;
```

If an enumerated type in C++ assigned to a string, it results in compiler error of assigning integer to a string type.

```cpp
enum class log_type {
	LOG_INFO = "Info",
};
```

Enum comparison can be directly performed with the `==` operator.

But surprisingly an integer to enum type comparison cannot be done as its not the same types literally.

### Operator Overloading

C++ Polymorphism provides operator overloading, which allows us to write programs by using same operators but logically between class objects etc.

Below program demonstrates operator overloading.

```cpp
#include <iostream>

class operator_example {
    public:
        explicit operator_example(int val) : val_(val) { }
        ~operator_example() = default;

        operator_example operator+(const operator_example &rhs) {
            operator_example o(0);
            o.val_ = this->val_ + rhs.val_;
            return o;
        }
        operator_example operator-(const operator_example &rhs) {
            operator_example o(0);
            o.val_ = this->val_ - rhs.val_;
            return o;
        }
        operator_example operator/(const operator_example &rhs) {
            operator_example o(0);
            o.val_ = this->val_ / rhs.val_;
            return o;
        }
        operator_example operator*(const operator_example &rhs) {
            operator_example o(0);
            o.val_ = this->val_ * rhs.val_;
            return o;
        }

        /* Prefix increment operator */
        void operator++() { this->val_ ++; }

        /* Prefix decrement operator */
        void operator--() { this->val_ --; }

        /* Postfix increment operator */
        void operator++(int) { this->val_ ++; }

        /* Postfix decrement operator */
        void operator--(int) { this->val_ --; }
        int operator()() { return this->val_; }

    private:
        int val_;
};

int main()
{
    operator_example o1(2), o2(1);
    operator_example o3 = o1 + o2;
    operator_example o4 = o1 - o2;
    operator_example o5 = o1 / o2;
    operator_example o6 = o1 * o2;

    ++o6;
    --o5;

    o6++;
    o5--;

    printf("o1 %d o2 %d o3 %d o4 %d o5 %d o6 %d\n",
            o1(), o2(), o3(), o4(), o5(), o6());
}
```


**const**

Be aware that `const` qualifier used to represent a constant value that never changes during the course of a program after the initialization.

Below is one example of how weird `const` can get. Not sure of the usecase of the below snippet but very much works.

```cpp
#include <iostream>
#include <stdlib.h>

int modify(int val)
{
    return val * 10;
}

int main(int argc, char **argv)
{
    int i = 10;

    i = modify(atoi(argv[1]));

    const int b = i;

    printf("%d %d\n", i, b);
}

```

i and b gives the same values. Any change to b directly after assigned to i, results in compiler errors.

For obvious reasons below vector definition can use `const`. However, g++ with `std=c++17` does not complain about it could be a const. However, underlying optimization might as well been performed during the compilation stage.

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4};

    for (auto i : v) {
        printf("%d\n", i);
    }
}

```

**nullptr**

The `nullptr` constant is introduced to avoid confusion between using `NULL` and `0` when doing operator overloading.

In `stdio.h` the constant `NULL` translates to a macro `#define NULL ((void *)0)`. Thus when doing the following,

```c
void set(char *ptr);
void set(int val);
```

The compiler may not be able to distinguish between the two `set` functions.

So whenever a pointer needs to be set to `NULL` use `nullptr`.


**constexpr**

Few rules:

1. `constexpr` variables must only get values from functions that named with `constexpr`. Otherwise compiler errors.

For example, if `square` is not declared as `constexpr` type, then compiler would throw an error.

```cpp
#include <iostream>

static constexpr double square(double v)
{
    return v * v;
}

int main()
{
    const int v = 10;
    int val = 10;

    constexpr double v2 = square(v);
    constexpr double val2 = square(val);
}

```

Notice that value to the `square` is not set as `const`. However, the result must always be a constant when declaring / using `constexpr`. And so the compiler will now complain again on the last element that the result will vary because passed argument `val` is not a `const`.

**Vectors**

Elements of a vector can be referenced with the `&` while iterating through them. For an example,

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v = {1,2,3,4};

    for (auto &x: v) {
        x ++;
    }

    for (auto x : v) {
        printf("%d\n", x);
    }
}

```

Adding an element in vector can be either used with `push_back` or `emplace_back`.

```cpp
#include <iostream>
#include <vector>

int main(int argc, char **argv)
{
    std::vector<std::string> strs;

    for (auto i = 1; i < argc; i ++) {
        strs.emplace_back(argv[i]);
    }

    for (auto p : strs) {
        printf("%s\n", p.c_str());
    }
}

```

Initializer lists are one of another features that when sometimes vector is requried but not necessarily need to be created. For example during program
or parameter initialization.

```cpp
#include <iostream>
#include <vector>
#include <initializer_list>

void sort(std::vector<int> l)
{
    for (auto i : l) {
        printf("%d\n", i);
    }
}

int main()
{
    std::initializer_list<int> l {1, 2, 3};

    sort(l);
}
```

The above `std::initializer_list` is auto converted into `std::vector` soon after being passed.

One issue is that if sort was taking a reference of the vector, then passing `initializer_list` to it will result in compiler error.

**References and Pointers**

1. Pointers Mostly harmless in C++ because every one uses managed memory allocators, unless they write low level code dealing with buffers or packets. Using reference or pointer doesn't really matter much.
2. However, passing references for variables of `shared_ptr` or `vector` etc would save a lot of time deducing how to deref them properly.

An example is that,

```cpp
void process_req(std::shared_ptr<request> &r);
```

In the below call,

```cpp
int string_copy(const char *src, char val[]);
int string_copy(const char *src, char *val);
```

Both calls are one and the same, since C++ treats `val[]` and `*val` as same.

The below code will copy `val` by reference and sets the value of `src` into `val`.

```cpp

#include <iostream>

void string_set(const char *str, char val[])
{
    int i = 0;

    for (i = 0; str[i] != '\0'; i ++) {
        val[i] = str[i];
    }

    val[i] = '\0';
}

int main()
{
    const char *src = "test";
    char val[10];

    string_set(src, val);
    printf("%s\n", val);
}

```

**Inheritance**

Concept of Inheritance is mostly to write reusable code. Below is one example of inheritance.

```cpp

class transport {
    public:
        explicit transport() { }
        ~transport() { }
        
        int write_to_transport_conn(int sock, char *data, int data_len) {
            return send(sock, data, data_len, 0);
        }
        
        int read_from_transport_conn(int sock, char *data, int data_len) {
            return recv(sock, data, data_len, 0);
        }
};

class file_transport : public transport {
    public:
        explicit file_transport() { }
        ~file_transport() { }
        
        int write(int sock, char *data, int data_len) {
            return write_to_transport_conn(sock, data, data_len);
        }
        
        int read(int sock, char *data, int data_len) {
            return read_from_transport_conn(sock, data, datalen);
        }
};


```

during the inheritance, destructors must be set as `virtual` so that the destructors of the derived class can be called upon.

For example, take a look at the below program.

```cpp
#include <iostream>
#include <vector>

class base {
    public:
        explicit base() = default;
        ~base() = default;

        virtual void print() { printf("hello\n"); }
};

class derived : public base {
    public:
        explicit derived() {
            ptr = new int;

            *ptr = 4;
        }
        ~derived() {
            delete ptr;
        }
        virtual void print() override
        {
            printf("%d\n", *ptr);
        }
    private:
        int *ptr;
};

int main()
{
    base *b = new derived;

    b->print();

    delete b;
}

```

Although `delete` called to destroy the base and derived classes, the destructor of `derived` never gets called, thus resulting in a 
leak of memory.

Thus, destructors must be declared with virtual to hint the compiler to call all of the destructors. After changing the destructors to virtual, the above code looks as follows.

```cpp
#include <iostream>
#include <vector>

class base {
    public:
        explicit base() = default;
        virtual ~base() = default;

        virtual void print() { printf("hello\n"); }
};

class derived : public base {
    public:
        explicit derived() {
            ptr = new int;

            *ptr = 4;
        }
        virtual ~derived() {
            delete ptr;
        }
        virtual void print() override
        {
            printf("%d\n", *ptr);
        }
    private:
        int *ptr;
};

int main()
{
    base *b = new derived;

    b->print();

    delete b;
}

```



So the base class `transport` being inherited by derived class `file_transport` so that `file_transport` could reuse the functions within the base class `transport`. The destructor sequence is always from derived class to the base.

Consider a multi-inheritance case below, `base` class is derived by `derived_1` and `derived_1` is derived by `derived_2`.

```cpp
#include <iostream>

class base {
    public:
        explicit base() = default;
        virtual ~base() {
            printf("base\n");
        }
};

class derived_1 : public base {
    public:
        explicit derived_1() = default;
        virtual ~derived_1() {
            printf("derived_1\n");
        }
};

class derived_2 : public derived_1 {
    public:
        explicit derived_2() = default;
        virtual ~derived_2() {
            printf("derived_2\n");
        }
};

int main()
{
    base *d = new derived_2;

    delete d;
}

```

The calling sequence of destructor is in reverse order from `derived_2 -> derived_1 -> base`.

**Function overriding**

Function overriding of the base class is possible in the derived class. The below code uses `override` keyword to override the
`print` member function of base class. or the inherited function can be declared `virtual`.

```cpp
#include <iostream>

class base {
    public:
        explicit base() = default;
        virtual ~base() = default;

        virtual void print() {
            printf("base\n");
        }
};

class derived : public base {
    public:
        explicit derived() = default;
        virtual ~derived() = default;

        virtual void print() override {
            printf("derived\n");
        }
};

int main()
{
    derived d;

    d.print();
}

```

To call the `print` of the base class, `base::print()` can be used.

When overriding the member functions within derived class, the base class member functions shall always be present. If the member functions
of derived class has defined as `override` and base do not have the same function, it results in compiler error.

The below code will not compile.

```cpp
#include <iostream>

struct base {
    public:
        virtual void set(int val) { v = val; }
    private:
        int v;
};

class derived : public base {
    public:
        virtual void set(int val) override { v = val; }
        virtual void set(double val) override { }

    private:
        int v;
};

int main()
{
    derived d;
}
```

So remove override if a virtual member function of a derived class is not present in the base class.

The below example also provide a calling convention of derived to base class.

```cpp
#include <iostream>

struct base {
    public:
        virtual void set(int val) { v = val; }
        virtual int get_int() { return v; }
    private:
        int v;
};

class derived : public base {
    public:
        virtual void set(int val) override { v = val; }
        virtual void set(double val) { v_d = val; }
        virtual int get_int() override { return v; }
        double get_double() { return v_d; }

    private:
        int v;
        double v_d;
};

int main()
{
    derived d;

    d.base::set(1);
    d.set(1);
    d.set(1.1);

    printf("%d %d %f\n", d.base::get_int(), d.get_int(), d.get_double());
}
```

Calling a base class from the derived class is basically `derived_obj.base_class::method`.


**Structure bindings**

Structure bindings are useful when returning structures directly as constants or when hiding structures.

for example,

```cpp
#include <iostream>

struct s {
	int a;
	std::string val;
};

s get_struct()
{
        // return as tuple packed in the structure
		return s{10, "val"};
}

int main()
{
	s a;

	a = get_struct();
	printf("%d %s\n", a.a, a.val.c_str());

    // unpack the received structure into the variables
	auto [_a, _val] = get_struct();
	printf("%d %s\n", _a, _val.c_str());
}

```

For example, when capturing the structure members with incorrect number of arguments result in compilation error.

```cpp
#include <iostream>

struct a {
	int a;
	char p;
	double v;
};

a get_struct()
{
	return a{1, 'p', 1.0};
}

int main()
{
	auto[_a, _b] = get_struct();
	printf("%d %c %f\n", _a, _b);
}

```

`std::map` can be iterated through as a tuple:

```cpp
#include <iostream>
#include <string>
#include <map>

int main()
{
	std::map<int, std::string> var_str_map = {
		{1, "apple"},
		{2, "lemon"},
		{3, "orange"},
		{4, "banana"},
	};

	for (const auto &[key,val] : var_str_map) {
		printf("key: %d val: %s\n", key, val.c_str());
	}
}

```

Structure binding with multiple elements with iterator.

```cpp
#include <iostream>
#include <vector>

struct book {
    int book_id;
    std::string author;
    double price;
};

int main()
{
    std::vector<book> books = {
        {1, "Stephen King", 100},
        {2, "Stephen King", 120},
    };

    for (auto &[id, author, price] : books) {
        printf("%d %s %f\n", id, author.c_str(), price);
    }
}
```

**function templates**

Function templates are really useful when writing generic code.

For example,

```cpp
#include <iostream>

template <typename T>
T max(T a, T b)
{
    return (a > b)? a: b;
}

template <typename T>
T min(T a, T b)
{
    return (a < b)? a: b;
}

int main()
{
    int a = 4;
    int b = 3;
    int max_val = 0;
    int min_val = 0;

    max_val = max(a, b);
    min_val = min(a, b);

    printf("max %d min %d\n", max_val, min_val);
}

```

Below code taken from : http://users.cis.fiu.edu/~weiss/Deltoid/vcstl/templates


```cpp
template <class T>
class Z
{
  public:
Z() {} ;
~Z() {} ;
void f(){} ;
void g(){} ;
} ;

int main()
{
Z<int> zi ;   //implicit instantiation generates class Z<int>
Z<float> zf ; //implicit instantiation generates class Z<float>
return 0 ;
}

````

See that the `zi` and `zf` are implictly instantiated.

once a call is made to `Z::f()` or `Z::g()` the code generation happens.

### Templates with non-type parameters:

```cpp
template <typename T, size_t N>
class array {
	private:
		T array_[N];
};

```

The above template is one example of template with non-type parameters. This is one of the examples of `Array` abstractions.

```cpp
array<int, 6> a;
```

defines an array of 6 integers.

**Variable Argument Templates**

Variable argument templates can be written with `typename... T`. The function arguments define the following way,

```cpp
template <typename... T>
void get_args(T... args)
```

To get number of arguments use `sizeof...(args)`.

```cpp
#include <iostream>

template <typename... T>
void get_args(T... args)
{
    printf("length %d\n", sizeof...(args));
}

int main()
{
    get_args(1);
    get_args(1, "test");
}
```

With the knowledge of the templates, we can write variable argument `print` function.

```cpp
#include <iostream>

enum class log_type {
    LOG_INFO,
    LOG_WARN,
    LOG_ERROR,
};

template <typename... T>
void log_msg(log_type t, T... args)
{
    switch (t) {
        case log_type::LOG_INFO:
            fprintf(stderr, "Info: ");
            fprintf(stderr, args...);
        break;
        case log_type::LOG_WARN:
            fprintf(stderr, "Warn: ");
            fprintf(stderr, args...);
        break;
        case log_type::LOG_ERROR:
            fprintf(stderr, "Error: ");
            fprintf(stderr, args...);
        break;
        default:
            return;
    }
}

int main()
{
    log_msg(log_type::LOG_INFO, "hello info\n");
    log_msg(log_type::LOG_WARN, "hello warn\n");
    log_msg(log_type::LOG_ERROR, "hello error\n");
}
```


## Design Patterns

### 1. Singleton design pattern

Singleton pattern is useful when a class needs to be accessed across various other classes / places. We just need not create an object to it, but get an
instance and use it everywhere in need. In order to do this, we make a static function within the class and hide the constructor so no more duplicate objects are created.

```cpp

class singleton {
	public:
		static singleton *instance() {
			static singleton s;
			return &s;
		}
		~singleton() { }
		singleton(const singleton &) = delete;
		const singleton &operator=(const singleton &) = delete;
		singleton(const singleton &&) = delete;
		const singleton &&operator=(const singleton &&) = delete;
		
		void function_1();
	private:
		explicit singleton() { }
};

```

Here the instance is created once and the object is obtained via the call `singleton *s = singleton::instance();`.

Any other member functions can be references with using the object `s`.

Singletons are generally used in cases such as configuration data, store / write to databases, loggers.

### 2. Builder pattern

Builder pattern is useful when initializing a series of steps together before program enters in execution.
Method behind is the return a reference / pointer of the class itself.
The caller utilizes this to repeatedly call the initializers.

```cpp

class builder {
	builder &setup_a();
	builder &setup_a2();
	builder &setup_a3();
	builder &setup_a4();
	
	void exec();
};

builder & builder::setup_a()
{
    // .. series of operations ..
    return *this;
}

builder b;

b.setup_a().setup_a2().setup_a3().setup_a4();

b.exec();

```


# Samples

### Implementing an Array


```cpp

template <typename T, size_t N>
class array {
	public:
		T &operator[](size_t idx) { return array_[idx]; }
		T at(size_t idx) { return array_[idx]; }
		size_t size() { return N; }
	private:
		T array_[N];
};

```
