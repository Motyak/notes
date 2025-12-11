
Init a git repository
```terminal
$ mkdir local; cd $_
$ git init
$ echo "fds" > file
$ git add file
$ git commit -m 'first commit'
```

Clone git repository to a bare repository
```terminal
$ cd ..
$ git clone --bare local/ remote.git
```

Clone a new git repository from the bare repository
```terminal
$ git clone remote.git -o local2 # no need to setup origin or upstream
```

Update the first repository to use the bare repository as remote origin and setup upstream branch
```terminal
$ cd local
$ git remote -v
$ git add remote origin ../remote.git # add remote origin
$ git branch -u origin/master # set upstream to origin/master
$ git remote -v
$ git branch -vv
```
