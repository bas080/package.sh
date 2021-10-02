# package.sh

Package manager for Bash and more.

## Features

- Works with simple files.
- Offers features to easily install executables, sourceable files, link files
  on the filesystem for either executing or sourcing.
- Supports packages which recursively resolve to end up with executable and
  sourceable files.
- Easy to install and configure as it builds on top of standard bash
  functionality. No need to rewrite your existing bash scripts.
- Allows project based dependencies. Simply define a package.sh in a directory
  and source the output of `package sourced`.

## Setup

To make this easy you can run the following script for installation.

```bash bash
rm "$HOME/.local/bin/package" || true
wget -O \
  "$HOME/.local/bin/package" \
  'https://raw.githubusercontent.com/bas080/package.sh/master/bin/package'
chmod +x "$HOME/.local/bin/package"
grep -q -F 'source <(package sourced)' "$HOME/.bashrc" ||
  echo 'source <(package sourced)' >> "$HOME/.bashrc"
```

> This script does assume you have the `~/.local/bin/` in your `$PATH`. You can
> check this with `echo "$PATH"`. You can add the `.local/bin` directory to
> your $PATH if that is not the case.

You can also clone the repo and link the ./package or ./bin/package script.

```bash bash
rm "$HOME/.local/bin/package" || true
ln -s "$PWD/package" "$HOME/.local/bin/package"
```

## Usage

We configure our dependencies in a file named `package.sh`. This file can be
defined in any directory.

For this example we'll show how to configure the different type of bash
dependencies.

Let's see how you could install [NVM][1].

### Sourced

NVM offers their functionality by sourcing the nvm script in a bash process. To
do this we use the sourced function(ality).

```bash
sourced https://raw.githubusercontent.com/creationix/nvm/v0.33.11/nvm.sh
sourced https://raw.githubusercontent.com/creationix/nvm/v0.33.11/bash_completion
```

> Please note that the order matters. This decides which script will be sourced
> first. In the case of NVM the order of sourcing is important.

We can see the contents of all sourced script by running `package sourced`. To
source the output we can do either `source <(package sourced)` or `package
sourced | source /dev/stdin`.

Besides files; one can also pass STDIN to the sourced function. This is handy
for a short bash expression. I use it to configure nvm.

```bash
sourced <<< 'export NVM_DIR="$HOME/.nvm"
nvm use stable'
```

### Sourceable

We also have the sourceable feature. This is a script which won't be printed
when running `package sourced`. The only thing it does is place the file in the
`.package` directory. Because the `.package` is added to `$PATH`; we can use
the bash functionality to source using an alias. For this example we'll install
bash-tap utility which expects one to source the file in their script.

```bash
sourceable bash-tap https://raw.githubusercontent.com/bas080/bash-tap/master/bash-tap
```

```bash bash
source bash-tap

plan 1

ok "Prints ok"
```
```
1..1
ok - Prints ok
```

### Executable

An executable is the same as sourceable except for that it has executable
permission.

> In some cases an executable bash script also offers sourcing that file
> instead. Some bash scripters will offer this to make working with their
> script within your own bash scripts easier.

```
executable tetris https://raw.githubusercontent.com/dkorolev/bash-tetris/master/tetris.sh
```

You can now run the tetris executable. Simply type `tetris` in your bash prompt
and press enter.

### Installed

Some projects offer a handy install script. Package.sh also has a use-case for
these.

> These scripts are only run on an install. Do make sure that the script allows
> re-running on previous failure or success runs. Most mature projects manage
> these cases.

```sh
installed https://cli-assets.heroku.com/install.sh
```

### Linked


You can choose to link instead of copying local files. This is handy for when
you are working on a file on the same file system which another script depends
on. No need to rerun an install when changing a script's dependency.

Because linking will also adopt the permissions of the dependency; we have no
need to describe if it is an executable or a sourceable.

```
linked package $PWD/package
```

It's also possible to link local applications defined on the $PATH. This allows
more explicit definitions of script dependencies.

```bash
linked mkdir
```

Those are all ways to configure dependencies.

## Issues

### Alias conflicts

Currently `package.sh` will overwrite with the last version defined in the
`./package.sh` without warnings. In the future the plan is to allow a concept
that allows different shell scripts to use their own version of dependencies.
You have been warned!

## Contributing

### Development

Before you can start developing we suggest you first follow the setup
instructions to get package.sh working. Once that is setup; run the following
in the project directory.

