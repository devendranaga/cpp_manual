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
