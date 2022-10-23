# CPP Manual

Few of the notes from personal projects, books, online material and trial and errors.

## Classes

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

**References and Pointers**

1. Pointers Mostly harmless in C++ because every one uses managed memory allocators, unless they write low level code dealing with buffers or packets. Using reference or pointer doesn't really matter much.
2. However, passing references for variables of `shared_ptr` or `vector` etc would save a lot of time deducing how to deref them properly.

An example is that,

```cpp
void process_req(std::shared_ptr<request> &r);
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
