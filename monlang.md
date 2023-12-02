
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

let lazy-eval (expr):{():{expr}}
let actions ["process":lazy-eval(process), "next":():{inputs |> next()}]
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

# functional OOP (without 'this')
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

    (msg):{
        let dispatch ["withdraw":withdraw, "deposit":deposit]
        dispatch[msg]
    }
}

var account make-account(100)
let acc-withdraw(x) account("withdraw", x)
let acc-deposit(x) account("deposit", x)

acc-withdraw(25) # 75
acc-withdraw(25) # 50
acc-deposit(10) # 60
acc-deposit(10) # 70
"deposit" |> account(30) # 100
```

# OOP (with 'this')

```
var account = {
    let initial-amount 100
    Account.new(initial-amount)
}
&account |> Account.withdraw(50) |> Account.deposit(25) # 75
```
