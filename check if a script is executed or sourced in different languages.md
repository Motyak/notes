
# HOW TO execute / source script in different languages

## bash:
the `script.sh` file:
```bash
#!/usr/bin/env bash

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
# now do_stuff is defined locally
do_stuff
```

how to execute the script from another script or a shell:
```bash
./script.sh # the main body IS executed
# but do stuff is also defined locally (we may not need it)
do_stuff
```

to prevent functions in a script from being imported when our script is called, we can unset our functions at the end of our script:
```bash
function do_stuff {
	:
}

if [ "${BASH_SOURCE[0]}" == "$0" ]; then
    # main body that probably uses our local functions
    # ...

	unset do_stuff # the caller wont need the function so lets undefine it before exiting
fi
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
