
# Prevent undefined behavior

For security reason, you better crash the program than letting it behave in an undefined way deliberately.

## Dereferencing a null pointer in C++

### Problem

```c++
something* ptr = nullptr;
*ptr = ...; // undefined behavior
```

### Solution

```c++
assert(ptr != nullptr);
*ptr = ...; // this line never gets executed unless ptr has an address
```

N.B: this code doesn't guarantee defined behavior but at least you prevented one obvious case of it.

# Lazy error/bug handling

Deliberately limiting your procedures capabilities, by enforcing them to only accept a limited range of inputs, allows you to focus exclusively on the "happy path" in the implementation and not worry about unknown/weird input that may slip from another process at runtime.

## Checking arguments "type" in python

```python
class Crate:
	def __init__(self, mark: str):
		assert isinstance(mark, str)
		assert len(mark) == 1
		self.mark = mark

class CrateStack:
	def __init__(self, crates: list[Crate]):
		assert isinstance(crates, list)
		for crate in crates:
			assert isinstance(crate, Crate)
		self.crates = crates
```



