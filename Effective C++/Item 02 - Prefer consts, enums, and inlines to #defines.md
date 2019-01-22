## Item 2: Prefer const, enum, and inline to #define.
> Prefer the compiler to the preprocessor

```c++
#define ASPECT_RATIO 1.653
```
ASPECT_RATIO may not get entered into the symbol table. Then, the error message may refer to 1.653, not ASPECT_RATIO.

```C++
const double AspectRAtio = 1.653;
```
AspectRatio is definitely seen by compilers and is certainly entered into their symbol tables.
In the case of a floating point constant, use of the constant may yield smaller code than using a #define because the constant AspectRatio should never result in more than on copy. 

When replacing #defines with constants, two special cases are woth mentioning. 
1. Define constant pointers
```C++
const char * const authorName = "Scott Meyers";
```
string objects are generally preferable to their char* based progenitors, so authorName is often better defined this way:
```C++
const std::string authorName("Scott Meyers");
```

2. class-specific constants. 
To limit, the scope of a constant to a class, you must make it a member.
To ensure there's at most one copy of the constant, you must make it a static member
```C++
class GamePlayer 
{
private:
    static const int NumTurns = 5;    // Constant declaration
    int scores[NumTurns];
}
```
Usually, C++ requires that you provide a definition for anything you use, but class-specific constants that are static and of integral type(e.g., integers, chars, bools) are an exception. As long as you don't take their address, you can declare them and use them without providing a definition. 
If you do take the address of a class constant, or if your compiler incorrectly insists on a definition even if you don't take the address, you provide a seperate definition like this:
```C++
const int GamePlayer::NumTurns;     // definition of NumTurns
```
Put this in an implementation file, not a header file. Because the initial value of class constants is provided where the constant is declared(e.g., Numturns is initialized to 5 when it is declared), no initial value is permitted at the point of definition.

Using a #define, there's no way to create a class-specific constant because #define don't respsect scope. There is no such thing as a "private" #define. 



Older compilers may not accept the syntax above. In case where the above syntax can't be used, you put the initial value at the point of definition:
```C++
class CostEstimate
{
private:
    static const double FudgeFactor;    // Declaration of static class constnat; goes in header file
    ...
};

const double CostEstimate::FudgeFactor = 1.35;  // Definition of static class constant; goes in implementation file
```

When you need the value of a class constant during compilation of the class, such as the array GamePlayer::scores, then the accepted way to compensate for compilers that forbid the in-class specification of initial values for static integral class constants it to use "Enum".
```C++
class GamePlayer
{
private:
    enum 
    {
        NumTurns = 5
    };
    
    int scores[numTurns];
    ...
}
```
The enum hack is worth knowing about for several reasons. 
1. It behaves in some ways more like a #define than a const does, and sometimes that's what you want. 
Like #define, it's not leagal to take the address of an enum and enums never result in unnecessary memory allocation.
2. It is purely pragmatic.

Another common isuse of the #define is using it to implement macros that look like functions but that don't incur the overhead of a function call. 
```C++
#define CALL_WITH_MAX(a, b)  f((a) > (b) ? (a) : (b))   // call f with the maximum of a and b
```
Whenever you write this kind of macro, you have to remember to parenthesize all the arguments in the macro body. Even if you get that right, weird things can happen:
```C++
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);        // a is incremented twice
CALL_WITH_MAX(++a, b + 10);   // a is incremented once
```

You can get all the efficiency of a macro plus all the predictable behavior and type safety of a regular function by using a template for an inline function:
```C++
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
    f(a > b ? a : b);       // Because we don't know what T is, we pass by reference to const
}
```
There's no need to parenthesize parameters inside the function body, no need to worry about evaluating parameters multiple times.




# Thisgs to Remember
## For simple constants, prefer const objects or enums to #defines.
## For function-like macros, prefer inline functions to #defines.