

It's recommended to run this script with timeout, to handle deadlock or lock acquiring taking too long (will send SIGTERM after 5 seconds if its still running):
```terminal
$ timeout 5s bash s.sh
```

Example of git scripting using bash: 
```bash
#!/usr/bin/env bash

set -o errexit

trap clean_and_exit EXIT
function clean_and_exit {
    local exit_code=$?

	set +o errexit; trap - ERR # cleaning shall not fail
	>&2 echo "Cleaning..."
    git_unlock

    exit $exit_code
}

g_locking=false

function git_lock {
	$g_locking && return 0 # git is already being locked => early return
	# try and acquire the lock
	until (set -o noclobber; > .git/index.lock) &>/dev/null; do
		sleep 1 # retry in a second
	done
    g_locking=true
}

function git_unlock {
    $g_locking || return 0 # git isnt being locked => early return
    rm -f .git/index.lock
    g_locking=false
}

git_lock # BEGIN TRANSACTION BLOCK
	current_branch_name=$(git branch --show-current)
	git fetch origin "$current_branch_name"
	git merge --ff-only "origin/$current_branch_name" || {
		# make a local backup of current branch #
		backup_branch_name="backupp/$current_branch_name/$(date +%Y-%m-%d_%H-%M-%S)"
		git branch "$backup_branch_name"
	
		# overwrite local branch with remote state
		git reset --hard "origin/$current_branch_name"
	}
git_unlock # END TRANSACTION BLOCK
```