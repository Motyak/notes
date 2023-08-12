

# "Data-oriented" programming

**EDIT 2023-08-12**:
Put more simply, think of a way you can model your data (classes / structures) so that the logic you have to write processing them become as straight-forward as possible. The algorithmic complexity shift from solving the problem to preparing the input data to be processed in a simple way.
Sometimes it's slower than just treating the input as it is, but also less expressive and difficult to understand. Sometimes it's both faster and easier to understand (suppose we leverage on data structures properties AND assume programmers know about them)


We can use a const (read only) unordered hash map to define a process that associates a particular state or context to a particular process (function pointer or functor) to do.
Instead of a long list of different branches and statements. What can be represented as data would be simpler to understand as so.

Most used data structures are simple :
- static array
- dynamic array
- hash table
- linked list
- binary tree

Data dominates.  If you've chosen the right data structures and organized things well, the algorithms will almost always be self­-evident.  Data structures, not algorithms, are central to programming.  (See Brooks p. 102.)

# Transforming Programming (src:Pragmatic Programmer p.147)

**EDIT 2023-08-12**:
We can generalize processes (programs, functions, ..) as units that read from an input and write to an output. By doing so, we leverage on their capabilities to cooperate with other processes designed this way and allow users/programmers to build programs defining a composition of generic processes (which is a lot easier to understand, implement and modify over time).
An obvious example of that is the built-in pipe mechanism in UNIX systems, iterators in C++, generators in python (functions yielding values progressively).
Another example are higher-order functions used extensively when programming in a functional style (map, reduce, filter, split, join).


All programs transform data, converting an input into an output.

Interactive/Exploratory programming (example: prototyping a shell script by typing commands directly in the shell, then putting them in an executable script file to automate the process).

Process as a series of transformations (using pipes in shell), compose with programs/processes.
Input data flows between individual steps and becomes the desired output.

In functional programming, we can define a specific function as a sequence of higher order (generic) functions (map, reduce, filter, split, join, ...)

TODO: continue on reading pragmatic programmer + Composing Software (Eric Elliott)

# Thinking in States (src:97 things every programmer should know thing_84)

People in the real world have a weird relationship with state. This morning I stopped by the local store to prepare for another day of converting caffeine to code. Since my favorite way of doing that is by drinking latte, and I couldn't find any milk, I asked the clerk.

"Sorry, we're super-duper, mega-out of milk."

To a programmer, that's an odd statement. You're either out of milk or you're not. There is no scale when it comes to being out of milk. Perhaps she was trying to tell me that they'd be out of milk for a week, but the outcome was the same — espresso day for me.

In most real-world situations, people's relaxed attitude to state is not an issue. Unfortunately, however, many programmers are quite vague about state too — and that is a problem.

Consider a simple webshop that only accepts credit cards and does not invoice customers, with an `Order` class containing this method:

```
 public boolean isComplete() {
     return isPaid() && hasShipped();
 }
```
 
Reasonable, right? Well, even if the expression is nicely extracted into a method instead of copy'n'pasted everywhere, the expression shouldn't exist at all. The fact that it does highlights a problem. Why? Because an order can't be shipped before it's paid. Thereby, `hasShipped` can't be true unless `isPaid` is true, which makes part of the expression redundant. You may still want `isComplete` for clarity in the code, but then it should look like this:

```
 public boolean isComplete() {
     return hasShipped();
 }
```

In my work, I see both missing checks and redundant checks all the time. This example is tiny, but when you add cancellation and repayment, it'll become more complex and the need for good state handling increases. In this case, an order can only be in one of three distinct states:

- *In progress:* Can add or remove items. Can't ship.
- *Paid:* Can't add or remove items. Can be shipped.
- *Shipped:* Done. No more changes accepted.

These states are important and you need to check that you're in the expected state before doing operations, and that you only move to a legal state from where you are. In short, you have to protect your objects carefully, in the right places.

But how do you begin thinking in states? Extracting expressions to meaningful methods is a very good start, but it is just a start. The foundation is to understand state machines. I know you may have bad memories from CS class, but leave them behind. State machines are not particularly hard. Visualize them to make them simple to understand and easy to talk about. Test-drive your code to unravel valid and invalid states and transitions and to keep them correct. Study the State pattern. When you feel comfortable, read up on Design by Contract. It helps you ensure a valid state by validating incoming data and the object itself on entry and exit of each public method.

If your state is incorrect, there's a bug and you risk trashing data if you don't abort. If you find the state checks to be noise, learn how to use a tool, code generation, weaving, or aspects to hide them. Regardless of which approach you pick, thinking in states will make your code simpler and more robust.

By [Niclas Nilsson](http://programmer.97things.oreilly.com/wiki/index.php/Niclas_Nilsson)


# Do a first integration fast, integrate frequently

When you need immediate feedback under actual conditions.
When you face a large number of unknowns.

Tracer bullets operate in the same environment and under the same constraint as the real bullets. They get to the target fast, so the gunner gets immediate feedback. And they're a cheap solution. To get the same effect in code, we look for something that gets us from a requirement to some aspect of the final system quickly, visibly and repeatably. Look for the important requirements, the ones that define the system. Look for the areas where you have doubts, and where you see the biggest risks. Then prioritize your development so that these are the first areas you code. 

Look for areas of uncertainty and add the skeleton to make it work.

TODO: continue on reading Pragmatic Programmer p.52


# Variables name and scope

The more "famous" a variable is, the more expressive its name should be.
(ex: int i for a local variable which lifespan ends within a for loop)

Variables shared between multiple files are uppercase.
Global variables local to a single module or file may have a `g_` prefix to identify their scope.
Constants are only uppercase if they are shared across multiple files (the variable "const"ness should be enforced by the compiler).

A variable should be defined as close as possible to where its used, if its used in multiple places then apply the well-known convention (variable initializations - process - return output).

Of course you should limit the scope of a variable to its minimum to avoid unnecessary complexity.



