# use the `exec` command in bash to create custom file descriptors, understanding redirections

## Redirect trace generated by xtrace bash option into a file
```bash
exec 6> trace.txt
    export BASH_XTRACEFD=6
    set -o xtrace; date; { set +o xtrace; } 6> /dev/null
    >&6 echo "this is also going into trace.txt"
    unset BASH_XTRACEFD
exec 6>&-
```

we create a file descriptor (6) and redirect it to `trace.txt` file (we create the file if it doesn't exist -- clobbering <> append) :
```bash
exec 6> trace.txt
```

we set the `BASH_TRACEFD` global variable to `6` (the freshly created fd), this is a variable used when we activate the xtrace bash option, which is commonly used for debugging a script :
```bash
export BASH_XTRACEFD=6
```

we execute the `date` command, that prints current date and time, enclosed by the xtrace option :
```bash
set -o xtrace; date; { set +o xtrace; } 6> /dev/null
```
instead of being printed in stdout, the trace is going to be written in the `trace.txt` file

this last bit is to close the file descriptor :
```bash
exec 6>&-
```

content of `trace.txt` at the end of the script:
```
+ date
this is also going into trace.txt

```

---

## Feed a literal string as input to a function/command reading from stdin
```bash
function read_a_line {
    read user_input
    echo "line was \`$user_input\`"
}

exec 6>&0 <<< "my literal string"
read_a_line

exec 0>&6 6>&-
read_a_line
```

let's define a simple function that reads a line from stdin then print it back to stdout :
```bash
function read_a_line {
    read user_input
    echo "line was \`$user_input\`"
}
```

now we create a new file descriptor and save current tty stdin to it, then forward a literal string to stdin. We then call read_a_line which will consume the string :
```bash
exec 6>&0 <<< "my literal string"
read_a_line
```

to restore stdin previous behavior, namely accepting user input again, we need to restore stdin fd from where we backed it up (fd 6) :
```bash
exec 0>&6
```

we no longer need file descriptor 6 so we close it (same command as previous script :
```bash
exec 6>&-
```

we can also combine the two previous commands into one (as we did in the script) :
```bash
exec 0>&6 6>&-
```

when we execute the script, this is what is printed :
```terminal
line was `my literal string`
>_
```

then it waits for a user input, suppose we type in `some text` then press ENTER, the script now ends :
```terminal
line was `my literal string`
>some text
line was `some text`
```

---

## Discard any potential output, coming from either stdout or stderr, in a block of commands
```bash
exec 6>&1 1> /dev/null; exec 7>&2 2> /dev/null
    echo "this message will never appear on screen"
    rm . || true
exec 1>&6 6>&-; exec 2>&7 7>&-
```

we create file descriptor 6, in which we save stdout (1), and forward stdout to /dev/null :
```bash
exec 6>&1 1> /dev/null
```

same goes for stderr (2), we will create file descriptor 7 to save it to :
```bash
exec 7>&2 2> /dev/null
```

`echo` command normally prints to stdout, `rm` command fails and normally prints an error message to stderr when we pass it a directory (here we use `|| true` in order to bypass `rm` non-zero exit code, but it would still prints the error message to stderr) :
```bash
echo "this message will never appear on screen"
rm . || true
```

finally, we restore both stdout and stderr, and close our file descriptors we no longer need (we joined the two `exec` commands in a single line with a `;` in the script) :
```bash
exec 1>&6 6>&-
exec 2>&7 7>&-
```

when we execute the script, this is what is printed :
```terminal
```
(nothing)

---

```bash
## duplicate stdout to stderr ##
exec 6>&1 > >(tee /dev/stderr)
trap 'exec 1>&6 6>&-' EXIT
```

---

# Appendix: a substantial runnable bash script with examples to learn from

```bash
#!/usr/bin/env bash
set -o errexit

trap 'echo Exited with status $?' EXIT

trap handle_errexit ERR
function handle_errexit {
    local status=$?

    trap - ERR

    declare -r E_COMMAND_FAILING=99

    local cmd=$BASH_COMMAND
    local func_name=${FUNCNAME[1]}
    local filename=${BASH_SOURCE[1]}
    local lineno=${BASH_LINENO[0]}

    >&2 echo "Command \`$cmd\` failed in $func_name with status $status"
    >&2 echo "  -> $filename:$lineno"

    exit $E_COMMAND_FAILING
}

# create a fd to a file (create file with > or append to file with >>)
# ex: redirect xtrace to a file using BASH_XTRACEFD variable and exec command #
exec 6> trace.txt
    export BASH_XTRACEFD=6
    set -o xtrace; date; { set +o xtrace; } 6> /dev/null
    >&6 echo "this is also going into trace.txt"
exec 6>&- # close fd

# discard all inputs coming from stdin (forward from /dev/null) #
exec 6>&0 < /dev/null
    cat # this will immediatly be fed with EOF (coming from /dev/null), so it doesn't wait for an input
exec 0>&6 6>&-

# hardcode stdin input with a literal string #
exec 6>&0 <<< "this is going to stdin"
    read user_input
    echo "we read from stdin \`$user_input\`"
exec 0>&6 6>&-

# hardcode stdin input with an EOF block #
exec 6>&0 <<'EOF'
    _\/_
     /\
     /\
    /  \
    /~~\o
   /o   \
  /~~*~~~\
 o/    o \
 /~~~~~~~~\~`
/__*_______\
     ||
   \====/
    \__/
EOF
    cat > christmas_tree.txt
exec 0>&6 6>&-

# hardcode stdin input with a file content #
exec 6>&0 < christmas_tree.txt
    cat
exec 0>&6 6>&-

# hardcode stdin input with a command output (using process substitution syntax) #
exec 6>&0 < <(date)
    read user_input
    echo "we read from stdin \`$user_input\`"
exec 0>&6 6>&-

# discard all output coming from stdout (forward to /dev/null) #
exec 6>&1 > /dev/null
    echo "this message will never appear on screen"
exec 1>&6 6>&-

# forward all output coming from stdout to a file #
exec 6>&1 > stdout.txt
    echo "this message will be in stdout.txt"
exec 1>&6 6>&-

# discard all output coming from stdout or stderr (forward to /dev/null) #
exec 6>&1 1> /dev/null; exec 7>&2 2> /dev/null
    echo "this message will never appear on screen"
    rm . || true
exec 1>&6 6>&-; exec 2>&7 7>&-

# forward all output coming from stdout as input to command (using process substitution syntax) #
exec 6>&1 > >(wc -l)
    date # 1 line...
    echo -en "1\n 2\n 3\n" # ...+ 3 lines
exec 1>&6 6>&-

exit 123

```
