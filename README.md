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

**Important Step**

To make this easy you can run the following script for installation.

```bash bash
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

## Usage

We configure our dependencies in a file named `package.sh`. This file can be
defined in any directory.

For this example we'll show how to configure the different type of bash
dependencies.

Let's see how you could install [NVM][1].

### Sourced

NVM offers their functionality by sourcing the nvm script in a bash process. To
do this we use the sourced function(ality).

```sh
sourced https://raw.githubusercontent.com/creationix/nvm/v0.33.11/nvm.sh
sourced https://raw.githubusercontent.com/creationix/nvm/v0.33.11/bash_completion
```

> Please note that the order matters. This decides which script will be sourced
> first. In the case of NVM the order of sourcing is important.

We can see the contents of all sourced script by running `package sourced`. To
source the output we can do either `source <(package sourced)` or `package
sourced | source /dev/stdin`.

### Sourceable

We also have the sourceable feature. This is a script which won't be printed
when running `package sourced`. The only thing it does is place the file in the
`.package` directory. Because the `.package` is added to `$PATH`; we can use
the bash functionality to source using an alias. For this example we'll install
bash-logger utility which expects one to source the file in their script.

```bash
sourceable bash-log https://raw.githubusercontent.com/bas080/logger/master/logger
```

```bash bash 2>&1
source bash-log

log_error "oops"
```
```
2021-09-28T23:27:52+02:00[1] oops
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
installed 'https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh'
```

### Linked


You can choose to link instead of copying local files. This is handy for when
you are working on a file on the same file system which another script depends
on. No need to rerun an install when changing a script's dependency.

Because linking will also adopt the permissions of the dependency; we have no
need to describe if it is an executable or a sourceable.

```
linked mache $HOME/projects/mache/mache
```

Those are all ways to configure dependencies.

## Issues

### Alias conflicts

Currently `package.sh` will overwrite with the last version defined in the
`./package.sh`. No warnings, in the future the plan is to allow a concept that
allows different shell scripts to use its own version of dependencies.

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
[package] fetched sourceable https://raw.githubusercontent.com/bas080/bash-tap/master/bash-tap with alias bash-tap
[package] fetched executable https://raw.githubusercontent.com/bas080/mache/master/mache with alias mache
[package] fetched executable https://raw.githubusercontent.com/bas080/markatzea/master/markatzea with alias markatzea
[package] linked /home/women/projects/bashionista/bashionista with alias bashionista
[package] linked /home/women/projects/package/package with alias package
[package] removing old install
[package] install was a success
source <(package sourced)
```

We also need to source a script in order to have the correct directory in the
`$PATH`. This can be done manually in the following manner.

```bash
source <(package sourced)
```

You might find it weird that the package.sh file contains the local package as
a dependency. This is done to make project development easier. Changes made to
the source-code will be immediately in effect.

> I myself use a tool named [projector][2] to create project specific bashrc
> files.

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

Documentation is generated with markatzea.

```bash bash
[[ -n "$NO_RECUR" ]] && exit 0

NO_RECUR=1 markatzea README.mz > README.md
```

## License

Not responsible for any damages!

Something

## TODO

- [ ] Man page and better help page.
- [ ] How to manage multiple package.sh files. One in `$HOME` and another in
  `$HOME/<project>`.
- [ ] Add tests.
