+++
title = "Filepath vs Path"
+++
What's in a path?  If you are like me and switch between Linux Mac OS,
you could be forgiven in thinking all paths are seaparted with the 
slash '/' character.  Therefore, creating a path to a file is nothing more than
concatenating the path segements with '/' characters to get the path 
to a file.  Directories are nothing more than splitting the path and then
using sub-slices of the split values, right?  That's a horrible idea.  There's 
a package that provides functions to correctly manipulate slash separated paths.

The [<code>path</code>](https://golang.org/pkg/path/)
package man ipulates '/' separated paths.  The Path package contains functions
to join and split paths correctly.   For example, the 
[<code>path.Join</code>](https://golang.org/pkg/path/#Join) function joins the
paths using the '/' separator.  The  It is therefore possible to write
working code on Linux, Mac OS, or FreeBSD that uses the Path package.  As soon
as it is run on Windows, it will most likely fail.  Even if your software is 
only running on *nix derived or Posix filesystems, separated with slashes, it is 
still wrong on some level.  Go may be ported to other operating systems 
([OpenVMS](https://www.vmssoftware.com/pdfs/VSI_Roadmap_20170607.pdf) is coming 
to x86 in 2019-ish), where paths or directories may look quite different.

### Cross Platform Code
It's possible to think that all you need to do to write platform independent
code is to check [<code>runtime.GOOS</code>](https://golang.org/pkg/runtime/#pkg-constants).
If you're on windows use '\' and on *nix/Posix use '/'.  This would be just as 
wrong as concatenating '/' characters.  If there's another operating system, 
then your code may not work on tht system.  It's also unnecessary.  

The [<code>path/filepath</code>](https://golang.org/pkg/path/filepath/) package is used to 
manipulate paths in a cross platform manner.  Using this package, along with a 
modest amount of care, will help create software that is runnable on other 
platforms.  In addition to the Filepath package, the 
[<code>io/ioutil</code>](https://golang.org/pkg/io/ioutil/) package contains
functions that simplify the writing of file access code, specifically the 
[<code>ioutil.ReadDir</code>](https://golang.org/pkg/io/ioutil/#ReadDir) function.

#### Creating Paths
Creating paths or joining paths can be done by stringing together parts of paths with
the [<code>filepath.Join</code>](https://golang.org/pkg/path/filepath/#Join) function.
The function takes any number of path elements and returns a path string.  The 
[<code>Abs</code>](https://golang.org/pkg/path/filepath/#Abs) function returns an 
absolute path relative to the current working directory.  This is preferred to 
concatenating together path elements with the 
[<code>filepath.Separator</code>](https://golang.org/pkg/path/filepath/#pkg-constants)
constant.  

```Go
path := filepath.Join("foo", "bar")
fmt.Println(path)

path, _ = filepath.Abs(path)
fmt.Println(path)
```

On Windows the code above returns the following relative and absolute directories.  

```
foo\bar
C:\Users\darcinc\Projects\go101examples\filesystems\filepathvspath\foo\bar
```

And on Linux:

```
foo/bar
/home/darcinc/foo/bar
```

If the path segments contain slash characters, <code>Join</code> will do the
"right thing" and on Windows convert the slash caracters to backslash characters.



#### List Files
The following example will list all the files in a directory, print the absolute
path, the directory, the file name (base), and the extension.  It does not require
my splitting or partioning the paths by hand.  It works on both platforms.  It should
also work on other platforms (for example, ones that separate paths with colons (':')).

```Go
// Spin through the files, print the absolute path,
// the file's directory, the base name, and the
// file extension.
files, _ := ioutil.ReadDir(".")
for _, f := range files {
    path, _ := filepath.Abs(f.Name())
    fmt.Println(path)

    dir := filepath.Dir(path)
    fmt.Println(dir)

    base := filepath.Base(path)
    fmt.Println(base)

    ext := filepath.Ext(path)
    fmt.Println(ext)
}
```

On Windows, it prints the following:

```
C:\Users\darcinc\Projects\go101examples\filesystems\filepathvspath\foo.txt
C:\Users\darcinc\Projects\go101examples\filesystems\filepathvspath
foo.txt
.txt
C:\Users\darcinc\Projects\go101examples\filesystems\filepathvspath\main.go
C:\Users\darcinc\Projects\go101examples\filesystems\filepathvspath
main.go
.go
```

On a Linux system, the same program prints:

```
/home/darcinc/Projects/go101examples/filesystems/filepathvspath/foo.txt
/home/darcinc/Projects/go101examples/filesystems/filepathvspath
foo.txt
.txt
/home/darcinc/Projects/go101examples/filesystems/filepathvspath/main.go
/home/darcinc/Projects/go101examples/filesystems/filepathvspath
main.go
.go
```

You can also [<code>filepath.Glob</code>](https://golang.org/pkg/path/filepath/#Glob)
which will find all files matching a pattern.  The pattern language is actually
defined in the documentation for [<code>filepath.Match</code>](https://golang.org/pkg/path/filepath/#Match).
For example, the code below lists all the files that end in "go".

```Go
matches, _ := filepath.Glob("*.go")
fmt.Printf("%v\n", matches)

### Unit Testing Cross Platform File System Code
The options for testing cross platform file system code are limited.  Ideally, 
a unit testing framework would create an in-memory file system which would respond
much like a physical filesystem.  For example, creating a file when the parent
directory does not exist should fail.  [Afero](https://github.com/spf13/afero) 
is an option but it does have some limitations.  For example, if you create a 
file in Afero, but the parent directory does not exist, the in-memory 
filesystem won't fail.  Run that code on a real file system and it will fail.

In addition, Mac OS and Windows are not case sensitive file systems.  On those 
operating systems, the files "foo.txt" and "FOO.txt" are the same file.  However,
on Mac OS it is possible to have the files "foo.txt" and "FOO.txt" in the same
directory.  (This caused me no end of grief when accidentally captializing and 
not captializing the names of some files.)  A testing framework should mimic 
these strange behaviors, but I'm sure there are many other gotchas and edge 
cases.  Maybe this is too detailed a mimicry.  Right now your best option for 
testing your file system code is to create integration tests that exercise a 
real file system.  That requires either having access to multiple virtual 
machines to validate your code.  

