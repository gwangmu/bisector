# bisector
Git bisection utility for memory corruption bugs that requires (almost) nothing.

## Introduction

Simple one-bash-script utility for git bisection. Receives **suspected commit range, PoC input, and PoC command**, produces the first bad commit (if it exists in the range).

## Requirements

* ASAN-enabled compiler
* (Obviously) git-based project
* Known bad/good commit hashes
* PoC input
* PoC command

## Usage

1. Place a `BISECT` file in the project directory, formatted as follows:

```
<known_bad_commit> <known_good_commit> <poc_input_path> -- <poc_command>
```

* `<known_bad_commit>`: Known bad commit hash. (tag name applicable as _tags/<name>_)
* `<known_good_commit>`: Known good commit hash.
* `<poc_input_path>`: Path to PoC input.
* `<poc_command>`: PoC command, with `@@` as a placeholder for the PoC input.

2. Place `bisector` in the project directory and execute it.

```
$ ./bisector
```

3. When finished, find the first bad commit in the `FIRSTBAD` file. See `{good,bad}.bis` for the ASAN dumps from the last good and first bad commit, respectively.

## Example: CVE-2016-3658 in libtiff 4.0.6

0. Prepare `libtiff` itself and the PoC input.

```
$ git clone https://gitlab.com/libtiff/libtiff.git libtiff
$ cd libtiff
$ curl http://bugzilla.maptools.org/attachment.cgi?id=601 > POC
```
The PoC input is provided by http://bugzilla.maptools.org/show_bug.cgi?id=2500.

1. Place a `BISECT` file.

```
d7db70c9 f7b79dc7 POC -- ./tools/tiffset @@
```
`d7db70c9` is the last commit before 07/01/2015, and we expect CVE-2016-3658 has been created after 4.0.0 (`f7b79dc7`).

2. Place the `bisector` script and run it.

```
$ ./bisector
[*] Initializing...
 +  Commit range: 'd7db70c9'(bad) to 'f7b79dc7'(good)
 +  PoC input: POC
 +  PoC command: ./tools/tiffset @@
[*] Generating most recent ASAN dump...
[*] Testing commit 'd7db70c9'...
 +  Bug type: heap-buffer-overflow
 +  Crash function: TIFFWriteDirectoryTagSampleformatArray
[*] Verifying good commit...
[*] Testing commit 'f7b79dc7'... Good
[*] Bisecting...
[*] Testing commit 'd1be5cb7'... (234 remaining) Bad
[*] Testing commit '871d1067'... (117 remaining) Bad
[*] Testing commit '1358a916'... (58 remaining) Bad
[*] Testing commit 'f502a159'... (29 remaining) Good
[*] Testing commit '920688aa'... (15 remaining) Bad
[*] Testing commit 'e54e9545'... (7 remaining) Good
[*] Testing commit 'c0733844'... (4 remaining) Bad
[*] Testing commit 'af47ad26'... (2 remaining) Good
[*] Verifying...
[*] Testing commit 'c0733844'... Bad
[*] Testing commit 'af47ad26'... Good
 +  First bad commit: c0733844461ceb1b05f41968c1d962480f0db5f6 (see FIRSTBAD)
 +  {Last-good, First-bad} ASAN log: {good, bad}.bis
 +  All done!
```

3. Check `FIRSTBAD` to see the first bad commit.

```
c0733844461ceb1b05f41968c1d962480f0db5f6
```

## Future Work

* Currently supports `configure` only.
* Supported bug types include: ASAN-detected memory corruptions, divide-by-zero (SIGFPE), null dereference (SIGSEGV).
* Random contribution very welcomed.
