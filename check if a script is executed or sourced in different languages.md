
# HOW TO execute / source script in different languages

## bash:
the `script.sh` file:
```bash
#!/usr/bin/env bash

some_variable="some value"

# defining a function that can be sourced
function do_stuff {
    :
}

if [ "${BASH_SOURCE[0]}" == "$0" ]; then
    # main body
fi
```

how to source the script from another script or a shell:
```bash
source script.sh # the main body ISNT executed
# now do_stuff and some_variable are defined locally
do_stuff && [ -v some_variable ] \
         && echo "do_stuff and some_variable are both defined"
```

how to execute the script from another script or a shell:
```bash
./script.sh # the main body IS executed
# do_stuff and some_variable are not defined because the script was executed in a subshell
```

---

## perl:
the `script.pl` file:
```perl
#!/usr/bin/env perl

package script;

our $a_public_variable;
my $a_private_variable;

sub do_stuff {
    ;
}

unless(caller) {
    # main body
}
```

how to import the script from another script:
```perl
require './script.pl' # the main body ISNT executed
# now do_stuff is defined locally as script::do_stuff()
script::do_stuff()
print $script::a_public_variable;
```

how to execute the script from a shell (sh/bash/...):
```bash
./script.pl # the main body IS executed
# obviously the perl functions arent exported to the shell
```

---

## python:

the `script.py` file:
```python
#!/usr/bin/env python3

def do_stuff():
    pass

if __name__ == "__main__":
    # main body
    pass
```

how to import the script from another script or the python interactive REPL:
```python
import script # the main body ISNT executed
# now do_stuff is defined locally as script.do_stuff()
script.do_stuff()
```

how to execute the script from a shell (sh/bash/...):
```bash
./script.py # the main body IS executed
# obviously the python functions arent exported to the shell
```
