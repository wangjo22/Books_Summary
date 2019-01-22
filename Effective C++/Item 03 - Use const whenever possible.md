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


Having a function return a constant value is generally inappropriate, but sometimes doing so can reduce the incidence of client errors without giving up safety or efficiency. 
```c++
    class Rational {...};
    `const` Rational operator*(const Rational& lhs, const Rational& rhs);
```
Many programmers might think why should the result of operator* be a `const` object? Because if it weren't, clients would be able to commit atrocities like this:
```c++
    Rational a, b, c;
    ...
    (a * b) = c;        // Invoke operator= on the result of a * b
```
Many programmers have tried to do it without wanting to. All it takes is a simple type (and a type that can be implicitly converted to bool): 
```c++
    if(a * b = c)...    // Oops, meant to do a comparison!
```
Declaring `operator*`'s return value `const` prevents it, and that's why it's the right thing todo in this case. It can save you from annoying errors such as the "I meant to type '==' but I accidently typed '='" mistake we just saw. 


### `const` Member Functions
The purpose of `const` on member functions is to identify which member functions may be invoked on `const` objects. Such member functions are important for two reasons.
1. They make the interface of a class easier to understand. 
2. They make it possible to work with `const` objects. That's a critical aspect of writing efficient code. One of the fundamental ways to improve a C++ program's performance is to pass objects by reference-to-const. That technique is viable only if there are `const` member functions with which to manipulate the resulting const-qualified objects. 


Many people overlook the fact that member functions differing only in their constness can be overloaded, but this is an important feature of C++. Consider a class for representing a block of test:
```C++
    class TextBlock
    {
    public:
        const char& operator[](std::size_t position) const
        {
            return text[position];             // operator[] for const objects
        }

        char& operator[](std::size_t position) 
        {
            return text[position];              // operator[] for non-const objects
        }
    }

    private:
        std::string text;
    };
    ...

    TextBlock tb("Hello");
    std::cout << tb[0];             // Calls non-const TextBlock::operator[]

    const TextBlock ctb("World");
    std::cout << ctb[0]             // Calls const TextBlock::operator[]
```
Incidentally, `const` objects most often arise in real programs as a result of being passed by pointer or reference-to-const. The example of `ctb` is artificial. This is more realistic:
```c++
    void print(const TextBlock& ctb)        // In this function, ctb is const
    {
        std::cout << ctb[0];                // Calls const TextBlock::operator[]
    }
```
By overloading operator[] and giving the different versions different return types, you can have `const` and `non-const` `TextBlocks` handled differently:
```c++
    std::cout << tb[0];         // Fine - reading a non-const TextBlock
    tb[0] = 'x';                // Fine - writing a non-const TextBlock
    std::cout << ctb[0];        // Find - reading a const TextBlock
    ctb[0] = 'x';               // Error!
```
Note that the error here has only todo with the return type of operator[] that is called; the calls to operator[] themselves are all fine. The error arises out of an attempt to make an assignment to a `const char&`, because that's the return type from the `const` version of operator[].


## Thisgs to Remember
* Declaring something `const` helps compilers detect usage errors. `const` can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
* Compilers enforce bitwise constness, but you should program using logical constness.
* When `const` and non-`const` member functions have essentially identical implementations, code duplication can be avoided by having non-`const` version call the `const version.