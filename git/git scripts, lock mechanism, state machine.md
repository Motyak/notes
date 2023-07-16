
# Example of git scripting using bash

## `gittransaction` wrapper script and transaction scripts

`gittransaction.sh` :
```bash
#!/usr/bin/env -S bash -i

# script should not be sourced
[ "${BASH_SOURCE[0]}" != "$0" ] && return 1

set -o errexit

# script should not be executed outside of a git repository
g_git_dir_abs_path="$(git rev-parse --show-toplevel)"
g_gitscripts_repo=""

trap clean_and_exit EXIT
function clean_and_exit {
    local exit_code=$?

    set +o errexit; trap - ERR # cleaning shall not fail
    >&2 echo "Cleaning..."
    git::unlock
    gitscripts::unlock

    exit $exit_code
}

function git::lock {
    # try and acquire the lock
    until (set -o noclobber; echo $$ > .git/index.lock) &>/dev/null; do
        sleep 1 # retry in a second
    done
}

function git::unlock {
    rm -f .git/index.lock
}

function uri_to_filename {
    sed -e 's/%/%25/g' \
        -e 's/\//%2f/g' \
        -e 's/:/%3a/g' \
        -e 's/?/%3f/g' \
        -e 's/\*/%2a/g' <<< "$@"
}

function get_cur_repo_id {
    local remote_origin_url; remote_origin_url="$(git config --get remote.origin.url)"
    local repository_name; repository_name="${remote_origin_url##*//}"
                           repository_name="$(uri_to_filename "${repository_name%.git}")"
    local hashed_git_dir_abs_path; hashed_git_dir_abs_path="$(md5sum <<< "$g_git_dir_abs_path")"

    local repository_id="$repository_name--${hashed_git_dir_abs_path::4}"
    echo -n "$repository_id"
}

function gitscripts::lock {
    local repository_id; repository_id="$(get_cur_repo_id)"
    # try and acquire the lock
    until (set -o noclobber; echo $$ > "/tmp/git-scripts/lock/$repository_id") &>/dev/null; do
        sleep 1 # retry in a second
    done
    g_gitscripts_repo="$repository_id"
}

function gitscripts::unlock {
    rm -f "/tmp/git-scripts/lock/$g_gitscripts_repo"
}

function begin_git_transaction {
    gitscripts::lock
    git::lock

    mkdir -p "/tmp/git-scripts/cache/$g_gitscripts_repo"; cd "$_"
    rsync -a --delete "$g_git_dir_abs_path/" .

    export GITTRANSACTION="occuring"
    >&2 echo "BEGIN TRANSACTION"
}

function end_git_transaction {
    cd -
    rsync -a --delete "$OLDPWD/" "$g_git_dir_abs_path"

    git::unlock
    gitscripts::unlock

    >&2 echo "END TRANSACTION"
}

# make sure these directories exist
mkdir -p /tmp/git-scripts/lock \
         /tmp/git-scripts/cache

begin_git_transaction
    bash --rcfile <(echo 'PS1="transaction...> "')
end_git_transaction
```

An example of a transaction script, `example.sh`:
```bash
# NEVER CALL `exec` IN A TRANSACTION SCRIPT..
# ..AS IT WILL NEVER RESUME TO PARENT SHELL EXECUTION FLOW !

# script should not be sourced
[ "${BASH_SOURCE[0]}" != "$0" ] && return 1

# make sure we only execute this using gittransaction
[ -v GITTRANSACTION ] || exit 1

current_branch_name=$(git branch --show-current)
git fetch origin "$current_branch_name"
git merge --ff-only "origin/$current_branch_name" || {
    # make a local backup of current branch #
    backup_branch_name="backupp/$current_branch_name/$(date +%Y-%m-%d_%H-%M-%S)"
    git branch "$backup_branch_name"

    # overwrite local branch with remote state
    git reset --hard "origin/$current_branch_name"
}
```

## how to use `gittransaction`

### interactive mode

```terminal
$ gittransaction
BEGIN TRANSACTION
# type in git commands
# ...
transaction...> exit
END TRANSACTION
Cleaning...
```

### passing a script file (bash or sh)

```terminal
$ gittransaction < example.sh
BEGIN TRANSACTION
# <script output if any>
END TRANSACTION
Cleaning...
```

### passing a script file (bash or sh) with a timeout

```terminal
$ timeout 10s gittransaction < example.sh
# suppose getting the lock takes too much time...
Cleaning...
```

### passing a block of bash commands

```bash
gittransaction <<EOF
    # git commands
    # ...
EOF
```
