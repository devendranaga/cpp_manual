# CPP Manual

Few of the notes from personal projects, books, online material and trial and errors.

**const**

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

So the base class `transport` being inherited by derived class `file_transport` so that `file_transport` could reuse the functions within the base class `transport`.
