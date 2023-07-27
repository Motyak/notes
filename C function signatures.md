
# My definition of terms I use

## variable
A memory location in which we can store data. Either a scalar, an array or a pointer.

## scalar
A variable that contain a single unit of data (as opposed to an array), also not a pointer.

## array
A variable that may contain multiple units of data of a same type. Has a known fixed size.

## pointer
A variable that holds an address of a variable.

# SIMPLE

```c
/* pass an int as input */
(int) <-- (scalar) // pass by value

/* pass an array of ints as input */
(const int*, size_t) <-- (array, scalar)

/* pass an int as input/output */
(int*) <-- (&scalar) // pass by reference

/* pass an array of ints as input/output */
(int*, size_t) <-- (array, scalar)



/* pass an int pointer as input */
(const int*) <-- (pointer)

/* pass an array of int pointers as input */
(const int* const *, size_t) <-- (array, scalar)

/* pass an int array pointer as input */
(const int* const *) <-- (pointer)

/* pass an int pointer as input/output */
(const int**) <-- (&pointer)

/* pass an array of int pointers as input/output */
(const int* const **, size_t) <-- (array, scalar)

/* pass an int array pointer as input/output */
(const int* const **) <-- (&pointer)
```


# EXTRA (specific use cases, mostly mistakes)

Careful on these signatures if you use them without knowing why, make sure it makes sense.

```c
/* pass an array of int pointers as input, BUT you are allowed to mutate int variables value as well */
(int* const *, size_t)

/* pass an int pointer as input/output AND you are allowed to mutate int variable value as well */
(int**)

/* pass an array of int pointers as input/output AND you are allowed to mutate int variables value as well */
(int**, size_t)



/* pass an int pointer pointer as input, BUT you are allowed to mutate int variable value as well */
(int* const *)
```


# DUMB "good" practice (don't do it)

Placing the const modifier after pointer types in functions prototype :

```c
/* Following the "good" practice */
void function(int* const var) {
    var = some_int_address; // compilation error
}

/* NOT following the "good" practice */
void function(int* var) {
    var = some_int_address; // OK but probably NOT what you intended to do
}
```

This is in case one would forget that passing an int pointer by value may allow you to mutate the pointed int variable (by dereferencing the pointer) but mutating the pointer variable itself would make no sense other than using the function local variable as a working variable, since it's a COPY of the actual pointer variable.

With the final const modifier, there is a compiler error saying you cannot mutate the pointer value (due to the const specifier that now prevents it).

The final const modifier isn't part of the signature (only in the prototype), so the user of the functions you declare won't see the difference. He knows the pointer itself cannot be modified anyway because IT'S COPIED.

# Code

## `(int) <-- (scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(int inner) {
    inner = 91;
} 

int main()
{
    int outer = 10;
    printf("%d\n", outer); // 10
    function(outer);
    printf("%d\n", outer); // 10
}
```

## `(const int*, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* inner, size_t) {
    int* new_arr = malloc(sizeof(int) * 2);
    new_arr[0] = 20;
    new_arr[1] = 21;
    inner = new_arr;
} 

int main()
{
    int outer[2];
    outer[0] = 10;
    outer[1] = 11;
    printf("%p: %d %d\n", 
        outer, outer[0], outer[1]); // 0x56013b16a2a0: 10 11
    as_is(outer, 2);
    printf("%p: %d %d\n", 
        outer, outer[0], outer[1]); // 0x56013b16a2a0: 10 11
}
```

```c
#include <stdlib.h>

