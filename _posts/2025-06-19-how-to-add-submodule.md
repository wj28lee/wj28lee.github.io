```bash
$ git clone https://github.com/path/to/repo.git
$ cd <repo name>
$ mkdir example && cd example
$ git submodule add https://github.com/path/to/submodule.git
$ cd <submodule name>
$ git status .
$ cd ..
$ git status .
```
```bash
$ cd /path/to/submodule
$ git add <modified file>
$ git commit -s -m"<Commit Message>"
$ git push
$ cd ..
$ git add <submodule name>
$ git commit -s -m"<Commit Message>"
$ git push
