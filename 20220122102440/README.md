# git clone with submodules

note: git submodules is often considered an anti-pattern.

If you are considering using git submodules in your own system, please consider the following alternatives:

* separate git libraries with proper packaging handled through tools such as go mod, mvn, and npm packages
* single git repository
* git subtree

## initial clone
```sh
git clone <address>
git submodule init
git submodule update
```

## updating submodules
```sh
cd <submodule dir>
git checkout <branch>
git pull
cd ..
git commit -a -m "update modules"
```
