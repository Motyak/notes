
```bash
#!/usr/bin/env bash

cd "$(dirname "$0")"

trap 'rm -f $tmp_file' EXIT

tmp_file="$(mktemp)"

###########################################################
# SPINNER IMPL BEGIN
###########################################################

# e.g.: "abc" => "a b c"
function split {
    fold -w1 <<< "$1" | paste -sd ' '
}

# e.g.: wait_for 1s ; wait_for 500ms
function wait_for {
    local t=${1%ms}
    if [ "$t" == "$1" ]; then
        sleep_val=${t%s}
    else
        sleep_val=$(bc <<<"scale=3; $t / 1000")
    fi
    sleep $sleep_val
}

# e.g.: animate '-\|/' 500ms
function animate {
    local spinner_states=$1
    local delta_t=$2

    while true; do
        for state in $(split $spinner_states); do
            printf "%s\r" $state
            wait_for $delta_t
        done
    done
}

###########################################################
# SPINNER IMPL END
###########################################################

# run in subshell as a background job
{
    exec 6>&1 > "$tmp_file"
    for n in $(./n_inputs.sh); do
        code="$(sed 's/27/'$n'/' ./continued_fraction.py)"
        echo -n "$n -> "
        python3 -c "$code" | grep -P '^\[.*'
    done
    exec 1>&6 6>&-
} & main_job_pid=$!


# print spinner in background job
animate '-\|/' 100ms & spinner_job_pid=$!

wait $main_job_pid
kill $spinner_job_pid

# print the final result
cat "$tmp_file"
```
