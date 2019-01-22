# Item 3: Use `const` whenever possible

The `const` keyword is remarkably versatile. 
For pointers, you can specify whether the pointer itself is `const`, the data it points to is `const`, both, or neither.
```C++
    char greeting[] = "Hello";
    char *p = greeting;                 // Non-const pointer, non-const data
    const char *p = greeting;           // Non-const pointer, const data
    char * const p = greeting;          // Const pointer, non-const data
    const char * const p = greeting;    // Const pointer, const data
```

`STL` iterators are modeled on pointers, so an iterator acts much like a `T*` pointer. Declaring an iterator `const` is like declaring a pointer `const`(i.g., `T* const pointer`): the iterator isn't allowed to point to something different, but the thing it points to may be modified. If you want an iterator that points to something that can't be modified (i.g., `const T* pointer`), you want a `const_iterator`.
```c++
    std::vector<int> vec;
    ...
    const std::vector<int>::iterator iter = vec.begin();        // iter acts like a T* const
    *iter = 10;                                                 // OK, changes what iter points to
    ++iter;                                                     // Error! iter is const

    std::vector<int>::const_iterator clter = vec.begin();       // clter acts like a const T*
    *clter = 10;                                                // Error! *clter is const
    ++clter;                                                    // Fine, changes clter
```



## Thisgs to Remember
* For simple constants, prefer const objects or enums to #defines
* For function-like macros, prefer inline functions to #defines