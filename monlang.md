
# les expressions primitives

## un str
"fds"

## une liste
["a", "b", "c"]

## une map
["key1":"val1", "key2":"val2"]

## une lambda (fonction anonyme), différent d'un bloc de code car l'expression est la fonction elle-même, pas la valeur qu'elle retourne après l'avoir évaluer
(x):{x}


# leurs types primitifs respectifs

str
list<T>
map<K,V>
lambda<R(P...)>

# construire une expression non-réduite

## en utilisant les opérateurs built-in
"a, b, c" |> split(", ") |> map((l):{"Letter " + l)}) |> join("\n")

# construire une abstraction

## en utilisant les étiquettes
```
let avg-duration {
    let sum (dur1, dur2):{
        dur1 + dur2
    }
    durations |> reduce(0, sum)
}
print("Avg duration is " + avg-duration)

let delay (expr):{():{expr}}
let actions ["process":delay(process), "next":():{inputs |> next()}]
let next-action actions[cur-state]
```

---

# state variables (<=> of static mutable var in c) and returning lambda
```
let make-account (initial-amount):{
    var balance = initial-amount
    let ret (amount):{
        balance -= amount
        balance
    }
    ret
}

var withdraw = {
    let initial-amount 100
    make-account(initial-amount) # returns a lambda with a state variable initialized
}
withdraw(25) # prints 75
withdraw(25) # prints 50
```

# functional OOP (without 'this') -- using closures
```
let make-account (balance):{
    var _balance (balance) # create state variable from arg value

    var withdraw (amount):{
        _balance -= amount
        _balance
    }

    var deposit (amount):{
        _balance += amount
        _balance
    }

    (msg, args...):{
        let dispatch ["withdraw":withdraw, "deposit":deposit]
        dispatch[msg](args)
    }
}

var account make-account(100)
let acc-withdraw(x) account("withdraw", x)
let acc-deposit account("deposit", x)

acc-withdraw(25) # 75
acc-withdraw(25) # 50
acc-deposit(10) # 60
acc-deposit(10) # 70
```

# OOP (with 'this')

```
var account = {
    let initial-amount 100
    Account.new(initial-amount)
}
&account |> Account.withdraw(50) |> Account.deposit(25) # 75
```

# reduce implementation
```
let reduce (reducer, init, list):{
    var acc init
    foreach list {
        acc = reducer(acc, $1)
    }
    acc
}

let sum (x, y):{x + y}
reduce(sum, 0, [1, 2, 3]) # 6
reduce(sum, 'T', "ommy") # Tommy
```

## what happens step by step (reduction)

```
reduce(sum, 0, [1, 2, 3])
```

resolve 'sum' identifier => substitute label with its equivalent

```
reduce((x, y):{x + y}, 0, [1, 2, 3])
```

eval reduce function => substitute label with its equivalent

```
(reducer, init, list):{
    var acc init
    foreach list {
        acc = reducer(acc, $1)
    }
    acc
}((x, y):{x + y}, 0, [1, 2, 3])
```

bind parameters and substitute them in the block

```
{
    var acc 0
    foreach [1, 2, 3] {
        acc = (x, y):{x + y}(acc, $1)
    }
    acc
}
```

reduce lambda + call to block of code

```
{
    var acc 0
    foreach [1, 2, 3] {
        acc = {acc + $1}
    }
    acc
}
```

evaluate block expression

```
6
```
