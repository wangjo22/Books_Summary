# Item 2: Prefer consts, enums, and inlines to #defines.

```C++
#define ASPECT_RATIO 1.653
```
The name ASPECT_RATIO may not get entered into the symbol table. This can be