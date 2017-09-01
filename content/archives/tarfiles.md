+++
title = "tar files"
+++
## Tar Files
Tar files are a *nix standard originally used to write files into
a tape to create backups and archives.  It is rarely used for 
actual tape drives.  It is more often used to create files that 
are archives of directories or files.  Tar files are not compressed.
In many cases they are compressed after they are created using 
gzip (.gz) or bzip (.bz2).  Most users on Windows 
need to use a separate program to unpack tar files on Windows.
[Here](https://en.wikipedia.org/wiki/Tar_(computing)) is the
WikiPedia article on tar.  

### Creating a Tar file
Let's unpack [the example](https://golang.org/pkg/archive/tar/#example_) on the Go documentation site a little.  A tar writer implements 
the [io.Writer](https://golang.org/pkg/io/#Writer) interface.  By
itself, however, you can't use it to write data.  Tar is intended
to wrap another type supporting io.Writer, like a file or byte
buffer.  Calling [<code>NewWriter</code>](https://golang.org/pkg/archive/tar/#NewWriter) will return a 
tar writer that will use some underlying writer to actually
write data to a buffer or filesystem.

```Go
// Create a bytes buffer and pass that to the NewWriter 
// function.
buffer := new(bytes.Buffer)
// Now the tar writer will write data to the bytes buffer.
writer := tar.NewWriter(buffer)
```

### Writing Data
Writing data is a two step process.  The first step is to writer a 
"header" that explains what the subsequent bytes will contain.  The 
[Header](https://golang.org/pkg/archive/tar/#Header) struct 
has fields for attributes such as the file name, the file 
mode (the type of file an the file permissions), the file owner, 
the size, etc.  Writing a header does not require any data and 
sometimes it is useful to write a header that represents a directory
and will not be follwed by data.

```Go
// Create a header
myheader := &tar.Header{Name:"foo", Mode:0755}
tarWriter.WriteHeader(myheader)
```
### Opening a Tar Archive
Much like the writer implments io.Writer interface, the tar reader
implements the [io.Reader](https://golang.org/pkg/io/#Reader) interface.
The [<code>NewReader</code>](https://golang.org/pkg/archive/tar/#NewReader)
function takes something that implements an <code>io.Reader</code>
and wraps it.

```Go
file, _ := os.Open("myfile.tar")
reader := tar.NewReader(file)
```

### Reading from a Tar Archive
Reading from the tar file is also a two-step process.  The first part
is reading the header and then (if applicable) reading the bytes 
following the header.  

```Go
// First step is to get the header, which can contain the file size.
header, _ := reader.Next()
// Create a buffer large enough to hold the data.
buffer := make([]byte, header.Size)
// Read the data into the buffer.
reader.Read(buffer)
```

### Putting it All Together