
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

/* pass an int as input/output */
(int*) <-- (&scalar) // pass by reference


/* pass an array of ints as input */
(const int*, size_t) <-- (array, scalar)

/* pass an array of ints as input/output */
(int*, size_t) <-- (array, scalar)


/* pass an int pointer as input */
(const int*) <-- (pointer)

/* pass an int pointer as input/output */
(const int**) <-- (&pointer)


/* pass an array of int pointers as input */
(const int* const *, size_t) <-- (array, scalar)

/* pass an array of int pointers as input/output */
(const int**, size_t) <-- (array, scalar)


/* pass an int array pointer as input */
(const int* const *) <-- (pointer)

/* pass an int array pointer as input/output */
(const int**) <-- (&pointer)
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
    as_is(outer);
    printf("%d\n", outer); // 10
}
```

## `(int*) <-- (&scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(int* inner) {
    int* new_scalar_address = malloc(sizeof(int));
    *new_scalar_address = 777;
    inner = new_scalar_address;
} 

int main()
{
    int outer = 10;
    printf("%d\n", outer); // 10
    as_is(&outer);
    printf("%d\n", outer); // 10
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
    int outer = 10;
    printf("%d\n", outer); // 10
    deref(&outer);
    printf("%d\n", outer); // 777
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
    printf("%d %d\n", 
        outer[0], outer[1]); // 10 11
    as_is(outer, 2);
    printf("%d %d\n", 
        outer[0], outer[1]); // 10 11
}
```

```c
#include <stdlib.h>

void deref(const int* inner, size_t) {
    inner[0] = 20; // compilation error
    inner[1] = 21; // ..
} 
```

## `(int*, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(int* inner, size_t) {
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
    printf("%d %d\n", 
        outer[0], outer[1]); // 10 11
    as_is(outer, 2);
    printf("%d %d\n", 
        outer[0], outer[1]); // 10 11
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(int* inner, size_t) {
    inner[0] = 20;
    inner[1] = 21;
} 

int main()
{
    int outer[2];
    outer[0] = 10;
    outer[1] = 11;
    printf("%d %d\n", 
        outer[0], outer[1]); // 10 11
    deref(outer, 2);
    printf("%d %d\n", 
        outer[0], outer[1]); // 20 21
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

## `(const int**) <-- (&pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int** inner) {
    int* new_pointer = malloc(sizeof(int));
    inner = &new_pointer;
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
    printf("%p\n", outer); // 0x56176a8562a0
    deref(&outer);
    printf("%p\n", outer); // 0x56176a8566d0
}
```

```c
#include <stdlib.h>

void deref_deref(const int** inner) {
    **inner = 777;
}
```

## `(const int* const *, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* const *inner, size_t) {
    int** new_arr = malloc(sizeof(int*) * 2);
    new_arr[0] = malloc(sizeof(int));
    new_arr[1] = malloc(sizeof(int));
    inner = new_arr;
} 

int main()
{
    int* outer[2];
    outer[0] = malloc(sizeof(int));
    outer[1] = malloc(sizeof(int));
    printf("%p %p\n", 
        outer[0], outer[1]); // 0x564e10c0b2a0 0x564e10c0b2c0
    as_is(outer, 2);
    printf("%p %p\n", 
        outer[0], outer[1]); // 0x564e10c0b2a0 0x564e10c0b2c0 (same)
}
```

```c
#include <stdlib.h>

void deref(const int* const *inner, size_t) {
    inner[0] = malloc(sizeof(int)); // compilation error
    inner[1] = malloc(sizeof(int)); // ..
}
```

```c
#include <stdlib.h>

void deref_deref(const int* const *inner, size_t) {
    *inner[0] = 20; // compilation error
    *inner[1] = 21; // ..
}
```

## `(const int**, size_t) <-- (array, scalar)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int** inner, size_t) {
    int** new_arr = malloc(sizeof(int*) * 2);
    new_arr[0] = malloc(sizeof(int));
    new_arr[1] = malloc(sizeof(int));
    inner = new_arr;
}

int main()
{
    int* outer[2];
    outer[0] = malloc(sizeof(int));
    outer[1] = malloc(sizeof(int));
    printf("%p %p\n", 
        outer[0], outer[1]); // 0x55d4ba2212a0 0x55d4ba2212c0
    as_is(outer, 2);
    printf("%p %p\n", 
        outer[0], outer[1]); // 0x55d4ba2212a0 0x55d4ba2212c0 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(const int** inner, size_t) {
    inner[0] = malloc(sizeof(int));
    inner[1] = malloc(sizeof(int));
}

int main()
{
    int* outer[2];
    outer[0] = malloc(sizeof(int));
    outer[1] = malloc(sizeof(int));
    printf("%p %p\n", 
        outer[0], outer[1]); // 0x562c9b3a42a0 0x562c9b3a42c0
    deref(outer, 2);
    printf("%p %p\n", 
        outer[0], outer[1]); // 0x562c9b3a46f0 0x562c9b3a4710
}
```

```c
#include <stdlib.h>

void deref_deref(const int** inner, size_t) {
    *inner[0] = 20; // compilation error
    *inner[1] = 21; // ..
}
```

## `(const int* const *) <-- (pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int* const *inner) {
    int* new_array = malloc(sizeof(int) * 5);
    inner = &new_array;
} 

int main()
{
    int (*outer)[2];
    outer = malloc(sizeof(int*));
    printf("%p\n", outer); // 0x55f0f867d2a0
    as_is(outer);
    printf("%p\n", outer); // 0x55f0f867d2a0 (same)
}
```

```c
#include <stdlib.h>

void deref(const int* const *inner) {
    int* new_array = malloc(sizeof(int) * 5);
    *inner = new_array; // compilation error
}
```

```c
#include <stdlib.h>

void deref_deref(const int* const *inner) {
    (*inner)[0] =20; // compilation error
    (*inner)[0] =21; // ..
}
```

## `(const int**) <-- (&pointer)`

```c
#include <stdlib.h>
#include <stdio.h>

void as_is(const int** inner) {
    int* new_array = malloc(sizeof(int) * 5);
    inner = &new_array;
} 

int main()
{
    int (*outer)[2];
    outer = malloc(sizeof(int*));
    printf("%p\n", outer); // 0x5587f07af2a0
    as_is(&outer);
    printf("%p\n", outer); // 0x5587f07af2a0 (same)
}
```

```c
#include <stdlib.h>
#include <stdio.h>

void deref(const int** inner) {
    int* new_array = malloc(sizeof(int) * 5);
    *inner = new_array;
} 

int main()
{
    int (*outer)[2];
    outer = malloc(sizeof(int*));
    printf("%p\n", outer); // 0x55f95ef092a0
    deref(&outer);
    printf("%p\n", outer); // 0x55f95ef096d0
}
```

```c
#include <stdlib.h>

void deref_deref(const int** inner) {
    (*inner)[0] =20; // compilation error
    (*inner)[0] =21; // ..
}
```
