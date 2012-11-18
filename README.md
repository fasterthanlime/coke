## A build tool for ooc

Uses bottles (JSON files) to set you up for development

## Basic premise

Programs have a Cokefile, that can contain various targets with various tasks.

Libraries have 'bottles', listed here in this Github repo. Think of bottles
as homebrew formulas. Under the hood, though, coke can install native
packages via your system's package manager (apt-get, brew, or a custom solution
on Win32).

coke requires rock, and will make a few basic sanity checks to make sure we have
stuff like 'pkg-config' in the path, and 'dylibbundler' on OSX for tasks that
require it (ie. bundling)

## Basic library structure

All ooc projects with bottles should have the same structure. Let's make a project
named 'foo':

  * foo
    * foo.use
    * source

`foo.use` should have a `SourcePath: source` directive so that rock can find the
source files.

## Bottle format

A bottle is a simple JSON file that tells you what the library is, how to install
the native packages (if any).

### name

The name of the bottle. It should match the name of the .json file.

### description

A human-readable description of the package - can contain spaces.

### version

The version of the software it provides. Free-form, string. So far coke
doesn't allow to manage more than one version of a software. Get over it :)

### source

Where to get the software from. This is an array with `[ source_type, source ] `

Examples:

```javascript
{
    source: [ 'github', 'nddrylliog/ooc-sdl' ]
}
```

Another example:

```javascript
{
    source: [ 'url', 'https://github.com/downloads/nddrylliog/rock/rock-0.9.4-prealpha4-bootstrap-only.tar.bz2' ]
}
```

#### GitHub sources

The reason we support GitHub sources but not 'git' sources is that we can't
rely on git being present on all platforms. For example, on Win32, you can
install msysgit and get a 'git bash', but the msys/mingw combo you install is
separate. So we won't necessarily have 'git' in PATH from where we run coke.

For GitHub, from a repo's name (e.g. 'nddrylliog/ooc-sdl') we can retrieve the latest
commit

```
curl -i https://api.github.com/repos/nddrylliog/rock/commits/HEAD
```

And we can get a tarball of the content without having to clone the git repo:

```
wget https://github.com/nddrylliog/ooc-sdl/archive/master.zip
```

#### URL sources

The only supported schemas are 'http' and 'https' for now. coke will use
libcurl to retrieve them, so that https is handled with no issue. This is
pretty much a subset of GitHub sources.

With http sources, to check for an update we don't have commit hashes to
compare, but we can compare the name of the last download with the new name,
in case the registry has been updated.

### platforms

Platforms defines the underlying packages we need to install for those libraries
to work. For pure ooc libraries, you don't need to specify a platforms block
at all, but for bindings of C libraries, this is a life saver.

The valid keys for this block are as follows:

  * os: can be 'linux', 'osx', 'windows', or '*' (default)
  * distribution: only valid for linux, can be 'ubuntu', 'arch' or '*' (default)
  * arch: can be '32', '64', or '*' (default) for now
  * prio: priority of the platform block: if several blocks apply, they'll be sorted
    by priority. Default value is 10, it's an integer, set it to whatever you want.
  * deps: see below
  * setup: see below
  * check: see below

#### deps

This is kinda redundant with .use files, but we'll figure it out as we go. For now,
the supported types are:

  * `pkgs`: a pkg-config package, e.g. `sdl`

#### setup

setup is a child of `platforms`, and it's an array of arrays. Each children of
setup is a setup 'step' that you need to take to make sure the underlying libraries
are there.

There are different valid types of steps:

  * `apt`: Install an apt package on an Ubuntu/Debian distribution
  * `brew`: Install a brew package on OSX
  * `mingw-archive`: Download and extract an archive to `/c/MinGW/MSYS/1.0/`. Such
     an archive will typically have folders such as `lib`, `include` and `bin`

#### check

check is the phase to make sur everything's fine. There are several types of groups.

The only supported group so far is `c`, which has `headers` and `libs`. The principle
of the `c` group is to compile a test file (a-la autotools, yep..) with an include,
linking it to libs, and make sure it compiles and links fine. We might also collect
additional information with otool/ldd/etc. later on for the lulz.