void deref(const int* inner, size_t) {
    inner[0] = 777; // compilation error
} 
```

## `(int*) <-- (&scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(int* inner) {
    int* new_scalar_address = malloc(sizeof(int));
    inner = new_scalar_address;
} 

int main()
{
    int outer;
    printf("%p: %d\n", &outer); // 0x7ffcfe838944
    as_is(&outer);
    printf("%p: %d\n", &outer); // 0x7ffcfe838944 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(int* inner) {
    *inner = 777;
}

int main()
{
    int outer = 0;
    printf("%d\n", *outer); // 0
    deref(outer);
    printf("%d\n", *outer); // 777
}
```

## `(int*, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(int* inner, size_t) {
    int* new_array = malloc(sizeof(int) * 5);
    inner = new_array;
} 

int main()
{
    int outer[2];
    printf("%p\n", outer); // 0x7ffe43dac4e0
    as_is(outer, 2);
    printf("%p\n", outer); // 0x7ffe43dac4e0 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(int* inner, size_t) {
    inner[0] = 777;
} 

int main()
{
    int outer[2];
    outer[0] = 10;
    outer[1] = 11;
    printf("%p: %d %d\n", 
        outer, outer[0], outer[1]); // 0x56013b16a2a0: 10 11
    as_is(outer, 2);
    printf("%p: %d %d\n", 
        outer, outer[0], outer[1]); // 0x56013b16a2a0: 777 11
}
```



## `(const int*) <-- (pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* inner) {
    int* new_pointer = malloc(sizeof(int));
    inner = new_pointer;
} 

int main()
{
    int* outer = malloc(sizeof(int));
    printf("%p\n", outer); // 0x7ffe43dac4e0
    as_is(outer);
    printf("%p\n", outer); // 0x7ffe43dac4e0 (same)
}
```

```c
#include <stdlib.h>

void deref(const int* inner) {
    *inner = 777; // compilation error
} 
```

## `(const int* const *, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* const *inner, size_t) {
    int** new_array = malloc(sizeof(int*) * 5);
    inner = new_array;
} 

int main()
{
    int* outer[2];
    outer[0] = malloc(sizeof(int));
    printf("%p\n", outer[0]); // 0x55a8a7c812a0
    as_is(outer, 2);
    printf("%p\n", outer[0]); // 0x55a8a7c812a0 (same)
}
```

```c
#include <stdlib.h>

void deref(const int* const *inner, size_t) {
    int* new_scalar_address = malloc(sizeof(int));
    inner[0] = new_scalar_address; // compilation error
} 
```

```c
#include <stdlib.h>

void deref_deref(const int* const *inner, size_t) {
    *inner[0] = 777; // compilation error
} 
```

## `(const int* const *) <-- (pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* const *inner, size_t) {
    int** new_pointer = malloc(sizeof(int*) * 5);
    inner = new_pointer;
} 

int main()
{
    int (*outer)[2];
    outer = malloc(sizeof(int*) * 2);
    printf("%p\n", outer); // 0x5626729202a0
    as_is(outer, 2);
    printf("%p\n", outer); // 0x5626729202a0 (same)
}
```

```c
#include <stdlib.h>

void deref(const int* const *inner, size_t) {
    int* new_array = malloc(sizeof(int) * 5);
    *inner = new_array; // compilation error
}
```

```c
#include <stdlib.h>

void deref_deref(const int* const *inner, size_t) {
    (*inner)[0] = 777; // compilation error
} 
```

## `(const int**) <-- (&pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int** inner) {
    int** new_pointer_address = malloc(sizeof(int*));
    inner = new_pointer_address;
} 

int main()
{
    int* outer = malloc(sizeof(int));
    printf("%p\n", outer); // 0x5632391922a0
    as_is(&outer);
    printf("%p\n", outer); // 0x5632391922a0 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(const int** inner) {
    int* new_pointer = malloc(sizeof(int));
    *inner = new_pointer;
} 

int main()
{
    int* outer = malloc(sizeof(int));
    printf("%p\n", outer); // 0x56200de2f2a0
    deref(&outer);
    printf("%p\n", outer); // 0x56200de2f6d0
}
```

```c
#include <stdlib.h>

void deref_deref(const int** inner) {
    **inner = 777;
}
```

## `(const int* const **, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* const **inner, size_t) {
    int*** new_array_address = malloc(sizeof(int**));
    *new_array_address = malloc(sizeof(int*) * 5);
    inner = new_array_address;
} 

int main()
{
    int* outer[2];
    outer[0] = malloc(sizeof(int));
    printf("%p\n", outer[0]); // 0x55a8a7c812a0
    as_is(outer, 2);
    printf("%p\n", outer[0]); // 0x55a8a7c812a0 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(const int* const **inner, size_t) {
    int** new_array = malloc(sizeof(int*) * 5);
    *inner = new_array;
} 

int main()
{
    int* outer[2];
    outer[0] = malloc(sizeof(int));
    printf("%p\n", outer[0]); // 0x557af8e1b2a0
    deref(outer, 2);
    printf("%p\n", outer[0]); // 0x557af8e1b6d0 (same)
}
```

```c
#include <stdlib.h>

void deref_deref(const int* const **inner, size_t) {
    int* new_pointer = malloc(sizeof(int));
    (*inner)[0] = new_pointer;
}
```

```c
#include <stdlib.h>

void deref_deref_deref(const int* const **inner, size_t) {
    *(*inner)[0] = 777;
} 
```

## `(const int* const **) <-- (&pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* const **inner) {
    int*** new_pointer_address = malloc(sizeof(int**));
    *new_pointer_address = malloc(sizeof(int*) * 5);
    inner = new_pointer_address;
} 

int main()
{
    int (*outer)[2];
    outer = malloc(sizeof(int*) * 2);
    printf("%p\n", outer); // 0x555abd5332a0
    as_is(&outer);
    printf("%p\n", outer); // 0x555abd5332a0 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(const int* const **inner) {
    int** new_pointer = malloc(sizeof(int*) * 5);
    *inner = new_pointer;
} 

int main()
{
    int (*outer)[2];
    outer = malloc(sizeof(int*) * 2);
    printf("%p\n", outer); // 0x555abd5332a0
    deref(&outer);
    printf("%p\n", outer); // 0x555abd5332a0 (same)
}
```

```c
#include <stdlib.h>

void deref_deref(const int* const **inner) {
    int* new_array = malloc(sizeof(int) * 2);
    **inner = new_array; // compilation error
}
```

```c
#include <stdlib.h>

void deref_deref_deref(const int* const **inner) {
    (**inner)[0] = 777; // compilation error
}
```

