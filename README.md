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