```bash bash 2>&1
package install
```
```
[package] fetched executable /home/women/projects/bashionista/bashionista with alias bashionista
[package] linked /usr/bin/tail with alias tail
[package] linked /usr/bin/head with alias head
[package] linked /usr/bin/basename with alias basename
[package] linked /usr/bin/envsubst with alias envsubst
[package] linked /bin/bash with alias bash
[package] linked /bin/cat with alias cat
[package] linked /bin/sed with alias sed
[package] fetched executable https://raw.githubusercontent.com/bas080/mache/master/mache with alias mache
[package] fetched executable https://raw.githubusercontent.com/bas080/markatzea/master/markatzea with alias markatzea
[package] fetched sourceable https://raw.githubusercontent.com/bas080/bash-tap/master/bash-tap with alias bash-tap
[package] linked /home/women/projects/package/package with alias package
[package] linked /bin/ls with alias ls
[package] linked /usr/bin/wget with alias wget
[package] linked /bin/chmod with alias chmod
[package] linked /bin/ln with alias ln
[package] linked /bin/rm with alias rm
[package] linked /bin/mkdir with alias mkdir
[package] linked /bin/which with alias which
[package] linked /bin/readlink with alias readlink
[package] linked /bin/mv with alias mv
[package] removing old install
[package] install was a success
```

We also need to source a script in order to have the correct directory in the
`$PATH`. This can be done manually in the following manner.

```bash
source <(package sourced)
```

You might find it weird that the package.sh file contains the local package as
a dependency. This is done to make project development easier. Changes made to
the source-code will be immediately in effect by using the `linked` feature.

> Consider using [projector][2] to create project specific bashrc files.

### Bundling

package.sh uses bashionista to make writing bash utilities a bit easier. To
make installation of the script easier we make a "bundle" which can be used by
others without installing bashionista. To create this bundle simply run `package
bundle` with the bashionista version of the script.

```bash bash
mkdir -p ./bin
./package bundle > ./bin/package
```

### Documentation

Documentation is generated with [markatzea][3]. To generate the `README.md`
simply run:

```bash
markatzea README.mz
```

When running markatzea it will evaluate the code below and prevent infinite
recursion. You can ignore this script.

```bash bash
[[ -n "$NO_RECUR" ]] && exit 0

NO_RECUR=1 markatzea README.mz > README.md
```

### Tests

Tests are defined in the `README.mz` in the form of executed code-blocks.

```bash bash
source bash-tap

plan 2

package ls | diagnostics
success "Can flatten package file"

package install 2>&1 | diagnostics
success "Can install deps"
```
```
1..2
# executable 'bashionista' '/home/women/projects/bashionista/bashionista'
# linked 'tail'
# linked 'head'
# linked 'basename'
# linked 'envsubst'
# linked 'bash'
# linked 'cat'
# linked 'sed'
# executable 'mache' 'https://raw.githubusercontent.com/bas080/mache/master/mache'
# executable 'markatzea' 'https://raw.githubusercontent.com/bas080/markatzea/master/markatzea'
# sourceable 'bash-tap' 'https://raw.githubusercontent.com/bas080/bash-tap/master/bash-tap'
# linked 'package' '/home/women/projects/package/package'
# linked 'ls'
# linked 'wget'
# linked 'chmod'
# linked 'ln'
# linked 'rm'
# linked 'bash'
# linked 'mkdir'
# linked 'which'
# linked 'readlink'
# linked 'mv'
# linked 'cat'
ok - Can flatten package file
# [package] fetched executable /home/women/projects/bashionista/bashionista with alias bashionista
# [package] linked /usr/bin/tail with alias tail
# [package] linked /usr/bin/head with alias head
# [package] linked /usr/bin/basename with alias basename
# [package] linked /usr/bin/envsubst with alias envsubst
# [package] linked /bin/bash with alias bash
# [package] linked /bin/cat with alias cat
# [package] linked /bin/sed with alias sed
# [package] fetched executable https://raw.githubusercontent.com/bas080/mache/master/mache with alias mache
# [package] fetched executable https://raw.githubusercontent.com/bas080/markatzea/master/markatzea with alias markatzea
# [package] fetched sourceable https://raw.githubusercontent.com/bas080/bash-tap/master/bash-tap with alias bash-tap
# [package] linked /home/women/projects/package/package with alias package
# [package] linked /bin/ls with alias ls
# [package] linked /usr/bin/wget with alias wget
# [package] linked /bin/chmod with alias chmod
# [package] linked /bin/ln with alias ln
# [package] linked /bin/rm with alias rm
# [package] linked /bin/mkdir with alias mkdir
# [package] linked /bin/which with alias which
# [package] linked /bin/readlink with alias readlink
# [package] linked /bin/mv with alias mv
# [package] removing old install
# [package] install was a success
ok - Can install deps
```

## TODO

- [ ] Man page and better help page. README to Man page?
- [ ] How to manage multiple package.sh files. One in `$HOME` and another in
  `$HOME/<project>`.

[1]:https://github.com/nvm-sh/nvm
[2]:https://github.com/bas080/projector
[3]:https://github.com/bas080/markatzea

